---
tags: [angular, components, inputs, outputs, model, signals, разбиране]
свързани: ["[[angular-components-basics]]", "[[angular-component-lifecycle]]", "[[change-detection-onpush]]", "[[change-detection-signals-zoneless]]", "[[angular-dependency-injection]]"]
---
# Angular: Inputs, outputs и `model()`

https://angular.dev/guide/components/inputs
https://angular.dev/guide/components/outputs

## Договорът на компонента

Компонентът е част от по-голямо дърво и общува с родителя си по два канала. **Inputs** са данните, които влизат отвън: родителят ги подава чрез property binding `[value]="..."`. **Outputs** са събитията, които излизат навън: компонентът ги излъчва, а родителят ги слуша чрез event binding `(changed)="..."`. Заедно те са публичният договор на компонента. Всичко останало в класа е вътрешна работа.

Образът е уред с гнезда. Inputs са входовете, в които родителят включва кабели. Уредът може да чете какво идва по тях, но не може да праща обратно по същия кабел. Outputs са изходите: уредът сигнализира навън, а кой слуша решава родителят. `model()` е двупосочно гнездо: по един и същ кабел стойността идва отвън и се връща навън.

Това разделение е основата на еднопосочния поток на данните в Angular. Данните текат надолу по дървото чрез inputs, а събитията текат нагоре чрез outputs. Същата идея стои зад NgRx, описан в [[ngrx-architecture]], само че на ниво компонент.

## Signal inputs: `input()`

От Angular 17.1 насам препоръчителният начин за деклариране на input е функцията `input()`. Тя връща `InputSignal`, тоест signal само за четене.

```typescript
export class PriceBox {
  readonly price = input(0);
  readonly currency = input<string>();
  readonly listingId = input.required<string>();
}
```

```html
<app-price-box [price]="listing.price" [listingId]="listing.id" />
```

Типът се извежда от стойността по подразбиране, затова `input(0)` е `InputSignal<number>`. Без стойност по подразбиране, както при `input<string>()`, типът е `string | undefined`, защото докато родителят не подаде стойност, тя е `undefined`.

`input.required<T>()` прави input-а задължителен. Ако темплейт използва компонента, без да подаде задължителен input, грешката е **при компилация**, а не по време на изпълнение. Типът не включва `undefined`. Задължителен input не може да се прочете в конструктора, защото тогава родителят още не го е подал. Опитът хвърля грешка. Първото място, където е сигурно наличен, е `ngOnInit`, а най-естественото място са `computed` и темплейтът.

Inputs се регистрират от компилатора. Затова не могат да се добавят или махат по време на изпълнение, а `input()` може да се извика **само във field initializer** на компонент или директива. Това е подобно на правилото за `inject()` от [[angular-dependency-injection]].

### Четене: signal като всеки друг

Input-ът се чете чрез извикване, като всеки signal. Затова участва в реактивния граф и може да се използва в `computed` и `effect`.

```typescript
export class PriceBox {
  readonly price = input.required<number>();
  readonly currency = input('EUR');

  protected readonly formattedPrice = computed(() => `${this.price().toLocaleString()} ${this.currency()}`);
}
```

Това е голямата практическа разлика спрямо стария `@Input`. За да се реагира на промяна в декораторен input, трябваше `ngOnChanges` със `SimpleChanges` или setter. С signal input реакцията е декларативна: `computed` се преизчислява сам, когато input-ът се промени. В повечето случаи `ngOnChanges` вече не е нужен.

### Inputs са само за четене

`InputSignal` няма `set` и `update`. Компонентът не може да промени собствения си input. Това е съзнателно. Стойността принадлежи на родителя, и ако детето можеше да я презаписва, двете страни биха имали различна представа за истината.

Честият въпрос е какво да се направи, когато компонентът трябва да започне със стойността от родителя, но после да я променя локално. Пример е поле за редакция, което получава начален текст. Има два правилни отговора, според това дали промяната трябва да стигне до родителя.

Ако промяната трябва да се върне към родителя, правилният инструмент е `model()`, описан по-долу.

Ако промяната е локална, правилният инструмент е `linkedSignal`. Той създава записваем signal, който се нулира към стойността на input-а всеки път, когато input-ът се промени.

```typescript
export class TitleEditor {
  readonly initialTitle = input.required<string>();
  protected readonly draft = linkedSignal(() => this.initialTitle());

  protected onInput(value: string) {
    this.draft.set(value);
  }
}
```

### Transforms

