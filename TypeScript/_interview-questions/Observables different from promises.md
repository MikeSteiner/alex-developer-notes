The first difference is that an Observable is **lazy** whereas a Promise is **eager**.

| **Promise** |  **Observable** |
|---|---|
|Emits a single value|Emits multiple values over a period of time|
|Not Lazy | Lazy. An observable is not called until we subscribe to the observable |
|Cannot be cancelled|Can be cancelled by using the unsubscribe() method|
| |Observable provides operators like map, forEach, filter, reduce, retry, retryWhen etc|

Consider the following Observable:

```typescript
const observable = rxjs.Observable.create(observer => {
   console.log('Text inside an observable');
   observer.next('Hello world!');
   observer.complete();
 });
 
 console.log('Before subscribing an Observable');
 observable.subscribe((message)=> console.log(message));    
```

 When you run the above Observable, you can see messages being displayed in the following order:

```plaintext
Before subscribing an Observable
Text inside an observable
Hello world!
```

As you can see, observables are lazy. Observable runs only when someone subscribes to them hence, the message “Before subscribing…” is displayed ahead of the message inside the observable.

Now let’s consider a Promise:    

```typescript
const promise = new Promise((resolve, reject) => {
   console.log('Text inside promise');
   resolve('Hello world!');
 });
 
 console.log('Before calling then method on Promise');
 greetingPoster.then(message => console.log(message));   
```

Running the above promise, the messages will be displayed in the following order:

```plaintext
Text inside promise
Before calling then method on Promise
Hello world!
```

As you can see the message inside Promise is displayed first. This means that a promise runs before the **then** method is called. Therefore, promises are **eager**.

The next difference is that Promises are always **asynchronous**. Even when the promise is immediately resolved. Whereas an Observable, can be both **synchronous** and **asynchronous**.

The above example of an observable is the case to show that an observable is synchronous. Let’s see the case where an observable can be asynchronous:

```typescript
 const observable = rxjs.Observable.create(observer => {
   setTimeout(()=>{
	   observer.next('Hello world');
	   observer.complete();
   },3000)
 });
 
 console.log('Before calling subscribe on an Observable');
 observable.subscribe((data)=> console.log(data));
 console.log('After calling subscribe on an Observable');    
```

The messages will be displayed in the following order:

```plaintext
Before calling subscribe on an Observable
After calling subscribe on an Observable
Hello world!
```

You can see in this case, observable runs asynchronously.

The next difference is that Observables can emit **multiple** values whereas Promises can emit only one value.

The biggest feature of using observables is the use of **operators**. We can use multiple operators on an observable whereas, there is no such feature in a promise.
