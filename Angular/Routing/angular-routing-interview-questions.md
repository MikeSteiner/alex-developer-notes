---
tags: [angular, routing, lazy-loading, guards, интервю, въпроси]
свързани: ["[[angular-routing]]", "[[angular-dependency-injection]]", "[[angular-http-client]]", "[[rxjs-flattening-operators]]"]
---
# Angular Routing: Въпроси за интервю

Въпросите са от изпитващата сесия на 18.10.2026. Отговорът под всеки въпрос е скрит. Първо отговори на глас, после отвори отговора и сравни.

## Въпрос 1

Каква е разликата между `loadComponent` и `loadChildren` и какво печели приложението от lazy loading? Какво е preloading и какъв компромис решава?

> [!note]- Отговор
> `component` слага компонента в същия bundle като конфигурацията на routes. `loadComponent` зарежда мързеливо **един компонент**, когато route-ът стане активен. `loadChildren` зарежда мързеливо **масив от дъщерни routes**, обикновено файл като `mortgage.routes.ts`, и това става по време на сравняването на пътя. Двете приемат функция с динамичен `import()`, а кодът отива в отделен chunk, който се изтегля при първото посещение. Това е главният инструмент за намаляване на началния bundle.
>
> Компромисът е, че първото посещение на lazy route добавя мрежова заявка. Preloading го решава: след първото зареждане router-ът изтегля chunks във фонов режим. Синтаксисът е `provideRouter(routes, withPreloading(PreloadAllModules))`. По подразбиране preloading няма. `PreloadAllModules` изтегля всичко, затова в голямо приложение се използва собствена `PreloadingStrategy`, която изтегля само routes с флаг в `data`.
>
> Chunk-ът съдържа **код**: компонента с компилирания му темплейт и стиловете. Данните от HTTP заявките не са част от него, затова preloading **не** ускорява бавно API. Вложен lazy loading на много нива създава водопад от последователни заявки.

## Въпрос 2

Какво връща един guard и как се пренасочва потребителят? Каква е разликата между `canActivate` и `canMatch` и защо guard в браузъра не е защита на сигурността?

> [!note]- Отговор
> Guard връща `true`, `false`, `UrlTree` или `RedirectCommand`, както и `Promise` или `Observable` с някоя от тях. Пренасочването се прави, като се **върне** `UrlTree`, а не чрез `router.navigate()` и `false`:
>
> ```typescript
> export const authGuard: CanActivateFn = (_, state) => {
>   const auth = inject(AuthService);
>   const router = inject(Router);
>
>   return auth.isLoggedIn() || router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
> };
> ```
>
> По-рано guards бяха класове с интерфейс `CanActivate`, който сега е deprecated. Днес са функции от тип `CanActivateFn`, които Angular изпълнява в injection context, затова в тях работи `inject()`.
>
> При `false` от `canActivate` навигацията **спира**. `canMatch` се изпълнява по време на сравняването, и при `false` router-ът **продължава** към следващата конфигурация. Затова един и същ път може да води до различни компоненти според роля, feature flag или A/B тест. Ако `canMatch` откаже, chunk-ът от `loadChildren` изобщо не се изтегля.
>
> Guard в браузъра не е сигурност, защото кодът е при потребителя и може да бъде заобиколен. Както при валидацията на форми, guards са за потребителския опит, а правата се проверяват на сървъра при всяка заявка.

## Въпрос 3

Компонентът показва обява по `/listings/:id`. Потребителят навигира от `/listings/1` към `/listings/2`. Защо компонент, който чете `id` от `route.snapshot` в `ngOnInit`, показва старата обява? Как днес четеш параметрите правилно?

> [!note]- Отговор
> Конфигурацията на route-а е същата, затова router-ът **преизползва същата инстанция** на компонента. Тя не се създава отново, `ngOnInit` не се изпълнява втори път, а `snapshot` е прочетен веднъж, със старото `id`.
>
> Модерният вариант е `provideRouter(routes, withComponentInputBinding())`. Router-ът подава като inputs по съвпадащо име параметрите на пътя, query параметрите, `data` и резултатите от resolvers. `id` става signal input, а `computed` или `httpResource(() => \`/api/listings/${this.id()}\`)` реагират на промяната и отменят старата заявка. Компонентът не зависи от `ActivatedRoute`, тестът подава inputs, а типът е точен.
>
> Традиционният вариант е абонамент за `route.paramMap` с `switchMap` към заявката, за да важи само последната обява при бърза смяна.
