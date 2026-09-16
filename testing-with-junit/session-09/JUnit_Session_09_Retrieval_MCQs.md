# JUnit Session 09 — Retrieval MCQs

Choose the single best answer. Distractors are intentionally close.

### 1. Why might a plain JUnit test be insufficient for testing a Spring-managed object?
A. JUnit cannot execute Java methods  
B. The object may depend on configuration or objects normally provided by Spring  
C. JUnit cannot perform assertions  
D. Spring objects cannot be tested

**Answer: B** — A Spring-managed object may rely on dependencies/configuration supplied by the Spring context.

### 2. What is an ApplicationContext in Spring?
A. A JUnit assertion mechanism  
B. A Mockito mock  
C. A Spring container that manages application objects  
D. A test method

**Answer: C** — For this session, think of ApplicationContext as the Spring container that manages beans.

### 3. What is the primary purpose of `@SpringJUnitConfig`?
A. Mark a method as a JUnit test  
B. Integrate a JUnit 5 test with Spring's test context/configuration support  
C. Create a Mockito mock  
D. Assert a returned value

**Answer: B** — It connects the JUnit 5 test with Spring's test context and configuration support.

### 4. Which annotation identifies a JUnit test method?
A. `@Bean`  
B. `@Autowired`  
C. `@Test`  
D. `@Configuration`

**Answer: C** — `@Test` identifies a test method.

### 5. Which statement best distinguishes `@Test` from `@SpringJUnitConfig`?
A. Both identify test methods  
B. `@Test` identifies a test method; `@SpringJUnitConfig` configures Spring/JUnit integration  
C. `@SpringJUnitConfig` identifies a test method; `@Test` starts Spring  
D. Both create Spring beans

**Answer: B** — They operate at different levels.

### 6. What does `@Configuration` indicate in the example used in this session?
A. A class providing Spring configuration  
B. A test method  
C. A Mockito mock  
D. A JUnit assertion

**Answer: A** — The configuration class supplies definitions Spring can use for the test context.

### 7. What does `@Bean` communicate to Spring?
A. Run this method as a JUnit test  
B. Define an object for Spring to manage  
C. Create a Mockito mock  
D. Verify a method call

**Answer: B** — A `@Bean` method defines an object that Spring manages in the context.

### 8. What is the high-level purpose of `@Autowired` in this session?
A. Execute a test  
B. Register a JUnit extension  
C. Ask Spring to provide/inject a managed object  
D. Stub a mock

**Answer: C** — `@Autowired` is used to obtain/inject a Spring-managed dependency.

### 9. Which sequence best represents the Spring-integrated test model?
A. Mockito → JUnit → Spring bean → test  
B. JUnit → Spring test support → ApplicationContext → Spring-managed object → test  
C. Spring bean → Mockito → JUnit → assertion  
D. `@Autowired` → `@Test` → ApplicationContext → JUnit

**Answer: B** — JUnit runs the test while Spring's test support provides the Spring context and managed objects.

### 10. Does every JUnit test require Spring?
A. Yes  
B. No  
C. Only if assertions are used  
D. Only if Mockito is used

**Answer: B** — Plain Java objects can often be tested directly with JUnit.

### 11. Which test most clearly does NOT need Spring?
A. A test of a simple `Calculator` created with `new Calculator()`  
B. A test requiring Spring-managed configuration  
C. A test requiring an ApplicationContext  
D. A test specifically checking Spring bean wiring

**Answer: A** — A simple independent Java object does not require Spring merely because it is being tested.

### 12. What is the main reason for using Spring test support?
A. To replace JUnit  
B. To make every test a Mockito test  
C. To allow tests to operate with Spring's managed objects and context  
D. To remove assertions

**Answer: C** — Spring test support makes Spring-managed behaviour available to the test.

### 13. Which statement about a Spring bean and a Mockito mock is correct?
A. They are always the same thing  
B. A Spring bean is a Spring-managed object; a Mockito mock is a controlled test double  
C. A Mockito mock is always created by ApplicationContext  
D. A Spring bean is always fake

**Answer: B** — They represent different concepts and are managed by different mechanisms.

### 14. Which framework normally runs the test method?
A. Spring  
B. Mockito  
C. JUnit  
D. ApplicationContext

**Answer: C** — JUnit is the testing framework responsible for executing the test.

### 15. Which framework provides the ApplicationContext?
A. JUnit  
B. Spring  
C. Mockito  
D. Java Collections

**Answer: B** — ApplicationContext is part of Spring's container infrastructure.

### 16. A student says "`@Autowired` creates the object." What is the best correction?
A. Correct; `@Autowired` always constructs objects directly  
B. `@Autowired` asks Spring to provide/inject a managed object  
C. `@Autowired` runs the test  
D. `@Autowired` creates a Mockito mock

**Answer: B** — The annotation expresses dependency injection; Spring's container provides the managed object.

### 17. Which annotation is most directly associated with defining Spring configuration in this session?
A. `@Configuration`  
B. `@Test`  
C. `@Mock`  
D. `@ExtendWith`

