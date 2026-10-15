---
tags: [rxjs, moc]
---

# RxJs — начало

Тази бележка не съдържа обяснения сама по себе си — тя е картата, по която темите се четат в естествения ред, в който се надграждат една върху друга. Добавянето на нов файл никога не изисква преномериране: просто решаваш къде в списъка му е мястото и добавяш нов ред тук.

## Разбиране

Основата тръгва от разликата между мързелив и нетърпелив продуцент на стойности, преди изобщо да стигнем до RxJS-специфични концепции.

- [[observable-vs-promise]] — защо Observable и Promise не са взаимозаменяеми, а решават различен проблем: мързелива срещу нетърпелива екзекуция, единична стойност срещу поток, отмяна, и мостовете между двете (`firstValueFrom`, `from`).
- [[observable-subject-behaviorsubject]] — Observable като рецепта, Subject като мултикаст мост, BehaviorSubject като носител на текущо състояние, и разликата между hot и cold observables през примера с `dialogRef.afterClosed()`.
- [[tapresponse-store-state]] — практическото приложение: `tapResponse` и защо съществува отделно от `tap`/`catchError`, плюс реален пример за state store с `BehaviorSubject`, изграден върху `FavouriteListingStore`.
- [[rxjs-switchMap]] — първият конкретен оператор в дълбочина: как и защо прекъсва предишен вътрешен observable, кога е капан вместо решение, и кога `concatMap`/`exhaustMap` са по-подходящи.
- [[rxjs-flattening-operators]] — четирите flattening оператора един до друг: `switchMap`, `mergeMap`, `concatMap` и `exhaustMap` се различават само по това какво правят, когато дойде нова стойност, докато предишната още тече. Въпрос номер едно на NgRx интервю.
- [[rxjs-error-handling]] — грешката е терминална: `catchError` подменя умрелия поток, а мястото му вътре или извън flattening оператора решава дали effect-ът оцелява. Плюс `retry`, `timeout`, `finalize` и какво става с неприхванатата грешка.
- [[rxjs-combination-operators]] — `combineLatest`, `withLatestFrom`, `forkJoin` и `concatLatestFrom` от NgRx: кой поток има право да предизвика излъчване и кога. Капанът с тихата загуба при `withLatestFrom`.
- [[rxjs-sharereplay-hot-cold]] — мостовете от cold към hot: `share` срещу `shareReplay`, капанът с `refCount` и кеширането на HTTP резултат в service.
- [[rxjs-startWith-distinctUntilChanged-skip-pairwise]] — как един поток разпознава „истинска“ промяна: `startWith`, `distinctUntilChanged`, `skip`, `pairwise` и `filter` върху реален пример с офертата в DSKHome.
- [[ngrx-architecture]] — приложението на всичко по-горе в NgRx: actions, чисти reducers, effects и мемоизирани selectors. Живее в папката на NgRx, но се чете след операторите.
- [[EMPTY-void-null-undefined]] — граничен случай, независим от останалата последователност: разликите между `of(undefined)`, `of()`, `of(void 0)` и `EMPTY`. Полезна справка по всяко време, не изисква предходно четене.

## Активиране

Кратки, прицелени карти за активно припомняне, изградени върху темите по-горе — компресирани за скорост на извличане под натиск, не за разбиране от нулата.

- [[flashcards-observable-basics]] — *в процес; все още празен файл.*
- [[flashcards-operators]] — *в процес; все още празен файл.*