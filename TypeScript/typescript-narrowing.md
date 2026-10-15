# TypeScript: Narrowing и Type Predicates

https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates

рецизирането на типа до нещо по-тясно от декларираното се нарича _narrowing_, а проверките, които го предизвикват, са _type guards_.

## Основната идея в един пример

Функция, която връща обикновен `boolean`, казва на компилатора само „да“ или „не“. Не казва какво означава това „да“ за аргумента.

```typescript
function isNotFoundStatus(status: number): boolean {
  return status === 404 || status === 410;
}

if (isNotFoundStatus(error.status)) {
  error.status; // все още number
}
```

Функция с type predicate описва в сигнатурата си какво следва от `true`. Компилаторът използва това, за да стесни типа на аргумента в съответния клон.

```typescript
type NotFoundStatus = 404 | 410;

function isNotFoundStatus(status: number): status is NotFoundStatus {
  return status === 404 || status === 410;
}

if (isNotFoundStatus(error.status)) {
  error.status; // може да бъде само NotFoundStatus — 404 или 410
}
```

По време на изпълнение двете функции са еднакви и връщат `true` или `false`. Анотацията `status is NotFoundStatus` изчезва при компилация. Разликата съществува само в типовата система, но тя решава дали следващият ред ще се компилира.

Една метафора: функцията е охранител на вход. С `boolean` охранителят пуска госта, но никой вътре не знае какво е проверено, и всички продължават да го третират като непознат `number`. С type predicate охранителят слага на госта гривна с надпис `404 | 410`. Всеки ред в `if` блока вижда гривната.

## Деклариран и наблюдаван тип

TypeScript разграничава два типа на една и съща стойност. *Декларираният тип* е този, с който стойността е обявена, например `HttpErrorResponse.status: number`. *Наблюдаваният тип* е по-конкретният тип, който компилаторът изчислява в дадена точка, като проследява пътищата на изпълнение: `if/else`, тернарни оператори, `switch`, ранни `return`. Този анализ се нарича *control flow analysis*. Прецизирането на типа до нещо по-тясно от декларираното се нарича *narrowing*, а проверките, които го предизвикват, са *type guards*.

Присвояванията винаги се проверяват спрямо декларирания тип. Стесняването не забранява на променливата по-късно да получи друга стойност от декларирания тип. То само описва какво знаем в конкретната точка.

## Вградените type guards

Компилаторът разпознава конструкции от JavaScript, без да е нужно нещо допълнително:

```typescript
function example(value: string | number | Date | null, status: number) {
  if (typeof value === 'string') { value; }   // string
  if (value instanceof Date) { value; }       // Date
  if (value !== null) { value; }              // string | number | Date
  if (status === 404 || status === 410) {
    status;                                   // 404 | 410
  }
}
```

Към тях спадат и операторът `in` за обекти, проверките за истинност (`if (value)`) и discriminated unions, при които общо литерално поле като `kind` разграничава членовете на union.

## Защо информацията се губи при функция

Докато проверката стои директно в `if`, компилаторът я вижда и стеснява типа. Щом я изнесем във функция, на мястото на извикване TypeScript вижда само сигнатурата, например `(status: number) => boolean`. Тялото на функцията се анализира, когато се проверява самата функция, но заключението не се пренася при извикването. Функцията е затворена кутия, от която излиза само `boolean`.

Type predicate решава точно това. Той пренася смисъла на `true` от тялото на функцията в сигнатурата ѝ, откъдето компилаторът може да го прочете при всяко извикване.

## Синтаксис и правила

Предикатът има формата `parameterName is Type` и стои на мястото на върнатия тип. `parameterName` трябва да е името на параметър от същата сигнатура. `Type` трябва да е съвместим с типа на параметъра, тоест негов подтип. `404 | 410` е подмножество на `number`, затова е валидно. `status is string` при параметър `number` би дало грешка.

Тялото трябва да връща `boolean`. Компилаторът приема, че `true` означава „аргументът е от тип `Type`“.

