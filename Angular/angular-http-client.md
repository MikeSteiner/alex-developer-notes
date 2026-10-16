---
tags: [angular, http, httpclient, interceptors, httpResource, xsrf, ssr, transfer-cache, разбиране]
свързани: ["[[angular-routing]]", "[[angular-dependency-injection]]", "[[rxjs-error-handling]]", "[[rxjs-sharereplay-hot-cold]]", "[[browser-storage]]", "[[angular-inputs-outputs-model]]"]
---
# Angular: HTTP Client — interceptors, `httpResource`, XSRF и transfer cache

https://angular.dev/guide/http/setup
https://angular.dev/guide/http/interceptors
https://angular.dev/guide/http/http-resource
https://angular.dev/best-practices/security
https://angular.dev/guide/ssr

## Настройка и какво се промени в последните версии

`HttpClient` е service-ът, чрез който Angular приложението говори със сървъра. Неговите методи връщат **cold observable**: заявката тръгва едва при абониране, и всеки абонат предизвиква нова заявка. Последствията от това, включително дублираните заявки, са описани в [[rxjs-sharereplay-hot-cold]].

Две промени от Angular 21 си заслужава да се знаят на интервю.

**`HttpClient` е достъпен по подразбиране.** Не е нужно да се предоставя, за да се инжектира. `provideHttpClient` се използва само за конфигурация, например за interceptors или XSRF настройки.

**Backend-ът по подразбиране е `fetch`, а не `XMLHttpRequest`.** `withFetch()` вече не е нужен. Старото поведение се връща с `withXhr()`. Основното ограничение на `fetch` е, че няма събития за прогрес при **качване** на файл. Затова за качване с прогрес бар се използва `withXhr()`. При SSR XHR е deprecated и е планиран за премахване, а документацията препоръчва `fetch`.

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor])),
  ],
};
```

`HttpClientModule` и `withJsonpSupport` са deprecated. JSONP е deprecated заради риска от XSS, а за заявки към друг домейн се използва CORS.

## Interceptors

Interceptor е функция, която стои между приложението и мрежата. Тя получава всяка изходяща заявка и функцията `next`, която предава заявката по-нататък по веригата. Връща observable от отговора. Така interceptor-ът може да промени заявката, да реагира на отговора или изобщо да не изпрати заявката.

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();

  if (!token) {
    return next(req);
  }

  return next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
};
```

Функционалните interceptors се изпълняват в **injection context** на injector-а, който ги е регистрирал, затова `inject()` работи в тях. Това е същият шаблон като guards и resolvers в [[angular-routing]] и същото правило като в [[angular-dependency-injection]].

### Редът: лукът

Interceptors се изпълняват в реда, в който са подадени на `withInterceptors`. Заявката минава през тях отвън навътре, а отговорът се връща **в обратен ред**, защото всеки interceptor обвива потока, върнат от `next`.

```mermaid
flowchart LR
    App["Компонент"] --> A["auth"]
    A --> E["error"]
    E --> L["loading"]
    L --> N["мрежа"]
    N -. "отговор" .-> L
    L -.-> E
    E -.-> A
    A -.-> App
```

Образът е лук. Заявката минава през слоевете към центъра, а отговорът излиза през същите слоеве обратно. Затова interceptor-ът за грешки, който трябва да види и грешките от другите, обикновено се поставя по-напред в списъка.

Документацията препоръчва функционалните interceptors пред старите, базирани на класове, именно заради реда. При класовете редът следва реда на регистриране на providers, който в голямо приложение с йерархичен DI е трудно предвидим. Класовите interceptors продължават да работят чрез `withInterceptorsFromDi()` и multi provider-а `HTTP_INTERCEPTORS`. Защо там е нужен `multi: true`, е обяснено в [[angular-dependency-injection]].

### Непроменими заявки

