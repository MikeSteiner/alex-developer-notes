
#### Unit tests

```typescript
[

].forEach(({minQuantity, totalQuantity}: { minQuantity: number, totalQuantity: number }) => {  }

```

```typescript
describe('highlightUnsavedDescription', () => {  

  [  
    { showUnsavedChanges: true, markAsDirty: false },  
    { showUnsavedChanges: false, markAsDirty: true },  
    { showUnsavedChanges: false, markAsDirty: false },  
  ].forEach(({ showUnsavedChanges, markAsDirty }: { showUnsavedChanges: boolean; markAsDirty: boolean }) => {  
    it('should return FALSE when form control IS NOT dirty or showUnsavedChanges is NOT set', () => {  
      // Arrange  
      component.showUnsavedChanges = showUnsavedChanges;  
      if(markAsDirty) {  
        component.boxDescriptionControl.markAsDirty();  
      }  
      // Act & Assert  
      expect(component.highlightUnsavedDescription).toEqual(false);  
    });  
  });  
});
```

#### Effects
