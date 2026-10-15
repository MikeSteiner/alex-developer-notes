---
tags: [rxjs, tapresponse, ngrx-component-store, state-management, разбиране]
свързани: ["[[observable-subject-behaviorsubject]]", "[[rxjs-switchMap]]"]
---
# RxJS в практиката: `tapResponse`, store/state с BehaviorSubject, cold и hot observables

## `tapResponse` — защо съществува отделно от `tap` и `catchError`

`tapResponse` идва от NgRx (исторически от `@ngrx/component-store`, в по-новите версии преместен в `@ngrx/operators`) и решава конкретен проблем, който възниква именно в контекста на `ComponentStore` ефекти като този от твоя пример. За да се разбере защо е нужен, трябва първо да се разбере какво се случва без него.

Ефект, дефиниран чрез `this.effect((source$) => ...)`, се абонира само веднъж — при създаването на store-а. Source observable-ът (в примера — `isLoggedIn$`) е дългоживеещ поток, който продължава да излъчва стойности през целия живот на компонента. Ако някъде надолу по веригата — например вътре в `switchMap` — се появи грешка, която не е прихваната, тя ще пропътува нагоре по цялата верига от оператори и ще стигне до самия `effect`, а неговата единствена абонация ще бъде терминирана заради `error`. Веднъж терминиран, ефектът никога повече няма да реагира на нови стойности от `isLoggedIn$`, дори последните вече да са напълно валидни — защото самият Observable контракт е категоричен: след `error`, потокът е мъртъв завинаги.

Точно тук идва `tapResponse`. Той приема същите аргументи като комбинация от `next` и `error` callback-и — по структура прилича на аргументите на `subscribe()` — но вместо просто да ги изпълни и да остави грешката да продължи нагоре, той я прихваща вътрешно чрез вграден `catchError`, изпълнява твоя `error` callback, и след това връща observable, който **завършва тихо**, без да разпространи грешката по-нагоре. Резултатът е, че вътрешният `switchMap` (или каквото и да е оператор, който съдържа `tapResponse`) не умира при грешка — той просто минава на изчакване за следващата стойност от source-а, точно както е предвидено при дългоживеещ ефект.

```typescript
readonly getIsFavoriteListing = this.effect((isLoggedIn$: Observable<boolean>) => {
  return isLoggedIn$.pipe(
    filter((isLoggedIn) => isLoggedIn),
    withLatestFrom(this.listingId$),
    tap(() => {
      this.resetErrorFavorite();
      this.resetSnackBarMessages();
    }),
    switchMap(([_, listingId]) => {
      return this.dskhomeFavouritesApiService.getSingleListingToFavorite(listingId).pipe(
        take(1),
        tapResponse(
          (favouriteListingResponse: FavouriteListingResponse) => {
            this.updateIsFavorite(favouriteListingResponse.favorite);
          },
          (error: HttpErrorResponse) => {
            this.setErrorFavorite(error);
            this.updateSnackBarMessages($localize`Възникна грешка, моля опитайте отново!`);
          }
        )
      );
    }),
  );
});
```

Разгледан отблизо, примерът показва и защо `tapResponse` е поставен именно вътре в `switchMap`, а не някъде по-нагоре по веригата — грешката трябва да бъде прихваната преди да напусне вътрешния observable, произведен от `switchMap`, а не след като вече е изтекла в основния поток на ефекта. `take(1)` тук е оправдан за разлика от предния пример с `afterClosed()` — HTTP заявката, върната от `getSingleListingToFavorite`, технически вече завършва сама след първия отговор, така че `take(1)` отново е формално излишен, но много екипи го запазват като изричен, самодокументиращ сигнал "тук очакваме точно една стойност", а не заради реална техническа необходимост.

`tapResponse` приема и трети, незадължителен аргумент — `finalize` callback, аналогичен на оператора `finalize()`, разгледан по-рано — който се изпълнява независимо дали `next` или `error` клонът е бил задействан, полезен за спиране на loading индикатор точно както в примера с `finalize()`.

```typescript
tapResponse(
  (response) => this.updateData(response),
  (error: HttpErrorResponse) => this.setError(error),
  () => this.setLoading(false), // трети, незадължителен finalize callback
)
```

## Store/state с `BehaviorSubject` и `Observable`: практически пример

`ComponentStore` абстрахира голяма част от ръчната работа с `BehaviorSubject`, но си струва да се разбере какво се случва под повърхността, защото същият модел се среща навсякъде, където се пише state management на ръка. Идеята е проста: state-ът живее вътре в `BehaviorSubject`, защото на state-а винаги му трябва текуща стойност, достъпна синхронно; навън се излага само readonly `Observable`, за да не може произволен консуматор да пише директно в потока в заобикаляне на контролираните методи за промяна.

