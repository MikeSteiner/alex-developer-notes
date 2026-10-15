## 4. Image loader abstraction

Angular има концепция за **ImageLoader**
***По default:***
```
provideImageKitLoader()  
provideCloudinaryLoader()  
provideImgixLoader()
или custom loader
```

## 📌 Какво прави loader-ът
Вместо: `/media/image-320.webp`
може да генерира: `https://cdn.com/image?w=320`

ngSrcset` не е само string replace, a може да стане URL generation pipeline

## Angualr `ngSrc` + `ngSrcset`

### Our source code
```html
<img ngSrc="media/homepage-hero-mobile-899w.webp"
     ngSrcset="320w,450w,700w,899w"
     width="899"
     height="450"
     priority
     sizes="100vw"
     alt="Ипотечен кредит за твоя дом"
     i18n-alt="@@homeHeaderImageLoan"
     class="hero__image" />
```
Here is what the official Angular docs say directly:
> **angular.dev/api/common/NgOptimizedImage**

To use the default loader: no additional code changes are necessary. **The URL returned by the generic loader will always match the value of `"src"`.** [Angular](https://angular.dev/api/common/NgOptimizedImage)
That's the key sentence. The default loader is essentially a **no-op** — it returns the `ngSrc` value unchanged. It does **not** append `?w=` params either — I was wrong about that in earlier replies. It just returns your `src` as-is for every srcset entry.

So with `ngSrcset="320w,450w,700w,899w"` and the default loader, all four srcset entries point to the exact same URL — `media/homepage-hero-mobile-899w.webp` — with no width manipulation at all.

### What Angular generates in the DOM

```html
<img
  src="media/homepage-hero-mobile-899w.webp"
  srcset="
    media/homepage-hero-mobile-899w.webp 320w,
    media/homepage-hero-mobile-899w.webp 450w,
    media/homepage-hero-mobile-899w.webp 700w,
    media/homepage-hero-mobile-899w.webp 899w
  "
  sizes="100vw"
  width="899"
  height="450"
  fetchpriority="high"
  loading="eager"
  alt="Ипотечен кредит за твоя дом"
  class="hero__image"
/>
```

### What Angular injects in `<head>` during SSR

```html
<link
  rel="preload"
  as="image"
  href="media/homepage-hero-mobile-899w.webp"
  imagesrcset="
    media/homepage-hero-mobile-899w.webp 320w,
    media/homepage-hero-mobile-899w.webp 450w,
    media/homepage-hero-mobile-899w.webp 700w,
    media/homepage-hero-mobile-899w.webp 899w
  "
  imagesizes="100vw"
  fetchpriority="high"
/>
```

This is the SSR-specific benefit of `priority` — the preload hint ships in the **initial HTML response**, so the browser starts fetching the image before JavaScript even executes.

!!! **if we want this to work, we need CDN**

---
## 🔥 5. Priority → какво реално става

Когато имаш: `priority` например:
```html
<img  
  ngSrc="media/homepage-hero-mobile-899w.webp"  
  ngSrcset="320w,450w,700w,899w"  
  sizes="100vw"  
  priority  
/>
```

### 1.  Angular добавя в `<img>` тага:
```
fetchpriority="high"  
loading="eager"
```
### 2. Angular добавя `<link rel="preload">`
в `<head>`:

```html
<link  
  rel="preload"  
  as="image"  
  href="media/homepage-hero-mobile-899w.webp"  
  imagesrcset="..."  
  imagesizes="100vw"  
/>
```

 `<link rel="preload">` е критично за LCP. По този начин Browser-ът
	- започва fetch **преди layout**    
	- не чака parsing на `<img>`
## 🧠 6. Lazy loading (ако няма priority)
### 1.  Angular добавя в `<img>` тага:
```
loading="lazy"  
decoding="async"
```
Но Angular НЕ контролира кога се зарежда  изображението. Това е **browser-level lazy loading** спесификация.