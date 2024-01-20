---
layout: post
title: "Testing in Spring & Code Coverage"
date: 2023-04-15
description: "Testing in Spring"
tag: Spring Framework
---

# Testing in Spring

## Unit Test vs Integration Test

`Unit Test` is a test focused on `functional groups` inside an application, generally we will test backend service layer by layer to make sure all layers and all methods work fine.

`Integration Test` is a test focused on the interaction between different modules. For example, test the redirction between two pages, test the interaction between frontend and backend.

## What do you use for testing in Spring? (Mockito)

`Mockito` is a `mocking framework` used for `unit testing` of Java applications. When we write unit tests, we need to mock the `external dependencies`. Mockito provide an easy way to mock the dependencies.

- Add dependency in pom file.

  ```xml
  <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>3.11.2</version>
      <scope>test</scope>
  </dependency>
  ```

- Several annotations in test class.

1. `@ExtendWith(MockitoExtension.class)`

   This annotation is used to tell JUnit to use Mockito's test runner when running the tests. It is a replacement of `@RunWith(MockitoJUnitRunner.class)`.

2. `@Mock`

   This annotation is used to create and inject mocked instances.

3. `@Spy`

   This annotation is used to create and inject partially mocked instances. It is used when we want to mock the behavior of a method of a class while real method invocations for other methods of the class.

4. `@InjectMocks`

   This annotation is used to inject mock fields into the tested object automatically. The class needs to be tested.

- Example

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Spy
    private UserMapper userMapper;

    @Test
    public void testGetUserById() {
        User user = new User();
        user.setId(1L);
        user.setName("test");
        user.setAge(20);
        Mockito.when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        User userById = userService.getUserById(1L);
        Assertions.assertEquals(userById.getName(), "test");
    }
}
```

### Difference between `@Mock`, `@Spy`, `@InjectMocks`

`@Mock` is used to create and inject mocked instances. Often used to mock the dao layer. It will not call the real method inside.

`@InjectMocks` will inject the mocked instances into the tested object automatically. Often used to mock the service layer. It will call the real method inside. (suppose we are testing service layer, the service should be marked as `@InjectMocks`. becasue we need to run the logic inside each method)

`@Spy` is used to create and inject partially mocked instances. It will call the real method inside. It is used when we want to call the real method and perform some logic inside the method. like sending an email and then test if the email is sent successfully.

## What is the difference between `@Mock` and `@MockBean`? (same with `@Spy` and `@SpyBean`)

They both are used to create a fake object. `@Mock` create object not related to the spring framework.

When testing controller, it need us to start the Spring Boot application. We use `@MockBean` to create a fake object that is part of the Spring context. such as beans declared in the application's configuration or beans created by Spring Boot auto-configuration.

## doReturn - when vs when - thenReturn

They both are used to mock the return value of a method.

`when - thenReturn` will not check the return type of the method. Try to use `doReturn - when` instead.

## Other annotations

- `@BeforeEach` and `@AfterEach` are used to run the method before and after each test methods. It is non-static method.

- `@DisplayName` is used to display the name of the test method.

- `@Disabled` is used to disable the test method.

- Junit: `@BeforeAll` and `@AfterAll` are used to run the method before and after all test methods. It is static method.

## Code Coverage

`Code Coverage` is a metric that measures the percentage of code that is executed during automated tests. It is used to measure the quality of the test.

We can use Jacoco to generate the code coverage report.

- Add dependency in pom file.

  ```xml
  <dependency>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.7</version>
  </dependency>
  ```

- Add plugin in pom file.

  ```xml
    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.7</version>
        <executions>
            <execution>
                <goals>
                    <goal>prepare-agent</goal>
                </goals>
            </execution>
            <execution>
                <id>report</id>
                <phase>test</phase>
                <goals>
                    <goal>report</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
  ```

- Run `mvn clean test` to generate the report.

- The report will be generated in `target/site/jacoco/index.html`.
