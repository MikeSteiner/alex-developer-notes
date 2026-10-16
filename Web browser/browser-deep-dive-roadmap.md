# Browser: Deep dive plan

Основа: https://javascript.info/ui, MDN https://developer.mozilla.org и web.dev https://web.dev ★ отбелязва личните приоритети. Връзката след точка сочи към вече написан файл. Точка без връзка още няма файл.

Свързан план: [[angular-deep-dive-roadmap]].

Фокусът е какво прави браузърът под Angular приложението, тоест знанията, които обясняват поведението на framework-а и производителността, а не пълно описание на Web API-тата.

---

## 1. ★ Event loop и изпълнение на JavaScript

- Macrotasks, microtasks и рендериране [[browser-event-loop]]
- Връзката с Zone.js и change detection [[browser-event-loop-angular]]
- `requestAnimationFrame` и `requestIdleCallback`
- Web Workers: изчисления извън main thread-а

## 2. Рендериране на страницата

- Critical rendering path: HTML, CSS, DOM, CSSOM, render tree
- Style, layout, paint и composite
- Reflow, repaint и layout thrashing [[browser-event-loop-angular]]
- Блокиращи ресурси: `defer` и `async` при скриптовете, CSS в `<head>`

## 3. ★ Производителност и Core Web Vitals

- LCP, INP и CLS: какво мерят и какво ги влошава
- Long tasks и отзивчивост [[browser-event-loop]]
- Resource hints: `preload`, `prefetch`, `preconnect`
- Профилиране в Chrome DevTools: Performance и Memory

## 4. DOM събития

- Capturing, target и bubbling фаза [[browser-dom-events]]
- Event delegation [[browser-dom-events]]
- `preventDefault` срещу `stopPropagation` [[browser-dom-events]]
- Passive listeners [[browser-dom-events]]

## 5. Мрежа и кеширане

- HTTP кеширане: `Cache-Control`, `ETag`, валидиране
- Same-origin policy и CORS
- Service Worker и стратегии за кеширане [[service-workers]]

## 6. Съхранение на данни

- Cookies, включително `HttpOnly`, `Secure` и `SameSite`
- `localStorage` и `sessionStorage`
- `IndexedDB` и `CacheStorage` [[service-workers]]
