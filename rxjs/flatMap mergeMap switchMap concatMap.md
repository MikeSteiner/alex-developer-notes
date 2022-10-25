https://stackoverflow.com/questions/49698640/flatmap-mergemap-switchmap-and-concatmap-in-rxjs

Taking this from a [previous answer](https://stackoverflow.com/questions/44593900/rxjs-one-observable-feeding-into-another/44597697#44597697):

-   **flatMap/mergeMap** - creates an Observable immediately for any source item, all previous Observables are kept alive. Note `flatMap` is an alias for `mergeMap` and `flatMap` will be removed in RxJS 8.
-   **concatMap** - waits for the previous Observable to complete before creating the next one
-   **switchMap** - for any source item, completes the previous Observable and immediately creates the next one
-   **exhaustMap** - source items are ignored while the previous Observable is not completed

Here is an example of how each of the operators behaves when the source is immediate items (0,1,2,3,4) and the map function creates an Observable that delays each item by 500ms: