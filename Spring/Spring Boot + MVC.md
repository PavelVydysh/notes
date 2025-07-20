# Глава 7. Spring Boot и веб-приложения
## Веб-приложения и встроенный Tomcat
Веб-приложение — это приложение, работающее на сервере и доступное по сети. Оно может использовать разные протоколы взаимодействия:

- HTTP/HTTPS (наиболее распространённый случай);

- gRPC (бинарный протокол поверх HTTP/2, часто используется в микросервисах);

- WebSocket (двусторонняя связь в реальном времени);

а также другие протоколы в зависимости от задач.

На практике, большинство веб-приложений взаимодействует с браузером по HTTP, но это не единственный вариант. Веб-приложение может обслуживать как браузеры, так и мобильные клиенты, другие серверы или IoT-устройства.

Tomcat — это встроенный в Spring Boot веб-сервер (Servlet-контейнер), который:

- слушает порт (по умолчанию 8080);

- принимает HTTP-запросы от клиентов;

- передаёт их в DispatcherServlet Spring'а;

- возвращает результат клиенту.

В классическом Spring требовалась установка Tomcat отдельно и упаковка WAR-файла. В Spring Boot — Tomcat встроен, и приложение запускается как обычный .jar:

Tomcat также поддерживает:

- сессии;

- фильтры и сервлеты;

- статические ресурсы;

- многопоточность и обработку параллельных запросов.

## Цель Spring Boot
Spring Boot создан, чтобы устранить боль начальной конфигурации в Spring. Он предоставляет:

- стартовые зависимости (starters),

- автоконфигурацию,

- встроенный сервер (Tomcat),

- запуск приложения одной командой.

## 7.1 Зависимости и стартеры
Проблема Spring
В классическом Spring нужно было:

- писать XML или Java-конфигурацию,

- вручную подключать зависимости (spring-context, spring-aop, и т.д.),

- конфигурировать логгеры, транзакции, веб-контейнер и т.д.

Spring Boot предлагает стартеры — это группы зависимостей.

Примеры:

- spring-boot-starter-web → Spring MVC + Jackson + Tomcat

- spring-boot-starter-data-jpa → Spring Data + JPA + Hibernate

- spring-boot-starter-security → Spring Security

- spring-boot-starter-thymeleaf → Thymeleaf

Автоматически подключается логгер (SLF4J + Logback).

## 7.2 Автоконфигурация
Spring Boot использует аннотацию @EnableAutoConfiguration, которая:

- ищет зависимости в classpath,

- применяет «наилучшую» конфигурацию по умолчанию.

Пример:
Если есть spring-boot-starter-data-jpa, то Boot:

- создает EntityManager,

- подключает DataSource,

- инициализирует H2 или другую базу, если она найдена.

## 7.3 Запуск приложения
С помощью аннотации @SpringBootApplication:
```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }
}
```

Эта аннотация объединяет:

- @Configuration

- @ComponentScan

- @EnableAutoConfiguration

## 7.4 Работа с вебом
Если подключить spring-boot-starter-web, то автоматически настраивается:

- DispatcherServlet,

- конвертеры (Jackson),

- RestController,

- встроенный Tomcat (порт 8080).

Пример контроллера:
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot!";
    }
}
```

Пример запроса:
curl http://localhost:8080/hello

## 7.5 application.properties
Файл для настройки приложения:

```properties
server.port=9000
spring.application.name=my-app
spring.jpa.show-sql=true
```

Или application.yml:

```yml
server:
  port: 9000