## Клоновете `if` и `else`

Когато параметърът е union, стесняването работи в двете посоки. Ако `isFish(pet: Fish | Bird): pet is Fish` върне `true`, `pet` е `Fish`. Ако върне `false`, компилаторът изважда `Fish` от union-а и `pet` става `Bird`.

При `isNotFoundStatus` параметърът е `number`, който не е union, който TypeScript може да разложи. Няма тип „всички числа без 404 и 410“, затова в `else` клона `error.status` остава `number`. Стесняването тук е еднопосочно. Ако параметърът беше `404 | 410 | 500`, `else` клонът щеше да стане `500`.

---

## Пример от реален код: Angular SSR и TransferState

В store-а за детайли на обява сървърът записва статус „не е намерено“ в `TransferState`, за да не изпраща браузърът втора заявка при hydration:

```typescript
const notFoundStatusKey = makeStateKey<NotFoundStatus>(`listing-details-not-found-status:${id}`);

if (this.platformDataService.isServer && isNotFoundStatus(error.status)) {
  this.transferState.set(notFoundStatusKey, error.status);
}
```

Ключът е `StateKey<NotFoundStatus>`, затова `set` приема само `404 | 410`. Без предиката `error.status` е `number` и редът дава грешка *Argument of type 'number' is not assignable to parameter of type 'NotFoundStatus'*. Единственото решение тогава би било `error.status as NotFoundStatus`. Такова твърдение не се проверява и остава в кода дори ако някой махне защитната проверка.

Същото важи за селектора, чийто тип е зададен явно:

```typescript
readonly notFoundErrorStatus$: Observable<NotFoundStatus | undefined> = this.select(this.error$, (error) =>
  error && isNotFoundStatus(error.status) ? error.status : undefined);
```

Тернарният оператор също е guard. След `?` `error.status` е `404 | 410`, затова изразът е `NotFoundStatus | undefined` и съвпада с анотацията.

Стесняването работи и върху достъп до свойство като `error.status`, не само върху локални променливи, стига между проверката и използването да няма присвояване.

## Стесняване през `const` псевдоним

От TypeScript 4.4 условието може да се запише в `const` и стесняването се запазва:

```typescript
const isNotFound = isNotFoundStatus(error.status);

if (isNotFound && this.platformDataService.isServer) {
  this.transferState.set(notFoundStatusKey, error.status); // 404 | 410
}
```

Това работи, когато стесняваната референция е `const`, параметър, който не се преприсвоява, или `readonly` свойство. `error` е непроменян параметър, а `HttpErrorResponse.status` е `readonly`, затова условията са изпълнени.

## Предикати във `filter`

Масивите и RxJS имат overload на `filter`, който приема предикат и стеснява типа на резултата:

```typescript
const statuses: number[] = [200, 404, 500, 410];
const notFound = statuses.filter(isNotFoundStatus); // NotFoundStatus[]

const notFoundStatus$ = errorStatus$.pipe(
  filter(isNotFoundStatus)
); // Observable<NotFoundStatus>
```

С функция, която връща `boolean`, резултатът остава `number[]` и `Observable<number>`.

## Свързани механизми

*Assertion functions* със сигнатура `asserts value is Type` не връщат `boolean`, а хвърлят грешка, ако проверката не мине. След успешното извикване стесняването важи за целия следващ код, не само за `if` клона:

```typescript
function assertNotFoundStatus(status: number): asserts status is NotFoundStatus {
  if (status !== 404 && status !== 410) {
    throw new Error(`Unexpected status ${status}`);
  }
}

assertNotFoundStatus(error.status);
error.status; // 404 | 410
```

От TypeScript 5.5 компилаторът в някои случаи сам извежда предикат за функции без явен върнат тип, чието тяло е просто стесняващо условие. Явната анотация е по-ясна за читателя и не зависи от правилата за извеждане. Ако обаче върнатият тип бъде анотиран като `: boolean`, извеждането се изключва и стесняването се губи.

## Упражнения

[[flashcards-typescript-narrowing]]
