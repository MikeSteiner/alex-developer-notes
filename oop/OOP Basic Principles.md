# OOP Object-Oriented Principles Questions and Answers  
  
  
### Кои са основните принципи на ООП  
 - Капсулация/Encapsulation    
 - Абстракция/Abstraction    
 - Наследяване/Inheritance\  
 - Полиморфизъм/Polymorphism\  
  
#### Капсулация/Encapsulation

**Капсулация - наричано още "скриване на данните"**.
  
За даден обект, данните и операциите върху тях са събрани в една структура, наречена клас. Така обекта се разглежда в "черна кутия". Скриват се ненужните детайли за обекта и се  показват само най-важните негови характеристики на външния свят. Капсулацирането на данните намалява глобалните променливи. Ограничава възможността на потребителя да променя вътрешното състояние на даден обект по неочакван начин. 

**Капсулацията** е един от основните принципи на обектно-ориентираното програмиране. Тя се нарича още "скриване на информацията" (**information hiding**). Един обект трябва да предоставя на ползвателя си само необходимите средства за управление. Една **Секретарка** ползваща един **Лаптоп** знае само за екран, клавиатура и мишка, а всичко останало е скрито. Тя няма нужда да знае за вътрешността на **Лаптопа**, защото не й е нужно и може да оплеска нещо. Тогава част от свойствата и методите остават скрити за нея.

Изборът какво е скрито и какво е публично видимо е на този, който пише класа. Когато програмираме, трябва да дефинираме като **private** (скрит) всеки метод или поле, които не искаме да се ползват от друг клас.
  
#### Абстракция/Abstraction

Абстракцията е нещо с което знаем как да работим, но не знаем как изглежда вътрешно. Например имаме лаптоп. Не е нужно да знаем как работи лаптопът отвътре, за да го използваме. Същото се получава и с обектите в Обектно-ориентираното програмиране. Абстракцията е нещо, което правим всеки ден. Игнорираме всички малки детайли и се вглеждаме само в тези, които имат значение за нас в текущия момент. Абстракцията е една от най-важните концепции в ООП.

Същото се получава и с обектите в ООП. Ако имаме обект **Лаптоп** и той се нуждае от процесор, просто използваме обекта **Процесор**. Не знаем (или по-точно не се интересуваме) как той смята вътрешно. За да го използваме, е достатъчно да извикваме метода **сметни()** с подходящи параметри.

Абстракцията е нещо, което правим всеки ден. Това е действие, при което игнорираме всички детайли, които не ни интересуват от даден обект, и разглеждаме само детайлите, които имат значение за проблема, който решаваме. Например в хардуера съществува абстракция "устройство за съхранение на данни", което може да бъде твърд диск, USB memory stick, флопи диск или CD-ROM устройство. Всяко от тях работи вътрешно по различен начин, но от гледна точка на операционната система и на програмите в нея те се използват по еднакъв начин – на тях се записват файлове и директории. В Windows имаме Windows Explorer и той умее да работи по еднакъв начин с всички устройства, независимо дали са твърд диск или USB stick. Той работи с абстракцията "устройство за съхранение на данни" (storage device) и не се интересува как точно данните се четат и пишат. За това се грижат драйверите за съответните устройства. Те се явяват конкретни имплементации на интерфейса "устройство за съхране­ние на данни".

Абстракцията е една от най-важните концепции в програмирането и в ООП. Тя ни позволява да пишем **код, който работи с абстрактни структури от данни** (например списък, речник, множество и други). Имайки абстрактния тип данни, ние можем да работим с него през неговия интерфейс, без да се интересуваме от имплементацията му. Например можем да запазим във файл всички елементи на списък, без да се интересуваме дали той е реализиран с масив, чрез свързан списък или по друг начин. Този код остава непроменен, когато работим с различни конкретни типове данни. Дори можем да пишем нови типове данни (които се появяват на по-късен етап) и те да работят с нашата програма, без да я променяме.

Абстракцията ни позволява и нещо много важно – **да дефинираме интерфейс на нашите програми**, т.е. да дефинираме всички задачи, които тази програма може да извърши, както и съответните входни и изходни данни. Така можем да направим няколко по-малки програми, всяка от които да извършва някаква по-малка задача. Като прибавим това към факта, че можем да работим с абстрактни данни, ни дава голяма гъвка­вост при свързването на тези по-малки програми в една по-голяма и ни дава повече възможности за преизползване на код. Тези малки подпрограми се наричат компоненти. Този начин на писане на програми намира широко приложение в практиката, защото ни позволява не само да преизползваме обекти, а дори цели подпрограми.

#### Наследяване/Inheritance  
  
Наследяването е принцип, при който ако един обект - **обект1** съдържа всички свойства и действия на друг обект - **обект2**, то обект1 може да наследи обект2. По този начин обект1 освен собствените си атрибути и операции приема и тези на базовия клас/родителя си, а именно на обект2. По този начин се избягва повторното дефиниране и има възможността от създаването на йерархии от класове. 

Например имаме клас Animal, който представлява абстракция за множеството от всички животни. Всички обекти от класа Animal имат общи характеристики(color, age) и обща функционалност(Eat,Walk). Класът Dog, който е множество от всички кучета, има операции Eat, Walk и Bark. Удачно е класът Dog да наследи класа Animal. По този начин Dog ще има описание на собственото си действие Bark, и ще наследи Eat и Walk от базовия клас - Animal. Чрез наследяването се моделира "is-a" отношението между обектите. Например можем да твърдим, че кучето е  животно, тъй като то може да прави всичко, което жи

#### Какво е Полиморфизъм/Polymorphism

Следващият основен принцип от обектно-ориентираното програмиране е "полиморфизъм". **Полиморфизмът** позволява третирането на обекти от наследен клас като обекти от негов базов клас. Например големите котки (базов клас) хващат жертвите си (метод) по различен начин. Лъвът (клас наследник) ги дебне, докато Гепардът (друг клас-наследник) просто ги надбягва.

Полиморфизмът дава възможността да третираме произволна голяма котка просто като голяма котка и да кажем "хвани жертвата си", без значение каква точно е голямата котка.

Полиморфизмът може много да напомня на абстракцията, но в програмирането се свързва най-вече с пренаписването (override) на методи в нас­ледените класове с цел промяна на оригиналното им поведение, наследено от базовия клас. Абстракцията се свързва със създаването на интерфейс на компонент или функционалност (дефиниране на роля). Пренаписва­нето на методи ще разгледаме в детайли след малко.
   
