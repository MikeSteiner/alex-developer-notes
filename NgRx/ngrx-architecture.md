---
tags: [ngrx, state-management, actions, reducers, effects, selectors, разбиране]
свързани: ["[[rxjs-flattening-operators]]", "[[rxjs-error-handling]]", "[[rxjs-combination-operators]]", "[[tapresponse-store-state]]", "[[angular-dependency-injection]]"]
---
# NgRx: архитектура — actions, reducers, effects и мемоизирани selectors

https://ngrx.io/guide/store
https://ngrx.io/guide/effects

## Идеята: еднопосочен поток на данните

NgRx Store е реализация на Redux шаблона за Angular, изградена върху RxJS. Целият шаблон стъпва на една идея: **данните текат само в една посока**, по затворен кръг.

```
Компонент ──dispatch(action)──▶ Reducer ──нов state──▶ Store
    ▲                                                    │
    └──────────────── selector ◀─────────────────────────┘

Effect ◀── слуша actions ── прави страничен ефект ── dispatch(нов action)
```

Компонентът не променя състоянието директно. Той само съобщава какво се е случило, като изпраща action. Reducer-ът изчислява новото състояние от старото и от action-а. Компонентът чете състоянието чрез selectors. Всичко, което излиза извън този кръг, като HTTP заявки, навигация или localStorage, се случва в effects.

Образът е банков трезор с дневник. Никой не влиза в трезора, за да промени сумите на ръка. Всеки, който иска промяна, подава заявление на гишето. Заявлението е action. Касиерът е reducer-ът. Той прилага заявлението по строги правила и записва новия баланс. Всички, които искат да знаят баланса, гледат таблото, тоест селекторите. Външните операции, като превод към друга банка, ги прави отделен отдел, тоест effects, който после подава ново заявление с резултата. Благодарение на тази строгост всяка промяна е проследима: дневникът на заявленията обяснява точно как се е стигнало до текущия баланс.

## Actions: описание на събитие

Action е прост обект, който описва **какво се е случило**. Той има тип и по желание данни.

```typescript
export const ListingDetailsActions = createActionGroup({
  source: 'Listing Details Page',
  events: {
    'Opened': props<{ id: string }>(),
    'Favorite Toggled': props<{ id: string }>(),
  },
});

export const ListingsApiActions = createActionGroup({
  source: 'Listings API',
  events: {
    'Load Success': props<{ listing: Listing }>(),
    'Load Failure': props<{ error: HttpErrorResponse }>(),
  },
});
```

`createActionGroup` създава група actions с общ източник и генерира типове като `[Listing Details Page] Opened`. Генерира и creator функции като `ListingDetailsActions.opened({ id })`.

Добрата практика, наричана action hygiene, е actions да описват **събития**, а не **команди**. `[Listing Details Page] Opened` казва какво е станало и къде. `Load Listing` е команда, която може да се изпраща от много места и затова не казва нищо за произхода си. Уникалните actions, по един за всяко място, правят дневника в Redux DevTools четим. Те позволяват и на различни reducers и effects да реагират на едно и също събитие, без то да знае за тях.

## Reducers: чиста функция от state и action към нов state

Reducer-ът е функция, която получава текущото състояние и action и връща **ново** състояние.

```typescript
export const listingDetailsFeature = createFeature({
  name: 'listingDetails',
  reducer: createReducer(
    initialState,
    on(ListingDetailsActions.opened, (state) => ({ ...state, loading: true, error: null })),
    on(ListingsApiActions.loadSuccess, (state, { listing }) => ({ ...state, listing, loading: false })),
    on(ListingsApiActions.loadFailure, (state, { error }) => ({ ...state, error, loading: false })),
  ),
});
```

`createFeature` освен това автоматично генерира selector за всяко поле на state-а, например `selectListing` и `selectLoading`.

### Защо reducer-ът трябва да е чиста функция

Чиста функция има две свойства. При едни и същи входни данни винаги връща един и същ резултат. И няма странични ефекти: не променя аргументите си, не прави заявки, не чете `Date.now()` или `Math.random()`. За reducer-а това означава, че не мутира стария state, а връща нов обект. Причините за това правило са няколко и всяка е силен аргумент на интервю.

**Предвидимост.** Ако reducer-ът е чист, текущото състояние е изцяло определено от началното състояние и поредицата от actions. Няма скрити зависимости от време, мрежа или глобални променливи.

**Тестваемост.** Чиста функция се тества с най-простия възможен тест: подаваш state и action и проверяваш резултата. Не е нужен `TestBed`, mock-ове или асинхронност.

**Redux DevTools и time-travel.** DevTools може да превърта приложението назад и напред и да изпълнява actions отново. Това работи само ако повторното изпълнение на reducer-а дава същия резултат и не предизвиква отново странични ефекти. Ако reducer-ът пращаше HTTP заявки, всяко превъртане би ги изпращало отново.

**Откриване на промени по референция.** Това е най-техническата причина и връзката с Change Detection. Ако reducer-ът мутира съществуващия обект, референцията остава същата. Тогава `distinctUntilChanged` в `store.select`, мемоизираните selectors и компонентите с `OnPush` сравняват по референция, виждат същия обект и решават, че нищо не се е променило. Данните са променени, но екранът не се обновява. Новият обект при всяка промяна прави всяка промяна видима с евтино сравнение `===`, без дълбоко обхождане.

