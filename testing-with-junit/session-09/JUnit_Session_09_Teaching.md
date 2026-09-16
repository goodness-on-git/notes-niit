# JUnit Session 09 — JUnit 5 with Spring: Testing Spring-Managed Objects

## Position

**Previous:** Session 8 — JUnit 5 Extensions  
**Today:** Understand why Spring changes test setup, and how JUnit 5 integrates with Spring's test context  
**Next:** Continue external-framework testing, then move into higher-value test design concepts

> **Important:** Students are beginners to Spring. Do not assume they already understand service, repository, Spring Boot, or application-layer architecture.

---

# 1. Start from what students already know

Recall the Mockito pattern:

```text
JUnit
  ↓
runs test

Mockito
  ↓
controls dependency

test class
  ↓
creates/configures what it needs
```

Ask:

> "What happens if the object we want to test is normally created and managed by Spring?"

Let students reason first.

Key problem:

> **The test may need Spring to create and configure the object instead of creating it as an ordinary Java object.**

# 2. Minimum Spring idea

Teach only this:

> **Spring can create and manage Java objects for an application.**

Example:

```java
@Component
class GreetingService {
    String greet(String name) {
        return "Hello " + name;
    }
}
```

Mental model:

```text
Spring
  ↓
creates/manages GreetingService

Test
  ↓
may need Spring to provide GreetingService
```

Do not turn this into a Spring container lecture.

# 3. The problem with ordinary JUnit

A simple object can be tested directly:

```java
class GreetingServiceTest {

    @Test
    void shouldGreet() {
        GreetingService service = new GreetingService();

        assertEquals("Hello John", service.greet("John"));
    }
}
```

But suppose the object depends on another Spring-managed object:

```java
@Component
class GreetingService {

    private final GreetingFormatter formatter;

    GreetingService(GreetingFormatter formatter) {
        this.formatter = formatter;
    }

    String greet(String name) {
        return formatter.format(name);
    }
}
```

Ask:

> "What now has to provide and configure the dependency?"

The point is not that plain JUnit is incapable of testing it. The point is that **a Spring-integrated test can reproduce the Spring-managed environment when that environment matters.**

# 4. Spring test support

Introduce:

> **Spring provides testing support that allows a JUnit 5 test to run with a Spring application context.**

Mental model:

```text
JUnit
  ↓
runs test

Spring test support
  ↓
provides Spring test environment

ApplicationContext
  ↓
contains Spring-managed objects

Test
  ↓
uses those objects
```

# 5. ApplicationContext

High-value definition:

> **ApplicationContext is a Spring container that manages application objects (beans).**

For today's purposes:

```text
ApplicationContext
        ↓
Spring-managed objects
        ↓
test can use them
```

# 6. `@SpringJUnitConfig`

Introduce:

```java
@SpringJUnitConfig
class GreetingServiceTest {
}
```

High-level meaning:

> `@SpringJUnitConfig` integrates JUnit 5 testing with Spring's test context and configuration support.

Connection to Session 8:

```text
Session 8
@ExtendWith(...)
        ↓
JUnit extension mechanism

Session 9
@SpringJUnitConfig
        ↓
Spring's JUnit 5 test integration
```

Core idea:

> **Spring's testing support integrates with JUnit 5 rather than replacing JUnit.**

# 7. Configuration

Use a deliberately small example:

```java
@Configuration
class TestConfig {

    @Bean
    GreetingService greetingService() {
        return new GreetingService();
    }
}
```

Then:

```java
@SpringJUnitConfig(TestConfig.class)
class GreetingServiceTest {

    @Autowired
    GreetingService service;

    @Test
    void shouldGreet() {
        assertEquals("Hello John", service.greet("John"));
    }
}
```

Explain only:

```text
@Configuration
    ↓
defines Spring test configuration

@Bean
    ↓
defines an object Spring manages

@SpringJUnitConfig
    ↓
connects JUnit 5 test with Spring

@Autowired
    ↓
asks Spring to provide the managed object
```

# 8. JUnit vs Spring

| Responsibility | JUnit | Spring |
|---|---:|---:|
| Run the test | ✓ | |
| `@Test` | ✓ | |
| Assertions | ✓ | |
| Create/manage Spring beans | | ✓ |
| Provide ApplicationContext | | ✓ |
| Spring test integration | | ✓ |

They work together; neither simply replaces the other.

# 9. Connect it to Mockito

Students have now seen two integrations:

### Mockito

```java
@ExtendWith(MockitoExtension.class)
```

### Spring

```java
@SpringJUnitConfig(TestConfig.class)
```

Deeper idea:

> **JUnit provides the testing framework. Other frameworks integrate with JUnit so their own behaviour can participate in testing.**

# 10. Spring-managed object vs mock

### Spring-managed object

```text
real object
managed by Spring
```

### Mockito mock

```text
controlled test double
managed by Mockito
```

They are not the same thing.

