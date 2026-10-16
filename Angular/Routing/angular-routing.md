---
tags: [angular, routing, lazy-loading, guards, resolvers, preloading, разбиране]
свързани: ["[[angular-dependency-injection]]", "[[angular-inputs-outputs-model]]", "[[angular-defer]]", "[[tapresponse-store-state]]", "[[angular-http-client]]"]
---
# Angular: Routing — lazy loading, guards, resolvers и route state

https://angular.dev/guide/routing/define-routes
https://angular.dev/guide/routing/loading-strategies
https://angular.dev/guide/routing/route-guards
https://angular.dev/guide/routing/data-resolvers
https://angular.dev/guide/routing/read-route-state

## Какво решава router-ът

Router-ът превръща адреса в браузъра в дърво от компоненти. При навигация той сравнява URL-а с конфигурацията на routes, решава кои компоненти да покаже в кои `<router-outlet>`, изпълнява guards и resolvers и зарежда кода на lazy частите. В SPA това се случва без презареждане на страницата.

За интервю четири неща от routing-а са важни: как routes разделят приложението на chunks, как guards решават дали навигацията е позволена, как resolvers зареждат данни преди страницата и как компонентът чете параметрите на route-а.

## Конфигурацията накратко

```typescript
export const routes: Routes = [
  { path: '', component: HomePage, title: 'Начало' },
  { path: 'listings/:id', loadComponent: () => import('./listing-details/listing-details') },
  { path: 'mortgage', loadChildren: () => import('./mortgage/mortgage.routes') },
  { path: 'offers', redirectTo: '/listings' },
  { path: '**', component: NotFoundPage },
];
```

**Първото съвпадение печели.** Щом път съвпадне, router-ът спира да търси. Затова по-конкретните пътища стоят преди по-общите: `users/new` преди `users/:id`, а wildcard-ът `**` винаги е последен.

**Вложени routes** се описват с `children`, а родителският компонент има собствен `<router-outlet>`, в който се показват децата.

**`title`** задава заглавието на страницата. Може да е и `ResolveFn`, а централизирано се управлява със собствен `TitleStrategy`.

## Lazy loading и bundle size

Ако route-ът използва `component`, компонентът влиза в същия bundle като конфигурацията на routes. При `loadComponent` и `loadChildren` кодът отива в **отделен chunk**, който се изтегля едва когато потребителят посети route-а. Това е главният инструмент за намаляване на началния bundle на приложението.

- `loadComponent` зарежда мързеливо един компонент, когато route-ът стане активен.
- `loadChildren` зарежда мързеливо цял масив от дъщерни routes, по време на сравняването на пътя.

Двете приемат функция, която връща Promise, най-често динамичен `import()`. Ако файлът експортира компонента или routes като `default`, `import()` се връща директно, без `.then(m => m.X)`.

Loader-ите се изпълняват в **injection context** на route-а, затова в тях работи `inject()`, по правилото от [[angular-dependency-injection]]. Това позволява компонентът да се избира според условие:

```typescript
{
  path: 'dashboard',
  loadComponent: () =>
    inject(FeatureFlags).isPremium ? import('./premium-dashboard') : import('./basic-dashboard'),
}
```

**Компромисът.** Lazy route спестява начален bundle, но добавя мрежова заявка при първото посещение. Документацията препоръчва главните входни страници да са eager, а останалите да са lazy. Тя предупреждава и за **вложен lazy loading на много нива**, защото всяко ниво е отделна последователна заявка.

### Preloading

Preloading решава компромиса отгоре. След като началната страница се зареди, router-ът изтегля chunks на lazy routes във фонов режим, преди потребителят да ги е поискал. Така първото зареждане остава леко, а следващите навигации са бързи.

```typescript
provideRouter(routes, withPreloading(PreloadAllModules));
```

По подразбиране няма preload. `PreloadAllModules` изтегля всички lazy routes. Собствена стратегия, клас, който имплементира `PreloadingStrategy`, може да изтегля само routes, маркирани с флаг в `data`. Тази част е от общото знание за router-а, а не от прегледаните страници.

### `@defer` срещу lazy route

