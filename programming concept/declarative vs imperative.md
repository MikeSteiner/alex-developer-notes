This "**what vs how**" is often used to compare both of these approaches

Declarative programming is a paradigm describing **WHAT** the program does, without explicitly specifying its control flow.

Imperative programming is a paradigm describing **HOW** the program should do something by explicitly specifying each instruction (or statement) step by step, which mutate the program's state.

Granted, at the end of the day, everything compiles to instructions for the CPU. So in a way, declarative programming is a layer of abstraction on top of imperative programming.

This is imperative, this is laying out every single step.
```javascript
function evenSum(numbers) {
    let result = 0;

    for (let i = 0; i < numbers.length; i++) {
        let number = numbers[i]
        if (number % 2 === 0) {
            result += number;
        }
    }

    return result;
}
```

Here's a more declarative solution:
```javascript
const evenSum = numbers => numbers
    .filter(i => i % 2 === 0)
    .reduce((a, b) => a + b)

```