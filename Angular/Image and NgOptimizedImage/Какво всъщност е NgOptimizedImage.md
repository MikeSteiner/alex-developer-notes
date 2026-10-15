`NgOptimizedImage` е **директива**, която:

> 👉 **декларира намерение (intent) за изображение**,  
> и позволява на Angular да приложи **compile-time + runtime оптимизации**

---
# 🔬 2. Какво прави на ниво DOM (реално)

Твоят код:

```html
<img  
  ngSrc="media/homepage-hero-mobile-899x.webp"  
  ngSrcset="320w,450w,700w,899w"  
  sizes="100vw"  
  priority  
/>
```
## 👉 Angular го трансформира до:

```html
<img  
  src="media/homepage-hero-mobile-899x.webp"  
  srcset="  
    media/homepage-hero-mobile-320x.webp 320w,  
    media/homepage-hero-mobile-450x.webp 450w,  
    media/homepage-hero-mobile-700x.webp 700w,  
    media/homepage-hero-mobile-899x.webp 899w  
  "  
  sizes="100vw"  
  fetchpriority="high"  
  loading="eager"  
  decoding="async"  
/>
```

[[Image Loading and Loader abstraction#🔥 5. Priority → какво реално става]]

---
## 🧠 3. Как Angular генерира `srcset`

Angular:
1. Взима `ngSrc` → базов URL
2. Парсва `ngSrcset` → списък от width descriptors (`320w`, `450w`…)
3. За всяка стойност:
    - извлича числото (`320`)
    - **замества width частта в URL-а**
## ⚠️ Това работи само при pattern за имената на файловете:

image-899x.webp  
→ image-320x.webp
→ image-450x.webp
→ image-700x.webp

👉 Angular НЕ „разбира“ файлове  
👉 прави **string transformation**

## ❗ Ако pattern не съвпада
Ако файловете са именувани по следния начин:
```
hero-small.webp  
hero-large.webp
```

👉 `ngSrcset` ще се счупи логически  
👉 трябва да минеш на `srcset` и файловете да се опишат по отделно с пълния им път.