Lazy route отлага код на ниво **страница** и тригерът е навигацията. `@defer` от [[angular-defer]] отлага код на ниво **част от темплейта** вътре във вече заредена страница. Двата механизма се допълват.

## Guards

Guard е функция, която решава дали навигацията може да продължи. Функционалните guards са обикновени функции, които Angular изпълнява в injection context, затова зависимостите се взимат с `inject()`. Същият шаблон имат resolvers и HTTP interceptors в [[angular-http-client]].

```typescript
export const authGuard: CanActivateFn = (_, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  return auth.isLoggedIn() || router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};
```

| Guard | Решава | Типичен случай |
|---|---|---|
| `canActivate` | дали може да се влезе в route | автентикация |
| `canActivateChild` | дали може да се влезе в всяко дете | защита на цял раздел |
| `canDeactivate` | дали може да се напусне route | незапазени промени във форма |
| `canMatch` | дали тази конфигурация изобщо съвпада | feature flags, A/B тестове, роли |

### Какво връща guard

Guard връща `true`, за да позволи навигацията, или `false`, за да я спре. Може да върне и `UrlTree` или `RedirectCommand`, за да **пренасочи**. Документацията препоръчва пренасочването чрез върната стойност пред `false` с ръчно извикване на `router.navigate`. Може да върне и `Promise` или `Observable`. Тогава router-ът взима първата стойност и се отписва.

### `canMatch` срещу `canActivate`

Това е най-честият въпрос за guards.

`canActivate` се изпълнява, **след като** route-ът е съвпаднал. При `false` навигацията спира.

`canMatch` се изпълнява **по време** на сравняването. При `false` router-ът не спира, а **продължава да търси** следваща конфигурация. Затова един и същ път може да води до различни компоненти:

```typescript
const routes: Routes = [
  { path: 'dashboard', component: AdminDashboard, canMatch: [adminGuard] },
  { path: 'dashboard', component: UserDashboard },
];
```

Има и практическа полза за bundle-а. Тя е от общото знание за router-а, а не от прегледаната страница. Ако `canMatch` върне `false`, кодът от `loadChildren` на този route изобщо не се изтегля. При `canActivate` chunk-ът с routes вече е изтеглен, преди guard-ът да откаже.

### Ред на изпълнение

Guards в един масив **не** се изчакват един друг. Router-ът ги стартира по реда в масива, но те се изпълняват паралелно. Ако някой не върне `true`, router-ът използва резултата на **първия по ред в масива**, дори ако по-късен е приключил по-рано. Затова един guard не трябва да разчита на работа, свършена от друг в същия масив, например зареждане на текущия потребител. Такава работа се споделя през service или двете проверки се обединяват в един guard.

### Guards не са сигурност

Документацията е категорична: guard в браузъра **никога не е единствената защита** на достъпа. Кодът се изпълнява при потребителя и може да бъде заобиколен. Guard-ът е за потребителския опит, а правата се проверяват на сървъра при всяка заявка за данни.

## Resolvers

Resolver е функция, която зарежда данни **преди** route-ът да се активира. Компонентът се показва едва когато данните са налични.

```typescript
export const listingResolver: ResolveFn<Listing> = (route) =>
  inject(ListingsApiService).getListing(route.paramMap.get('id')!);
```

```typescript
{ path: 'listings/:id', component: ListingDetails, resolve: { listing: listingResolver } }
```

Ключът в `resolve` става име на свойство в данните на route-а.

### Кога resolver е антипатерн

Най-силната част от темата за интервю е кога **не** да се използва resolver. Навигацията **чака** resolver-а. Ако заявката е бавна, потребителят кликва и нищо не се случва, докато не дойде отговорът. Документацията изброява три случая, в които resolver е грешният избор.

Първият е, когато страницата може смислено да се покаже със skeleton или индикатор за зареждане. Тогава е по-добре страницата да се отвори веднага, а данните да дойдат в нея.

Вторият е при вложени routes, всеки със свой resolver. Те образуват **водопад**: детето чака родителя, затова всяко ниво добавя време.

