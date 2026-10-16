---
tags: [angular, components, content-projection, ng-content, host-element, queries, viewChild, contentChild, разбиране]
свързани: ["[[angular-component-lifecycle]]", "[[angular-dependency-injection]]", "[[angular-inputs-outputs-model]]", "[[change-detection-onpush]]", "[[angular-components-basics]]"]
---
# Angular: Content projection, host element и queries

https://angular.dev/guide/components/content-projection
https://angular.dev/guide/components/host-elements
https://angular.dev/guide/components/queries

## Отвън, отвътре и стената

Всеки компонент има три части, които трябва да се различават ясно.

**View** е собственият темплейт на компонента: всичко, което той сам описва и създава. **Content** са децата, които родителят поставя между отварящия и затварящия таг на компонента и които компонентът показва чрез `<ng-content>`. **Host element** е самият таг на компонента в DOM, например `<app-card>`. Той е границата между двете. Темплейтът на компонента се рендерира вътре в него, а родителят го вижда като обикновен елемент от своя темплейт.

Тази разлика вече се появи на две места. В [[angular-component-lifecycle]] тя разделя `ngAfterContentInit` и `ngAfterViewInit`. В [[angular-dependency-injection]] тя разделя `providers` и `viewProviders`. Този файл я разглежда от трета страна: как компонентът показва чуждото съдържание, как управлява собствената си стена и как достига до децата си.

## Content projection с `<ng-content>`

`<ng-content>` е място в темплейта на компонента, където Angular поставя децата, подадени от родителя. Документацията го сравнява с native елемента `<slot>` от Web Components, с допълнителни възможности.

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content />
    </div>
  `,
})
export class Card {}
```

```html
<app-card>
  <p>Описание на обявата</p>
</app-card>
```

Параграфът се рендерира вътре в `.card`. Той е **content** на `Card`, а `div.card` е част от неговото **view**.

### Няколко слота: `select`

Атрибутът `select` приема CSS селектори, същите като при селектора на компонент. Всеки проектиран елемент отива в слота, чийто селектор отговаря.

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <header class="card__header">
        <ng-content select="[card-title]" />
      </header>
      <ng-content />
      <footer class="card__footer">
        <ng-content select="[card-actions]" />
      </footer>
    </div>
  `,
})
export class Card {}
```

```html
<app-card>
  <h3 card-title i18n>Двустаен апартамент</h3>
  <p i18n>Слънчев, с изглед към Витоша.</p>
  <button card-actions i18n>Запази</button>
</app-card>
```

`<ng-content>` без `select` е **слотът по подразбиране**. В него отива всичко, което не е отговорило на друг слот. Ако компонентът няма слот по подразбиране, такова съдържание изобщо не се рендерира. Няма грешка, то просто изчезва.

Съдържание, поставено вътре в `<ng-content>`, е **резервно**. Показва се, ако нищо не е проектирано в този слот: `<ng-content select="[card-actions]">Няма действия</ng-content>`.

`ngProjectAs` позволява елемент да попадне в слот, на чийто селектор сам не отговаря. Например `<ng-container ngProjectAs="[card-title]">` се третира като заглавие. Стойността трябва да е статична и не може да е израз.

### `<ng-content>` не е DOM елемент

`<ng-content>` е място, което компилаторът обработва при build. Той не е компонент и не е DOM възел. Затова не може да се добавя, маха или променя по време на изпълнение. Не може да му се сложат директиви, стилове или атрибути.

### Най-важният капан: проектираното съдържание винаги се създава

Документацията изрично предупреждава `<ng-content>` да не се поставя в `@if`, `@for` или `@switch`. Angular **винаги** създава инстанциите и DOM възлите на проектираното съдържание, дори когато мястото, където трябва да се покаже, е скрито.

```html
<!-- Не прави това -->
@if (expanded()) {
  <ng-content />
}
```

Тук съдържанието се създава веднага, независимо от `expanded()`. Компонентите в него минават `ngOnInit`, правят HTTP заявките си и заемат памет, но не се виждат. Причината следва от правилото по-горе: съдържанието е написано в темплейта на родителя, затова родителят го създава, а не компонентът, който го проектира.

Ако съдържанието наистина трябва да се създава само при нужда, то се подава като **template fragment**, тоест `<ng-template>`. Компонентът го взима с content query и го рендерира с `NgTemplateOutlet`, когато прецени.

```typescript
@Component({
  selector: 'app-expander',
  imports: [NgTemplateOutlet],
  template: `
    <button (click)="expanded.set(!expanded())" i18n>Покажи още</button>
    @if (expanded()) {
      <ng-container [ngTemplateOutlet]="content()" />
    }
  `,
})
export class Expander {
  protected readonly expanded = signal(false);
  protected readonly content = contentChild.required(TemplateRef);
}
```

```html
<app-expander>
  <ng-template>
    <app-listing-documents />
  </ng-template>
</app-expander>
```