# 11. Guided practical

Start with:

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }
}
```

Ask:

> "Do we actually need Spring for this test?"

Expected answer: **No.**

Plain JUnit is enough:

```java
class CalculatorTest {

    @Test
    void shouldAdd() {
        Calculator calculator = new Calculator();

        assertEquals(5, calculator.add(2, 3));
    }
}
```

Then ask:

> "So when does Spring testing become useful?"

Expected direction:

> When code depends on objects/configuration that Spring normally creates and manages.

# 12. Second practical — Spring integration

Give students:

```java
@Configuration
class TestConfig {

    @Bean
    GreetingService greetingService() {
        return new GreetingService();
    }
}
```

Ask them to create:

```java
@SpringJUnitConfig(TestConfig.class)
class GreetingServiceTest {
}
```

Requirements:

1. Connect the test to the Spring configuration.
2. Obtain `GreetingService` from Spring.
3. Call `greet("John")`.
4. Assert the expected result.

### Hint ladder

1. Which annotation connects a JUnit 5 test with Spring's test context?
2. What class contains the Spring test configuration?
3. How do we ask Spring for a managed object?
4. What assertion should check the returned greeting?

Do not show the full solution before students attempt it.

# 13. "Why not just `new`?"

Ask:

> "If I can write `new GreetingService()`, why use Spring?"

Do not teach "always use Spring."

For a simple object:

```java
new Calculator()
```

may be perfectly appropriate.

For an object whose dependencies/configuration are supplied by Spring, using the Spring-managed environment may be relevant to the test.

Important principle:

> **Do not start Spring merely because the production application uses Spring. Start/integrate Spring when Spring behaviour or configuration is relevant to what the test is testing.**

# 14. Plain unit test vs Spring-integrated test

### Plain unit test

```text
JUnit
  ↓
test object directly
```

### Spring-integrated test

```text
JUnit
  ↓
Spring test support
  ↓
Spring context
  ↓
test Spring-managed object
```

The second generally involves more framework setup.

# 15. Common beginner mistakes

### Mistake 1

"`@SpringJUnitConfig` is the same as `@Test`."

Correction:

- `@Test` identifies a test method.
- `@SpringJUnitConfig` configures Spring/JUnit integration for the test.

### Mistake 2

"`@Autowired` creates the object."

Correction:

> Spring provides/injects the managed object.

### Mistake 3

"Every JUnit test needs Spring."

Correction:

> No. Use plain JUnit when Spring behaviour is not part of the test.

### Mistake 4

"A Spring bean is a Mockito mock."

Correction:

> A bean is a Spring-managed object. A mock is a test double created/controlled by Mockito.

# 16. What NOT to teach today

Do not spend the session on:

- `@SpringBootTest`
- Spring Boot auto-configuration
- web testing
- MockMvc
- database testing
- `@MockBean`
- transactions
- profiles
- test slices
- repository testing
- service/repository architecture
- Spring internals

Today's target is much smaller.

# 17. Retrieval during teaching

Ask without notes:

1. Why might plain JUnit be insufficient for a Spring-managed object?
2. What is an ApplicationContext?
3. What does Spring's test support provide?
4. What is the purpose of `@SpringJUnitConfig`?
5. What does `@Configuration` represent?
6. What does `@Bean` tell Spring?
7. What does `@Autowired` do at a high level?
8. Does every JUnit test require Spring?
9. What is the difference between a Spring bean and a Mockito mock?
10. How does Session 9 connect to the extension concept from Session 8?

# 18. Exit test

Students should be able to explain:

```text
Plain JUnit
    ↓
test object directly

Spring-integrated JUnit
    ↓
JUnit
    ↓
Spring test support
    ↓
ApplicationContext
    ↓
Spring-managed object
    ↓
test
```

And distinguish:

```text
@Test
    → identifies a test method

@SpringJUnitConfig
    → integrates the test with Spring's test context

@Configuration
    → provides configuration

@Bean
    → defines a Spring-managed object

@Autowired
    → obtains/injects a Spring-managed object
```

# 19. Two-hour structure

| Time | Activity |
|---:|---|
| 0–15 min | Retrieval from Sessions 6–8 |
| 15–30 min | Problem: why plain JUnit may not be enough |
| 30–45 min | Minimum Spring concepts |
| 45–60 min | ApplicationContext + `@SpringJUnitConfig` |
| 60–80 min | Configuration + managed object |
| 80–105 min | Guided student coding |
| 105–115 min | Plain JUnit vs Spring test comparison |
| 115–120 min | Exit retrieval |

## Teacher rule

Do **not** let this become a Spring Framework lesson.

The Pareto target is:

> **Students understand that Spring can manage objects, that Spring provides JUnit 5 integration for tests, and that `@SpringJUnitConfig` connects a JUnit 5 test to Spring's test context.**

The bigger Spring concepts can be taught properly in the separate Spring course.
