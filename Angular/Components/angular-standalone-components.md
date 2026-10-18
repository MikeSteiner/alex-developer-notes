# Angular: Standalone компоненти

https://angular.dev/guide/components

## Сърцевината на standalone

За да използва компонент, директива или pipe в темплейта си, един компонент трябва да ги добави в масива `imports` на своя `@Component`. От Angular 19 компонентите са standalone по подразбиране, затова се добавят директно в `imports` на друг компонент. По-старите компоненти може да имат изрично `standalone: false`. Тях не импортираш директно, а импортираш `NgModule`-а, в който са декларирани. Преди версия 19.0.0 стойността по подразбиране на `standalone` е била `false`.

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Profile photo" i18n-alt />`,
})
export class ProfilePhoto {}

@Component({
  selector: 'user-profile',
  imports: [ProfilePhoto],
  template: `<profile-photo />`,
})
export class UserProfile {}
```

## История: от `NgModule` към standalone

Standalone компонентите не започват с версия 19. Те се появяват в **Angular 14** като developer preview и са стабилни от **версия 15**. Версия 19 само ги прави поведението по подразбиране, затова `standalone: true` вече не се пише.

## Какво решаваше `NgModule` и защо беше проблем

В света на модулите компонентът сам не казваше какво използва. Темплейтът му виждаше всичко, което е декларирано в същия модул, и всичко, което **импортираните модули експортират**. Модулът беше компилационният контекст на компонента.

Класическият пример за проблема е `SharedModule`, който импортира и експортира всички общи компоненти, директиви и pipes. Всеки модул, който го импортира, вижда всичко, независимо колко от него използва. Зависимостите стават невидими, а при lazy loading общият модул може да вкара в chunk-а код, който не е нужен.

## Компилация срещу DI

В `NgModule` се смесваха две различни неща, които е добре да се разделят ясно на интервю.

`declarations`, `imports` и `exports` определят **какво вижда темплейтът**. Това е въпрос на компилацията.

`providers` определя **какво може да се инжектира**. Това е въпрос на dependency injection.

В standalone света services **не** влизат в `imports` на компонента. Те се предоставят с `providedIn: 'root'` или `@Service`, или в `providers` на route, на компонент или на приложението. Подробностите са в [[angular-dependency-injection]].

## Стартиране и конфигурация без модули

`AppModule` с `bootstrap: [AppComponent]` е заменен от `bootstrapApplication`:

```typescript
bootstrapApplication(App, appConfig);

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes), provideHttpClient(withInterceptors([authInterceptor]))],
};
```

Модулите с `forRoot()` бяха начинът да се конфигурира библиотека. Техният заместител са функциите `provide*`. `RouterModule.forRoot(routes)` става `provideRouter(routes)`, а `HttpClientModule` става `provideHttpClient()`. От версия 21 `HttpClient` е достъпен и без него, както е описано в [[angular-http-client]].

## Какво печели lazy loading

Standalone компонентът е самодостатъчна входна точка. `loadComponent` в [[angular-routing]] зарежда точно този компонент и неговите зависимости, без да зарежда цял модул, от който се използва само една страница.

`@defer` от [[angular-defer]] също разчита на standalone. Той може да изнесе в отделен chunk само **standalone** зависимости. Компонент, деклариран в `NgModule`, не може да бъде отложен.

## Изречението за интервю

> Standalone компонентът декларира зависимостите си директно в `imports` на `@Component`, вместо да ги получава от `NgModule`. Стабилен е от Angular 15, а от 19 е по подразбиране. В модулите темплейтът виждаше всичко, декларирано в модула или експортирано от импортираните модули, и затова зависимостите бяха невидими, а `SharedModule` вкарваше излишен код. Standalone ги прави явни на ниво компонент. `imports` е за темплейта, а services не влизат там, а се предоставят чрез DI. `AppModule` е заменен от `bootstrapApplication` с `ApplicationConfig`, а `forRoot` модулите от функции като `provideRouter` и `provideHttpClient`. Lazy loading и `@defer` печелят, защото компонентът е самодостатъчна входна точка и не тегли цял модул след себе си.