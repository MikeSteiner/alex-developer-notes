# Changelog

All notable changes to these notes are recorded here, newest first.

## 2026-10-16: Complete Angular fundamentals and split roadmaps by topic

Completes section 1 of the Angular roadmap (components and templates) and adds the directives, pipes, routing and HTTP notes. Splits the non-Angular topics into their own roadmaps, and adds browser and JavaScript fundamentals notes.

### Added

* Angular/Components/
  * `angular-inputs-outputs-model`: signal inputs, `model()` and two-way binding, `output()`, `outputFromObservable`
  * `angular-content-projection-queries`: `<ng-content>`, host elements and the `host` property, `viewChild`/`contentChild` queries
* Angular/Templates/ (new)
  * `angular-control-flow`: `@if`, `@for` with `track`, `@switch`, `@let`
  * `angular-defer`: `@defer` triggers, prefetching, placeholder/loading/error blocks, incremental hydration
  * `angular-directives-pipes`: attribute and structural directives, `*` desugaring to `<ng-template>`, `TemplateRef`/`ViewContainerRef`, `hostDirectives` composition, pure vs impure pipes, why `async` is impure, pipe vs template method
* Angular/Routing/ (new): `angular-routing`, covering lazy loading and preloading, functional guards, `canMatch` vs `canActivate`, resolvers and when to avoid them, `withComponentInputBinding`
* Angular/HTTP/ (new): `angular-http-client`, covering fetch as the default backend since v21, functional interceptors and their order, `HttpContext`, `httpResource`, XSRF, SSR transfer cache
* JavaScript/ (new): `javascript-fundamentals`, covering equality and falsy values, `??` and `?.`, value vs reference, hoisting and TDZ, closures, `this`, arrow vs regular functions, prototypes, promise combinators, mutating array methods, debounce, memory leaks
* Web browser/
  * `browser-dom-events`: bubbling and capturing, delegation, `preventDefault` vs `stopPropagation`, passive listeners
  * `browser-storage`: cookies, `localStorage`, `sessionStorage`, IndexedDB, token storage and XSS/CSRF trade-offs
  * `browser-deep-dive-roadmap`
* rxjs/: `rxjs-deep-dive-roadmap`, moved out of the Angular roadmap
* TypeScript/: `typescript-deep-dive-roadmap`
* `testing-security-deep-dive-roadmap`

### Changed

* `angular-deep-dive-roadmap`: restructured into 11 sections, with directives and pipes as a separate section. RxJS, TypeScript, testing/security and browser topics moved to their own roadmaps, linked from the header. Links now point to the current note names.
* `change-detection-signals-zoneless`: new section on how signals trigger change detection. The trigger is global, but the work is local (targeted vs global mode, `HasChildViewsToRefresh`).
* `change-detection-frameworks`: added a paragraph on Astro after the SolidJS section.

### Known issues

* Wikilinks in `rxjs/operators/rxjs-switchMap` still point to old note names.
* Angular/Forms is unfinished.

---

## 2026-10-15: Merge study notes from different sources

Brings the Angular interview-prep notes and the supporting RxJS, NgRx, TypeScript and browser notes together in one tree. All notes share one format: YAML frontmatter with tags and related links, prose explanations, code only where needed, and an "Изречението за интервю" summary at the end.

### Added

- **Angular/Change Detection/**: a four-part series and two supporting notes
  - `change-detection-zonejs`: dirty checking, trigger vs mechanism, Zone.js and `NgZone`
  - `change-detection-onpush`: OnPush rules, the four documented scenarios, `markForCheck`/`detectChanges`/`detach`, OnPush as the default since v22 (`Eager`)
  - `change-detection-signals-zoneless`: signals, zoneless as the default since v21, migration pitfalls, `PendingTasks` for SSR, testing
  - `change-detection-frameworks`: comparison with React, Vue, Svelte and Solid
  - `zonejs-monkey-patching`: line-by-line walkthrough of how Zone.js wraps `setTimeout`
  - `browser-event-loop-angular`: where change detection sits in the event loop, NG0100 workarounds, layout thrashing, RxJS schedulers, SSR stability
- **Angular/Components/**: `angular-components-basics`, `angular-standalone-components`, `angular-component-lifecycle`
- **Angular/Dependency Injection/**: `angular-dependency-injection`, covering `inject()` and injection context, providers, the two injector hierarchies, resolution modifiers, `providers` vs `viewProviders`, `injectAsync` and 18 interview questions
- **Angular/Image and NgOptimizedImage/**: three NgOptimizedImage notes
- **Angular/Reactive Forms/**: `patchValue()`
- **Angular/**: `angular-deep-dive-roadmap`, a topic map for interview preparation
- **NgRx/**: `ngrx-architecture`, covering unidirectional data flow, actions, pure reducers, effects, memoized selectors, and Store vs ComponentStore vs SignalStore
- **TypeScript/**: `TypeScript` index, `typescript-narrowing`, `Активиране/flashcards-typescript-narrowing`
- **Web browser/**
  - `browser-event-loop`: macrotasks, microtasks, rendering, execution-order exercise, long tasks, splitting heavy work
  - `service-workers`: lifecycle, fetch interception, caching strategies, `CacheStorage` vs `IndexedDB`, and the Angular service worker (`ngsw-config.json`, version pinning, `SwUpdate`, production operations)
- **rxjs/**
  - `rxjs-flattening-operators`: `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`
  - `rxjs-error-handling`: `catchError` placement, `retry`, `timeout`, `finalize`, uncaught errors
  - `rxjs-combination-operators`: `combineLatest`, `withLatestFrom`, `forkJoin`, `concatLatestFrom`
  - `rxjs-sharereplay-hot-cold`: `share` vs `shareReplay`, the `refCount` trap, caching
  - `observable-vs-promise`, `observable-subject-behaviorsubject`, `tapresponse-store-state`, `EMPTY-void-null-undefined`
  - `operators/`: `rxjs-switchMap`, `rxjs-startWith-distinctUntilChanged-skip-pairwise`
  - `Активиране/`: flashcard placeholders

### Changed

- `service-workers`: rewritten from an older note. Removed duplicated and conversational sections, and corrected claims about version handling, cross-origin interception, metadata storage, push and Background Sync. Checked against MDN and angular.dev.
- `rxjs/rxjs`: index now lists the four new operator notes, `rxjs-startWith-distinctUntilChanged-skip-pairwise` and `ngrx-architecture`.
- `rxjs-combination-operators`: reworked the `withLatestFrom` "silent loss" section and removed a duplicated sentence.

### Fixed

- Restored Cyrillic file and folder names that the zip export had encoded as `#Uxxxx` (`Активиране/`, `Какво всъщност е NgOptimizedImage.md`).

### Known issues

- 8 wikilinks in `Angular/angular-deep-dive-roadmap` and `rxjs/operators/rxjs-switchMap` point to old note names and don't resolve yet.
