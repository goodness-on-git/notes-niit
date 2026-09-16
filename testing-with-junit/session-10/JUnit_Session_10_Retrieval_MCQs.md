# JUnit Session 10 — Retrieval MCQs

Choose the single best answer. Distractors are intentionally close.

### 1. When testing `UserService`, why might `UserRepository` be mocked?
A. Because repositories cannot be tested  
B. To isolate `UserService` from the repository's real behaviour  
C. To replace JUnit  
D. To make `UserService` a mock

**Answer: B** — A mock can control the dependency so the test focuses on the unit under test.

### 2. In a unit test of `UserService`, which is normally the object under test?
A. `UserRepository`  
B. `UserService`  
C. `MockitoExtension`  
D. `ApplicationContext`

**Answer: B** — The object under test is the component whose behaviour the test evaluates.

### 3. Which arrangement best represents an isolated unit test of `UserService`?
A. Mock `UserService`, real `UserRepository`  
B. Real `UserService`, mock `UserRepository`  
C. Mock both  
D. Real repository only

**Answer: B** — The target is real; its dependency is controlled.

### 4. What is the primary reason for controlling a dependency with Mockito?
A. To make every test faster  
B. To isolate the unit and control dependency behaviour  
C. To turn the dependency into a Spring bean  
D. To replace assertions

**Answer: B** — Mocking is primarily about controlling and isolating dependencies.

### 5. Which statement best describes a Mockito mock?
A. The production object being tested  
B. A controlled test double representing a dependency  
C. A Spring ApplicationContext  
D. A JUnit test runner

**Answer: B** — A mock stands in for a dependency and allows its behaviour/interactions to be controlled.

### 6. Which framework primarily provides mocks?
A. JUnit  
B. Spring  
C. Mockito  
D. Java

**Answer: C** — Mockito provides mock objects and stubbing/verification operations.

### 7. Which framework primarily runs the test?
A. Mockito  
B. Spring  
C. JUnit  
D. ApplicationContext

**Answer: C** — JUnit is responsible for test execution.

### 8. What is the main role of Spring in a Spring-integrated test?
A. Replace Mockito  
B. Provide the Spring-managed environment/context and objects  
C. Replace JUnit assertions  
D. Automatically mock every dependency

**Answer: B** — Spring test support makes its managed environment available to the test.

### 9. Which statement is most accurate?
A. Every test involving Spring must use Mockito  
B. Every Mockito test must use Spring  
C. Spring and Mockito solve different problems and can be used together  
D. Spring and Mockito are interchangeable

**Answer: C** — Spring manages its objects/context; Mockito provides controlled test doubles.

### 10. When might Spring not be necessary?
A. When testing a simple object whose behaviour does not depend on Spring  
B. Whenever assertions are used  
C. Whenever Mockito is used  
D. Whenever a constructor exists

**Answer: A** — Plain JUnit or JUnit + Mockito may be sufficient.

### 11. What does this assertion primarily check?
```java
assertEquals("John", result);
```
A. Whether the repository was called  
B. Whether the result matches the expected value  
C. Whether Spring created the bean  
D. Whether Mockito registered an extension

**Answer: B** — An assertion checks an expected outcome/value.

### 12. What does this verification primarily check?
```java
verify(repository).findById(10);
```
A. The returned user's name  
B. Whether the repository interaction occurred  
C. Whether the Spring context started  
D. Whether JUnit discovered the test

**Answer: B** — Mockito verification checks an interaction with the mock.

### 13. Which distinction is correct?
A. Assertion → interaction; verification → result  
B. Assertion → result/outcome; verification → interaction  
C. Assertion → mock creation; verification → Spring configuration  
D. Both always check exactly the same thing

**Answer: B** — They answer different questions.

### 14. If `UserService` is the unit under test, which dependency would most naturally be mocked?
A. `UserService` itself  
B. `UserRepository`  
C. JUnit  
D. The test class

**Answer: B** — The repository is a dependency of the service.

### 15. Why might a real database be undesirable in a unit test of `UserService`?
A. Java cannot connect to databases  
B. It introduces external state and behaviour that can reduce isolation  
C. JUnit prohibits databases  
D. Mockito cannot work with classes

**Answer: B** — A real database introduces additional variables outside the unit's core behaviour.

### 16. Which statement about a Spring bean is correct?
A. It is always a Mockito mock  
B. It is an object managed by Spring  
C. It is always a test double  
D. It is always created by JUnit

**Answer: B** — A Spring bean is an object managed by Spring.

### 17. Which statement about a mock is correct?
A. It must be managed by Spring  
B. It represents a controlled dependency in a test  
C. It is always the unit under test  
D. It replaces JUnit