Сега `app-listing-documents` се създава едва при първото разгъване. Същият шаблон се използва в [[angular-dependency-injection]], когато проектираното съдържание трябва да достигне `viewProviders` на компонента.

### Проектираното съдържание принадлежи на родителя

Документацията посочва още две следствия от същото правило.

**Change detection.** Проектираното съдържание е част от view-то на родителя и се проверява, когато се проверява родителят. OnPush компонентът, който го проектира, не може да го „прескочи“. Това е важно при четенето на сценариите в [[change-detection-onpush]].

**Dependency injection.** Проектираното съдържание търси зависимостите си от injector-а на родителя. То вижда `providers` на компонента, в който е проектирано, но не и неговите `viewProviders`.

**Библиотечни компоненти.** Менюта, табове и списъци в библиотеките често намират децата си с `contentChildren`. Ако децата им се обвият в допълнителен компонент и се проектират вътре, query-то не ги намира, защото не минава през чужд компонент, както е описано по-долу. Клавиатурната навигация, фокусът и ARIA атрибутите може да се счупят тихо, без грешка.

## Host element

Host елементът е DOM елементът, който отговаря на селектора на компонента. Angular създава по една инстанция на компонента за всеки такъв елемент, а темплейтът се рендерира вътре в него. Затова host елементът е единственото място, до което компонентът има достъп отвън на собствения си темплейт.

### Свойството `host`

Bindings и listeners върху host елемента се декларират в свойството `host` на декоратора. Синтаксисът е същият като в темплейт.

```typescript
@Component({
  selector: 'app-rating',
  template: `...`,
  host: {
    'role': 'slider',
    '[attr.aria-valuenow]': 'value()',
    '[class.rating--disabled]': 'disabled()',
    '[tabIndex]': 'disabled() ? -1 : 0',
    '(keydown.arrowRight)': 'increment()',
  },
})
export class Rating {
  readonly value = model(0);
  readonly disabled = input(false, { transform: booleanAttribute });

  protected increment() {
    this.value.update((value) => Math.min(value + 1, 5));
  }
}
```

Така компонентът сам управлява собствената си достъпност, класове и клавиатура, без потребителят да трябва да ги добавя всеки път. Глобални събития се слушат с префиксите `document:`, `window:` и `body:`, например `'(window:resize)': 'onResize()'`.

Имената на клавиши като `keydown.enter` отговарят на `KeyboardEvent.key`, който зависи от клавиатурната подредба. За физически клавиш се използва модификаторът `code`, например `keydown.code.Enter`. Това е важно в приложение, което работи с кирилица.

Декораторите `@HostBinding` и `@HostListener` правят същото, но документацията казва, че те съществуват **само заради обратна съвместимост**. За нов код се препоръчва свойството `host`.

### Кой печели при конфликт

Ако компонентът задава host binding, а родителят задава binding на същото свойство в темплейта си, победителят се определя по таблица:

| Binding в темплейта на родителя | Host binding на компонента | Печели |
|---|---|---|
| Статичен | Статичен | Темплейтът на родителя |
| Статичен | Динамичен | Host binding-ът |
| Динамичен | Статичен | Темплейтът на родителя |
| Динамичен | Динамичен | Host binding-ът |

Правилото за запомняне: **динамичното печели пред статичното**, а при равенство статичната стойност на родителя печели, докато при два динамични binding-а печели компонентът.

### Атрибути и CSS променливи

Статичен атрибут на host елемента се чете с `inject(new HostAttributeToken('variation'))`. Ако атрибутът липсва, това хвърля грешка, освен ако инжектирането не е маркирано като `optional`. Подходящо е за стойности, които не се променят, като вариант на бутон. За стойности, които се променят, правилният инструмент е input.

CSS custom property може да се зададе на host елемента чрез `'[style.--card-accent]': 'accent()'`. Така стиловете на компонента и децата му се обновяват автоматично, когато signal-ът се промени.

## Queries: как компонентът достига до децата си

Queries позволяват на компонента да получи референция към елементи, компоненти, директиви или инжектирани стойности от своето view или content. Функциите `viewChild`, `viewChildren`, `contentChild` и `contentChildren` връщат **signals**, които винаги отразяват текущия резултат.

```typescript
export class ListingGallery {
  private readonly slider = viewChild.required(Slider);
  private readonly thumbnails = viewChildren(Thumbnail);
  private readonly caption = contentChild('caption', { read: ElementRef });

  protected readonly thumbnailCount = computed(() => this.thumbnails().length);
}
```

**View queries**, тоест `viewChild` и `viewChildren`, търсят в собствения темплейт на компонента. **Content queries**, тоест `contentChild` и `contentChildren`, търсят в проектираното съдържание.

`viewChild` и `contentChild` връщат един резултат или `undefined`, например когато елементът е скрит от `@if`. `viewChildren` и `contentChildren` връщат масив. С `.required()` типът не включва `undefined`, а липсата на резултат е грешка.

### Queries никога не минават през граница на компонент

