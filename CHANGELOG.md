# Changelog

All notable changes to these notes are recorded here, newest first.

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