`HttpRequest`, `HttpResponse`, `HttpHeaders` и `HttpParams` са непроменими. Промяна се прави чрез `clone()`, а резултатът на `headers.set()` трябва да се присвои. Причината е, че interceptor-ът може да се изпълни **няколко пъти за една и съща заявка**, например при `retry`. Затова той трябва да е идемпотентен. Тялото на заявката обаче не е защитено от дълбока мутация, и ако се променя, промяната трябва да е безопасна при повторно изпълнение.

### Типичните interceptors

**Автентикация.** Добавя токена към заявките. Добре е да проверява и дали заявката е към собственото API, за да не изпрати токена към външен домейн.

**Грешки.** Реагира централно на грешки, които изискват еднаква реакция навсякъде. Пример е 401, който води към страницата за вход. Той препраща грешката нататък с `throwError`, за да може и локалният код да реагира. Цялата картина на обработката на грешки на три нива е в [[rxjs-error-handling]].

**Индикатор за зареждане.** Увеличава брояч при заявка и го намалява във `finalize`. Не трябва да е в `tap` при успех и в `catchError` при грешка, защото така пропуска отменените заявки, например при `switchMap`.

**Кеш.** Interceptor-ът не е длъжен да извика `next`. Той може да върне собствен отговор с `of(new HttpResponse({ body: cached }))`, без да стига до мрежата.

### `HttpContext`: настройка на отделна заявка

`HttpContext` носи метаданни, предназначени за interceptors, а не за сървъра. Така отделна заявка може да промени поведението на interceptor, без да се добавят специални хедъри.

```typescript
export const SKIP_LOADING = new HttpContextToken(() => false);

this.http.get('/api/listings/autocomplete', {
  context: new HttpContext().set(SKIP_LOADING, true),
});
```

В interceptor-а стойността се чете с `req.context.get(SKIP_LOADING)`. Factory функцията дава стойността по подразбиране. За разлика от останалите свойства на заявката, контекстът е **променлив** и промяната в него оцелява при `retry`. Затова е подходящ за броене на опитите.

## `httpResource`: заявки като signals

`httpResource` обвива `HttpClient` и представя заявката и отговора ѝ като **signals**. Той използва същия `HttpClient`, затова всички interceptors важат и за него.

```typescript
export class ListingDetails {
  readonly id = input.required<string>();

  protected readonly listing = httpResource<Listing>(() => `/api/listings/${this.id()}`);
}
```

```html
@if (listing.hasValue()) {
  <app-listing-card [listing]="listing.value()" />
} @else if (listing.error()) {
  <p i18n>Обявата не може да се зареди.</p>
} @else if (listing.isLoading()) {
  <app-spinner />
}
```

Функцията, подадена на `httpResource`, е реактивна. Когато signal в нея се промени, например `id` от route-а чрез `withComponentInputBinding` в [[angular-routing]], ресурсът изпраща нова заявка. Ако предишната още тече, тя се **отменя**. Това е поведението на `switchMap` от [[rxjs-flattening-operators]], без нито ред RxJS.

Няколко детайла, които се питат:

- За разлика от `HttpClient`, `httpResource` е **eager**: заявката тръгва веднага, без абонамент.
- `value()` хвърля грешка, ако ресурсът е в състояние на грешка. Затова в темплейта първо се проверява `hasValue()`.
- `httpResource.text`, `httpResource.blob` и `httpResource.arrayBuffer` са варианти за други типове отговор.
- Опцията `parse` приема функция за валидация, например схема на Zod. Типът на `value` е типът, който тя връща. Така невалиден отговор от сървъра се открива на границата, а не дълбоко в компонента.

**Кога `httpResource` и кога `HttpClient`.** Документацията е категорична: `httpResource` е за **четене** на данни, които показва интерфейсът. За **промени**, тоест POST, PUT и DELETE, се използва `HttpClient`. Причината следва от поведението му. Ресурс, който сам изпраща и отменя заявки при промяна на signal, е подходящ за GET, но опасен за запис. Същото разграничение има между `switchMap` и `concatMap`.

## XSRF защита