Това е най-важното правило и най-честият източник на объркване. View query търси само в собствения темплейт на компонента, а не в темплейтите на децата му. Content query търси само в съдържанието, написано в **същия темплейт, където е използван компонентът**. Ако родител търси `viewChild(Button)`, а бутонът е вътре в темплейта на `Card`, която е в темплейта на родителя, родителят не го намира. Намира само `Card`.

Това е капсулацията, която прави компонентите независими. Ако родителят трябва да стигне до нещо по-дълбоко, правилният път е договорът на детето: inputs, outputs или публичен метод, описани в [[angular-inputs-outputs-model]].

### Какво може да се търси: locators

Query-то приема един от три вида locator.

- **Тип на компонент или директива**: `viewChild(Slider)`
- **Име на template reference променлива**: `viewChild<ElementRef<HTMLInputElement>>('search')` за `<input #search>`. Ако няколко елемента имат същото име, връща се първият.
- **Injection token**, предоставен от компонент или директива: `contentChild(TAB_ITEM)`

CSS селектори **не** се поддържат.

Опцията `read` указва какво точно да се върне от намерения елемент. Без нея за компонент се връща инстанцията му, а за обикновен елемент се връща `ElementRef`. С `read` може да се поиска `ElementRef` на компонент, `TemplateRef`, `ViewContainerRef` или `Injector`. Пример: `viewChild(Slider, { read: ElementRef })` връща host елемента на слайдера, а не инстанцията му.

### `descendants`

`contentChildren` по подразбиране търси само **преките** деца в проектираното съдържание. `contentChild` по подразбиране търси във всички нива. С `descendants: true` и `contentChildren` търси във всички нива на същия темплейт. View queries винаги търсят във всички нива и нямат такава опция. И в двата случая търсенето спира на границата на чужд компонент.

### Кога резултатите са налични

При signal queries въпросът почти изчезва. Signal-ът може да се чете по всяко време, а стойността му е текущото състояние. Той участва в `computed` и `effect` и се обновява сам, когато например `@if` покаже или скрие елемента.

При декораторните queries таймингът е важен и се пита на интервю. Резултатът на `@ViewChild` е наличен в `ngAfterViewInit`, а на `@ContentChild` в `ngAfterContentInit`. Преди това е `undefined`. С `static: true` резултатът е наличен още в `ngOnInit`, но не се обновява след това, затова целта трябва да е винаги налична, тоест да не е вътре в `@if` или `@for`.

### Signal срещу декораторни queries

| Signal функция | Декоратор | Резултат |
|---|---|---|
| `viewChild`, `viewChild.required` | `@ViewChild` | един резултат |
| `viewChildren` | `@ViewChildren` | масив срещу `QueryList` |
| `contentChild`, `contentChild.required` | `@ContentChild` | един резултат |
| `contentChildren` | `@ContentChildren` | масив срещу `QueryList` |

`QueryList` има методи като на масив, например `map`, `forEach` и `toArray()`, а промените се следят чрез observable-а `changes`. Signal queries заменят и двете неща: масивът е обикновен масив, а промяната е самият signal. Екипът на Angular препоръчва signal queries за нов код.

### Капаните, които документацията изброява

Първият е опитът за query през граница на компонент, описан по-горе.

Вторият е **два източника на истината**. Ако едно и също състояние се пази и в родителя, и в детето, двете неизбежно се разминават.

Третият и най-коварният е **записване на състояние в дете или родител през резултата от query**. Пример е `this.slider().value = 5` или промяна на поле на родителя от `ngAfterViewInit` на детето. Освен че е крехко, това води до `ExpressionChangedAfterItHasBeenCheckedError` (NG0100), описана в [[angular-component-lifecycle]]. Към момента на `ngAfterViewInit` bindings са вече проверени, а записът ги променя. Правилният път за данни към детето е input, а към родителя е output.

Queries са най-подходящи за **императивни** операции, които нямат естествен израз в темплейта. Примери са фокус на поле, измерване на размер, превъртане до елемент и извикване на метод на библиотечен компонент, като `slider.reset()`. За данни правилният път остава договорът от [[angular-inputs-outputs-model]].

## Изречението за интервю

> Компонентът има view, тоест собствения си темплейт, content, тоест децата, проектирани с `<ng-content>`, и host element, тоест собствения си таг. Проектираното съдържание е написано в темплейта на родителя и му принадлежи. Затова винаги се създава, дори `<ng-content>` да е в `@if`, и затова за мързеливо съдържание се подава `<ng-template>` и се рендерира с `NgTemplateOutlet`. По същата причина се проверява с change detection-а на родителя и не вижда `viewProviders`. Host елементът се управлява със свойството `host`, а `@HostBinding` е само за обратна съвместимост. Queries достигат децата: `viewChild` в собствения темплейт, `contentChild` в проектираното съдържание. Те никога не минават през граница на компонент. Signal queries са налични по всяко време, докато декораторните чакат `ngAfterViewInit` или `ngAfterContentInit`. Използвам ги за императивни неща като фокус и измерване, а не за записване на данни, защото това води до NG0100.
