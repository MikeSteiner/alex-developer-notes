# Angular: Deep dive plan

Основа: официалната документация https://angular.dev (v22). ★ отбелязва личните приоритети. Връзката след точка сочи към вече написан файл. Точка без връзка още няма файл.

Свързани планове извън Angular: [[rxjs-deep-dive-roadmap]], [[typescript-deep-dive-roadmap]], [[browser-deep-dive-roadmap]], [[testing-security-deep-dive-roadmap]].

---

## 1. Основи на компонентите и темплейтите

- Анатомия на компонента [[angular-components-basics]]
- Standalone компоненти [[angular-standalone-components]]
- View encapsulation: `Emulated`, `ShadowDom`, `None` и `:host`
- Lifecycle hooks [[angular-component-lifecycle]]
- Signal inputs, `model()` и outputs [[angular-inputs-outputs-model]]
- Content projection с `<ng-content>` [[angular-content-projection-queries]]
- Host elements [[angular-content-projection-queries]]
- Queries: `viewChild`, `contentChild` [[angular-content-projection-queries]]
- Control flow: `@if`, `@for`, `@switch` [[angular-control-flow]]
- `track` в `@for`, наследник на `trackBy` в `*ngFor` [[angular-control-flow]]
- Deferrable views и incremental hydration [[angular-defer]]

## 2. Directives и pipes

- Attribute директиви
- Структурни директиви и защо `*` е захар за `<ng-template>`
- `hostDirectives`: композиция вместо наследяване
- Pure и impure pipes, защо `async` pipe е impure
- Собствени pipes и вградените pipes

## 3. ★ Change Detection

- Механизмът и тригерът, Zone.js и `NgZone` [[change-detection-zonejs]]
- Zone.js и monkey patching [[zonejs-monkey-patching]]
- Мястото на change detection в event loop-а [[browser-event-loop-angular]]
- `ExpressionChangedAfterItHasBeenCheckedError` (NG0100) [[angular-component-lifecycle]]
- OnPush, `markForCheck` и пропускане на поддървета [[change-detection-onpush]]
- Signals, zoneless и глобален тригер с локална работа [[change-detection-signals-zoneless]]
- Как другите frameworks решават същия проблем [[change-detection-frameworks]]

## 4. ★ Signals

https://angular.dev/guide/signals

- `signal`, `computed`, `effect` като основа [[change-detection-signals-zoneless]]
- `linkedSignal` за локално копие на input [[angular-inputs-outputs-model]]
- `asReadonly()`
- Reactive context: producer и consumer
- `computed`: мързелив, мемоизиран, с динамични зависимости
- `untracked`
- Загуба на tracking след `await`
- Equality функции
- `resource`
- Debounced signals
- `effect` и `afterRenderEffect`
- RxJS interop: `toSignal`, `toObservable`

## 5. ★ Dependency Injection

https://angular.dev/guide/di

Всички подточки са в [[angular-dependency-injection]].

- `inject()` и injection context
- Създаване и използване на services, `providedIn: 'root'`
- Новият декоратор `@Service` във v22
- Lazy loading на services с `injectAsync`
- Видове providers: `useClass`, `useValue`, `useFactory`, `useExisting`, `InjectionToken`
- Hierarchical injectors: environment injector и element injector
- Resolution modifiers
- `providers` срещу `viewProviders`
- Lightweight injection tokens, без файл засега
- Debugging на DI, без файл засега

## 6. Архитектура и state management

- Smart и dumb компоненти, или container и presentational
- Facade services: компонентът не знае откъде идват данните
- Feature-based структура на папките
- Nx monorepo: библиотеки и граници между тях
- ★ Кога signals, кога RxJS, кога NgRx
- Състояние в service със signals
- Състояние в service с `BehaviorSubject` [[tapresponse-store-state]]
- NgRx: actions, чисти reducers, effects, мемоизирани selectors [[ngrx-architecture]]
- Store, ComponentStore и SignalStore [[ngrx-architecture]]
- SignalStore в дълбочина: `withState`, `withComputed`, `withMethods`, `rxMethod`
- NgRx Entity

## 7. Routing

- Lazy loading: `loadComponent`, `loadChildren`
- Функционални guards и resolvers
- Route-level providers, връзка с DI [[angular-dependency-injection]]
- Bundle size и code splitting

## 8. Forms

- Reactive Forms и typed forms
- `patchValue()` [[patchValue()]]
- Signal Forms
- Custom validators
- Dynamic forms

## 9. HTTP Client

- Функционални interceptors
- Глобална обработка на грешки в interceptor [[rxjs-error-handling]]

## 10. Рендериране и производителност

- `NgOptimizedImage` [[Какво всъщност е NgOptimizedImage]], [[Image Loading and Loader abstraction]], [[Image optimization NgOptimizedImage Browser]]
- `@defer` [[angular-defer]]
- OnPush и zoneless като инструменти за производителност [[change-detection-onpush]], [[change-detection-signals-zoneless]]
- Lazy loading на routes и bundle size, в точка 7

## 11. SSR и hydration

- SSR с `@angular/ssr`, наследник на Angular Universal
- Hydration
- Incremental hydration [[angular-defer]]
- Стабилност на приложението и `PendingTasks` [[change-detection-signals-zoneless]]
- `TransferState`: реален случай с `NotFoundStatus` [[typescript-narrowing]]
- Angular Service Worker и навигации при SSR [[service-workers]]
