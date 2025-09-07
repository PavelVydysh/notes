# Основы Spring Security

## Использование по умолчанию

Для подключения секьюрити к приложению достаточно добавить зависимость security-starter.
В этом случае будет использоваться стандартная реализация security, которая использует
аутентификацию HTTP Basic.

Basic аутентификация предполагает передачу логина и пароля в HTTP заголовке Authorization в формате:
Basic userId:passwd. Комбинация userId:passwd кодируется в Base64 

Этот тип аутентификации очень небезопасен, так как передает логин и пароль пользователя в запросе
и любой злоумышленник, перехвативший запрос, сможет без проблем достать эти данные.
Basic можно использовать только для тестирования и дев среды, но не на проде.
Так же лучше совмещать Basic с https.

## Устройство Spring Security

Последовательность аутентификации:

1. **Фильтр аутентификации(Authentication Filter)** перехватывает запрос и делегирует его менеджеру аутентификации 
и на основе того, что вернет менеджер - настраивает контекст безопасности.
2. **Менеджер аутентификации(Authentication Manager)** использует провайдер аутентификации для обработки аутентификации
3. **Провайдер аутентификации(Authentication Provider)** реализует конкретную логику аутентификации
4. **Служба сведений о пользователе(User Details Service)** реализует ответственность за управление пользователем,
которую провайдер аутентификации использует в логике аутентификации
5. **Кодировщик паролей(Password Encoder)** реализует управление паролями, которое провайдер
аутентификации использует в логике аутентификации
6. **Контекст безопасности(Security Context)** сохраняет данные аутентификации после процесса аутентификации. 
Контекст будет хранить данные до тех пор, пока действие не завершится. Обычно используется подход "один поток на действие".

## UserDetailsService по умолчанию

Реализация UserDetailsService по умолчанию хранит учетные данные в памяти приложения.
По умолчанию - это пользователь user и пароль UUID. Пароль генерируется случайно при загрузке контекста Spring(запуске приложения)

## PasswordEncoder по умолчанию

Делает две вещи:

- Кодирует пароль(обычно с использованием алгоритма шифрования или хеширования)
- Проверяет, совпадает ли пароль с существующим закодированным значением

Дефолтная реализация не кодирует пароли и использует их в виде обычного текста.

По умолчанию PasswordEncoder работает совместно с UserDetailsService. Когда мы заменяем UserDetailsService,
мы так же должны заменить PasswordEncoder

## Переопределение конфигурации
По умолчанию для реализации UserDetailsServices используется InMemoryUserDetailsManager, он несколько шире, чем просто реализация UserDetailsService.

Пример переопределения UserDetailsService и PasswordEncoder:

```java
@Bean
    UserDetailsService userDetailsService() {
        UserDetails details = User.withUsername("john")
                .password("123456")
                .authorities("read")
                .build();

        return new InMemoryUserDetailsManager(details);
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
```

Установка закрытости конечных точек:

```java
@Bean
    SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.httpBasic(Customizer.withDefaults());
        httpSecurity.authorizeHttpRequests(
                c -> c.anyRequest().authenticated()
        );

        return httpSecurity.build();
    }
```

Customizer - это контракт, который реализуется для определения настройки любого элемента Spring Security. Это функциональный интерфейс.

Так же можно сконфигурировать UserDetailsService напрямую в методе конфигурации SecurityFilterChain: 

```java
@Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.httpBasic(Customizer.withDefaults());
        httpSecurity.authorizeHttpRequests(
                c -> c.anyRequest().authenticated()
        );

        UserDetails details = User.withUsername("john")
                .password("123456")
                .authorities("read")
                .build();

        UserDetailsService service = new InMemoryUserDetailsManager(details);

        httpSecurity.userDetailsService(service);
        return httpSecurity.build();
    }
```

Переопределение AuthenticationProvider:

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = String.valueOf(authentication.getCredentials());

        if("john".equals(username) && "123456".equals(password)) {
            return new UsernamePasswordAuthenticationToken(username, password, List.of());
        } else {
            throw new AuthenticationCredentialsNotFoundException("Error!");
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken
                .class
                .isAssignableFrom(authentication);
    }

}
```

Так же провайдера можно зарегистрировать в методе определения SecurityFilterChain,
вызывав у объекта HttpSecurity метод `authenticationProvider`

## Итоги

- Spring Boot предоставляет ряд конфигураций по умолчанию при
добавлении Spring Security к зависимостям приложения.

- Реализуются следующие базовые компоненты для аутентифика
ции и авторизации: UserDetailsService, PasswordEncoder и Authen
ticationProvider.

- Можно определить пользователей с по мощью класса User.
У пользователя должны быть как минимум имя пользователя, пароль и полномочия.
Полномочия – это действия, которые разрешается пользователю выполнять в
контексте приложения.

- Простая реализация UserDetailsService, которую предоставляет
Spring Security, – это InMemoryUserDetailsManager. Можно добавлять пользователей
в такой экземпляр UserDetailsService для управления пользователем в памяти приложения.

- NoOpPasswordEncoder – это реализация контракта PasswordEncoder,
который использует пароли в открытом тексте. Эта реализация
хороша для изучения примеров и (возможно) проверки работо
способности идей, но не для приложений, готовых к производству.

- Можно использовать контракт AuthenticationProvider для
реализации пользовательской логики аутентификации в приложении.

- Существует несколько способов написания конфигураций, но
в рамках приложения следует придерживаться одного выбранного подхода. 
Это помогает сделать ваш код чище и проще для понимания.