**Answer: A** — `@Configuration` identifies a configuration class.

### 18. Which annotation is most directly associated with defining a Spring-managed object in the example?
A. `@Test`  
B. `@Bean`  
C. `@Mock`  
D. `@ExtendWith`

**Answer: B** — `@Bean` defines an object for Spring to manage.

### 19. What is the most accurate description of `@SpringJUnitConfig`?
A. A Mockito annotation  
B. A JUnit 5 test-method annotation  
C. Spring's convenient JUnit 5 test integration annotation for test context/configuration  
D. An assertion annotation

**Answer: C** — It provides Spring/JUnit 5 integration and can point to test configuration.

### 20. How does Session 9 connect to Session 8?
A. It abandons extensions completely  
B. It shows another example of framework integration with JUnit 5  
C. It replaces Mockito with assertions  
D. It teaches custom extension implementation

**Answer: B** — Session 8 introduced the extension/integration idea; Spring provides JUnit 5 test integration.

### 21. Which statement is FALSE?
A. JUnit can run tests without Spring  
B. Spring can manage application objects  
C. A Spring bean and Mockito mock are conceptually identical  
D. Spring provides testing support

**Answer: C** — A Spring bean is a managed object; a Mockito mock is a test double.

### 22. Consider:

```java
@SpringJUnitConfig(TestConfig.class)
class GreetingServiceTest {
}
```

What does `TestConfig.class` represent?

A. A Mockito mock  
B. Spring test configuration  
C. A test method  
D. A JUnit assertion

**Answer: B** — It supplies configuration for the Spring test context.

### 23. Why might this test be unnecessary?

```java
@SpringJUnitConfig(TestConfig.class)
class CalculatorTest {
    @Autowired
    Calculator calculator;
}
```

if `Calculator` has no Spring-specific dependencies or behaviour?

A. JUnit cannot test calculators  
B. Spring may add unnecessary setup when plain JUnit is sufficient  
C. Assertions cannot be used with Spring  
D. `Calculator` cannot be instantiated

**Answer: B** — The goal is not to start Spring for every test.

### 24. Which is the best description of a plain unit test in this session?
A. JUnit starts a Spring ApplicationContext for every test  
B. JUnit tests an object directly without requiring Spring's managed environment  
C. Mockito must always be used  
D. Spring creates every dependency

**Answer: B** — A plain unit test can directly construct and test an ordinary Java object.

### 25. Which is the best description of a Spring-integrated test?
A. A test that cannot use JUnit  
B. A test where Spring's test support provides a Spring-managed environment/context  
C. A test that must use Mockito  
D. A test without assertions

**Answer: B** — Spring's test support allows the test to work with the Spring context and managed objects.

### 26. A student asks: "Why can't I always just use `new GreetingService()`?"
A. Java forbids `new` in tests  
B. If the object depends on Spring-managed dependencies/configuration, constructing it manually may not reproduce the Spring-managed environment  
C. JUnit does not allow constructors  
D. Mockito requires Spring

**Answer: B** — Manual construction may bypass dependency/configuration management supplied by Spring.

### 27. Which responsibility belongs primarily to Spring?
A. Identifying `@Test` methods  
B. Running assertions  
C. Managing Spring beans and the application context  
D. Verifying Mockito interactions

**Answer: C** — Spring manages the container and its beans.

### 28. Which responsibility belongs primarily to JUnit?
A. Managing Spring beans  
B. Running test methods  
C. Creating Mockito mocks  
D. Providing ApplicationContext

**Answer: B** — JUnit is responsible for test execution.

### 29. Which concept should NOT be assumed to be necessary for every JUnit test?
A. Test method  
B. Assertion where appropriate  
C. Spring ApplicationContext  
D. Test class

**Answer: C** — Spring is only needed when Spring-managed behaviour/configuration is relevant.

### 30. What is the Pareto target of Session 9?
A. Learn the entire Spring testing framework  
B. Memorise every Spring testing annotation  
C. Understand why Spring integration can be needed, what ApplicationContext represents, and how `@SpringJUnitConfig` connects Spring with JUnit 5  
D. Learn Spring Boot web testing

**Answer: C** — That is the high-value conceptual foundation without turning the session into a full Spring course.

---

# Quick Retrieval

**JUnit** — runs tests and provides the testing framework.

**Spring bean** — an object managed by Spring.

**ApplicationContext** — a Spring container that manages application objects.

**`@SpringJUnitConfig`** — connects a JUnit 5 test with Spring's test context/configuration support.

**`@Configuration`** — defines Spring configuration.

**`@Bean`** — defines an object for Spring to manage.

**`@Autowired`** — requests injection/provision of a Spring-managed object.

## Core distinction

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

## Most important exam distinctions

```text
@Test
→ identifies a test method

@SpringJUnitConfig
→ integrates JUnit 5 with Spring's test context

@Configuration
→ configuration class

@Bean
→ defines a Spring-managed object

@Autowired
→ obtains/injects a Spring-managed object

Spring bean
→ real object managed by Spring

Mockito mock
→ controlled test double managed by Mockito
```
