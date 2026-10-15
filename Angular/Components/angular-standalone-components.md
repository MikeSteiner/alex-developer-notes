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