```

## 7.6 Статические ресурсы
Spring Boot обслуживает ресурсы из следующих путей:

- classpath:/static/ - статические файлы, доступные при прямом обращении к ним

- classpath:/templates/ - файлы шаблонов представлений, обычно используется шаблонизаторами(Thymeleaf, Mustache и тд) 

- classpath:/resources/ - ресурсы проекта(yml, liqui-скрипты и тд)

## 7.7 Работа с шаблонами
Spring Boot поддерживает шаблонизаторы:

- Thymeleaf

- FreeMarker,

- Mustache и др.

Шаблоны размещаются в resources/templates/.

Пример Thymeleaf-шаблона:

```html
<h1 th:text=\"${message}\">Hello!</h1>
```
## 7.8 DevTools
Spring Boot DevTools — это набор удобных инструментов для ускорения разработки приложений на Spring Boot.
DevTools помогает быстро видеть изменения в коде без полной перезагрузки приложения, а также предоставляет дополнительные удобства для отладки.

**Основные возможности Spring Boot DevTools**:
- Автоматическая перезагрузка приложения (Auto Restart)
При изменении кода DevTools автоматически перезапускает приложение. Это происходит быстрее, чем полная перезагрузка JVM, потому что используется специальный ClassLoader.

- LiveReload
Если у тебя подключён LiveReload-браузер-плагин (или IDE с поддержкой), браузер автоматически обновится при изменении ресурсов (например, HTML, CSS, JS).

- Улучшенная конфигурация для разработки
DevTools автоматически включает/отключает некоторые настройки (например, кеширование шаблонов, логирование), чтобы облегчить разработку.

- Удалённая отладка (Remote Debugging)
Можно настроить DevTools для работы с удалённым приложением, позволяя использовать те же удобства в продакшене (обычно с ограничениями).

- Property Defaults
DevTools переопределяет некоторые настройки Spring Boot при запуске, чтобы они подходили для режима разработки.

**Особенности:**
- DevTools отключается в production — по умолчанию она не включается в собранный jar для продакшена, чтобы не влиять на производительность.

- Если в IDE не работает автообновление, нужно убедиться, что проект компилируется автоматически (например, в IntelliJ включить Build project automatically).

- DevTools перезапускает приложение при изменениях классов и ресурсов, но не при изменениях в зависимостях.

## 7.9 Профили
Позволяют создавать конфигурации для разных окружений.

Примеры:

application-dev.properties

application-prod.properties

Активация:

```properties
spring.profiles.active=dev
```

В совокупности с аннотацией @Profile, позволяет создавать определенные бины, только при использовании конкретного профиля

## 7.10 Логгирование
Spring Boot использует SLF4J + Logback по умолчанию.

SLF4J (Simple Logging Facade for Java) — это абстрактный слой для логирования в Java, который предоставляет простой и единый интерфейс для разных систем логирования.

SLF4J не пишет логи сам, а служит "прослойкой" (фасадом) между приложением и конкретной библиотекой логирования (например, Logback, Log4j, java.util.logging и т.п.)

Код логирования пишется через SLF4J API — а под капотом уже используется выбранная реальная библиотека логирования, например Logback

Spring Boot:

устраняет необходимость ручной настройки,

позволяет запускать Spring-приложение за минуты,

делает конфигурацию декларативной и прозрачной.

## MVC

**MVC = Model - View - Controller**
Паттерн проектирования, разделяющий ответственность:

**Model** - Данные, бизнес-логика - Java-объекты (DTO, Entity)

**View** - Отображение данных (HTML, JSON, PDF и т.п.) - Thymeleaf, JSP, JSON и т.д.

**Controller** - Принимает запросы, подготавливает модель, выбирает view - @Controller, @RestController

Controller — это класс, в котором описываются методы, реагирующие на запросы от клиента (браузера, мобильного приложения и т.п.) и возвращающие данные или представления.

Аннотаций:
```java
@Controller
class MainController {}
```
Контроллер содержит методы, обозначающие конечные точки запросов, каждый метод соответствует своему запросу, например:
```java
@Controller
class MainController {
    
    @RequestMapping("/home")
    public String home() {
        return "home.html";
    }

    @RequestMapping("/login")
    public String login() {
        return "login.html";
    }
    
}
```

ViewResolver — это интерфейс, который отвечает за преобразование имени представления (view name), возвращённого из контроллера, в объект View, который знает, как отобразить (отрендерить) ответ клиенту.

Spring Boot реализует из коробки InternalResourceViewResolver и BeanNameViewResolver(используется редко),
Если название представления указать без расширения, то Spring сделает редирект, например:

```java
@Controller
public class MainController {


    @RequestMapping("/login")
    public String login() {
        return "reg";
    }

