# RxJS: Deep dive plan

Основа: официалната документация https://rxjs.dev ★ отбелязва личните приоритети. Връзката след точка сочи към вече написан файл. Точка без връзка още няма файл.

Входната точка с реда на четене е [[rxjs]]. Свързан план: [[angular-deep-dive-roadmap]].

---

## 1. Основи

- Observable срещу Promise [[observable-vs-promise]]
- Observable, Subject, BehaviorSubject [[observable-subject-behaviorsubject]]
- Cold и hot observables [[observable-subject-behaviorsubject]], [[tapresponse-store-state]]
- `of(undefined)`, `of()` и `EMPTY` [[EMPTY-void-null-undefined]]
- `ReplaySubject` и `AsyncSubject`

## 2. ★ Оператори

- Flattening: `switchMap`, `mergeMap`, `concatMap`, `exhaustMap` [[rxjs-flattening-operators]]
- `switchMap` в дълбочина [[rxjs-switchMap]]
- Комбиниране: `combineLatest`, `withLatestFrom`, `forkJoin`, `concatLatestFrom` [[rxjs-combination-operators]]
- Разпознаване на истинска промяна: `startWith`, `distinctUntilChanged`, `skip`, `pairwise` [[rxjs-startWith-distinctUntilChanged-skip-pairwise]]
- Създаване на потоци: `from`, `fromEvent`, `timer`, `interval`, `defer`
- Филтриране по време: `debounceTime`, `throttleTime`, `auditTime`
- Ограничаване: `take`, `first`, `takeUntil`, `takeWhile`
- Собствени оператори като функции от observable към observable

## 3. ★ Грешки и споделяне

- Обработка на грешки: `catchError`, `retry`, `timeout`, `finalize` [[rxjs-error-handling]]
- `share`, `shareReplay` и превръщането на cold в hot [[rxjs-sharereplay-hot-cold]]

## 4. ★ Абонаменти и изтичане на памет

- Кога абонаментът трябва да се прекрати и кога не
- `takeUntilDestroyed`, `async` pipe и `toSignal` [[angular-dependency-injection]]
- Типични изтичания: безкрайни потоци, вложени `subscribe`

## 5. RxJS и Angular

- `tapResponse` в ComponentStore [[tapresponse-store-state]]
- Schedulers и връзката им с event loop-а [[browser-event-loop-angular]]
- Interop със signals: `toSignal`, `toObservable`, `rxResource`
- `outputFromObservable` и `outputToObservable` [[angular-inputs-outputs-model]]

## 6. Тестване

- Marble тестове с `TestScheduler`
- Тестване на потоци с време без истинско чакане