**Answer: B** — A mock is a test double used to control a dependency.

### 18. Does using Spring automatically mean a test is an integration test?
A. Yes, always  
B. No; the purpose and scope of the test matter  
C. Yes, if `@Autowired` is used  
D. Yes, if an assertion is used

**Answer: B** — The presence of Spring alone does not determine the test's purpose.

### 19. Which situation most clearly calls for Spring test support?
A. Testing `Calculator.add(2, 3)` with no Spring-specific behaviour  
B. Testing behaviour that depends on Spring-managed configuration or wiring  
C. Testing an integer calculation  
D. Testing string concatenation

**Answer: B** — Spring integration is relevant when Spring-managed behaviour/configuration matters.

### 20. Which situation most clearly fits a plain Mockito unit test?
A. Testing a service's logic while controlling its repository dependency  
B. Testing Spring bean configuration itself  
C. Testing whether Spring creates an ApplicationContext  
D. Testing Spring component scanning

**Answer: A** — Mockito is appropriate for isolating the service from its dependency.

### 21. A student says, "If Spring is involved, every dependency must be real." What is correct?
A. Correct  
B. No; Spring and Mockito can be combined so selected dependencies are controlled  
C. Only repositories can be mocked  
D. Only services can be mocked

**Answer: B** — A Spring-integrated test can still use test doubles where appropriate.

### 22. What is the key question when deciding whether to mock a dependency?
A. Is the dependency a class?  
B. What are we actually trying to test, and do we need to control this dependency?  
C. Does the dependency have methods?  
D. Can JUnit compile it?

**Answer: B** — Testing scope and isolation should drive the decision.

### 23. Which sequence best represents an isolated service unit test?
A. Mock target → execute repository → assert service  
B. Real target → controlled dependency → execute → assert/verify  
C. Real target → real database → no assertion  
D. Mock target → mock test

**Answer: B** — The target remains real while selected dependencies are controlled.

### 24. What does stubbing do?
A. Checks whether a dependency was called  
B. Controls a mock's behaviour for a specified call  
C. Starts Spring  
D. Runs a JUnit test

**Answer: B** — Stubbing specifies what the mock should return/do.

### 25. What does verification do?
A. Controls the return value of a mock  
B. Checks an interaction with a mock  
C. Creates the Spring context  
D. Creates the unit under test

**Answer: B** — Verification checks interactions.

### 26. Why can testing a service against a real repository make isolation harder?
A. The repository may involve database state and other external behaviour  
B. The service cannot have methods  
C. JUnit cannot use repositories  
D. Assertions stop working

**Answer: A** — External dependencies introduce additional variables.

### 27. Which statement best describes the roles of the three tools?
A. JUnit mocks, Mockito runs tests, Spring asserts results  
B. JUnit runs tests, Mockito controls test doubles, Spring manages Spring objects/context  
C. Spring runs tests, JUnit creates beans, Mockito configures Spring  
D. All three perform the same role

**Answer: B** — Their responsibilities are complementary.

### 28. A test uses a real `UserService` and a mock `UserRepository`. What does the mock primarily allow the test to do?
A. Test the repository's database implementation  
B. Control repository behaviour and isolate the service  
C. Replace the service's logic  
D. Run the Spring application

**Answer: B** — The mock isolates the service and gives the test control over the dependency.

### 29. Which statement is FALSE?
A. A unit test can use Mockito without Spring  
B. A Spring-integrated test can use Mockito  
C. Every test should start a Spring context  
D. The object under test is normally kept real in a unit test

**Answer: C** — Starting Spring is unnecessary when Spring behaviour is not relevant.

### 30. What is the Pareto target of Session 10?
A. Memorise every combination of Spring and Mockito annotations  
B. Learn Spring Boot testing internals  
C. Understand what is being tested, when to control dependencies, and the distinct roles of JUnit, Spring, and Mockito  
D. Learn database integration testing

**Answer: C** — The high-value skill is making the correct testing-scope decision rather than memorising annotation combinations.

---

# Quick Retrieval

## Core model

```text
JUnit
→ runs the test

Spring
→ manages Spring objects/context

Mockito
→ controls selected dependencies

Real object
→ target whose behaviour we are testing

Mock
→ controlled replacement for a dependency
```

## Two critical distinctions

```text
assertEquals(...)
→ checks outcome/result

verify(...)
→ checks interaction
```

```text
Plain JUnit/Mockito
→ useful when Spring itself is not part of the behaviour being tested

Spring-integrated test
→ useful when Spring-managed configuration/wiring/behaviour matters
```

## The question to remember

> **What are we actually trying to test?**

That question should determine the test environment and which dependencies, if any, should be mocked.
