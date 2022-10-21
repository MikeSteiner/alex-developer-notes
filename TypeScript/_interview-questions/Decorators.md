# What is Decorator

> “Decorators provide a way to add both annotations and a meta-programming syntax for class declarations and members. Decorators are a [stage 2 proposal](https://github.com/tc39/proposal-decorators) for JavaScript and are available as an experimental feature of TypeScript.”

So what are those Decorators, really? As you can see from the gist below it is a function that returns a function of a specific Type:

-   ClassDecorator
-   PropertyDecorator
-   MethodDecorator
-   ParameterDecorator

```typescript
function classDecorator(...args: any): ClassDecorator {
    return function <TFunction extends Function>(
        target: TFunction
    ): TFunction | void {
        // do something
    }
}

function propertyDecorator(...args: any): PropertyDecorator {
    return function (
        target: Object,
        propertyKey: string | symbol
    ): void {
        // do something
    }
}

function methodDecorator(...args: any): MethodDecorator {
    return function <T>(
        target: Object,
        propertyKey: string | symbol,
        descriptor: TypedPropertyDescriptor<T>
    ): TypedPropertyDescriptor<T> | void {
        // do something
    }
}

function parameterDecorator(...args: any): ParameterDecorator {
    return function (
        target: Object,
        propertyKey: string | symbol,
        parameterIndex: number
    ): void {
        // do something
    }
}
```