Опцията `transform` преобразува стойността, преди да бъде записана в input-а. Типът на параметъра на функцията определя какво може да се подаде от темплейта, а типът на резултата е типът на input-а.

```typescript
readonly label = input('', { transform: trimString });

function trimString(value: string | undefined) {
  return value?.trim() ?? '';
}
```

Angular предлага два вградени transform-а. `booleanAttribute` работи като HTML булев атрибут: самото присъствие на атрибута означава `true`, а низът `"false"` става `false`. `numberAttribute` превръща стойността в число и връща `NaN`, ако това е невъзможно.

```typescript
readonly disabled = input(false, { transform: booleanAttribute });
```

```html
<app-button disabled />
```

Документацията поставя две изисквания. Transform-ът трябва да може да се анализира статично, тоест не може да се присвоява условно или от израз. И трябва да е **чиста функция**, защото зависимост от външно състояние води до непредвидимо поведение.

### Aliases

Опцията `alias` променя името на input-а в темплейта, без да променя името на свойството в класа: `input(0, { alias: 'sliderValue' })` се свързва като `[sliderValue]`. Документацията не го препоръчва като общ подход. Използва се при преименуване, когато старото име трябва да продължи да работи, или за да се избегне сблъсък с native DOM свойство. В `SimpleChanges` ключът винаги е името на свойството, а не alias-ът, както е описано в [[angular-component-lifecycle]].

## Outputs: `output()`

`output()` декларира събитие на компонента. Тя връща `OutputEmitterRef`, а събитието се излъчва с `emit()`.

```typescript
export class FavoriteButton {
  readonly listingId = input.required<string>();
  readonly favoriteToggled = output<string>();

  protected onClick() {
    this.favoriteToggled.emit(this.listingId());
  }
}
```

```html
<app-favorite-button [listingId]="listing.id" (favoriteToggled)="onFavoriteToggled($event)" />
```

В темплейта `$event` съдържа излъчената стойност. Тя може да е примитив или обект. За събитие без данни се използва `output<void>()`.

Има няколко свойства, които на интервю е добре да се знаят.

Първо, собствените събития **не правят bubbling**. За разлика от native DOM събитие като `click`, `favoriteToggled` стига само до прекия родител, който го слуша в темплейта си. Ако събитието трябва да мине през няколко нива, всяко ниво трябва изрично да го препредаде или състоянието трябва да се премести в споделен service.

Второ, `output()`, подобно на `input()`, може да се извика само във field initializer.

Трето, за имената важат същите правила като за inputs: без сблъсък с имена на DOM събития и без префикс `on`. Името описва какво се е случило, `favoriteToggled`, а не какво да се направи, `onToggle`. Това е същата хигиена като при NgRx actions в [[ngrx-architecture]].

### Абониране от кода

Ако компонентът е създаден динамично, например с `ViewContainerRef.createComponent`, няма темплейт, в който да се слуша събитието. Тогава се абонира директно към output-а през инстанцията.

```typescript
const ref = this.viewContainerRef.createComponent(FavoriteButton);
ref.setInput('listingId', listing.id);

ref.instance.favoriteToggled.subscribe((listingId) => this.store.toggleFavorite(listingId));
```

`subscribe` връща `OutputRefSubscription`. Angular премахва абонамента автоматично, когато унищожи компонента, но той може да се прекрати и ръчно с `unsubscribe()`. `setInput` е правилният начин да се подаде input на динамичен компонент. Той известява Angular за промяната и затова е едно от известията, които задействат change detection в zoneless приложение, описани в [[change-detection-signals-zoneless]].

### `OutputEmitterRef` не е `EventEmitter`

Старият `@Output` използва `EventEmitter`, който наследява RxJS `Subject`. Това позволяваше да се прилагат оператори директно към output-а, но и смесваше понятията: output-ът беше пълноценен observable с `error` и `complete`. `OutputEmitterRef` е по-тесен. Той има само `emit` и `subscribe` и не зависи от RxJS.

Мостът към RxJS е в `@angular/core/rxjs-interop`. `outputFromObservable(obs$)` превръща observable в output, а `outputToObservable(ref.instance.someOutput)` прави обратното. Това е полезно, когато събитието естествено идва от поток, например от `valueChanges` на форма с `debounceTime`.

## `model()`: двупосочна връзка

`model()` създава input, в който компонентът **може и да пише**. Записът се връща към родителя чрез двупосочно свързване. Това е синтаксисът `[(...)]`, наричан „банан в кутия“.