    @RequestMapping("/reg")
    public String reg() {
        return "reg.html";
    }

}
```

Если обратиться на ручку "/login", то Spring перенаправит запрос на "/reg" и в итоге получим страницу reg.html
Это происходит, потому что InternalResourceViewResolver получив название представления, просто делает редирект,
поэтому дял его корректной работы, нужно возвращать полное название файла и файл должен лежать в static ресурсах проекта

## Полный путь запроса в MVC

1. **Клиент -> Tomcat** 

Клиент делает запрос, Tomcat перехватывает его

2. **Tomcat -> Dispatcher Servlet**

Сервлет — это Java-класс, который обрабатывает HTTP-запросы и формирует HTTP-ответы. Это основа веб-приложений на Java.

В Spring MVC есть только один сервлет - Dispatcher Servlet, он является точкой входа в веб приложение
и при получении запроса, определяет какому методу какого контроллера переджать управление

3. **Dispatcher Servlet -> Controller**
Dispacther Servlet, с помощью карты контроллеров определяет, какой метод какого контроллера, должен обработать пришедший запрос,
если контроллер не найден, то вернется ошибка 404, если же найден - сервлет вызовет его,
после выполнения, контроллер вернет название представления

4. **Dispatcher Servlet -> View Resolver**
Получив, название представления, сервлет передает управление цепочке View Resolver(их может быть несколько одновременно),
Получая в ответ построенное и готовое представление

5. **Dispatcher Servlet -> Tomcat**
Сервлет формирует ответ и передает его Tomcat

6. Tomcat -> Клиент
Tomcat передает ответ клиенту.

**На месте Tomcat может быть любой контейнер сервлетов**

Забегая вперед, при использовании RestController процесс такой же, только вместо ViewResolver вызывается HttpMessageConvertre,
чаще всего - Jackson

## Реализация MVC

**@Controller**

Отмечает класс как контроллер, обрабатывающий HTTP-запросы.

По умолчанию возвращаемое значение из методов является обозначением представлений.

**@ResponseBody**

Каждое возвращаемое значение будет сразу сериализовано в тело ответа(например, в JSON или XML)

**@RestController**

Отмечает класс как контроллер, обрабатывающий HTTP-запросы.

Каждое возвращаемое значение из методов будет сразу сериализовано в тело ответа (например, в JSON или XML), вместо рендеринга шаблона.

Фактически объединяет @Controller + @ResponseBody

**@RequestMapping**

Универсальная аннотация для обработки HTTP-запросов.

Позволяет указать метод запроса, путь, заголовки, params и т.д.

```java
@RequestMapping(value = "/hello", method = RequestMethod.GET)
public String hello() { ... }
```

**@GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping**

Это специализации @RequestMapping.

Просто сокращённая запись:

```java
@GetMapping("/hello")            // эквивалентен @RequestMapping(method = GET)
@PostMapping("/submit")          // POST
@PutMapping("/update")           // PUT
@DeleteMapping("/delete/{id}")   // DELETE
```

**@PathVariable, @RequestParam, @RequestBody**

@PathVariable

Извлекает часть пути из URL.

```java
@GetMapping("/user/{id}")
public String getUser(@PathVariable Long id) { ... }
// /user/42 → id = 42
```

@RequestParam

Извлекает параметры запроса из строки URL (?name=Alice&age=30).

```java
@GetMapping("/search")
public String search(@RequestParam String name) { ... }
// /search?name=Bob → name = Bob
```

@RequestBody

Считывает тело запроса (чаще всего JSON) и конвертирует в объект Java.

```java
@PostMapping("/user")
public String createUser(@RequestBody UserDto user) { ... }
// тело запроса: { "name": "Alice", "age": 25 }
```

## Model

Model — это объект, через который ты передаёшь данные из контроллера в представление (HTML-шаблон).

```java
@GetMapping("/hello")
public String hello(Model model) {
    model.addAttribute("name", "Alice");
    return "hello";
}
```

## Thymeleaf

Thymeleaf — это серверный шаблонизатор, который позволяет встраивать переменные и условия прямо в HTML, и Spring Boot его поддерживает из коробки.

Предоставляет свой ViewResolver.

Пример html:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head><title>Hello</title></head>
<body>
  <h1 th:text="'Hello, ' + ${name}">Hello</h1>
</body>
</html>
```

Работает в совокупности с Model.

# Web области видимости бинов

@RequestScope - область видимости в рамках запроса.
На каждый запрос выдается новый объект.

@SessionScope - область видимости в рамках одной сессии.
если пользователь делает запрос - создается сессия, которая запоминается на сервере, следующий запрос от этого же пользователя,
будет в рамках этйо же сессии, если она не была удалена и при запросе вернется тот же бин, что и при первом запросе

Для идентификации сессии используется JSESSIONID, который передается вместе с запросом.

**Важно:**

Для внедрения бинов с областью видимости запроса/сессии необходимо использовать прокси,
т.к. бин внедряется только при инициализации запроса/сессии, а не при старте приложения. 
Прокси высутпает в роли заглушки и при создании запроса/сессии обратится к контексту для получения бина и вызова его методов

@ApplicationScope - область видимости в рамках приложения. Бин активен, пока активно приложение.

Отличие Application от Singleton:

1. singleton scope — по имени бина:

- В Spring singleton означает "один экземпляр на контейнер по имени бина", а не по классу.

- Это значит, что можно создать несколько singleton-бинов одного и того же класса, если у них разные имена.

2. application scope — по типу (на уровне ServletContext):
   В application-scope (как и в session, request) Spring создаёт один бин на весь ServletContext.

## Обработка ошибок

@ControllerAdvice — это аннотация в Spring, позволяющая централизованно обрабатывать исключения

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public String handleIllegalArgument(IllegalArgumentException ex, Model model) {
        model.addAttribute("error", ex.getMessage());
        return "error"; // имя HTML-шаблона (если используется MVC)
    }
}
```

@ExceptionHandler - обозначает какие исключения попадают под обработку этим методом.

Controller Advice действует на все контроллеры(но толко на контроллеры, тк является частью Spring MVC и работает в рамках HTTP запросов)