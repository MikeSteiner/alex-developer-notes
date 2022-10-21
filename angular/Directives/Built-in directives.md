https://angular.io/guide/built-in-directives#built-in-directives

Directives are classes that add additional behavior to elements in your Angular applications. Use Angular's built-in directives to manage forms, lists, styles, and what users see.


### Tracking items with `*[ngFor](https://angular.io/api/common/NgFor)` `trackBy`

Reduce the number of calls your application makes to the server by tracking changes to an item list. With the [ngFor](https://angular.io/api/common/NgFor) `trackBy` property, Angular can change and re-render only those items that have changed, rather than reloading the entire list of items.

1.  Add a method to the component that returns the value [NgFor](https://angular.io/api/common/NgFor) should track. In this example, the value to track is the item's `id`. If the browser has already rendered `id`, Angular keeps track of it and doesn't re-query the server for the same `id`.
   
```typescript
trackByItems(index: number, item: Item): number { return item.id; }
```
   
2. In the shorthand expression, set `trackBy` to the `trackByItems()` method.

```html
<div *ngFor="let item of items; trackBy: trackByItems">
  ({{item.id}}) {{item.name}}
</div>
```

**Change ids** creates new items with new `item.id`s. In the following illustration of the `trackBy` effect, **Reset items** creates new items with the same `item.id`s.

-   With no `trackBy`, both buttons trigger complete DOM element replacement.
-   With `trackBy`, only changing the `id` triggers element replacement.

![Animation of trackBy](https://angular.io/generated/images/guide/built-in-directives/ngfor-trackby.gif)
