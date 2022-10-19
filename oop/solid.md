### Кои са **SOLID** принципите  
  
https://en.wikipedia.org/wiki/SOLID  
  
SOLID е акроним от първите букви на 5 основни принципа в обектно ориентираното програмиране, които имат за цел да направят кода по гъвкав, лесен за поддръжка и промяна/разширение, както и по разбираем.    
Предожели за от не безизвестният софтуерен инженер и преподавател Robert C. Martin през 2000 г. Основните идеи са:  
  
- **S**ingle-responsibility principle: "There should never be more than one reason for a class to change." In other words, every class should have only one responsibility;  
- **O**pen–closed principle: "Software entities ... should be open for extension, but closed for modification.";  
- **L**iskov substitution principle: "Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.";  
- **I**nterface segregation principle: "Clients should not be forced to depend upon interfaces that they do not use.";  
- **D**ependency inversion principle: "Depend upon abstractions, concretions.".  
  
Най-важните според Виктор Цветков по време не Телерик Академията, са първият и последния и всички останали биха могли да се нарекат второстепенни и органичо идващи си, ако започнем да ползвяме пътвият и послендия принцип.  
А именно **S**ingle-responsibility principle и **D**ependency inversion.  
  
#### Какво е **S**ingle-responsibility principle  
Single Responsibility Principle - A _class or function should only have one reason to change_

#### Какво е **O**pen–closed principle  
O: Open-Closed Principle - _A software artifact should be open for extension but closed for modification_

#### Какво е **L**iskov substitution principle  
L: Liskov-Substitution Principle - Introduced by Barbara Liskov in the 1980s. This principle defines that objects of a superclass shall be replaceable with objects of its subclasses without breaking the application. That requires the objects of the subclasses to behave in the same way as the objects of the superclass.

#### Какво е **I**nterface segregation principle  
I: Interface Segregation Principle - Prevent classes from relying on things that they don’t need

#### Какво е **D**ependency inversion principle
D: Dependency Inversion Principle - Abstractions should not depend on details. Details should depend on abstractions