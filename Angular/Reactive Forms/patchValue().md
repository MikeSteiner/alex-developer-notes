`patchValue()` does **not** mark controls as touched — it only updates values and recalculates validity. So your `markAllAsTouched()` call is necessary and correct.

Here's what each method does in your context:

**`patchValue()`** — updates values + triggers `updateValueAndValidity()` internally. The control will _be_ invalid if the value is out of range, but the error message won't _show_ yet.

**`markAllAsTouched()`** — required. Most Angular error message patterns (`*ngIf="control.invalid && control.touched"`) gate on `touched`. Without it the control is invalid but the template keeps the message hidden.

---

Keep `{ emitEvent: false }`. Here's why:

The child component's `valueChanges` subscription uses `startWith(creditCalculatorInitialValues)` — meaning it already emits once on subscription with the initial values. When `parseCreditDataFormValue` runs during `ngOnInit`, it patches the form with query param values. If you allow the event to emit, you get **two emissions back-to-back**:

1. `startWith` fires with initial config values
2. `patchValue` fires with query param values

This means `creditCalculatorValueChanges` emits twice and the parent recalculates credit results twice on page load — wasted computation.

With `{ emitEvent: false }` the patch is silent, `startWith` still fires once with the initial values as the "seed" emission, and then `valueChanges` takes over for any subsequent user interaction. One emission, correct behaviour.

The only case where you'd want to **remove** `{ emitEvent: false }` is if the child subscribed _after_ the patch runs and had no `startWith` — then suppressing the event would mean the child misses the initial patched values entirely. But that's not your situation here.