## 1.Spring в реальном мире

### 1.1 Зачем нужны фреймворки

Современные приложения состоят из множества компонентов, взаимодействующих между собой. Без фреймворков разработчику пришлось бы каждый раз с нуля реализовывать:

- Инфраструктурный код (логгирование, управление транзакциями, обработку исключений);

- Настройку подключения к базе данных;

- Тестируемую архитектуру и инъекцию зависимостей.

- Фреймворки обеспечивают:

- Повторное использование архитектурных решений.

- Унифицированные подходы к конфигурации, внедрению зависимостей, тестированию.

- Снижение количества шаблонного кода.

Spring стал стандартом де-факто в Java-разработке.

### 1.2 Экосистема Spring

Spring — это набор модулей, образующих экосистему:

### 1.2.1 Spring Core

- Основа контейнера инверсии управления (IoC Container).

- Управляет созданием, настройкой и жизненным циклом объектов (бинов).

- Поддерживает DI (внедрение зависимостей).

### 1.2.2 Spring Data Access

- Упрощает работу с JDBC и ORM-фреймворками (Hibernate).

- Позволяет работать с БД без шаблонного кода.

- Поддерживает транзакции через аннотацию @Transactional.

1.2.3 Spring MVC

- Поддержка создания REST и web-приложений.

- Использует архитектуру Model-View-Controller.

- Аннотации @Controller, @RequestMapping, @RestController.

1.2.4 Spring Testing

- Интеграция с JUnit и TestNG.

- Возможность запуска контейнера Spring при тестировании.

- Использование @SpringBootTest, @MockBean, @WebMvcTest.

1.2.5 Другие проекты Spring

- Spring Boot — автоматическая конфигурация и запуск приложений.

- Spring Security — обеспечение аутентификации и авторизации.

- Spring Cloud — микросервисная инфраструктура.

- Spring Batch, Spring Integration, Spring WebFlux — для разных задач.

1.3 Реальные сценарии применения Spring

- Бэкенды корпоративных приложений.

- REST API.

- Микросервисные системы.

- Веб-приложения.

- Системы автоматизации и десктопные приложения (редко).

1.4 Когда не стоит использовать Spring

- В малых скриптах и утилитах, где Spring добавляет избыточность.

- В системах с критичными ограничениями по размеру/памяти.

- Если необходимо 100% ручное управление безопасностью или временем выполнения (например, в real-time системах).

## 2. Контекст Spring: что такое бины

### 2.1 Контейнер IoC

Spring IoC Container — это механизм создания и управления объектами. Контейнер получает инструкции по созданию объектов (бинов) через конфигурацию.

Bean — управляемый объект в контейнере Spring.

Контейнер отвечает за создание, инициализацию, внедрение зависимостей и уничтожение бина.

Контейнер запускается через AnnotationConfigApplicationContext, SpringApplication.run() или ClassPathXmlApplicationContext (если XML).

2.2 Определение бинов

**Способ 1: @Bean + @Configuration**

```java
@Configuration
public class AppConfig {
    @Bean
    public Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
}
```



**Способ 2: Стереотипные аннотации**

@Component — базовая аннотация.

@Service — для сервисов бизнес-логики.

@Repository — для DAO и доступа к БД.

@Controller — для MVC-контроллеров.

```java
@Component
public class Parrot {
    private String name;
    // сеттеры и геттеры
}
```

**Способ 3: Программное добавление бина**

```java
var context = new AnnotationConfigApplicationContext();
context.registerBean("parrot", Parrot.class, () -> new Parrot("Koko"));
context.refresh();
```

## 3. Создание новых бинов и внедрение зависимостей

### 3.1 Внедрение зависимостей (Dependency Injection)

DI — основной механизм Inversion of Control в Spring. Контейнер создает объекты и внедряет в них зависимости, избавляя разработчика от необходимости ручного связывания компонентов.

Варианты внедрения:

1. **Через поля**

Преимущество — краткость. Недостаток — трудность при тестировании (нельзя передать зависимость через конструктор).

```java
@Component
public class Person {
    @Autowired
    private Parrot parrot;
}
```
2. **Через конструктор**

