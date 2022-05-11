--- 
layout: post 
title: Шпаргалка по шаблонам проектирования с примерами кода на С++ 
permalink: design-patterns-cpp
tags: patterns C++
--- 

Описание паттернов, uml-схемы, плюс код на плюсах.

---

За небольшое время своей профессиональной  деятельности я редко сознательно прибегал к использованию шаблонов проектирования. Решил себе сделать шпаргалку. Пост ничего принципиально нового не несет, он лишь способ для меня систематизации знаний. Скомпилировано на основе найденного в интернете([1](http://itdumka.com.ua/index.php?cmd=shownode&node=11), [2](http://habrahabr.ru/post/210288/), [3](http://design-pattern.ru/), [4](http://sourcemaking.com/design_patterns), [5](https://sites.google.com/site/akstechtalks/tech-topics/Architecture/patterns), <abbr title="Gand of Four - Банда четырех">GOF</abbr>, гугл и википедия).
Пост постепенно будет дописываться и обновляться.

# Содержание
- [Условные обозначения](#legend)
- [Базовые типовые решения](#base_solutions)
    - [Шлюз (Gateway)](#gateway)
    - [Преобразователь (Mapper)](#mapper)
    - [Супертип слоя (Layer Supertype)](#layer_supertype)
    - [Отдельный интерфейс (Separated Interface)](#separated_interface)
    - [Реестр (Registry)](#registry)
    - [Объект-значение (Value Object)](#value_object)
    - [Деньги (Money)](#money)
    - [Частный случай (Special Case)](#special_case)
    - [Дополнительный модуль (Plugin)](#plugin)
    - [Фиктивная служба (Service Stub)](#service_stub)
    - [Множество записей (Record Set)](#record_set)
- [Порождающие паттерны](#creational_patterns)
    - [Фабричный метод (Factory Method)](#factory_method)
    - [Абстрактная фабрика (Abstract Factory)](#abstract_factory)
    - [Одиночка (Singleton)](#singleton)
    - [Прототип (Prototype)](#prototype)
    - [Строитель (Builder)](#builder)
    - [Объектный пул (Object Pool)](#object_pool)
- [Структурные паттерны](#structural_pattern)
    - [Адаптер (Adapter)](#adapter)
    - [Декоратор (Decorator)](#decorator)
    - [Заместитель (Proxy)](#proxy)
    - [Компоновщик (Composite)](#composite)
    - [Мост (Bridge)](#bridge)
    - [Приспособленец (Flyweight)](#flyweight)
    - [Фасад (Facade)](#facade)
- [Паттерны поведения](#behavioral_patterns)
    - [Интерпретатор (Interpreter)](#interpreter)
    - [Шаблонный метод (Template Method)](#template_method)
    - [Итератор (Iterator)](#iterator)
    - [Команда (Command)](#command)
    - [Наблюдатель (Observer)](#observer)
    - [Посетитель (Visitor)](#visitor)
    - [Посредник (Mediator)](#mediator)
    - [Состояние (State)](#state)
    - [Стратегия (Strategy)](#strategy)
    - [Хранитель (Memento)](#memento)
    - [Цепочка обязанностей (Chain of Responsibility)](#chain_of_responsibility)
- [Представление бизнес-логики](#representation_of_business_logic)
    - [Сценарий транзакции (Transaction Script)](#transaction_script)
    - [Модель предметной области (Domain Model)](#domain_model)
    - [Модуль таблицы (Table Module)](#table_module)
    - [Слой служб (Service Layer)](#service_layer)
- [Архитектурные типовые решения источника данных](#architecture_of_standard_solutions_data_source)
    - [Шлюз таблицы данных (Table Data Gateway)](#table_data_gateway)
    - [Шлюз записи данных (Row Data Gateway)](#row_data_gateway)
    - [Активная запись (Active Record)](#active_record)
    - [Преобразователь данных (Data Mapper)](#data_mapper)
- [Объектно-реляционные типовые решения, предназначенные для моделирования поведения](#object-relational_model_solutions_designed_to_simulate_the_behavior)
    - [Единица работы (Unit of Work)](#unit_of_work)
    - [Коллекция объектов (Identity Map)](#identity_map)
    - [Загрузка по требованию (Lazy Load)](#lazy_load)
- [Объектно-реляционные типовые решения, предназначенные для моделирования структуры](#object-relational_model_solutions_for_modeling_the_structure)
    - [Поле идентификации (Identity Field)](#identity_field)
    - [Отображение внешних ключей (Foreign Key Mapping)](#foreign_key_mapping)
    - [Отображение с помощью таблицы ассоциаций (Association Table Mapping)](#association_table_mapping)
    - [Отображение зависимых объектов (Dependent Mapping)](#dependent_mapping)
    - [Внедренное значение (Embedded Value)](#embedded_value)
    - [Сериализованный крупный объект (Serialized LOB)](#serialized_lob)
    - [Наследование с одной таблицей (Single Table Inheritence)](#single_table_inheritence)
    - [Наследование с таблицами для каждого класса (Class Table Inheritence)](#class_table_inheritence)
    - [Наследование с таблицами для каждого конкретного класса (Concrete Table Inheritence)](#concrete_table_inheritence)
    - [Преобразователи наследования (Inheritance Mappers)](#inheritance_mappers)
- [Шаблоны, предназначенные для представления данных в Web](#the_templates_are_designed_to_provide_data_on_the_web)
    - [Модель-представление-контроллер (Model View Controller)](#model_view_controller)
    - [Контроллер страниц (Page Controller)](#page_controller)
    - [Контроллер запросов (Front Controller)](#front_Controller)
    - [Контроллер приложения (Application Controller)](#application_controller)
    - [Представление с преобразованием (Transform View)](#transform_view)
    - [Двухэтапное представление (Two Step View)](#two_step_view)

# Условные обозначения   {#legend}

- ![](/assets/2015-01-22-design-patterns-cpp/01.png) - агрегация (aggregation) — описывает связь «часть»–«целое», в котором «часть» может существовать отдельно от «целого». Ромб указывается со стороны «целого».
- ![](/assets/2015-01-22-design-patterns-cpp/02.png) композиция (composition) — подвид агрегации, в которой «части» не могут существовать отдельно от «целого».
- ![](/assets/2015-01-22-design-patterns-cpp/03.png) - зависимость (dependency) — изменение в одной сущности (независимой) может влиять на состояние или поведение другой сущности (зависимой). Со стороны стрелки указывается независимая сущность.
- ![](/assets/2015-01-22-design-patterns-cpp/04.png) - обобщение (generalization) — отношение наследования или реализации интерфейса. Со стороны стрелки находится суперкласс или интерфейс.

# Базовые типовые решения    {#base_solutions}

## Шлюз (Gateway)    {#gateway}
![](/assets/2015-01-22-design-patterns-cpp/gateway.gif)  
Это объект, инкапсулирующий доступ к внешней схеме или источнику данных. Смысл применения шлюза - обеспечить более удобный интерфейс для доступа к внешней системе. Весь специализированный код API помещается в класс-шлюз, интерфейс которого не отличается от интерфейса обычного объекта. Интерфейс шлюза может даже представлять собой точную копию инкапсулируемого интерфейса. Шлюз разрабатывается клиентом для использования внешнего приложения в отличие от интерфейса ([Facade](#facade)). В отличие от адаптера ([Adapter](#adapter)) интерфейс шлюза не нужно подстраивать под какой-либо существующий интерфейс. И в отличие от медиатора ([Mediator](#mediator)) шлюз разделяет только два объекта, причем источник данных не знает о существовании шлюза. Одно из главных назначений шлюза - обеспечить основу для реализации фиктивных служб ([Service Stub](#service_stub)).

## Преобразователь (Mapper)  {#mapper}
![](/assets/2015-01-22-design-patterns-cpp/mapper.gif)  
Это объект, устанавливающий взаимодействие между двумя независимыми объектами. Основное назначение преобразователя - отделить друг от друга различные части программной системы. Преобразователь представляет собой некий "изоляционный" слой, проложенный между двумя подсистемами. Он управляет взаимодействием подсистем, зачастую лишь перемещая данные из одного слоя в другой. Разделяемые подсистемы не зависят друг от друга, поэтому ни одна из них не может напрямую вызвать запуск преобразователя. Этот вызов обычно возлагают на третью подсистему, либо реализуют преобразователь в виде типового решения обозреватель ([Observer](#observer)). В отличие от медиатора ([Mediator](#mediator)), объекты, разделенные преобразователем, не знают о наличии последнего.

## Супертип слоя (Layer Supertype)   {#layer_supertype}
Это тип, выполняющий роль суперкласса для всех классов своего слоя. Супертип слоя используется тогда, когда все объекты соответствующего слоя имеют некоторые общие свойства или поведение. Чтобы избежать повторения, всё общее поведение можно вынести в супертип слоя. Если в рассматриваемом приложении находятся объекты нескольких различных типов, может понадобиться создать несколько супертипов слоя.

## Отдельный интерфейс (Separated Interface) {#separated_interface}
![](/assets/2015-01-22-design-patterns-cpp/separated-interface.gif)  
Этот шаблон предполагает размещение интерфейса и его реализации в разных пакетах. Отдельный интерфейс используется тогда, когда нужно уничтожить зависимость между двумя подсистемами, обеспечить поддержку нескольких независимых реализаций или осуществить вызовы методов, противоречащие общей структуре зависимостей. Если реализация имеет только одного клиента или все клиенты реализации находятся в одном пакете, интерфейс может быть размещен прямо в пакете клиента. Если имеется несколько клиентских пакетов или определение интерфейса не входит в обязанности разработчиков клиентского пакета, то интерфейс помещается в пакет, отдельный от клиентских.

## Реестр (Registry) {#registry}
![](/assets/2015-01-22-design-patterns-cpp/registry.gif)  
Это "глобальный" (в действительности не обязательно таковой) объект, который используется другими объектами для поиска общих объектов или служб. В качестве интерфейса реестров рекомендуется применять статические методы. Обычно глобальный по отношению к процессу реестр реализуется в виде единственного элемента ([Singleton](#singleton)). Рекомендуется использовать явные классы с явными методами, которые позволяют проследить, какие ключи применяются для поиска объекта. Явный класс позволяет сохранить типовую безопасность в статически типизированных языках, а также инкапсулировать структуру реестра, чтобы при последующем росте системы её можно было вынести в нужный класс или слой. Разные области видимости требуют различных реализаций, однако интерфейс при этом может быть общий.

## Объект-значение (Value Object)    {#value_object}
***Также может иметься в виду объект переноса данных (Data Transfer Object)***  
Это небольшие простые объекты наподобие денежных значений или диапазонов дат, равенство которых не основано на равенстве идентификаторов. Применяется для моделирования тех сущностей, равенство которых определяется по значениям полей. Эти объекты делаются неизменяемыми, т.е. чтобы после создания объекта значения его полей не изменялись. Такие объекты можно передавать в виде значения, а не в виде ссылки на объект. Два объекта считаются равными если равны значения их полей. Обычно изменение полей объекта подразумевает полную замену старого объекта новым. Как правило для хранения объектов-значений применяют внедренные значения ([Embedded Value](#embedded_value)) или крупные сериализованные объекты ([Serialized LOB](#serialized_lob)).

## Деньги (Money)    {#money}
![](/assets/2015-01-22-design-patterns-cpp/money.gif)  
Этот шаблон представляет денежное значение. Денежные величины представляют собой объекты-значения ([Value Object](#value_object)), поэтому и хранить их желательно в виде внедренного значения ([Embedded Value](#embedded_value)). Величина денежной суммы обычно представляется значением целочисленного типа или же действительным значением с фиксированным количеством десятичных знаков. Это препятствует созданию некорректных значений, обусловленных работой с числами с плавающей запятой. Таким образом инкапсулируется обработка округления, что позволяет избежать большинства ошибок округления до фиксированного количества десятичных знаков. Еще одним преимуществом является возможность одновременного выполнения вычислений в нескольких единицах измерения (валютах).

## Частный случай (Special Case) {#special_case}
![](/assets/2015-01-22-design-patterns-cpp/special-case.gif)  
Производный класс, описывающий поведение объекта в особых ситуациях. Суть заключается в возврате специального объекта с тем интерфейсом, который ожидает увидеть вызывающий метод. Следует применять для описания повторяющегося поведения, связанного с обработкой значений NULL или других особых ситуаций после выполнения соответствующих проверок на их наличие. Обычно, но не всегда, для реализации можно воспользоваться типовым решением приспособленец ([Flyweight](#flyweight)). Нередки ситуации, когда при переопределении методов в частном случае последний возвращает еще один частный случай.

## Дополнительный модуль (Plugin)    {#plugin}
![](/assets/2015-01-22-design-patterns-cpp/plugin.gif)  
Связывает классы во время настройки, а не во время компиляции приложения. Это типовое решение применяется тогда, когда одно и то же поведение может иметь несколько реализаций в зависимости от выбранной исполняющей среды. Также дополнительный модуль обеспечивает централизованную настройку приложения во время выполнения. Обычно определяется Отдельный интерфейс ([Separated Interface](#separated_interface)), а затем пишется объект-фабрика ([Factory Method](#factory_method)) дополнительного модуля, который будет выполнять отображение интерфейса на необходимые реализации. Также дополнительный модуль комбинируется с типовым решением единственный элемент ([Singleton](#singleton)). Применение дополнительного модуля наиболее предпочтительно в языках, поддерживающих отражение, поскольку объект-фабрика может создавать объекты реализаций без необходимости устанавливать зависимости во время компиляции.

## Фиктивная служба (Service Stub)   {#service_stub}
***Синонимы: объект-имитатор (Mock Object)***  
![](/assets/2015-01-22-design-patterns-cpp/Service_Stub_Design_Pattern.jpg)  
Устраняет зависимости приложения от труднодоступных или проблемных служб на время тестирования. Используется тогда, когда зависимость приложения от конкретной внешней службы значительно затрудняет процесс разработки и тестирования. Обычно устанавливают доступ к внешней службе с помощью шлюза ([Gateway](#gateway)). Последний следует реализовать не как класс, а в виде отдельного интерфейса ([Separated Interface](#separated_interface)), имеющего одну реализацию для обращения к реальной службе и, как минимум, еще одну реализацию, являющуюся фиктивной службой. Нужная реализация шлюза должна загружаться с помощью дополнительного модуля ([Plugin](#plugin)). Сама реализация фиктивной службы должна быть достаточно простой, чтобы ускорить процесс разработки.

## Множество записей (Record Set)    {#record_set}
![](/assets/2015-01-22-design-patterns-cpp/record-set.gif)  
Представление табличных данных в оперативной памяти. Структура множества записей в точности копирует результат выполнения запроса к базе данных, однако может быть сгенерирована и использована другими частями системы. Обычно множество записей можно легко отсоединить от источника данных, что позволяет передавать множество записей по сети, не беспокоясь о наличии соединения с базой данных. Если множество записей легко поддается сериализации, оно может выступать в качестве объекта переноса данных ([Data Transfer Object](#value_object)). Зачастую множество записей реализуют в виде единицы работы ([Unit of Work](#unit_of_work)), благодаря чему все изменения множества записей могут быть внесены в отсоединенном режиме и затем зафиксированы в базе данных. Источник данных может воспользоваться оптимистической автономной блокировкой ([Optimistic Offline Lock](http://design-pattern.ru/patterns/optimistic-offline-lock.html)), чтобы проверить параллельные транзакции на наличие конфликтов и в случае отсутствия последних записать все внесенные изменения в базу данных. Большинство реализаций множества записей используют неявный интерфейс (implicit interface), хотя он и обладает такими существенными недостатками как рассредоточение кода поиска и утрата компилятором информации о типах. Настоящая ценность множества записей раскрывается в тех средах, которые могут применять его в качестве стандартного способа манипулирования данными. При наличии подобных сред для организации логики домена следует применять модуль таблицы ([Table Module](#table_module)).


# Порождающие паттерны   {#creational_patterns}

- Иногда порождающие паттерны конкурируют между собой: Есть случаи, когда либо [Абстрактная Фабрика](#abstract_factory) либо [Прототип](#prototype) может использоваться с выгодой. В других случаях они дополняют друг друга: [Абстрактная Фабрика](#abstract_factory) может содержать множество  [Прототипов](#prototype), из которых клонируются и возвращаются необходимые объекты, [Строитель](#builder) может использовать один из этих паттернов для постройки какого либо компонента. [Абстрактная Фабрика](#abstract_factory), [Строитель](#builder), [Прототип](#prototype) могут использовать [Синглтон](#singleton) в их реализации.
- [Абстрактная Фабрика](#abstract_factory) часто реализуется при помощи [Фабричного метода](#factory_method), так же это можно сделать применяя [Прототип](#prototype).
- [Абстрактная Фабрика](#abstract_factory) может использоваться как альтернатива [Фасаду](#facade) для сокрытия платформенно специфичных классов.
- [Строитель](#builder) фокусируется на построении комплексного объекта шаг за шагом. [Абстрактная Фабрика](#abstract_factory) акцентируется на семействе объектов (простых и сложных). [Строитель](#builder) возвращает результирующий объект на финальном шаге, в то время как [Абстрактная Фабрика](#abstract_factory) порождает объект сразу.
- [Строитель](#builder) - для создания(порождения), [Стратегия](#strategy) - для алгоритмов.
- [Строитель](#builder) часто используют для порождения [Компоновщика](#composite).
- [Фабричный метод](#factory_method) обычно используется внутри [Шаблонного метода](#template_method).
- [Фабричный метод](#factory_method): порождение через наследование. [Прототип](#prototype): порождение через делегирование.
- Часто проектировщик начинает с использования [Фабричного метода](#factory_method)(менее сложный, более настраиваемый, легко наследовать) и при усложнении проектируемой системы движется в сторону применения [Абстрактной Фабрики](#abstract_factory), [Прототипа](#prototype) или [Строителя](#builder) (более гибкие, более сложные).
- [Прототип](#prototype) не требует наследования, но нуждается в инициализации. [Фабричный метод](#factory_method) требует наследование, но не требует инициализацию.
- Проекты, которые интенсивно используют [Компоновщик](#composite) и [Декоратор](#decorator) часто могут извлечь выгоду также из [Прототипа](#prototype).

## Фабричный метод (Factory Method)  {#factory_method}
***Синонимы: Виртуальный конструктор (Virtual Constructor)***  
Определяет интерфейс для создания объектов, при этом выбранный класс инстанцируется подклассами, то есть оставляет подклассам решение о том, какой класс инстанцировать. Фабричный метод позволяет классу делегировать инстанцирование подклассам. Фабричные методы избавляют проектировщика встраивать в код зависящие от приложения классы.

***Применимость***  
Используется в случае:

- классу заранее неизвестно, объекты каких классов ему нужно создавать;
- класс спроектирован так, чтобы объекты, которые он создает, специфицировались подклассами;
- класс делегирует свои обязанности одному из нескольких вспомогательных подклассов, и планируется локализовать знание о том, какой класс принимает эти обязанности на себя.

![](/assets/2015-01-22-design-patterns-cpp/19_factorymethod-min.png)
***Родственные паттерны***  
[Абстрактная фабрика](#abstract_factory) часто реализуется с помощью фабричных методов. Паттерн **Фабричный метод** часто вызывается внутри шаблонных методов.

***Пример реализации***  
Паттерн реализуется через статический метод класса Stooge, возвращаемое знание типа Stooge. Но в отличие от обычного конструктора, возможно получить инстанс обьекта, который является наследником Stooge. Еще одно из преимуществ **Фабричного метода** - класс может многократно возвращать уже созданный инстанс объекта.
{% highlight c++ %}
class Stooge
{
  public:
    // Factory Method
    static Stooge *make_stooge(int choice);
    virtual void slap_stick() = 0;
};

int main()
{
  vector<Stooge*> roles;
  int choice;
  while (true)
  {
    cout << "Larry(1) Moe(2) Curly(3) Go(0): ";
    cin >> choice;
    if (choice == 0)
      break;
    roles.push_back(Stooge::make_stooge(choice));
  }
  for (int i = 0; i < roles.size(); i++)
    roles[i]->slap_stick();
  for (int i = 0; i < roles.size(); i++)
    delete roles[i];
}

class Larry: public Stooge
{
  public:
    void slap_stick()
    {
        cout << "Larry: poke eyes\n";
    }
};
class Moe: public Stooge
{
  public:
    void slap_stick()
    {
        cout << "Moe: slap head\n";
    }
};
class Curly: public Stooge
{
  public:
    void slap_stick()
    {
        cout << "Curly: suffer abuse\n";
    }
};

Stooge *Stooge::make_stooge(int choice)
{
  if (choice == 1)
    return new Larry;
  else if (choice == 2)
    return new Moe;
  else
    return new Curly;
}
{% endhighlight %}

## Абстрактная фабрика (Abstract Factory)    {#abstract_factory}
***Синонимы: Инструментарий (Kit)***  
Предоставляет интерфейс для создания семейств, связанных между собой, или независимых объектов, конкретные классы которых неизвестны.

***Применимость***  
Применять паттерн абстрактная фабрика необходимо, когда:

- система не должна зависеть от того, как создаются, компонуются и представляются входящие в нее объекты;
- входящие в семейство взаимосвязанные объекты должны использоваться вместе и необходимо обеспечить выполнение этого ограничения;
- система должна конфигурироваться одним из семейств составляющих ее объектов;
- необходимо предоставить библиотеку объектов, раскрывая только их интерфейсы, но не реализацию.

![](/assets/2015-01-22-design-patterns-cpp/14_abstractfactory-min.png)
***Родственные паттерны:*** Классы Abstract Factory часто реализуются фабричными методами (паттерн [фабричный метод](#factory_method)), но могут быть реализованы и с помощью паттерна [прототип](#prototype). **Конкретная фабрика часто описывается паттерном [одиночка](#singleton).**

***Пример реализации***  
Обычно, пытаясь сохранить переносимость между несколькими платформами, требуется много предпроцессорных `#ifdef...#else...#endif` по всей простыне кода. Один из способов избежать этого - использовать паттерн "Абстрактная Фабрика".
{% highlight c++ %}
#define WINDOWS

class Widget {
public:
   virtual void draw() = 0;
};

class MotifButton : public Widget {
public:
   void draw() { cout << "MotifButton\n"; }
};
class MotifMenu : public Widget {
public:
   void draw() { cout << "MotifMenu\n"; }
};

class WindowsButton : public Widget {
public:
   void draw() { cout << "WindowsButton\n"; }
};
class WindowsMenu : public Widget {
public:
   void draw() { cout << "WindowsMenu\n"; }
};

class Factory {
public:
   virtual Widget* create_button() = 0;
   virtual Widget* create_menu() = 0;
};

class MotifFactory : public Factory {
public:
   Widget* create_button() {    return new MotifButton; }
   Widget* create_menu()   {    return new MotifMenu; }
};

class WindowsFactory : public Factory {
public:
   Widget* create_button() {    return new WindowsButton; }
   Widget* create_menu()   {    return new WindowsMenu; }
};

void display_window_one(Factory* factory) {
   Widget* w[] = { factory->create_button(), factory->create_menu() };
   w[0]->draw();  w[1]->draw();
}

void display_window_two(Factory* factory) {
   Widget* w[] = { factory->create_menu(), factory->create_button() };
   w[0]->draw();  w[1]->draw();
}

int main() {
    Factory* factory = 0;
#ifdef MOTIF
   factory = new MotifFactory;
#else // WINDOWS
   factory = new WindowsFactory;
#endif

   Widget* w = factory->create_button();
   w->draw();
   display_window_one(factory);
   display_window_two(factory);
}
{% endhighlight %}

Вывод:

    WindowsButton
    WindowsButton
    WindowsMenu
    WindowsMenu
    WindowsButton

## Одиночка (Singleton)  {#singleton}
Гарантирует, что некоторый класс может иметь только один экземпляр, и предоставляет глобальную точку доступа к нему.

***Применимость***  

- должен быть ровно один экземпляр некоторого класса, легко доступный всем клиентам;
- единственный экземпляр должен расширяться путем порождения подклассов, и клиентам нужно иметь возможность работать с расширенным экземпляром без модификации своего кода.

![](/assets/2015-01-22-design-patterns-cpp/23_singleton-min.png)
***Пример реализации***  
Реализация не thread-safe.
{% highlight c++ %}
class Singleton 
{
public:
    static Singleton& instance()
    {
        static Singleton instance;
        return instance;
    }
protected:
    Singleton();
    Singleton(const Singleton &classRef);
    Singleton& operator=(const Singleton &classRef);
};
{%endhighlight %}

## Прототип (Prototype)  {#prototype}
Описывает виды создаваемых объектов с помощью прототипа и создаёт новые объекты путём его копирования. У прототипа те же самые результаты, что у абстрактной фабрики ([Abstract Factory](#abstract_factory)) и строителя ([Builder](#builder)): он скрывает от клиента конкретные классы продуктов, уменьшая тем самым число известных клиенту имён. Кроме того все эти паттерны позволяют клиенту работать со специфичными для приложения классами без модификаций.

***Применимость***  
Паттерн применяется, когда система не должна зависеть от того, как в ней создаются, компонуются и представляются продукты:

- инстанцируемые классы определяются во время выполнения, например, с помощью динамической загрузки;
- для того чтобы избежать построения иерархий классов или фабрик, параллельных иерархии классов продуктов;
- экземпляры класса могут находиться в одном из не очень большого числа различных состояний. Может оказаться удобнее установить соответствующее число прототипов и клонировать их, а не инстанцировать каждый раз класс вручную в подходящем состоянии.

![](/assets/2015-01-22-design-patterns-cpp/20_prototype-min.png)
***Отношения***  
Клиент обращается к прототипу, чтобы тот создал свою копию.

***Родственные паттерны***  
В некоторых отношениях прототип и [абстрактная фабрика](#abstract_factory) являются конкурентами. Но их используют и совместно. Абстрактная фабрика может хранить набор прототипов, которые клонируются и возвращают изготовленные объекты. В тех проектах, где активно применяются паттерны [компоновщик](#composite) и [декоратор](#decorator), тоже можно извлечь пользу из прототипа.

***Пример реализации***  
{% highlight c++ %}
class Stooge {
public:
   virtual Stooge* clone() = 0;
   virtual void slap_stick() = 0;
};

class Factory {
public:
   static Stooge* make_stooge( int choice );
private:
   static Stooge* s_prototypes[4];
};

Stooge* Factory::s_prototypes[] = {
   0, new Larry, new Moe, new Curly
};
Stooge* Factory::make_stooge( int choice ) {
   return s_prototypes[choice]->clone();
}

class Larry : public Stooge {
public:
   Stooge*   clone() { return new Larry; }
   void slap_stick() {
      cout << "Larry: poke eyes\n"; }
};
class Moe : public Stooge {
public:
   Stooge*   clone() { return new Moe; }
   void slap_stick() {
      cout << "Moe: slap head\n"; }
};
class Curly : public Stooge {
public:
   Stooge*   clone() { return new Curly; }
   void slap_stick() {
      cout << "Curly: suffer abuse\n"; }
};

int main() {
   vector roles;
   int             choice;

   while (true) {
      cout << "Larry(1) Moe(2) Curly(3) Go(0): ";
      cin >> choice;
      if (choice == 0)
         break;
      roles.push_back(
         Factory::make_stooge( choice ) );
   }

   for (int i=0; i < roles.size(); ++i)
      roles[i]->slap_stick();
   for (int i=0; i < roles.size(); ++i)
      delete roles[i];
}
{% endhighlight %}

## Строитель (Builder)   {#builder}
Отделяет конструирование сложного объекта от его представления, позволяя использовать один и тот же процесс конструирования для создания различных представлений.

***Применимость***  

- алгоритм создания сложного объекта не должен зависеть от того, из каких частей состоит объект и как они стыкуются между собой;
- процесс конструирования должен обеспечивать различные представления конструируемого объекта.

![](/assets/2015-01-22-design-patterns-cpp/17_builder-min.png)
***Отношения*** 

- клиент создает объект-распорядитель Director и конфигурирует его нужным объектом-строителем Builder;
- распорядитель уведомляет строителя о том, что нужно построить очередную часть продукта;
- строитель обрабатывает запросы распорядителя и добавляет новые части к продукту;
- клиент забирает продукт у строителя.

Следующая диаграмма взаимодействий иллюстрирует взаимоотношения строителя и распорядителя с клиентом:
![](/assets/2015-01-22-design-patterns-cpp/builder_flow2.png)
***Родственные паттерны***  
[Абстрактная фабрика](#abstract_factory) похожа на **Строитель** в том смысле, что может конструировать сложные объекты. Основное различие между ними в том, что Строитель делает акцент на пошаговом конструировании объекта, а Абстрактная фабрика - на создании семейств объектов (простых или сложных). Строитель возвращает продукт на последнем шаге, тогда как с точки зрения абстрактной фабрики продукт возвращается немедленно. Паттерн [компоновщик](#composite) - это то, что часто создает Строитель.

***Пример реализации***  
Сильная сторона Builder - это построение объекта шаг за шагом. Абстрактный класс декларирует шаги, конкретный наследник реализует эти шаги. В примере "distributed work packages" абстрагирован от платформенных особенностей.

Это означает, что для конкретной платформы механизм реализации файлов, очередей и путей определяется в конкретной производной класса. Объект "Reader" получает спецификацию из `PersistenceAttribute` для `DistrWorkPackage` и конфигурирует `Builder`, который был зарегистрирован клиентом, соответственно. После завершения клиент получает конечный результат от Builder'a.
{% highlight c++ %}
#include <iostream.h>
#include <stdio.h>
#include <string.h>

enum PersistenceType
{
  File, Queue, Pathway
};

struct PersistenceAttribute
{
  PersistenceType type;
  char value[30];
};

class DistrWorkPackage
{
  public:
    DistrWorkPackage(char *type)
    {
        sprintf(_desc, "Distributed Work Package for: %s", type);
    }
    void setFile(char *f, char *v)
    {
        sprintf(_temp, "\n  File(%s): %s", f, v);
        strcat(_desc, _temp);
    }
    void setQueue(char *q, char *v)
    {
        sprintf(_temp, "\n  Queue(%s): %s", q, v);
        strcat(_desc, _temp);
    }
    void setPathway(char *p, char *v)
    {
        sprintf(_temp, "\n  Pathway(%s): %s", p, v);
        strcat(_desc, _temp);
    }
    const char *getState()
    {
        return _desc;
    }
  private:
    char _desc[200], _temp[80];
};

class Builder
{
  public:
    virtual void configureFile(char*) = 0;
    virtual void configureQueue(char*) = 0;
    virtual void configurePathway(char*) = 0;
    DistrWorkPackage *getResult() { return _result; }
  protected:
    DistrWorkPackage *_result;
};

class UnixBuilder: public Builder
{
  public:
    UnixBuilder()
    {
        _result = new DistrWorkPackage("Unix");
    }
    void configureFile(char *name)
    {
        _result->setFile("flatFile", name);
    }
    void configureQueue(char *queue)
    {
        _result->setQueue("FIFO", queue);
    }
    void configurePathway(char *type)
    {
        _result->setPathway("thread", type);
    }
};

class VmsBuilder: public Builder
{
  public:
    VmsBuilder()
    {
        _result = new DistrWorkPackage("Vms");
    }
    void configureFile(char *name)
    {
        _result->setFile("ISAM", name);
    }
    void configureQueue(char *queue)
    {
        _result->setQueue("priority", queue);
    }
    void configurePathway(char *type)
    {
        _result->setPathway("LWP", type);
    }
};


class Reader
{
  public:
    void setBuilder(Builder *b) {   _builder = b;   }
    void construct(PersistenceAttribute[], int);
  private:
    Builder *_builder;
};

void Reader::construct(PersistenceAttribute list[], int num)
{
  for (int i = 0; i < num; i++)
  {
    if (list[i].type == File)
      _builder->configureFile(list[i].value);
    else if (list[i].type == Queue)
      _builder->configureQueue(list[i].value);
    else if (list[i].type == Pathway)
      _builder->configurePathway(list[i].value);
  }
}

const int NUM_ENTRIES = 6;
PersistenceAttribute input[NUM_ENTRIES] = 
{
  { File, "state.dat" }
  , 
  { File, "config.sys" }
  , 
  { Queue, "compute"   }
  , 
  { Queue, "log"  }
  , 
  { Pathway, "authentication"  }
  , 
  { Pathway, "error processing" }
};

int main()
{
  UnixBuilder unixBuilder;
  VmsBuilder vmsBuilder;
  Reader reader;

  reader.setBuilder(&unixBuilder);
  reader.construct(input, NUM_ENTRIES);
  cout << unixBuilder.getResult()->getState() << endl;

  reader.setBuilder(&vmsBuilder);
  reader.construct(input, NUM_ENTRIES);
  cout << vmsBuilder.getResult()->getState() << endl;
}
{% endhighlight %}
Вывод программы:

    Distributed Work Package for: Unix
      File(flatFile): state.dat
      File(flatFile): config.sys
      Queue(FIFO): compute
      Queue(FIFO): log
      Pathway(thread): authentication
      Pathway(thread): error processing
    Distributed Work Package for: Vms
      File(ISAM): state.dat
      File(ISAM): config.sys
      Queue(priority): compute
      Queue(priority): log
      Pathway(LWP): authentication
      Pathway(LWP): error processing

## Объектный пул (Object Pool) {#object_pool}
Объектный пул — набор инициализированных и готовых к использованию объектов. Когда системе требуется объект, он не создается, а берется из пула. Когда объект больше не нужен, он не уничтожается, а возвращается в пул. Применяется для повышения производительности, когда создание объекта в начале работы и уничтожение его в конце приводит к большим затратам. Особенно заметно повышение производительности, когда объекты часто создаются-уничтожаются, но одновременно существует лишь небольшое их число.
![](/assets/2015-01-22-design-patterns-cpp/Object_pool1-2x.png)
***Рекомендации*** 

- Фабричный метод может использоваться для инкапсуляции логики создания объектов. 
- Объектный пул обычно делают синглтоном.

***Ловушки***  

- После того, как объект возвращен, он должен вернуться в состояние, пригодное для дальнейшего использования. Если объекты после возвращения в пул оказываются в неправильном или неопределенном состоянии, такая конструкция называется объектной клоакой (англ. object cesspool).
- Повторное использование объектов также может привести к утечке информации. Если в объекте есть секретные данные (например, номер кредитной карты), после освобождения объекта эту информацию надо затереть.

***Пример реализации***
{% highlight cpp %}
#include <vector>
 
class Object
{
    // ...
};
 
 
class ObjectPool
{
    private:
        struct PoolRecord
        {
            Object* instance;
            bool    in_use;
        };
 
        std::vector<PoolRecord> m_pool;
 
    public:
        Object* createNewObject()
        {
            for (size_t i = 0; i < m_pool.size(); ++i)
                        {
                if (! m_pool[i].in_use)
                {
                    // переводим объект в список используемых
                    m_pool[i].in_use = true;
                    return m_pool[i].instance;
                }
                        }
            // если не нашли свободный объект, то расширяем пул
            PoolRecord record;
            record.instance = new Object;
            record.in_use   = true;
 
            m_pool.push_back(record);
 
            return record.instance;
        }
 
        void deleteObject(Object* object)
        {
            // в реальности не удаляем, а лишь помечаем, 
            // что объект свободен
            for (size_t i = 0; i < m_pool.size(); ++i)
                        {
                if (m_pool[i].instance == object)
                {
                    m_pool[i].in_use = false;
                    break;
                }
                        }
        }
 
        virtual ~ObjectPool()
        {
            // теперь уже "по-настоящему" удаляем объекты
            for (size_t i = 0; i < m_pool.size(); ++i)
                delete m_pool[i].instance;
        }
};
 
 
int main()
{
    ObjectPool pool;
    for (size_t i = 0; i < 1000; ++i)
    {
        Object* object = pool.createNewObject();
        // ...
        pool.deleteObject(object);
    }
    return 0;
}
{% endhighlight %}

# Структурные паттерны   {#structural_pattern}

- Adapter makes things work after they're designed; Bridge makes them work before they are.
- Назначение [моста](#bridge) - чтобы абстракция и реализация могли изменяться независимо. Предназначение адаптера - чтобы несвязанные классы могли работать вместе.
- [Адаптер](#adapter) обеспечивает различные интерфейсы к объекту, [заместитель](#proxy) обеспечивает одинаковый интерфейс, [декоратор](#decorator) расширяет интерфейс.
- [Адаптер](#adapter) изменяет интерфейс объекта, [декоратор](#decorator) - расширяет. Поэтому он более прозрачный для клиента. Как следствие, декоратор поддерживает рекурсивную композицию, что невозможно с "чистым" адаптером.
- Composite can be traversed with Iterator. Visitor can apply an operation over a Composite. Composite could use Chain of responsibility to let components access global properties through their parent. It could also use Decorator to override these properties on parts of the composition. It could use Observer to tie one object structure to another and State to let a component change its behavior as its state changes.
- Composite can let you compose a Mediator out of smaller pieces through recursive composition.
- Декоратор позволяет изменить внешний "облик"(интерфейсы) объекта. Стратегия позволяет изменить внутреннее устройство(реализацию) объекта.
- Декоратор предназначен для добавления обязанностей объекту без его наследования. Фокус внимания компоновщика сосредоточен на представлении. Эти намерения различны, но являются взаимодополняющими. Следовательно, компоновщик и декоратор часто используются вместе.
- Декоратор и Заместитель имеют разные цели, но похожие структуры. Both describe how to provide a level of indirection to another object, and the implementations keep a reference to the object to which they forward requests.
- Facade defines a new interface, whereas Adapter reuses an old interface. Remember that Adapter makes two existing interfaces work together as opposed to defining an entirely new one.
- Фасад зачастую синглтон, потому что только один обьект фасада нужен.
- Mediator is similar to Facade in that it abstracts functionality of existing classes. Mediator abstracts/centralizes arbitrary communication between colleague objects, it routinely "adds value", and it is known/referenced by the colleague objects. In contrast, Facade defines a simpler interface to a subsystem, it doesn't add new functionality, and it is not known by the subsystem classes.
- Whereas Flyweight shows how to make lots of little objects, Facade shows how to make a single object represent an entire subsystem.
- Flyweight is often combined with Composite to implement shared leaf nodes.
- [Приспособленец](#flyweight) explains when and how State objects can be shared.

## Адаптер (Adapter)     {#adapter}
***Синонимы: Обертка (Wrapper).***  
Преобразует интерфейс класса в некоторый другой интерфейс, ожидаемый клиентами. Обеспечивает совместную работу классов, которая была бы невозможна без данного паттерна из-за несовместимости интерфейсов.
![](/assets/2015-01-22-design-patterns-cpp/12_adapter-min.png)
***Применимость***  

- есть существующий класс, но его интерфейс не соответствует потребностям;
- необходимо создать повторно используемый класс, который должен взаимодействовать с заранее неизвестными или не связанными с ним классами, имеющими несовместимые интерфейсы;
- (только для адаптера объектов!) нужно использовать несколько существующих подклассов, но непрактично адаптировать их интерфейсы путем порождения новых подклассов от каждого. В этом случае адаптер объектов может приспосабливать интерфейс их общего родительского класса.

***Отношения***  
Клиенты вызывают операции экземпляра адаптера Adapter. В свою очередь адаптер вызывает операции адаптируемого объекта или класса Adaptee, который и выполняет запрос.

***Родственные паттерны***  
Структура паттерна [Мост](#bridge) аналогична структуре адаптера, но у моста иное назначение. Он отделяет интерфейс от реализации, чтобы то и другое можно было изменять независимо. Адаптер же призван изменить интерфейс существующего объекта. Паттерн [декоратор](#decorator) расширяет функциональность объекта, изменяя его интерфейс. Таким образом, декоратор более прозрачен для приложения, чем адаптер. Как следствие, декоратор поддерживает рекурсивную композицию, что для «чистых» адаптеров невозможно. [Заместитель](#proxy) определяет представителя или суррогат другого объекта, но не изменяет его интерфейс.

***Пример реализации***  
Реализация внешнего полиморфизма для трех несовместимых классов при помощи адаптера.
{% highlight cpp %}
#include <iostream>

using namespace std;
class ExecuteInterface {
  public:
    // 1. Specify the new interface
    virtual ~ExecuteInterface(){}
    virtual void execute() = 0;
};

// 2. Design a "wrapper" or "adapter" class
template <class TYPE>
class ExecuteAdapter: public ExecuteInterface {
  public:
    ExecuteAdapter(TYPE *o, void(TYPE:: *m)()) {
      object = o;
      method = m;
    }
    ~ExecuteAdapter() {
      delete object;
    }

    // 4. The adapter/wrapper "maps" the new 
    //      to the legacy implementation
    void execute() {  /* the new */
      (object->*method)();
    }
  private:
    // ptr-to-object attribute
    TYPE *object;
    // ptr-to-member-function attribute
    void(TYPE:: *method)(); /* the old */
};


// The old: three totally incompatible classes
// no common base class,
class Fea {
  public:
  // no hope of polymorphism
  ~Fea() {
    cout << "Fea::dtor" << endl;
  }
  void doThis() {
    cout << "Fea::doThis()" << endl;
  }
};

class Feye {
  public:~Feye() {
    cout << "Feye::dtor" << endl;
  }
  void doThat() {
    cout << "Feye::doThat()" << endl;
  }
};

class Pheau {
  public:
  ~Pheau() {
    cout << "Pheau::dtor" << endl;
  }
  void doTheOther() {
    cout << "Pheau::doTheOther()" << endl;
  }
};


/* the new is returned */
ExecuteInterface **initialize() {
  ExecuteInterface **array = new ExecuteInterface *[3];

  /* the old is below */
  array[0] = new ExecuteAdapter < Fea > (new Fea(), &Fea::doThis);
  array[1] = new ExecuteAdapter < Feye > (new Feye(), &Feye::doThat);
  array[2] = new ExecuteAdapter < Pheau > (new Pheau(), &Pheau::doTheOther);
  return array;
}

int main() {
  ExecuteInterface **objects = initialize();
  for (int i = 0; i < 3; i++) {
   objects[i]->execute();
  }
 
  // 3. Client uses the new (polymporphism)
  for (i = 0; i < 3; i++) {
    delete objects[i];
  }
  delete objects;
  return 0;
}
{% endhighlight %}

    Fea::doThis()
    Feye::doThat()
    Pheau::doTheOther()
    Fea::dtor
    Feye::dtor
    Pheau::dtor

Более классический пример:  
Интерфейс `LegacyRectangle` не совместимый с системой, в которой его хотят использовать. При помощи абстрактного класса специфицируется желаемый интерфейс. Класс-адаптер публично наследует абстрактный класс с желаемым интерфейсом и приватно `LegacyRectangle`. В своей реализации класс-адаптер "мапит" вызовы нового интерфейса к "старым", уже реализованым интерфейсам `LegacyRectangle`.
{% highlight c++ %}
#include <iostream.h>

typedef int Coordinate;
typedef int Dimension;

// Desired interface
class Rectangle
{
  public:
    virtual void draw() = 0;
};

// Legacy component
class LegacyRectangle
{
  public:
    LegacyRectangle(Coordinate x1, Coordinate y1, 
                    Coordinate x2, Coordinate y2)
    {
        x1_ = x1;
        y1_ = y1;
        x2_ = x2;
        y2_ = y2;
        cout << "LegacyRectangle:  create.  (" << x1_ 
            << "," << y1_ << ") => (" << x2_ << "," 
            << y2_ << ")" << endl;
    }
    void oldDraw()
    {
        cout << "LegacyRectangle:  oldDraw.  (" << x1_ 
            << "," << y1_ << ") => (" << x2_ << "," 
            << y2_ << ")" << endl;
    }
  private:
    Coordinate x1_;
    Coordinate y1_;
    Coordinate x2_;
    Coordinate y2_;
};

// Adapter wrapper
class RectangleAdapter: public Rectangle, private LegacyRectangle
{
  public:
    RectangleAdapter(Coordinate x, Coordinate y, 
                    Dimension w, Dimension h):
      LegacyRectangle(x, y, x + w, y + h)
    {
        cout << "RectangleAdapter: create.  (" << x << "," << y <<
          "), width = " << w << ", height = " << h << endl;
    }
    virtual void draw()
    {
        cout << "RectangleAdapter: draw." << endl;
        oldDraw();
    }
};

int main()
{
  Rectangle *r = new RectangleAdapter(120, 200, 60, 40);
  r->draw();
}
{% endhighlight %}

    LegacyRectangle:  create.  (120,200) => (180,240)
    RectangleAdapter: create.  (120,200), width = 60, height = 40
    RectangleAdapter: draw.
    LegacyRectangle:  oldDraw.  (120,200) => (180,240)

## Декоратор (Decorator) {#decorator}
Также может иметься в виду обертка (Wrapper).
Динамически возлагает на объект новые функции. Декораторы применяются для расширения имеющейся функциональности и являются гибкой альтернативой порождению подклассов.
![](/assets/2015-01-22-design-patterns-cpp/18_decorator-min.png)
***Применимость***  

- для динамического, прозрачного для клиентов добавления обязанностей объектам;
- для реализации обязанностей, которые могут быть сняты с объекта;
- когда расширение путем порождения подклассов по каким-то причинам неудобно или невозможно. Иногда приходится реализовывать много независимых расширений, так что порождение подклассов для поддержки всех возможных комбинаций приведет к комбинаторному росту их числа. В других случаях определение класса может быть скрыто или почему-либо еще недоступно, так что породить от него подкласс нельзя.

***Отношения***  
Декоратор переадресует запросы объекту. Может выполнять и дополнительные операции до и после переадресации.

***Родственные паттерны***  
[Адаптер](#adapter): если декоратор изменяет только обязанности объекта, но не его интерфейс, то адаптер придает объекту совершенно новый интерфейс. [Компоновщик](#composite): декоратор можно считать вырожденным случаем составного объекта, у которого есть только один компонент. Однако декоратор добавляет новые обязанности, агрегирование объектов не является его целью. [Стратегия](#strategy): декоратор позволяет изменить внешний облик объекта, стратегия - его внутреннее содержание. Это два взаимодополняющих способа изменения объекта.

***Пример реализации***  
{% gist 48d75babfda7377f1c86 decorator.cpp %}

    TextField: 80, 24
       ScrollDecorator
       BorderDecorator
       BorderDecorator

## Заместитель (Proxy)   {#proxy}
***Синонимы: суррогат (Surrogate)***  
Подменяет другой объект для контроля доступа к нему, то есть является суррогатом другого объекта. Заместитель применим во всех случаях, когда возникает необходимость сослаться на объект более изощренно, чем это возможно, если использовать простой указатель. С помощью этого паттерна при доступе к объекту вводится дополнительный уровень косвенности.
![](/assets/2015-01-22-design-patterns-cpp/13_proxy-min.png)
***Применимость***  
Паттерн заместитель применим во всех случаях, когда возникает необходимость сослаться на объект более изощренно, чем это возможно, если использовать простой указатель. Вот несколько типичных ситуаций, где заместитель оказывается полезным:

- удаленный заместитель предоставляет локального представителя вместо объекта, находящегося в другом адресном пространстве;
- виртуальный заместитель создает «тяжелые» объекты по требованию;
- защищающий заместитель контролирует доступ к исходному объекту. Такие заместители полезны, когда для разных объектов определены различные права доступа;
- "умная" ссылка - это замена обычного указателя. Она позволяет выполнить дополнительные действия при доступе к объекту. К типичным применениям такой ссылки можно отнести:
    - подсчет числа ссылок на реальный объект, с тем чтобы занимаемую им память можно было освободить автоматически, когда не останется ни одной ссылки (такие ссылки называют еще «умными» указателями);
    - загрузку объекта в память при первом обращении к нему;
    - проверку и установку блокировки на реальный объект при обращении к нему, чтобы никакой другой объект не смог в это время изменить его.

***Отношения***  
Proxy при необходимости переадресует запросы объекту RealSubject. Детали зависят от вида заместителя.

***Родственные паттерны***  
Паттерн [адаптер](#adapter) предоставляет другой интерфейс к адаптируемому объекту. Напротив, заместитель в точности повторяет интерфейс своего субъекта. Однако, если заместитель используется для ограничения доступа, он может отказаться выполнять операцию, которую субъект выполнил бы, поэтому на самом деле интерфейс заместителя может быть и подмножеством интерфейса субъекта. Несколько замечаний относительно [декоратора](#decorator). Хотя его реализация и похожа на реализацию заместителя, но назначение совершенно иное. Декоратор добавляет объекту новые обязанности, а заместитель контролирует доступ к объекту. Степень схожести реализации заместителей и декораторов может быть различной. Защищающий заместитель мог бы быть реализован в точности как декоратор. С другой стороны, удаленный заместитель не содержит прямых ссылок на реальный субъект, а лишь косвенную ссылку, что-то вроде «идентификатор хоста и локальный адрес на этом хосте».

***Пример реализации***  
"->" и "." операторы дают разный результат.
{% gist 46143c0cac0f829dbda6 proxy.cpp %}

    the quick fox jumped over the dog
    the brown fox jumped over the dog

## Компоновщик (Composite)   {#composite}
Группирует объекты в древовидные структуры для представления иерархии типа "часть-целое". Позволяет клиентам работать с единичными объектами так же, как с группами объектов.
![](/assets/2015-01-22-design-patterns-cpp/16_composite-min.png)
***Применимость***  

- нужно представить иерархию объектов вида часть-целое;
- необходимо, чтобы клиенты единообразно трактовали составные и индивидуальные объекты.

***Отношения***  
Клиенты используют интерфейс класса Component для взаимодействия с объектами в составной структуре. Если получателем запроса является листовый объект
Leaf, то он и обрабатывает запрос. Когда же получателем является составной объект Composite, то обычно он перенаправляет запрос своим потомкам, возможно, выполняя некоторые дополнительные операции до или после перенаправления.

***Родственные паттерны***  
Отношение компонент-родитель используется в паттерне [цепочка обязанностей](#chain_of_responsibility). Паттерн [декоратор](#decorator) часто применяется совместно с компоновщиком. Когда декораторы и компоновщики используются вместе, у них обычно бывает общий родительский класс. Поэтому декораторам придется поддержать интерфейс компонентов такими операциями, как Add, Remove и GetChild. Паттерн [приспособленец](#flyweight) позволяет разделять компоненты, но ссылаться на своих родителей они уже не могут. [Итератор](#iterator) можно использовать для обхода составных объектов. [Посетитель](#visitor) локализует операции и поведение, которые в противном случае пришлось бы распределять между классами Composite и Leaf.

***Пример реализации***  
{% gist 1e2de68db5ee6927192d composite.cpp %}

    Row1  Col2  7  Col3  Row4  9  Row5  10  8  6

## Мост (Bridge)     {#bridge}
***Синонимы: Описатель Тело (Handle Body)***  
Отделяет абстракцию от реализации, благодаря чему появляется возможность независимо изменять то и другое.
![](/assets/2015-01-22-design-patterns-cpp/15_bridge-min.png)
***Применимость***  

- необходимо избежать постоянной привязки абстракции к реализации. Так, например, бывает, когда реализацию необходимо выбирать во время выполнения программы;
- и абстракции, и реализации должны расширяться новыми подклассами. В таком случае паттерн мост позволяет комбинировать разные абстракции и реализации и изменять их независимо;
- изменения в реализации абстракции не должны сказываться на клиентах, то есть клиентский код не должен перекомпилироваться;

***Отношения***  
Объект Abstraction перенаправляет своему объекту Implementor запросы клиента.

***Пример реализации***  
{% gist 8332a34a25ce56503c9f bridge.cpp %}

    UseCase1 on Windows :-!
    UseCase1 on Linux! :-)
    UseCase2 on Windows :-!
    UseCase2 on Linux! :-)

## Приспособленец (Flyweight)    {#flyweight}
Использует разделение для эффективной поддержки большого числа мелких объектов. При использовании приспособленцев не исключены затраты на передачу, поиск или вычисление внутреннего состояния, особенно если раньше оно хранилось как внутреннее. Однако такие расходы с лихвой компенсируются экономией памяти за счет разделения объектов-приспособленцев.
![](/assets/2015-01-22-design-patterns-cpp/22_flyweight-min.png)
***Применимость***  
Эффективность паттерна приспособленец во многом зависит от того, как
и где он используется. Применяйть этот паттерн целесообразно, когда выполнены все ниже перечисленные условия:

- в приложении используется большое число объектов;
- из-за этого накладные расходы на хранение высоки;
- большую часть состояния объектов можно вынести вовне;
- многие группы объектов можно заменить относительно небольшим количеством разделяемых объектов, поскольку внешнее состояние вынесено;
- приложение не зависит от идентичности объекта. Поскольку объекты-приспособленцы могут разделяться, то проверка на идентичность возвратит "истину" для концептуально различных объектов.

***Отношения***  

- состояние, необходимое приспособленцу для нормальной работы, можно охарактеризовать как внутреннее или внешнее. Первое хранится в самом объекте ConcreteFlyweight. Внешнее состояние хранится или вычисляется клиентами. Клиент передает его приспособленцу при вызове операций;
- клиенты не должны создавать экземпляры класса ConcreteFlyweight напрямую, а могут получать их только от объекта FlyweightFactory. Это позволит гарантировать корректное разделение.

***Родственные паттерны***  
Паттерн приспособленец часто используется в сочетании с [компоновщиком](#composite) для реализации иерархической структуры в виде ациклического направленного графа с разделяемыми листовыми вершинами. Часто наилучшим способом реализации объектов [состояния](#state) и [стратегии](#strategy) является паттерн приспособленец.

***Пример реализации***  
{% gist 041096bdf4a8a9228806 flyweight.cpp %}

    A ( PointSize 12 )
    A ( PointSize 12 )
    Z ( PointSize 12 )
    Z ( PointSize 12 )
    B ( PointSize 12 )
    B ( PointSize 12 )
    Z ( PointSize 12 )
    B ( PointSize 12 )

## Фасад (Facade)    {#facade}
Предоставляет унифицированный интерфейс к множеству интерфейсов в некоторой подсистеме. Определяет интерфейс более высокого уровня, облегчающий работу с подсистемой. Клиенты общаются с подсистемой, посылая запросы фасаду. Он переадресует их подходящим объектам внутри подсистемы. Хотя основную работу выполняют именно объекты подсистемы, фасаду, возможно, придется преобразовать свой интерфейс в интерфейсы подсистемы. Клиенты, пользующиеся фасадом, не имеют прямого доступа к объектам подсистемы.
![](/assets/2015-01-22-design-patterns-cpp/21_facade-min.png)
***Применимость***  

- необходимо предоставить простой интерфейс к сложной подсистеме. Часто подсистемы усложняются по мере развития. Применение большинства паттернов приводит к появлению меньших классов, но в большем количестве. Такую подсистему проще повторно использовать и настраивать под конкретные нужды, но вместе с тем применять подсистему без настройки становится труднее. Фасад предлагает некоторый вид системы по умолчанию, устраивающий большинство клиентов. И лишь те объекты, которым нужны более широкие возможности настройки, могут обратиться напрямую к тому, что находится за фасадом;
- между клиентами и классами реализации абстракции существует много зависимостей. Фасад позволит отделить подсистему как от клиентов, так и от других подсистем, что, в свою очередь, способствует повышению степени независимости и переносимости;

***Отношения***  
Клиенты общаются с подсистемой, посылая запросы фасаду. Он переадресует их подходящим объектам внутри подсистемы. Хотя основную работу выполняют именно объекты подсистемы, фасаду, возможно, придется преобразовать свой интерфейс в интерфейсы подсистемы. Клиенты, пользующиеся фасадом, не имеют прямого доступа к объектам подсистемы.

***Родственные паттерны***  
Паттерн [абстрактная фабрика](#abstract_factory) допустимо использовать вместе с фасадом, чтобы предоставить интерфейс для создания объектов подсистем способом, не зависимым от этих подсистем. Абстрактная фабрика может выступать и как альтернатива фасаду, чтобы скрыть платформенно-зависимые классы. Паттерн [посредник](#proxy) аналогичен фасаду в том смысле, что абстрагирует функциональность существующих классов. Однако назначение посредника - абстрагировать произвольное взаимодействие между «сотрудничающими» объектами. Часто он централизует функциональность, не присущую ни одному из них. Коллеги посредника обмениваются информацией именно с ним, а не напрямую между собой. Напротив, фасад просто абстрагирует интерфейс объектов подсистемы, чтобы ими было проще пользоваться. Он не определяет новой функциональности, и классам подсистемы ничего неизвестно о его существовании. Обычно требуется только один фасад. Поэтому объекты фасадов часто бывают [одиночками](#singleton).

***Пример реализации***  
{% gist 774b16f2d9c55d8472db facade.cpp %}

    submitted to Facilities - 1 phone calls so far
    submitted to Electrician - 12 phone calls so far
    submitted to MIS - 25 phone calls so far
    job completed after only 35 phone calls

# Паттерны поведения {#behavioral_patterns}

## Интерпретатор (Interpreter)   {#interpreter}
Для заданного языка определяет представление его грамматики, а также интерпретатор предложений языка, использующий это представление.
![](/assets/2015-01-22-design-patterns-cpp/06_interpreter-min.png)
***Пример реализации***  
Uses a class hierarchy to represent the grammar given below. When a roman numeral is provided, the class hierarchy validates and interprets the string. RNInterpreter "has" 4 sub-interpreters. Each sub-interpreter receives the "context" (remaining unparsed string and cumulative parsed value) and contributes its share to the processing. Sub-interpreters simply define the Template Methods declared in the base class RNInterpreter.

`romanNumeral ::= {thousands} {hundreds} {tens} {ones} thousands, hundreds, tens, ones ::= nine | four | {five} {one} {one} {one} nine ::= "CM" | "XC" | "IX" four ::= "CD" | "XL" | "IV" five ::= 'D' | 'L' | 'V' one ::= 'M' | 'C' | 'X' | 'I'`
{% gist 7c7992884efbcd8858d9 interpreter.cpp %}

    Enter Roman Numeral: MCMXCVI
       interpretation is 1996
    Enter Roman Numeral: MMMCMXCIX
       interpretation is 3999
    Enter Roman Numeral: MMMM
       interpretation is 0
    Enter Roman Numeral: MDCLXVIIII
       interpretation is 0
    Enter Roman Numeral: CXCX
       interpretation is 0
    Enter Roman Numeral: MDCLXVI
       interpretation is 1666
    Enter Roman Numeral: DCCCLXXXVIII
       interpretation is 888


## Шаблонный метод (Template Method) {#template_method}
Определяет скелет алгоритма, перекладывая ответственность за некоторые его шаги на подклассы. Позволяет подклассам переопределять шаги алгоритма, не меняя его общей структуры. Шаблонные методы - один из фундаментальных приемов повторного использования кода. Они особенно важны в библиотеках классов, поскольку предоставляют возможность вынести общее поведение в библиотечные классы. Шаблонные методы приводят к инвертированной структуре кода, которую иногда называют принципом Голливуда, подразумевая часто употребляемую в этой кино-империи фразу "Не звоните нам, мы сами позвоним". В данном случае это означает, что родительский класс вызывает операции подкласса, а не наоборот.
![](/assets/2015-01-22-design-patterns-cpp/09_templatemethod-min.png)
***Пример реализации***  
{% gist 01c7522e5f75ec3a8239 template_method.cpp %}

    a  b  c  d  e
    a  2  c  4  e

## Итератор (Iterator)   {#iterator}
***Синонимы: Курсор (Cursor)***  
Дает возможность последовательно обойти все элементы составного объекта, не раскрывая его внутреннего представления.
![](/assets/2015-01-22-design-patterns-cpp/08_iterator-min.png)
***Пример реализации***  
{% gist 81edaa76480d09255825 iterator.cpp %}

    1 == 2 is 1
    1 == 3 is 0
    1 == 4 is 0
    1 == 5 is 0

## Команда (Command) {#command}
***Синонимы: Действие (Action), Сценарий транзакции (Transaction Script)***  
Инкапсулирует запрос в виде объекта, позволяя тем самым параметризовывать клиентов типом запроса, устанавливать очередность запросов, протоколировать их и поддерживать отмену выполнения операций.
![](/assets/2015-01-22-design-patterns-cpp/04_command-min.png)

## Наблюдатель (Observer)    {#observer}
***Синонимы: Подчиненные (Dependents), Издатель Подписчик (Publish Subscribe)***  
Определяет между объектами зависимость типа один-ко-многим, так что при изменении состояния одного объекта все зависящие от него получают извещение и автоматически обновляются. Шаблон позволяет изменять субъекты и наблюдатели независимо друг от друга. Субъекты разрешается повторно использовать без участия наблюдателей и наоборот. Это дает возможность добавлять новых наблюдателей без модификации субъекта или других наблюдателей.
![](/assets/2015-01-22-design-patterns-cpp/03_observer-min.png)

## Посетитель (Visitor)  {#visitor}
Представляет операцию, которую надо выполнить над элементами объекта. Позволяет определить новую операцию, не меняя классы элементов, к которым он применяется.
![](/assets/2015-01-22-design-patterns-cpp/10_visitor-min.png)

## Посредник (Mediator)  {#mediator}
Определяет объект, в котором инкапсулировано знание о том, как взаимодействуют объекты из некоторого множества. Способствует уменьшению числа связей между объектами, позволяя им работать без явных ссылок друг на друга. Это, в свою очередь, дает возможность независимо изменять схему взаимодействия. Коллеги посылают запросы посреднику и получают запросы от него. Посредник реализует кооперативное поведение путем переадресации каждого запроса подходящему коллеге (или нескольким коллегам).
![](/assets/2015-01-22-design-patterns-cpp/11_mediator-min.png)

## Состояние (State)     {#state}
Позволяет объекту варьировать свое поведение при изменении внутреннего состояния. При этом создается впечатление, что поменялся класс объекта.
![](/assets/2015-01-22-design-patterns-cpp/05_state-min.png)

## Стратегия (Strategy)  {#strategy}
***Синонимы: Политика (Policy)***  
Определяет семейство алгоритмов, инкапсулируя их все и позволяя подставлять один вместо другого. Можно менять алгоритм независимо от клиента, который им пользуется.
![](/assets/2015-01-22-design-patterns-cpp/07_strategy-min.png)

## Хранитель (Memento)   {#memento}
***Синонимы: Лексема (Token)***  
Позволяет, не нарушая инкапсуляции, получить и сохранить во внешней памяти внутреннее состояние объекта, чтобы позже объект можно было восстановить точно в таком же состоянии.
![](/assets/2015-01-22-design-patterns-cpp/01_memento-min.png)

## Цепочка обязанностей (Chain of Responsibility)    {#chain_of_responsibility}
Можно избежать жесткой зависимости отправителя запроса от его получателя, при этом запросом начинает обрабатываться один из нескольких объектов. Объекты-получатели связываются в цепочку, и запрос передается по цепочке, пока кокой-то объекта его не обработает.
![](/assets/2015-01-22-design-patterns-cpp/02_chain-min.png)

# Представление бизнес-логики    {#representation_of_business_logic}

## Сценарий транзакции (Transaction Script)  {#transaction_script}
![](/assets/2015-01-22-design-patterns-cpp/transaction-script.gif)  
Способ организации бизнес-логики по процедурам, каждая из которых обслуживает один запрос, инициализируемый слоем представления. Главным достоинством сценария транзакции является простота решения. Вам не приходится беспокоиться о наличии и вариантов функционирования других параллельных транзакций. Обычно сценарий транзакции размещается в классах, отличных от тех, которые относятся к слоям представления и источника данных. Может использоваться один класс для реализации нескольких сценариев транзакции или вариант, следующий схеме типового решения команда ([Command](#command)) - собственный класс для каждого сценария транзакции. В последнем возможность манипулировать экземплярами сценариев как объектами в период выполнения помогает преодолевать проблемы потоков вычислений и облегчает изоляцию данных.

## Модель предметной области (Domain Model)  {#domain_model}
![](/assets/2015-01-22-design-patterns-cpp/domain-model.gif)  
Это объектная модель домена, охватывающая поведение (функции) и свойства (данные). Предусматривает создание сети взаимосвязанных объектов, каждый из которых представляет некую осмысленную сущность. Реализация модели предметной области означает пополнение приложения целым слоем объектов, описывающих различные стороны определенной области бизнеса. С моделью предметной области связано большое количество различных контекстов. Модели предметной области особенно хороши, когда существует множество схожих условий протекания процесса, которые могут быть отображены в структуре объектов как таковых. В этом случае сложность алгоритмов вычислений перемещается на уровень связей между объектами. Чем более близка логика, тем с большей вероятностью различные части кода системы должны пользоваться одними и теми же связями. Любому алгоритму вычислений соответствует определенная сеть объектов. Также назначение модели предметной области во многом состоит в сокрытии базы данных от верхних слоев системы.

## Модуль таблицы (Table Module) {#table_module}
![](/assets/2015-01-22-design-patterns-cpp/table-module.gif)  
Объект, охватывающий логику обработки всех записей хранимой или виртуальной таблицы базы данных. Предусматривает создание по одному классу на каждую таблицу базы данных, и единственный экземпляр класса содержит всю логику обработки данных таблицы. Основное отличие модуля таблицы от модели предметной области ([Domain Model](#domain_model)) состоит в том, что если, например, приложение обслуживает множество заказов, в соответствии с моделью предметной области придется сконструировать по одному объекту на каждый заказ, а при использовании модуля таблицы понадобится всего один объект, представляющий одновременно все заказы. Сильная сторона решения модуль таблицы заключается в том, что оно позволяет сочетать данные и функции для их обработки и в то же время эффективно использовать ресурсы реляционной базы данных. На первый взгляд модуль таблицы во многом напоминает обычный объект, но отличается тем, что не содержит какого бы то ни было упоминания об идентификационном признаке объекта. Типовое решение модуль таблицы во многом основывается на табличной структуре данных и потому допускает очевидное применение в ситуациях, где доступ к информации обеспечивается при посредничестве множеств записей ([Record Set](#record_set)). Структуре данных отводится центральная роль, так что методы обращения к данным в структуре должны быть прямолинейными и эффективными.

## Слой служб (Service Layer)    {#service_layer}
![](/assets/2015-01-22-design-patterns-cpp/service-layer.gif)  
Устанавливает множество доступных действий и координирует отклик приложения на каждое действие. Слой служб определяет границы приложения и множество операций, предоставляемых им для интерфейсных клиентских слоев кода. Он инкапсулирует бизнес-логику приложения, управляет транзакциями и координирует реакции на действия. Подобно сценарию транзакции ([Transaction Script](#transaction_script)) и модели предметной области ([Domain Model](#domain_model)), слой служб представляет собой типовое решение по организации бизнес-логики. Слой служб предусматривает распределение "разной" логики по отдельным слоям, что обеспечивает традиционные преимущества расслоения, а также большую степень свободы применения классов домена в разных приложениях. Двумя базовыми вариантами реализации слоя служб являются создание интерфейса доступа к домену (Domain Facade) и конструирование сценария операции (Operation Script). Преимуществом использования слоя служб является возможность определения набора общих операций, доступных для применения многими категориями клиентов, и координация откликов приложения на выполнение каждой операции.

# Архитектурные типовые решения источника данных {#architecture_of_standard_solutions_data_source}

## Шлюз таблицы данных (Table Data Gateway)  {#table_data_gateway}
![](/assets/2015-01-22-design-patterns-cpp/tabledatagateway.png)  
Объект, выполняющий роль шлюза ([Gateway](#gateway)) к базе данных. Он содержит в себе все команды SQL, необходимые для извлечения, вставки, обновления и удаления данных из таблицы или представления. Методы шлюза таблицы данных используются другими объектами для взаимодействия с базой данных. Как правило, для каждой таблицы базы данных создается собственный шлюз таблицы данных. Это наиболее простое типовое решение интерфейса базы данных, поскольку оно замечательно отображает таблицы или записи баз данных на объекты. Одно из преимуществ использования шлюза таблицы данных для инкапсуляции доступа к базе данных состоит в том, что этот интерфейс может применяться и для обращения к базе данных с помощью средств языка SQL, и для работы с хранимыми процедурами. Данное решение подходит практически для любой платформы, так как представляет собой не более чем оболочку для операторов SQL.

## Шлюз записи данных (Row Data Gateway)     {#row_data_gateway}
![](/assets/2015-01-22-design-patterns-cpp/dbgateRow.gif)  
Объект, выполняющий роль шлюза (Gateway) к отдельной записи источника данных. Каждой строке таблицы базы данных соответствует свой экземпляр шлюза записи данных. Шлюз выступает в роли интерфейса к строке данных и прекрасно подходит для применения в сценариях транзакции (Transaction Script). Реализация шлюза записи данных включает в себя только логику доступа к базе данных и никакой логики домена. Добавление логики домена в шлюз записи данных превращает его в активную запись ([Active Record](#active_record)).

## Активная запись (Active Record)   {#active_record}
![](/assets/2015-01-22-design-patterns-cpp/active-record.gif)  
Объект, выполняющий роль оболочки для строки таблицы или представления базы данных. Он инкапсулирует доступ к базе данных и добавляет к данным логику домена. В основе типового решения активная запись лежит модель предметной области ([Domain Model](#domain_model)). Каждая активная запись отвечает за сохранение и загрузку информации в базу данных, а также за логику домена, применяемую к данным. Это может быть вся бизнес-логика приложения. Впрочем, иногда некоторые фрагменты логики домена содержатся в сценариях транзакции ([Transaction Script](#transaction_script)), а общие элементы кода, ориентированные на работу с данными, - в активной записи. Активная запись очень похожа на шлюз записи данных ([Row Data Gateway](row_data_gateway)). Принципиальное отличие между ними в том, что шлюз записи данных содержит только логику доступа к базе данных, в то время как активная запись содержит и логику доступа к данным, и логику домена. Активная запись хорошо подходит для реализации не слишком сложной логики домена, в частности операций создания, считывания, обновления и удаления. Кроме того, она прекрасно справляется с извлечением и проверкой на правильность отдельной записи.

## Преобразователь данных (Data Mapper)  {#data_mapper}
![](/assets/2015-01-22-design-patterns-cpp/data-mapper.gif)  
Это слой преобразователей (Mapper), который осуществляет передачу данных между объектами и базой данных, сохраняя последние независимыми друг от друга и от самого преобразователя. То есть это слой программного обеспечения, которое отделяет объекты, расположенные в оперативной памяти, от базы данных. В функции преобразователя данных входит передача данных между объектами и базой данных и изоляция их друг от друга. В результате этого объекты, расположенные в оперативной памяти, могут даже не "подозревать" о самом факте присутствия базы данных. Им не нужен SQL-интерфейс и тем более схема базы данных. (В свою очередь, схема базы данных никогда не "знает" об объектах, которые её используют.) Поскольку преобразователь данных является разновидностью преобразователя ([Mapper](#mapper)), он полностью скрыт от уровня домена. В большинстве случаев преобразователь данных применяется для того, чтобы схема базы данных и объектная модель могли изменяться независимо друг от друга. Как правило, подобная необходимость возникает при использовании модели предметной области ([Domain Model](#domain_model)). Основным преимуществом преобразователя данных является возможность работы с моделью предметной области без учета структуры базы данных как в процессе проектирования, так и во время сборки и тестирования проекта. В этом случае объектам домена ничего не известно о структуре базы данных, поскольку все отображения выполняются преобразователями.


# Объектно-реляционные типовые решения, предназначенные для моделирования поведения  {#object-relational_model_solutions_designed_to_simulate_the_behavior}

## Единица работы (Unit of Work)     {#unit_of_work}
![](/assets/2015-01-22-design-patterns-cpp/unit-of-work.gif)  
Содержит список объектов, охватываемых бизнес-транзакцией, координирует запись изменений в базу данных и разрешает проблемы параллелизма. Типовое решение единица работы позволяет контролировать все действия, выполняемые в рамках бизнес-транзакции, которые так или иначе связаны с базой данных. По завершении всех действий единица работы определяет окончательные результаты работы, которые и будут внесены в базу данных. Таким образом не приходится отслеживать, что было изменено, или беспокоиться о том, в каком порядке необходимо выполнить нужные действия, чтобы не нарушить целостность на уровне ссылок - это все сделает единица работы. Основным назначением единицы работы является отслеживание действий, выполняемых над объектами домена, для дальнейшей синхронизации данных, хранящихся в оперативной памяти, с содержимым базы данных. Эта синхронизация выполняется единицей работы в конце бизнес-транзакции, для избежания большого количества мелких обращений к базе данных.

## Коллекция объектов (Identity Map)     {#identity_map}
![](/assets/2015-01-22-design-patterns-cpp/identity-map.gif)  
Гарантирует, что каждый объект будет загружен из базы данных только один раз, сохраняя загруженный объект в специальной коллекции. При получении запроса просматривает коллекцию в поисках нужного объекта. Коллекция объектов может быть явной (explicit), кода доступ осуществляется посредством специализированных методов, соответствующих конкретному типу искомого объекта, или универсальной (generic), когда для доступа ко всем типам объектов используется общий метод. Как правило, коллекция объектов применяется для управления любыми объектами, которые были загружены из базы данных и затем подверглись изменениям. Основное назначение коллекции объектов - не допустить возникновения ситуации, когда два разных объекта приложения будут соответствовать одной и той же записи базы данных, поскольку изменение этих объектов может происходить несогласованно и, следовательно, вызывать трудности с отображением на базу данных. Еще одним преимуществом коллекции объектов является возможность использования ее в качестве кэша записей, считываемых из базы данных. Это избавляет от необходимости повторного обращения к базе данных, если снова понадобиться какой-нибудь объект.

## Загрузка по требованию (Lazy Load)    {#lazy_load}
![](/assets/2015-01-22-design-patterns-cpp/lazy-load.gif)  
Объект, который не содержит все требующиеся данные, однако может загрузить их в случае необходимости. Существует четыре основных способа реализации загрузки по требованию. Самый простой - инициализация по требованию (Lazy Initialization). Основная идея данного подхода заключается в том, что при каждой попытке доступа к полю выполняется проверка, не содержит ли оно значение NULL. Если поле содержит NULL, метод доступа загружает значение поля и лишь затем его возвращает. Если используется преобразователь данных ([Data Mapper](#data_mapper)), то может понадобиться реализовать виртуальный прокси-объект (Virtual Proxy). Виртуальный прокси-объект имитирует объект, являющийся значением поля, однако в действительности ничего в себе не содержит. В этом случае загрузка реального объекта будет выполнена только тогда, когда будет вызван один из методов виртуального прокси-объекта. Диспетчер значений (Value Holder) - еще один способ реализации - это объект, который выполняет роль оболочки для какого-нибудь другого объекта. Чтобы добраться к значению базового объекта, необходимо обратиться за ним к диспетчеру значения. При первом обращении диспетчер значения извлекает необходимую информацию из базы данных. И наконец, фиктивный объект (Ghost) - это реальный объект с неполным состоянием. Когда подобный объект загружается из базы, он содержит только свой идентификатор. При первой же попытке доступа к одному из его полей объект загружает значения всех остальных полей. Помимо всего прочего, использование загрузки по требованию может повлечь за собой чрезмерное количество обращений к базе данных. Бессмысленно использовать загрузку по требованию, чтобы извлечь значения поля, хранящегося в той же строке, что и остальное содержимое объекта, даже если речь идет о полях большого размера, таких как крупный сериализованный объект ([Serialized LOB](#serialized_lob)) (тут я не совсем согласен с автором). Применение загрузки по требованию существенно усложняет приложение, поэтому прибегать к ней нужно только тогда, когда без нее действительно не обойтись.


# Объектно-реляционные типовые решения, предназначенные для моделирования структуры  {#object-relational_model_solutions_for_modeling_the_structure}

## Поле идентификации (Identity Field)   {#identity_field}
![](/assets/2015-01-22-design-patterns-cpp/identity-field.gif)  
Сохраняет идентификатор записи базы данных для поддержки соответствия между объектом приложения и строкой базы данных. Суть поля идентификации до смешного проста: требуется лишь сохранить первичный ключ таблицы реляционной базы данных в полях объекта, однако в реализации этого шаблона существует множество спорных моментов. Поле идентификации используется тогда, когда необходимо построить отображение между объектами, расположенными в оперативной памяти, и строками таблицы базы данных. Как правило, такая необходимость возникает при использовании модели предметной области ([Domain Model](#domain_model)) или шлюза записи данных ([Row Data Gateway](#row_data_gateway)). Подобное отображение не нужно, если вы используете сценарий транзакции ([Transaction Script](#transaction_script)), модуль таблицы ([Table Module](#table_module)) или шлюз таблицы данных ([Table Data Gateway](#table_data_gateway)).

## Отображение внешних ключей (Foreign Key Mapping)  {#foreign_key_mapping}
![](/assets/2015-01-22-design-patterns-cpp/foreign-key-mapping.gif)  
Отображает ассоциации между объектами на ссылки внешнего ключа между таблицами базы данных. Обычно реализуется посредством добавления обратного указателя на "объект-хозяин", что приводит к появлению ссылки, аналогичной создаваемой внешним ключом. Это изменяет объектную модель, однако позволяет значительно упростить обновление данных путем простого обновления однозначных полей "с другой стороны" ссылки. Отображение внешних ключей может применяться для моделирования практически всех связей между классами. Наиболее распространенный случай, когда отображение внешних ключей применить нельзя, - это связи "многие ко многим". Внешние ключи являются одномерными значениями, а из определения первой нормальной формы следует, что в одном поле нельзя хранить множественные значения внешних ключей. В этом случае вместо отображения внешних ключей необходимо воспользоваться отображением с помощью таблицы ассоциаций ([Association Table Mapping](#association_table_mapping)).

## Отображение с помощью таблицы ассоциаций (Association Table Mapping)  {#association_table_mapping}
![](/assets/2015-01-22-design-patterns-cpp/association-table-mapping.gif)  
Сохраняет множество ассоциаций в виде таблицы, содержащей внешние ключи таблицы, связанных ассоциациями. В основе отображения с помощью таблицы ассоциаций лежит хранение ассоциаций в дополнительной таблицы отношений. Последняя содержит только значения внешних ключей двух таблиц, связанных отношением. Таким образом, каждой паре взаимосвязанных объектов соответствует одна строчка таблицы отношений.

## Отображение зависимых объектов (Dependent Mapping)    {#dependent_mapping}
![](/assets/2015-01-22-design-patterns-cpp/dependent-mapping.gif)  
Передает некоторому классу полномочия по выполнению отображения для дочернего класса. Один класс (зависимый объект) передает другому классу (владельцу) все свои полномочия по взаимодействию с базой данных. При этом у каждого зависимого объекта должен быть один и только один владелец. Данный принцип проявляет себя в терминах классов, выполняющих отображение. В случае с активной записью ([Active Record](#active_record)) и шлюзом записи данных ([Row Data Gateway](#row_data_gateway)) зависимый класс не будет содержать никакого кода, касающегося выполнения отображения на базу данных; этот код будет реализован в классе владельце. В случае с преобразователем данных ([Data Mapper](#data_mapper)) у зависимого класса не будет своего преобразователя; всё необходимое отображение будет выполняться преобразователем класса-владельца. И наконец, в случае шлюза таблицы данных ([Table Data Gateway](#table_data_gateway)) зависимого класса не будет вообще; всю обработку зависимых данных будет осуществлять класс-владелец. Зависимый объект не имеет поля идентификации ([Identity Field](#identity_field)) и, следовательно, не заноситься в коллекцию объектов ([Identity Map](#identity_map)). Для зависимого объекта вообще не предусмотрено отдельных методов поиска, так как весь поиск выполняется объектом-владельцем. Отображение зависимых объектов может применяться только при соблюдении следующих условий: у каждого зависимого объекта должен быть строго один владелец; на зависимый объект может ссылаться только его владелец.

## Внедренное значение (Embedded Value)  {#embedded_value}
![](/assets/2015-01-22-design-patterns-cpp/embedded-value.gif)  
Отображает объект на несколько полей таблицы, соответствующей другому объекту. Обычно происходит отображение значения полей объекта на поля записи его владельца. Или другими словами, поля таблицы некоего объекта, как бы группируются в одном поле этого объекта, представляя собой подчиненный объект, содержащий значения полей объекта-владельца. Он может представлять собой, например, объект-значение ([Value Object](#value_object)). Когда-объект владелец загружается из базы данных или сохраняется в ней, вместе с ним загружаются или сохраняются и зависимые объекты. Зависимые классы не имею собственных методов загрузки и сохранения, поскольку все эти операции выполняются их владельцем. Для большей наглядности внедренное значение можно рассматривать как частный случай отображения зависимых объектов ([Dependent Mapping](#dependent_mapping)), где значения поля таблицы является отдельным зависимым объектом. Огромным преимуществом внедренного значения является возможность постановки SQL-запросов к полям зависимого объекта.

## Сериализованный крупный объект (Serialized LOB)   {#serialized_lob}
![](/assets/2015-01-22-design-patterns-cpp/serialized-lob.gif)  
Сохраняет граф объектов путем их сериализации в единый крупный объект и помещает его в поле базы данных. Это разновидность типового решения хранитель ([Memento](#memento)). Может представлять собой как крупный двоичный объект (BLOB), так и крупный символьный объект (CLOB). Первый сохранятся в виде двоичного формата данных, а второй представляет собой текстовую строку, например, в виде XML.

## Наследование с одной таблицей (Single Table Inharitence)  {#single_table_inheritence}
![](/assets/2015-01-22-design-patterns-cpp/single-table-inheritance.gif)  
Представляет иерархию наследования классов в виде одной таблицы, столбцы которой соответствуют всем полям классов, входящих в иерархию. Наследование с одной таблицей отображает все поля всех классов структуры на столбцы одной и той же таблицы. То есть структура наследования отображается на одну таблицу, которая содержит в себе все данные всех классов, входящих в иерархию наследования. Каждому классу (а точнее, его экземпляру) соответствует одна строка таблицы; при этом поля таблицы, которых нет в данном классе, остаются пустыми. Основное представление объектов, выполняющих отображение, соответствует общей схеме преобразователей наследования ([Inheritance Mappers](#inheritance_mappers)).

## Наследование с таблицами для каждого класса (Class Table Inheritance) {#class_table_inheritence}
***Синонимы: отображение "корень-лист" (Root-Leaf Mapping)***  
![](/assets/2015-01-22-design-patterns-cpp/class-table-inheritance.gif)  
Представляет иерархию наследования классов, используя по одной таблице для каждого класса. Идея наследование с таблицами для каждого класса проста и понятна: каждому классу модели предметной области соответствует своя таблица базы данных. Поля класса домена отображаются непосредственно на столбцы соответствующей таблицы. Как и в других схемах отображения иерархии наследования, в данном типовом решении применяется фундаментальный принцип преобразователей наследования ([Inheritance Mappers](#inheritance_mappers)).

## Наследование с таблицами для каждого конкретного класса (Concrete Table Inheritance)  {#concrete_table_inheritence}
![](/assets/2015-01-22-design-patterns-cpp/concrete-table-inheritance.gif)  
Представляет иерархию наследования классов, используя по одной таблице для каждого конкретного класса. При этом каждая таблица содержит столбцы, соответствующие полям конкретного класса и всех его "предков", а потому поля суперкласса дублируются во всех таблицах его производных классов. Как и остальные схемы отображения иерархии наследования, данное типовое решение основано на фундаментальном принципе преобразователей наследования ([Inheritance Mappers](#inheritance_mappers)). Наследование с таблицами для каждого конкретного класса часто рассматривают как разновидность наследования с таблицами для каждого листа (Leaf Table Inheritance), когда создается по одной таблице для каждого листа иерархии наследования, а не для каждого конкретного класса.

## Преобразователи наследования (Inheritance Mappers)    {#inheritance_mappers}
![](/assets/2015-01-22-design-patterns-cpp/inheritance-mappers.gif)  
Структура, предназначенная для организации преобразователей, которые работают с иерархиями наследования. Минимизирует количество кода, необходимого для загрузки и сохранения содержимого базы данных, при отображении объектно-ориентированной иерархии наследования, благодаря точному распределению обязанностей по структуре преобразователей, реализующих как абстрактное так и конкретное поведение. Хотя детали поведения могут различаться в зависимости от выбранной схемы отображения (наследование с одной таблицей ([Single Table Inheritence](#single_table_inheritence)), наследование с таблицами для каждого класса ([Class Table Inheritence](#class_table_inheritence)), наследование с таблицами для каждого конкретного класса ([Concrete Table Inheritence](#concrete_table_inheritence))), общая структура остается одной и той же.


# Шаблоны, предназначенные для представления данных в Web    {#the_templates_are_designed_to_provide_data_on_the_web}

## Модель-представление-контроллер (Model View Controller)   {#model_view_controller}
![](/assets/2015-01-22-design-patterns-cpp/mvc.png)  
Распределяет обработку взаимодействия с пользовательским интерфейсом между тремя участниками. Модель - это объект, предоставляющий некоторую информацию о домене. Содержит в себе все данные и поведение и не связана с пользовательским интерфейсом. В наиболее "чистой" форме представляет собой объект модели предметной области ([Domain Model](#domain_model)). Представление отображает содержимое модели средствами графического интерфейса. Все изменения информации обрабатываются контроллером. Получая входные данные от пользователя, он выполняет операции над моделью и указывает представлению на необходимость соответствующего обновления. Данное типовое решение реализует два принципиальных типа разделения: отделение представления от модели, являющееся одним из фундаментальных принципов проектирования программного обеспечения, и отделение контроллера от представления. Последний тип разделения практически не играет такой важной роли в системах с толстым клиентом, однако в Web-интерфейсах отделение весьма полезно.

## Контроллер страниц (Page Controller)  {#page_controller}
![](/assets/2015-01-22-design-patterns-cpp/page-controller.gif)  
Объект, который обрабатывает запрос к Web-странице или выполнение конкретного действия на Web-сайте. Предполагает наличие отдельного контроллера для каждой логической страницы Web-сайта. Этим контроллером может быть сама страница или отдельный объект, соответствующий данной странице. То есть контроллер страниц может быть реализован в виде сценария (сценария CGI, сервлета и т.п.) или страницы сервера (ASP, PHP, JSP и т.п.). Контроллер страниц не обязательно должен представлять собой единственный класс, зато все классы контроллеров могут использовать одни и те же вспомогательные объекты. Контроллер страниц более прост в работе чем контроллер запросов (Front Controller) и представляет собой естественный механизм структуризации, при котором конкретные действия обрабатываются соответствующими страницами сервера или классами сценариев. Поэтому контроллер страниц хорошо применять для сайтов с достаточно простой логикой контроллера.

## Контроллер запросов (Front Controller)    {#front_Controller}
![](/assets/2015-01-22-design-patterns-cpp/front-controller.gif)  
Контроллер, который обрабатывает все запросы к Web-сайту. Он объединяет все действия по обработке запросов в одном месте, распределяя их выполнение посредством единственного объекта обработчика. Как правило этот объект реализует общее поведение, которое может быть изменено во время выполнения с помощью декораторов ([Decorator](#decorator)). Для выполнения конкретного запроса обработчик вызывает соответствующий объект команды ([Command](#command)). Выбор команды может происходить статически или динамически.

## Контроллер приложения (Application Controller)    {#application_controller}
![](/assets/2015-01-22-design-patterns-cpp/application-controller.gif)  
Точка централизованного управления порядком отображения интерфейсных экранов и потоком функций приложения. Входные контроллеры обращаются к контроллеру приложения за командами, которые следует применить к модели, и за представлениями, которые необходимо использовать в определенном контексте приложения. Контроллер приложения выполняет две основные функции: выбор логики домена, которую нужно применить в конкретной ситуации, и выбор представления, которое следует отобразить в ответ на запрос. Для осуществления этих функций контроллер приложения поддерживает две коллекции ссылок на классы - одну для команд, выполняющихся в слое домена, и одну для представлений. Приложение может иметь несколько контроллеров приложений, управляющих его составными частями. Благодаря этому сложная логика выбора интерфейсных экранов может быть распределена по нескольким классам. Более простому приложению достаточно и одного контроллера. Если логика Web-приложения, описывающая порядок отображения интерфейсных экранов, довольно проста (другими словами, если пользователь может открывать экраны приложения практически в любом порядке), применять контроллер приложения не имеет смысла. Основное преимущество данного типового решения состоит именно в определении порядка отображения страниц и выборе тех или иных представлений в зависимости от состояний объектов. Необходимость использования контроллера приложения очевидна, если различные изменения хода приложения требуют применения схожей логики, касающиеся выбора команд и представлений особенно если такие изменения возникают во многих местах приложения.

## Представление по шаблону (Template View)  {#template_view}
![](/assets/2015-01-22-design-patterns-cpp/template-view.gif)  
Преобразует результат выполнения запрос в формат HTML путём внедрения маркеров в HTML-страницу. В основном выполняется вставка маркеров в текст готовой статической HTML-страницы, при вызове которой для обслуживания запроса эти маркеры будут преобразованы в вызовы функций, предоставляющих динамическую информацию. Подобная схема позволяет создавать статическую часть страницы с помощью обычных средств, например WYSIWYG-редакторов. Для избежания внедрения в страницу большого количества программной логики применяют вспомогательный объект (Helper Object). Он будет содержать в себе всю фактическую логику домена, а сама страница - только вызовы вспомогательного объекта. Это значительно упростит структуру страницы и максимально приблизит её к "чистой" форме представления по шаблону. Преимущество представления по шаблону перед представлением с преобразованием ([Transform View](#transform_view)) в том, что оно позволяет оформлять представление в соответствии со структурой страницы, а также воплотить в жизнь идею, когда дизайнер занимается проектированием страницы, а программист работает над вспомогательным объектом. Однако, реализуя представление в виде страницы сервера, последнюю весьма легко переполнить логикой. Также представление по шаблону сложнее тестировать, чем представлением с преобразованием ([Transform View](#transform_view)), так как большинство реализаций представления по шаблону ориентированы на использование в рамках Web-сервера.

## Представление с преобразованием (Transform View)  {#transform_view}
![](/assets/2015-01-22-design-patterns-cpp/transform-view.gif)  
Представление, которое поочередно обрабатывает элементы данных домена и преобразует их в код HTML. На вход подаются данные из модели, а на выходе принимается код HTML. При этом программа последовательно проходит по структуре данных домена и, обнаруживая новый фрагмент данных создает их описание в терминах HTML. Отличие представления с преобразованием от представления по шаблону ([Template View](#template_view)) заключается в способе организации представления. Представление по шаблону организовано с учетом размещения на экране выходных данных. Представление с преобразованием ориентировано на использование отдельных преобразований для каждого вида входных данных. Преобразованиями управляет нечто наподобие простого цикла, который поочередно просматривает каждый входной элемент, подбирает для него подходящее преобразование и применяет это преобразование. Таким образом, правила представления с преобразованием могут быть организованы в любом порядке - на результат это не повлияет. Преобразования изначально направлены на визуализацию данных в формат HTML, что позволяет избежать внедрения в представление слишком большого количества логики. Представление с преобразованием легче тестировать, поскольку, в основном, его реализация не требует наличия функционирующего Web-сервера. Представления по шаблону ([Template View](#template_view)) не выдерживает последних двух достоинств представления с преобразованием.

## Двухэтапное представление (Two Step View) {#two_step_view}
![](/assets/2015-01-22-design-patterns-cpp/two-step-view.gif)  
Выполняет визуализацию данных домена в два этапа: вначале формирует некое подобие логической страницы, после чего преобразует логическую страницу в формат HTML. На первом этапе информация, полученная от модели, организуется в некую логическую структуру, которая описывает визуальные элементы будущего отображения, однако еще не содержит кода HTML. На втором этапе полученная логическая структура преобразуется в код HTML. Методы второго этапа "знают", какие элементы есть в логической структуре и как визуализировать каждый из этих элементов. Таким образом, система с множеством экранов может быть трансформирована в код HTML путём единственного прохождения второго этапа, благодаря чему решение о варианте преобразования в HTML принимается в одном месте. Двухэтапное преобразование может быть построено как на основе представления с преобразованием ([Transform View](#transform_view)) так и на основе представления по шаблону ([Template View](#template_view)). Главным преимуществом двухэтапного представления является возможность разбить преобразование данных на два этапа, что облегчает проведение глобальных изменений. Одноэтапный вариант представления, будь то представление по шаблону или представление с преобразованием, предусматривает по одному компоненту представления для каждой Web-страницы приложения. В двухэтапном представлении визуализация данных выполняется в два этапа, что требует по одному представлению первого этапа для каждой страницы приложения и единственное представление второго этапа для всего приложения. Последняя схема значительно облегчает изменение внешнего вида сайта на втором этапе, поскольку каждое такое изменение распространяется сразу на весь сайт. Таким образом, для приложений с несколькими вариантами внешнего вида используется меньше элементов регулировки отображения и чем больше интерфейсных экранов и вариантов внешнего вида есть у приложения, тем большим будет выигрыш.
 