`HttpClient` има вградена защита срещу CSRF, описана и в [[browser-storage]]. Той чете cookie с име `XSRF-TOKEN` и изпраща стойността му като хедър `X-XSRF-TOKEN`. Това става само при заявки, които **променят данни**, като POST, и само към **относителни или същия origin** адреси. За GET и HEAD хедър не се добавя. Имената се променят с `withXsrfConfiguration({ cookieName, headerName })`.

Защитата работи, защото само код от собствения домейн може да прочете cookie-то. Злонамерен сайт може да накара браузъра да изпрати cookies, но не може да ги прочете и да ги сложи в хедър.

Документацията подчертава, че Angular покрива само **половината от схемата**. Сървърът трябва да постави cookie-то, без `HttpOnly`, защото JavaScript трябва да може да го чете. Трябва и да проверява при всяка заявка, че хедърът съвпада с cookie-то. Без това вградената защита е безполезна.

`HttpClient` освен това премахва автоматично префикса `)]}',\n` от JSON отговорите. Той е защита на сървъра срещу XSSI, тоест четене на JSON чрез `<script>` таг.

## Transfer cache при SSR

При SSR всяка заявка, направена по време на рендерирането на сървъра, би се повторила в браузъра, когато приложението се хидратира. Transfer cache-ът решава това. На сървъра `HttpClient` кешира отговорите и ги сериализира в HTML-а. В браузъра при първоначалното рендериране той използва тези данни, вместо да изпраща нови заявки. Когато приложението стане стабилно в браузъра, кешът спира да се използва.

По подразбиране се кешират само GET и HEAD заявки. **Не се кешират** заявки с хедъри `Authorization`, `Proxy-Authorization` или `Cookie`, заявки с `withCredentials`, отговори със `Set-Cookie` и отговори с `Cache-Control: no-store`, `no-cache` или `private`.

Това е капан, който се среща в реални приложения. Ако interceptor добавя `Authorization` към всяка заявка, включително към публични данни, нищо не се кешира, и всяка заявка се изпраща втори път в браузъра. Изглежда като че ли transfer cache-ът не работи.

Настройката е глобална с `withHttpTransferCacheOptions` в `provideClientHydration()`. Опции като `includeRequestsWithAuthHeaders`, `includePostRequests` и `filter` разширяват или стесняват кеширането. За отделна заявка се използва опцията `transferCache`, включително `transferCache: false`.

Още два детайла от документацията за SSR. По време на SSR `fetch` backend-ът ограничава тялото на отговора до 1 MB по подразбиране, а по-голям отговор дава грешка NG02825. Ако сървърът и браузърът използват **различни адреси** за API-то, както в примера с `API_BASE_URL_TOKEN` в [[angular-dependency-injection]], където сървърът ходи към вътрешния домейн, а браузърът към публичния, ключовете в кеша не съвпадат. За това има `HTTP_TRANSFER_CACHE_ORIGIN_MAP`, който съпоставя двата origin-а.

## Изречението за интервю

> `HttpClient` връща cold observables, затова заявката тръгва при абониране. От Angular 21 той е достъпен по подразбиране, а backend-ът по подразбиране е `fetch`, като `withXhr` остава само за прогрес при качване. Функционалните interceptors са функции в injection context, които работят като лук: заявката минава през тях по реда в `withInterceptors`, а отговорът в обратен ред. Заявките са непроменими и се променят с `clone`, защото interceptor-ът може да се изпълни повторно при `retry`. `HttpContext` настройва interceptor за отделна заявка. `httpResource` е signal подходът за четене: реактивен, eager, отменя предишната заявка, а за запис остава `HttpClient`. XSRF защитата е наполовина в Angular, който изпраща `X-XSRF-TOKEN` при мутиращи заявки към същия origin, и наполовина в сървъра, който трябва да проверява. При SSR transfer cache-ът пренася отговорите от сървъра в HTML-а, но по подразбиране пропуска заявки с `Authorization`, което е честа причина заявките да се повтарят в браузъра.
