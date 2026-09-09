# JUnit — Session 07 Retrieval
## Mockito in Practice

**Format:** MCQ-heavy exam retrieval  
**Purpose:** Distinguish closely related Mockito/JUnit concepts.

### 1. What is the primary reason for mocking a dependency in a unit test?

A. To test the dependency instead of the class under test  
B. To isolate the class under test from the real dependency  
C. To make the production dependency execute faster  
D. To replace all assertions with Mockito

**Answer: B**

**Explanation:** Mocking allows the unit under test to be tested without relying on the real dependency.

### 2. Which statement best describes stubbing?

A. Checking whether a dependency was called  
B. Creating the class under test  
C. Controlling how a mock responds to a method call  
D. Running all tests automatically

**Answer: C**

**Explanation:** Stubbing specifies mock behavior, commonly with `when(...).thenReturn(...)`.

### 3. What does this code do?

```java
when(repository.findById(10))
    .thenReturn(user);
```

A. Verifies that `findById(10)` was called  
B. Makes the real repository return `user`  
C. Configures the mock to return `user` when called with `10`  
D. Asserts that `user` is not null

**Answer: C**

**Explanation:** `when(...).thenReturn(...)` is stubbing.

### 4. What does this code primarily check?

```java
verify(repository).findById(10);
```

A. The value returned by `findById`  
B. That the repository interaction occurred with argument `10`  
C. That the repository is a real object  
D. That the service returned `10`

**Answer: B**

**Explanation:** `verify` checks interaction with the mock.

### 5. Which pairing is correct?

A. `when` → verification; `verify` → stubbing  
B. `when` → assertion; `verify` → injection  
C. `when` → stubbing; `verify` → verification  
D. `when` → injection; `verify` → assertion

**Answer: C**

**Explanation:** This is one of the central distinctions in Mockito.

### 6. In a typical unit test, which object should contain the real business logic being executed?

A. The mock dependency  
B. The class under test  
C. The test framework  
D. The verification object

**Answer: B**

**Explanation:** The real unit should be exercised while its dependencies are controlled.

### 7. Which is the most appropriate object to mock?

```text
OrderService → OrderRepository → Database
```

A. `OrderService`  
B. `OrderRepository`  
C. The test class  
D. JUnit

**Answer: B**

**Explanation:** The repository is the service's dependency and can be replaced to avoid the real database.

### 8. Why is mocking the class under test usually inappropriate for a basic unit test?

A. Mockito cannot mock classes  
B. It prevents the real class logic from being the thing under test  
C. JUnit cannot run mocked classes  
D. Mocked classes cannot have methods

**Answer: B**

**Explanation:** Mocking the service means you are no longer primarily exercising its real implementation.

### 9. What does this accomplish?

```java
UserService service = new UserService(repository);
```

A. It creates a Mockito mock  
B. It verifies the repository  
C. It supplies the dependency to the service  
D. It runs the test

**Answer: C**

**Explanation:** Constructor injection is ordinary Java dependency provision.

### 10. Which statement is correct?

A. Mockito replaces JUnit assertions  
B. JUnit creates all mocks automatically  
C. Mockito can control dependencies while JUnit can assert test outcomes  
D. `verify` replaces `assertEquals`

**Answer: C**

**Explanation:** The tools have different roles.

### 11. Consider:

```java
String result = service.getUserName(10);

assertEquals("Alice", result);
```

What does the assertion primarily check?

A. Whether the repository was called  
B. Whether the service produced the expected result  
C. Whether the mock was created  
D. Whether Mockito was initialized

**Answer: B**

**Explanation:** The assertion checks the observed result.

### 12. Consider:

```java
verify(repository).findById(10);
```

Does this by itself prove that the service returned `"Alice"`?

A. Yes  
B. No

**Answer: B**

**Explanation:** Verification checks the interaction, not the service's returned value.

### 13. Which statement about a Mockito mock is most accurate?

A. It is always a real implementation of the dependency  
B. It automatically behaves exactly like the production dependency  
C. It is a controlled substitute for the dependency  
D. It is the same thing as an assertion

**Answer: C**

**Explanation:** A mock is used to control and observe dependency interactions.

### 14. What is the purpose of `@Mock`?

A. To indicate that a field should be supplied as a Mockito mock when the integration is initialized  
B. To mark a method as a JUnit test  
C. To verify a method call  
D. To create a real database connection

**Answer: A**

**Explanation:** `@Mock` declares a Mockito mock field.

### 15. What is the conceptual purpose of `@InjectMocks`?

A. To tell JUnit which assertion to execute  
B. To create/configure the object under test and inject available mocks where appropriate  
C. To turn every dependency into a real object  
D. To verify every method call automatically

**Answer: B**

**Explanation:** `@InjectMocks` helps set up the object under test with Mockito-managed dependencies.

### 16. Which annotation commonly activates Mockito's JUnit 5 integration?