Рекомендуемый способ внедрения. Обеспечивает неизменность зависимостей, удобство для модульных тестов.

```java
@Component
public class Person {
private final Parrot parrot;

    @Autowired  // начиная с Spring 4.3 можно опустить, если один конструктор
    public Person(Parrot parrot) {
        this.parrot = parrot;
    }
}
```

3. **Через сеттер**

Удобно для необязательных зависимостей.

```java
@Component
public class Person {
private Parrot parrot;

    @Autowired
    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }
}
```

### 3.2 Дополнительные возможности внедрения

@Primary

Используется, если есть несколько реализаций одного типа:

```java
@Bean
@Primary
public Parrot defaultParrot() {
return new Parrot("Default");
}
```

@Qualifier

Указывает, какой именно бин нужно внедрить:

```java
@Bean(name = "parrot1")
public Parrot parrot1() { 
    return new Parrot("Koko"); 
}

@Bean(name = "parrot2")
public Parrot parrot2() { 
    return new Parrot("Miki"); 
}

@Autowired
@Qualifier("parrot2")
private Parrot parrot;
```

@Value

Внедрение значений из конфигурации:

@Value("${app.parrot.name}")
private String name;

3.3 Особые случаи внедрения

Внедрение коллекций

Spring может внедрять список всех реализаций интерфейса:

@Autowired
private List<Parrot> parrots;

Внедрение по условию (@ConditionalOn...)

В Spring Boot можно внедрять зависимости по определенным условиям:

@Bean
@ConditionalOnMissingBean
public Parrot fallbackParrot() { return new Parrot("Fallback"); }

### 3.4 Циклические зависимости и способы их избегать

Spring не позволяет создавать циклические зависимости при внедрении через конструктор.

Решения:

Использовать @Lazy:

```java
@Autowired
@Lazy
private A a;
```

Изменить архитектуру: применить фасад, промежуточный сервис и т.п.

Внедрять через сеттер или поле, если необходимо.

### 3.5 Примеры

Классическая зависимость

```java
@Component
public class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

@Component
public class Car {
private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

Таким образом, внедрение зависимостей — это фундамент Spring, обеспечивающий модульность, расширяемость и легкость тестирования приложений.

## 4. Использование абстракций

### 4.1 Интерфейсы как контракт

Определение зависимостей по интерфейсу позволяет легко менять реализацию:

```java
public interface NotificationService {
    void send(String message);
}

