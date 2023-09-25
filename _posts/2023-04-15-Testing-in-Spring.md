---
layout: post
title: "Testing in Spring"
date: 2023-04-15
description: "Testing in Spring"
tag: Spring-Testing
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

## What is the difference between `@Mock` and `@MockBean`?

`@Mock` is used to create and inject mocked instances. It is used in unit test.

`@MockBean` is used to create and inject mocked instances. It is used in integration test. (controller layer)

## Other annotations

- `@BeforeAll` and `@AfterAll` are used to run the method before and after all test methods. It is static method.

- `@BeforeEach` and `@AfterEach` are used to run the method before and after each test methods. It is non-static method.

- `@DisplayName` is used to display the name of the test method.

- `@Disabled` is used to disable the test method.