Третият е в приложение със signals. Документацията препоръчва route resources като неблокиращ, реактивен подход. Те се включват с `withRouterResources`. Конкретният им API трябва да се провери в документацията, защото страниците, които прегледах, само ги споменават.

Ако resolver все пак се използва, той трябва да е лек, да има timeout и да обработва грешките. Грешките се обработват или централно с `withNavigationErrorHandler` в `provideRouter`, или в самия resolver с `catchError`, който връща `RedirectCommand` към резервна страница.

## Как компонентът чете route state

### `ActivatedRoute`

`ActivatedRoute` предоставя данните на текущия route като **observables**: `params`, `queryParams`, `data` и `url`. Той има и `snapshot` със стойностите в момента.

Разликата е важна. Ако потребителят навигира от `/listings/1` към `/listings/2`, router-ът **преизползва същата инстанция** на компонента, защото конфигурацията е същата. Компонентът не се създава отново, а `ngOnInit` не се изпълнява втори път. Ако компонентът е прочел `snapshot` веднъж, той показва старата обява. Затова параметрите се четат от observable-а или чрез input binding.

```typescript
export class ListingDetails {
  private readonly route = inject(ActivatedRoute);

  protected readonly listing = toSignal(
    this.route.paramMap.pipe(
      map((params) => params.get('id')!),
      switchMap((id) => inject(ListingsApiService).getListing(id)),
    ),
  );
}
```

`switchMap` е правилният оператор тук, защото при бърза смяна на обявите важи само последната. Причината е описана в [[rxjs-flattening-operators]].

### `withComponentInputBinding`

С `withComponentInputBinding()` в `provideRouter` router-ът подава параметрите на пътя, query параметрите и данните на route-а, включително резултатите от resolvers, **директно като inputs** на компонента, по съвпадащо име.

```typescript
provideRouter(routes, withComponentInputBinding());
```

```typescript
export class ListingDetails {
  readonly id = input.required<string>();
  readonly listing = input.required<Listing>();
}
```

Така компонентът не зависи от `ActivatedRoute`, тестът подава inputs като на всеки друг компонент, а типът е точен. Тъй като inputs са signals, `computed` и `httpResource` реагират на смяната на `id` без `switchMap`. Това се връзва с [[angular-inputs-outputs-model]]. Прегледаните страници описват подробно само връзката с resolver данните. Съпоставянето на параметрите на пътя и query параметрите с inputs е от общото знание за router-а.

### Router events

`Router.events` е **hot observable**. Той излъчва събития като `NavigationStart`, `NavigationEnd`, `NavigationCancel` и `NavigationError`, независимо дали някой слуша. Компонент, който се абонира след навигацията, не получава минали събития. Типичните употреби са аналитика и глобален индикатор за зареждане по време на навигация. Разликата hot срещу cold е описана в [[tapresponse-store-state]].

## Route-level providers

Масивът `providers` на route прави services достъпни само за този route и децата му, включително за неговите guards и resolvers. Router-ът създава за него отделен environment injector. Подробностите, включително защо тези services нямат директен достъп до параметрите на route-а, са в [[angular-dependency-injection]]. В комбинация с `loadChildren` services на цял feature влизат в неговия chunk, а не в основния bundle.

## Изречението за интервю

> Router-ът превръща URL-а в дърво от компоненти, като първото съвпадение печели. `loadComponent` и `loadChildren` изнасят кода в отделни chunks и са главният инструмент за начален bundle, а preloading изтегля тези chunks във фонов режим след първото зареждане. Loader-ите, guards и resolvers са функции в injection context, затова в тях работи `inject()`. Guard-ът връща `true`, `false` или `UrlTree` за пренасочване. `canMatch` се различава от `canActivate` по това, че при отказ router-ът продължава към следващата конфигурация, затова е за feature flags и роли. Guards са за потребителския опит, а не за сигурност, защото правата се проверяват на сървъра. Resolver-ът блокира навигацията, затова го избягвам, когато страницата може да се покаже със skeleton, и при вложени routes, които правят водопад. Параметрите чета като inputs с `withComponentInputBinding`, а не от `snapshot`, защото при навигация към същия route компонентът се преизползва.
