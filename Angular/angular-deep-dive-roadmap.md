# Angular: Deep dive plan

Основа: официалната документация https://angular.dev (v22). ★ отбелязва личните приоритети. [[връзка]] сочи към вече написан файл.

---

## 1. Основи на компонентите и темплейтите

- Анатомия на компонента и standalone компоненти [[angular-standalone-components]]
- Lifecycle hooks [[angular-component-lifecycle]]
- Signal inputs, `model()` и outputs
- Content projection с `<ng-content>`
- Host elements
- Queries: `viewChild`, `contentChild`
- Control flow: `@if`, `@for`, `@switch`
- `track` в `@for` (наследник на `trackBy` в `*ngFor`)
- Deferrable views (`@defer`)

## 2. ★ Change Detection

- Zone.js и monkey patching
- Цикълът: обхождане на дървото отгоре надолу
- `ExpressionChangedAfterItHasBeenCheckedError` (NG0100)
- OnPush, `markForCheck` и пропускане на поддървета: https://angular.dev/best-practices/skipping-subtrees
- Signals в OnPush темплейт и автоматичното маркиране
- Zone срещу Zoneless: старият и новият модел
- `NgZone` като исторически контекст

## 3. ★ Signals

https://angular.dev/guide/signals

- `signal`, `computed`, `asReadonly()`
- Reactive context: producer и consumer
- `computed`: мързелив, мемоизиран, с динамични зависимости
- `untracked`
- Загуба на tracking след `await`
- Equality функции
- `linkedSignal`
- `resource`
- Debounced signals
- `effect` и `afterRenderEffect`
- RxJS interop: `toSignal`, `toObservable`

## 4. ★ Dependency Injection

https://angular.dev/guide/di

- `inject()` и injection context
- Създаване и използване на services, `providedIn: 'root'`
- Новият декоратор `@Service` във v22
- Lazy loading на services
- Видове providers: `useClass`, `useValue`, `useFactory`, `useExisting`, `InjectionToken`
- Hierarchical injectors: environment injector и element injector
- Resolution modifiers
- Lightweight injection tokens
- Debugging на DI

## 5. RxJS

- Observable срещу Promise [[rxjs-observables-subjects]]
- Cold и hot observables, Subject, BehaviorSubject [[rxjs-observables-subjects]]
- Оператори: `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`
- Композиция на потоци
- Асинхронни UI потоци
- `tapResponse` и store с `BehaviorSubject` [[rxjs-tapresponse-store-hot-cold]]

## 6. Архитектура и state management

- Smart и dumb компоненти
- Feature-based структура на папките
- State със Signals (модерният подход)
- State с RxJS
- NgRx и ComponentStore
- Шаблони за дизайн на services

## 7. Routing

- Lazy loading: `loadComponent`, `loadChildren`
- Функционални guards и resolvers
- Route-level providers (връзка с DI)

## 8. Forms

- Reactive Forms и typed forms
- Signal Forms
- Custom validators
- Dynamic forms

## 9. HTTP Client

- Функционални interceptors

## 10. Рендериране и производителност

- `NgOptimizedImage`
- `@defer`
- OnPush и zoneless като инструменти за производителност
- Lazy loading на routes
- Bundle size и code splitting

## 11. SSR и hydration

- SSR с `@angular/ssr` (наследник на Angular Universal)
- Hydration и incremental hydration
- `TransferState`: реален случай с `NotFoundStatus` [[typescript-narrowing]]

## 12. Testing и Security

- Unit и e2e тестове
- XSS и sanitization

## 13. TypeScript

- Narrowing и type predicates [[typescript-narrowing]]