```typescript
@Injectable()
export class FavouriteListingStore {
  private readonly stateSubject = new BehaviorSubject<FavouriteState>({
    isFavorite: false,
    error: null,
    loading: false,
  });

  // навън само Observable — консуматорите четат, но не могат да пишат директно
  readonly state$: Observable<FavouriteState> = this.stateSubject.asObservable();

  readonly isFavorite$: Observable<boolean> = this.state$.pipe(
    map((state) => state.isFavorite),
    distinctUntilChanged(),
  );

  updateIsFavorite(isFavorite: boolean): void {
    this.stateSubject.next({ ...this.stateSubject.value, isFavorite, error: null });
  }

  setError(error: HttpErrorResponse): void {
    this.stateSubject.next({ ...this.stateSubject.value, error });
  }
}
```

Видимостта на `stateSubject` тук не е случаен избор на модификатор — тя е приложение на общото правило, разгледано в [[observable-subject-behaviorsubject]]: Subject, който носи state, остава `private`, за да не може консуматор извън класа да инжектира произволна стойност в потока чрез директно извикан `.next()`. Ако `FavouriteListingStore` бъде преработен като базов клас, от който наследяват специализирани store-ове (например отделен store за други типове "любими" обекти), `private` естествено става `protected`, за да могат наследниците да пишат в същия `stateSubject`, докато навън, извън цялата йерархия, все пак остава достъпен само readonly `state$`.

Два детайла тук заслужават внимание. Първо, `.asObservable()` не е козметика — той връща обект, чийто тип е стеснен до `Observable`, лишен от методите `.next()`, `.error()` и `.complete()`, така че компонент, инжектирал `state$`, физически няма достъп до `.next()`, дори да опита. Второ, четенето на `this.stateSubject.value` вътре в `updateIsFavorite` е възможно единствено защото `BehaviorSubject` пази текущата стойност синхронно достъпна чрез своето `.value` свойство — нещо, което обикновен `Subject` или `Observable` никога не предлагат, защото и двата нямат понятие за "текуща" стойност извън контекста на активен абонамент.

`distinctUntilChanged()` в `isFavorite$` е добавка, която си заслужава отделно споменаване — тя предотвратява ненужни emissions надолу по веригата, когато нова стойност на state обекта се появи, но конкретното поле `isFavorite` остава непроменено (например ако `setError` презапише state-а, но `isFavorite` си остане същото). Без нея всеки `.next()` върху целия state би тригнал повторно всички derived observables, дори когато частта, от която реално се интересуват, не се е променила.

## Пример с cold observables

HTTP заявките в Angular, направени чрез `HttpClient`, са класически cold observables, и `getSingleListingToFavorite` от примера по-горе е точно такъв случай. Всеки отделен `.subscribe()` върху резултата от `this.http.get(...)` предизвиква нова, независима HTTP заявка — ако забравиш `shareReplay()` и случайно се абонираш два пъти за същия observable, ще излетят две отделни мрежови заявки, а не една, споделена между двата абоната.

```typescript
getListing(id: string): Observable<Listing> {
  return this.http.get<Listing>(`/api/listings/${id}`);
}

// две отделни абонирания = две отделни HTTP заявки
const listing$ = this.getListing('123');
listing$.subscribe((listing) => console.log('Първо абониране:', listing));
listing$.subscribe((listing) => console.log('Второ абониране:', listing));
```

Ако целта е реално споделяне на един-единствен резултат между множество абонати — например когато `listing$` се ползва едновременно в темплейта чрез `async` pipe и в компонентен метод — `shareReplay({ bufferSize: 1, refCount: true })` превръща cold observable-а в ефективно hot, кеширайки последния резултат и препредавайки го на всеки нов абонат, без да предизвиква нова HTTP заявка.

## Пример с hot observables — router и подобни

Router `events` в Angular е каноничен пример за hot observable, защото навигационните събития се случват в резултат на действия на потребителя или на извиквания на `router.navigate()`, напълно независимо от това дали някой компонент в момента слуша. Ако компонент се абонира за `router.events` едва след като навигацията вече е приключила, той просто изпуска събитията, случили се преди абонирането — точно както обикновен `Subject`, разгледан в предишния файл, няма памет за минали emissions.

```typescript
this.router.events.pipe(
  filter((event): event is NavigationEnd => event instanceof NavigationEnd),
).subscribe((event) => {
  console.log('Навигация завърши:', event.url);
});
```

`fromEvent(document, 'click')` е другият типичен hot пример, разгледан и в предишния файл — кликовете на потребителя текат независимо от кода, а ролята на `subscribe()` е единствено да реши дали и кога да "влезе в стаята" и да започне да слуша. Разликата спрямо cold observable като HTTP заявка е фундаментална: тук абонирането никога не предизвиква самото събитие — то просто определя от кой момент нататък абонатът ще получава известия за нещо, което би се случило независимо от всякакъв абонамент.