```typescript
export class QuantityPicker {
  readonly quantity = model(1);

  protected increment() {
    this.quantity.update((value) => value + 1);
  }
}
```

```html
<app-quantity-picker [(quantity)]="selectedQuantity" />
```

В родителя `selectedQuantity` може да е signal или обикновено свойство. Ако е signal, се подава **самият signal**, без извикване: `[(quantity)]="selectedQuantity"`, а не `selectedQuantity()`. Това е често срещана грешка.

Как работи под повърхността: декларацията на `quantity = model(1)` създава едновременно input `quantity` и **неявен output** `quantityChange`. Всяко извикване на `set` или `update` излъчва `quantityChange`. Затова `[(quantity)]="x"` е съкратен запис на `[quantity]="x" (quantityChange)="x = $event"`, и родителят може да слуша `quantityChange` и отделно.

`model` поддържа `required` и `alias` като обикновен input. **Не поддържа** `transform`, защото стойността трябва да може да се върне към родителя в същия вид.

### Кога `model`, кога `input` с `output`

Документацията препоръчва `model()`, когато целта на компонента е да **променя стойност** в резултат на действие на потребителя. Типичният случай са собствени контроли: избор на дата, combobox, превключвател, слайдер.

Когато компонентът само съобщава, че **нещо се е случило**, правилният избор са отделни `input` и `output`. Пример е бутон за любима обява, който излъчва `favoriteToggled`, а решението какво да се промени остава у родителя или в store. Ако такъв компонент използва `model`, той сам решава състоянието на родителя и границата на отговорността се размива.

Добро правило: `model` е за **стойност**, която двете страни споделят, а `output` е за **събитие**, на което родителят реагира, както прецени.

## Inputs и change detection

Signal inputs се вписват естествено в OnPush и zoneless модела от [[change-detection-onpush]] и [[change-detection-signals-zoneless]]. Нова стойност на input през template binding е една от причините OnPush компонент да бъде проверен. Сравнението е с `Object.is`, тоест по референция. Затова мутация на обект, подаден като input, без нова референция не се вижда, независимо дали input-ът е декораторен или signal. Signal input, прочетен в темплейта, се държи като всеки друг signal: промяната му маркира само view-тата, които го четат.

## Декораторите `@Input` и `@Output`

Декораторният API е напълно поддържан, но екипът на Angular препоръчва `input()` и `output()` за нов код. Синтаксисът в темплейта е един и същ, затова двата подхода могат да съжителстват в един проект.

```typescript
@Input({ required: true }) listingId!: string;
@Input({ transform: booleanAttribute }) disabled = false;
@Output() favoriteToggled = new EventEmitter<string>();
```

Разликите, които си заслужава да се назоват на интервю:

- `@Input` е обикновено свойство, затова реакцията на промяна изисква `ngOnChanges` или setter. `input()` е signal, затова реакцията е `computed` или `effect`.
- `@Input` може да се презапише от самия компонент, което е източник на бъгове. `input()` е само за четене.
- `@Input({ required: true })` изисква `!` в TypeScript, защото компилаторът не знае, че Angular ще го попълни. `input.required` има точен тип без трикове.
- `@Output` използва `EventEmitter`, който е RxJS `Subject`. `output()` използва по-тесния `OutputEmitterRef`.

Декораторен input може да е и двойка getter и setter. Документацията препоръчва transform вместо това, когато е възможно, и предупреждава setter-ът да е евтин, защото Angular може да го извиква многократно. При наследяване inputs и outputs от базовия клас се наследяват. Те могат и да се декларират изрично в масивите `inputs` и `outputs` на `@Component`, по желание с alias след двоеточие.

## Изречението за интервю

> Inputs и outputs са публичният договор на компонента: данните влизат отгоре чрез inputs, а събитията излизат нагоре чрез outputs, което пази еднопосочния поток. `input()` връща signal само за четене, затова реакцията на промяна е `computed` вместо `ngOnChanges`. `input.required` се проверява при компилация. Transforms като `booleanAttribute` трябва да са чисти функции. Ако компонентът трябва да промени стойността, тя не принадлежи само на него. Затова за локално копие използвам `linkedSignal`, а за двупосочна връзка `model()`, който създава input и неявен output `nameChange`, и така `[(name)]` работи. `output()` връща `OutputEmitterRef`, а не RxJS `Subject`, и събитията му не правят bubbling. Правилото ми е: `model` за стойност, която двете страни споделят, а `output` за събитие, на което родителят реагира сам.
