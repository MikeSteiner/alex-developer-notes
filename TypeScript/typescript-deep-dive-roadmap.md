# TypeScript: Deep dive plan

Основа: официалният TypeScript Handbook https://www.typescriptlang.org/docs/handbook/ ★ отбелязва личните приоритети. Връзката след точка сочи към вече написан файл. Точка без връзка още няма файл.

Входната точка за бележките по TypeScript е [[TypeScript]]. Свързан план: [[angular-deep-dive-roadmap]].

Фокусът е TypeScript така, както се използва в Angular приложение и се пита на Senior интервю за frontend, а не пълният език.

---

## 1. Основи на типовата система

- Структурно типизиране: съвместимост по форма, а не по име
- `type` срещу `interface`: кога кое, declaration merging
- `any`, `unknown` и `never`
- Literal types и `as const`
- `enum` срещу union от literal types

## 2. ★ Narrowing и безопасност

- Вградени type guards, type predicates, assertion functions и `filter` [[typescript-narrowing]]
- Discriminated unions
- Exhaustive проверка с `never`
- `satisfies` срещу анотация на тип и срещу `as`
- Strict режим: `strictNullChecks`, `noImplicitAny`, `noUncheckedIndexedAccess`

## 3. ★ Generics

- Generic функции, класове и интерфейси
- Ограничения с `extends`
- Стойности по подразбиране на generic параметри
- Как Angular използва generics: `InjectionToken<T>`, `input<T>()`, `signal<T>()`

## 4. Трансформации на типове

- `keyof`, `typeof` и indexed access types
- Utility types: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `ReturnType`, `Parameters`, `Awaited`
- Mapped types
- Conditional types и `infer`
- Template literal types

## 5. TypeScript в Angular проекта

- Декоратори: експерименталните TypeScript декоратори срещу стандартните
- `import type` и изтриването на типове при компилация, връзка с DI токените [[angular-dependency-injection]]
- Типизирани reactive forms
- Strict templates и проверка на типовете в темплейта [[angular-control-flow]]
- `tsconfig`: `paths`, `strict`, `target`