A. `@TestExtension`  
B. `@MockitoTest`  
C. `@ExtendWith(MockitoExtension.class)`  
D. `@UseMockito`

**Answer: C**

**Explanation:** Mockito's JUnit 5 extension integrates Mockito initialization with the JUnit 5 test lifecycle.

### 17. Which sequence best represents the basic Mockito workflow?

A. Verify → mock → assert → stub  
B. Mock → stub → inject → execute → assert/verify  
C. Assert → inject → mock → verify  
D. Execute → mock → verify → stub

**Answer: B**

**Explanation:** This captures the basic workflow established in Session 7.

### 18. Which line controls the mock's response?

A. `assertEquals("Alice", result);`  
B. `verify(repository).findById(10);`  
C. `when(repository.findById(10)).thenReturn(user);`  
D. `new UserService(repository);`

**Answer: C**

**Explanation:** This is stubbing.

### 19. Which line checks an interaction?

A. `when(repository.findById(10)).thenReturn(user);`  
B. `verify(repository).findById(10);`  
C. `assertEquals("Alice", result);`  
D. `new User(10, "Alice");`

**Answer: B**

**Explanation:** `verify` checks that the interaction occurred.

### 20. Which is an assertion rather than Mockito verification?

A. `verify(repository).findById(10);`  
B. `when(repository.findById(10)).thenReturn(user);`  
C. `assertEquals("Alice", result);`  
D. `mock(UserRepository.class);`

**Answer: C**

**Explanation:** JUnit's assertion checks the result.

### 21. A student writes:

```java
UserRepository repository = mock(UserRepository.class);
UserService service = new UserService(new UserRepository());
```

What is the problem?

A. The repository cannot be mocked  
B. The service receives a real repository instead of the mock  
C. JUnit cannot create services  
D. `mock()` only works for services

**Answer: B**

**Explanation:** Creating a mock does not automatically mean the class under test uses it.

### 22. A student writes:

```java
UserService service = mock(UserService.class);
```

and expects the real `getUserName()` implementation to execute. What is wrong?

A. Nothing  
B. A mock is intended to replace the class under test here  
C. The service is being mocked instead of testing its real implementation  
D. `UserService` must be an interface

**Answer: C**

**Explanation:** The basic pattern is to execute the real class under test and mock its dependency.

### 23. Which statement best distinguishes an assertion from verification?

A. Assertions check interactions; verification checks results  
B. Assertions check observed outcomes; verification checks mock interactions  
C. They are exactly the same  
D. Assertions create mocks; verification injects them

**Answer: B**

**Explanation:** Assertions and verification provide different evidence.

### 24. Suppose:

```java
when(repository.exists("A1")).thenReturn(true);

boolean result = service.accountExists("A1");
```

What should a following assertion most directly test?

A. That Mockito exists  
B. That the repository is real  
C. That `result` is `true`  
D. That the repository has a database

**Answer: C**

**Explanation:** The stub controls the dependency response; the assertion checks the service result.

### 25. Which statement is FALSE?

A. A mock can be stubbed  
B. A mock can be verified  
C. A mock is normally used to control a dependency  
D. A mock automatically performs the real dependency's business logic

**Answer: D**

**Explanation:** A mock is a controlled substitute, not the real implementation.

### 26. Why might a test use both `assertEquals` and `verify`?

A. They check exactly the same thing  
B. One checks the result while the other checks dependency interaction  
C. `verify` is required for every assertion  
D. `assertEquals` only works with Mockito

**Answer: B**

**Explanation:** They test different aspects of behavior.

### 27. Why should the session not begin with `@Mock` and `@InjectMocks`?

A. The annotations are invalid  
B. Students first need the underlying mental model of mocking, stubbing, injection, and verification  
C. Mockito does not support annotations  
D. JUnit 5 does not support extensions

**Answer: B**

**Explanation:** Syntax is easier to understand after the underlying problem and workflow are clear.

### 28. Which statement about constructor injection is correct?

A. It is a Mockito-specific feature  
B. It is ordinary Java dependency passing  
C. It verifies a repository call  
D. It automatically stubs methods

**Answer: B**

**Explanation:** Constructor injection is a Java design technique.

### 29. If this line is removed:

```java
when(repository.findById(10)).thenReturn(user);
```

what is the most likely consequence?

A. Verification automatically supplies the return value  
B. The mock may return Mockito's default value instead of the expected `user`  
C. The real database is automatically called  
D. JUnit stops existing

**Answer: B**

**Explanation:** Removing the stub removes the configured response.

### 30. Which is the best summary of Mockito's role in this session?

A. It replaces all testing concepts  
B. It provides controlled substitutes for dependencies and lets tests control/observe their interactions  
C. It is a build tool  
D. It is a reporting system

**Answer: B**

**Explanation:** That is the practical role established in this session.