NgRx помага да се спази правилото. В development режим runtime проверките `strictStateImmutability` и `strictActionImmutability` са включени по подразбиране. Те замразяват state-а и actions и хвърлят грешка при опит за мутация.

## Effects: страничните ефекти извън reducer-а

Щом reducer-ът трябва да е чист, асинхронната работа трябва да живее другаде. Effect-ът слуша потока от actions, извършва страничния ефект и обикновено изпраща нов action с резултата.

```typescript
export const loadListing = createEffect(
  (actions$ = inject(Actions), listingsApi = inject(ListingsApiService)) =>
    actions$.pipe(
      ofType(ListingDetailsActions.opened),
      switchMap(({ id }) =>
        listingsApi.getListing(id).pipe(
          map((listing) => ListingsApiActions.loadSuccess({ listing })),
          catchError((error) => of(ListingsApiActions.loadFailure({ error }))),
        ),
      ),
    ),
  { functional: true },
);
```

Това е функционален effect. Зависимостите се инжектират като стойности по подразбиране на параметрите. Това работи, защото NgRx изпълнява функцията в injection context, по правилото от [[angular-dependency-injection]].

Тук се срещат двете RxJS теми от предишните файлове. Изборът на flattening оператор, тук `switchMap`, защото важи само последно отворената обява, е описан в [[rxjs-flattening-operators]]. `catchError` вътре във вътрешния поток, за да оцелее effect-ът след грешка, е описан в [[rxjs-error-handling]]. Effects, които само извършват действие, без да изпращат нов action, например навигация или логване, се маркират с `{ dispatch: false }`.

## Selectors: мемоизирано четене на състоянието

Selector-ът е функция, която извлича или изчислява част от състоянието. `createSelector` комбинира други selectors и прилага към техните резултати **projector** функция.

```typescript
export const selectVisibleListings = createSelector(
  selectListings,
  selectFilter,
  (listings, filter) => listings.filter((listing) => matchesFilter(listing, filter)),
);
```

### Мемоизацията

Selectors, създадени с `createSelector`, са **мемоизирани**. Selector-ът помни последните си входни стойности и последния резултат. Ако при следващото извикване входните стойности са същите по референция, той връща запомнения резултат, без да изпълнява projector-а отново.

Това има две ползи. Първата е производителността: скъпото филтриране по-горе се изпълнява само когато `listings` или `filter` наистина са се променили. Втората е по-важна: резултатът е **същата референция**, ако нищо не е променено. Без мемоизация `filter` би връщал нов масив при всяко изпълнение, дори със същото съдържание. Тогава `OnPush` компонентите и `distinctUntilChanged` биха виждали „промяна“ при всяка промяна на state-а, дори в несвързана част. Мемоизацията разчита на същото правило като reducer-а: промените се откриват по референция. Затова чистият reducer и мемоизираният selector са две половини на една и съща идея.

Мемоизацията пази само **последните** входни данни, не история. Projector функцията е достъпна и отделно като `selectVisibleListings.projector(...)`, което позволява тя да се тества като чиста функция, без Store.

Компонентът чете selector-а чрез `store.select(selector)`, който връща observable, или чрез `store.selectSignal(selector)`, който връща signal.

## Store, ComponentStore и SignalStore

NgRx предлага три подхода към състоянието и на интервю е добре да можеш да кажеш кога кой е подходящ.

**Store** е глобалното състояние по описания по-горе Redux шаблон. Той е подходящ за данни, споделени между много несвързани части на приложението, като текущия потребител или количка. Цената е повече код: actions, reducers, effects и selectors за всяка функционалност.

**ComponentStore** е по-лек, локален store, изграден върху `BehaviorSubject`. Обикновено се предоставя в `providers` на компонент, затова живее и умира заедно с него, по правилата за `ElementInjector` от [[angular-dependency-injection]]. Няма actions и reducers, а `updater`, `select` и `effect`. Това е подходът в DSKHome, описан в [[tapresponse-store-state]].

**SignalStore** е най-новият подход, изграден върху Signals. Той е функционален и композируем чрез `withState`, `withComputed` и `withMethods`. Може да се използва и локално, и глобално.

## Изречението за интервю

> NgRx реализира еднопосочен поток на данните. Компонентът изпраща action, който описва събитие. Reducer-ът изчислява нов state като чиста функция, effects извършват страничните ефекти и изпращат нови actions, а компонентът чете състоянието през мемоизирани selectors. Reducer-ът трябва да е чист заради предвидимостта, тестваемостта и time-travel в DevTools. Най-вече обаче промените се откриват по референция: мутиран обект запазва референцията си и `OnPush` и selectors не биха видели промяната. Мемоизираните selectors са другата половина на същата идея: при същите входове връщат същата референция, затова изгледът не се преизчислява излишно. За локално състояние на компонент използвам ComponentStore, предоставен в `providers` на компонента, а за глобално — Store или SignalStore.
