https://javascript.info/event-loop

Browser JavaScript execution flow, as well as in Node.js, is based on an _event loop_.

Understanding how event loop works is important for optimizations, and sometimes for the right architecture.

In this chapter we first cover theoretical details about how things work, and then see practical applications of that knowledge.

## [Event Loop](https://javascript.info/event-loop#event-loop)

The _event loop_ concept is very simple. There’s an endless loop, where the JavaScript engine waits for tasks, executes them and then sleeps, waiting for more tasks.

The general algorithm of the engine:

1.  While there are tasks:
	-   execute them, starting with the oldest task.
1.  Sleep until a task appears, then go to 1.

That’s a formalization for what we see when browsing a page. The JavaScript engine does nothing most of the time, it only runs if a script/handler/event activates.

Examples of tasks:

-   When an external script `<script src="...">` loads, the task is to execute it.
-   When a user moves their mouse, the task is to dispatch `mousemove` event and execute handlers.
-   When the time is due for a scheduled `setTimeout`, the task is to run its callback.
-   …and so on.

Tasks are set – the engine handles them – then waits for more tasks (while sleeping and consuming close to zero CPU).

It may happen that a task comes while the engine is busy, then it’s enqueued.

The tasks form a queue, so-called “macrotask queue” (v8 term):

![[Pasted image 20221021154805.png]]

For instance, while the engine is busy executing a `script`, a user may move their mouse causing `mousemove`, and `setTimeout` may be due and so on, these tasks form a queue, as illustrated on the picture above.

Tasks from the queue are processed on “first come – first served” basis. When the engine browser is done with the `script`, it handles `mousemove` event, then `setTimeout` handler, and so on.

So far, quite simple, right?

Two more details:

1.  Rendering never happens while the engine executes a task. It doesn’t matter if the task takes a long time. Changes to the DOM are painted only after the task is complete.
2.  If a task takes too long, the browser can’t do other tasks, such as processing user events. So after a time, it raises an alert like “Page Unresponsive”, suggesting killing the task with the whole page. That happens when there are a lot of complex calculations or a programming error leading to an infinite loop.

That was the theory. Now let’s see how we can apply that knowledge.