@Service
public class EmailService implements NotificationService {
    public void send(String message) {
        System.out.println("Отправка письма: " + message);
    }
}
```

### 4.2 Внедрение интерфейса

```java
@Autowired
private NotificationService service;
```

Если более одной реализации:
```java
@Autowired
@Qualifier("smsService")
private NotificationService service;
```

### 4.3 Роли компонентов

@Service — бизнес-логика.

@Repository — DAO, поддержка обработки исключений.

@Component — универсальная аннотация.

## 5. Области видимости и жизненный цикл бинов

### 5.1 Области видимости (@Scope)

- singleton — один экземпляр на всё приложение (по умолчанию).

- prototype — создается новый объект при каждом запросе.

- request, session, application — используются в веб-контексте.

```java
@Bean
@Scope("prototype")
public Parrot parrot() {
    return new Parrot();
}
```

### 5.2 Ленивые бины (@Lazy)

По умолчанию все singleton-бины создаются при инициализации контекста.
С помощью аннотации @Lazy можно отложить создание бина до первого обращения:

```java
@Bean
@Lazy
public Parrot parrot() {
    return new Parrot();
}
```
@Lazy можно применять и на уровне класса:

```java
@Lazy
@Component
public class HeavyBean { ... }
```

### 5.3 Жизненный цикл бина

Жизненный цикл бина в Spring включает несколько этапов:

Создание экземпляра — через конструктор или фабричный метод.

Внедрение зависимостей — через конструктор, поле, сеттер.

Выполнение методов инициализации — если задано.

Использование — бин участвует в работе приложения.

Удаление (для singleton) — вызываются методы завершения (если указаны).

Методы жизненного цикла:

1. @PostConstruct — метод, вызываемый после создания и внедрения зависимостей:

```java
@PostConstruct
public void init() {
    System.out.println("Инициализация бина");
}
```

2. @PreDestroy — вызывается перед уничтожением бина:

```java
@PreDestroy
public void destroy() {
    System.out.println("Удаление бина");
}
```

3. InitializingBean и DisposableBean

Интерфейсы, которые можно реализовать для управления жизненным циклом:

```java
public class Bird implements InitializingBean, DisposableBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Инициализация через интерфейс");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Уничтожение через интерфейс");
    }
}
```
4. Настройка методов в @Bean

```java
@Bean(initMethod = "customInit", destroyMethod = "customDestroy")
public Parrot parrot() {
    return new Parrot();
}
```

5.4 Отличия singleton и prototype

Singleton-бин создается один раз на весь ApplicationContext.

Prototype-бин создается каждый раз при запросе, но Spring не управляет его уничтожением.

```java
@Component
@Scope("prototype")
public class Toy {
    public Toy() {
        System.out.println("Создан новый Toy");
    }
}
```
При prototype Spring не вызывает @PreDestroy и не управляет очисткой памяти.

## 6. Аспекты и AOP в Spring (подробно)

### 6.1 Что такое AOP (Aspect-Oriented Programming)

AOP — парадигма программирования, позволяющая отделить сквозную функциональность от основной бизнес-логики.

Сквозная функциональность — поведение, повторяющееся в нескольких частях приложения:

- логгирование,

- аутентификация/авторизация,

- управление транзакциями,

- кэширование,

- метрики и мониторинг.

С помощью AOP можно внедрить такую функциональность без изменения основного кода.

### 6.2 Основные понятия AOP

- Aspect - Класс, содержащий сквозную логику.

- Advice - Метод, содержащий поведение, внедряемое в приложение (например, @Before, @After).

- Join point - Точка в программе, куда можно внедрить аспект (обычно метод).

- Pointcut - Условие, определяющее, где применяется advice.

- Weaving - Процесс связывания аспектов с основным кодом (во время компиляции, загрузки или выполнения).

### 6.3 Виды Advice

- @Before — выполняется перед вызовом метода.

- @After — выполняется после завершения метода.

- @AfterReturning — выполняется после успешного завершения метода.

- @AfterThrowing — выполняется при выбрасывании исключения.

- @Around — оборачивает вызов метода (до и после). Позволяет изменить поведение, вернуть другое значение или отменить вызов.

### 6.4 Пример аспекта

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Вызов метода: " + joinPoint.getSignature());
    }

    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result")
    public void logAfter(Object result) {
        System.out.println("Метод успешно завершен. Результат: " + result);
    }

    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex")
    public void logException(Exception ex) {
        System.out.println("Произошло исключение: " + ex.getMessage());
    }
}
```
### 6.5 Синтаксис Pointcut выражений

Pointcut выражения описывают, к каким методам применяется аспект:

execution(* com.example.service.*.*(..)) — все методы всех классов в пакете com.example.service.

within(com.example.service.*) — все методы классов в пакете.

this(...) и target(...) — бин реализует или расширяет указанный тип.

@annotation(...) — методы, помеченные конкретной аннотацией.

### 6.6 Использование @Around

```java
@Aspect
@Component
public class TimingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object measureExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long end = System.currentTimeMillis();
        System.out.println("Время выполнения метода: " + (end - start) + " мс");
        return result;
    }
}
```

### 6.7 Ограничения Spring AOP

Используются прокси — оборачивают бин, реализующий интерфейс или класс.

Работают только с публичными методами.

По умолчанию работает JDK Dynamic Proxy (если бин реализует интерфейс) или CGLIB (если нет интерфейса).

Вызовы внутри одного бина не перехватываются (self-invocation). Но для обхода можно бин внедрить сам в себя.
При этом не будет цикличности, Spring понимает, что внедрение идет в самого себя.

### 6.8 Применение в реальных проектах

Логгирование всех вызовов REST-методов.

Обработка исключений централизованно.

Автоматическое открытие/закрытие транзакций.

Валидация или аудит вызовов.