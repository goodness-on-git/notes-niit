# JUnit Session 8 — Retrieval MCQs

Choose the single best answer. Distractors are intentionally close.

### 1. What is the primary purpose of a JUnit 5 extension?
A. Replace the JUnit test engine  
B. Add behaviour to the JUnit testing process  
C. Create all test objects automatically  
D. Replace assertions  

**Answer: B** — Extensions provide additional behaviour that can participate in JUnit's test lifecycle.

### 2. What does `@ExtendWith` primarily do?
A. Creates a Mockito mock  
B. Executes a test method  
C. Registers an extension with JUnit 5  
D. Stubs a method call  

**Answer: C** — `@ExtendWith` is the JUnit mechanism for registering an extension.

### 3. Which statement about `@ExtendWith` is correct?
A. It is a Mockito-only annotation  
B. It is a JUnit 5 annotation  
C. It is an assertion annotation  
D. It creates the object under test  

**Answer: B** — `@ExtendWith` belongs to JUnit 5.

### 4. In `@ExtendWith(MockitoExtension.class)`, what is `MockitoExtension.class`?
A. A mock object  
B. A test class  
C. A Mockito-provided JUnit 5 extension  
D. A JUnit assertion  

**Answer: C** — Mockito provides the extension for JUnit 5 integration.

### 5. Why is `MockitoExtension` useful?
A. It replaces JUnit assertions  
B. It integrates Mockito with JUnit 5 test execution  
C. It converts every Java class into a mock  
D. It runs production code  

**Answer: B** — It provides the integration needed for Mockito's JUnit 5 features.

### 6. Which pairing is correct?
A. `@ExtendWith` — creates a mock  
B. `@Mock` — registers a JUnit extension  
C. `@ExtendWith` — registers an extension  
D. `verify()` — registers an extension  

**Answer: C** — `@ExtendWith` registers an extension.

### 7. Which component normally runs the JUnit test?
A. Mockito  
B. JUnit  
C. `@Mock`  
D. `verify()`  

**Answer: B** — JUnit executes the tests.

### 8. Which component provides mock creation and verification?
A. JUnit alone  
B. Mockito  
C. `@Test`  
D. `@ExtendWith`  

**Answer: B** — Mockito provides mocking, stubbing and verification.

### 9. What is the best description of the relationship between JUnit and Mockito?
A. Mockito replaces JUnit  
B. JUnit replaces Mockito  
C. JUnit runs tests while Mockito supplies mocking capabilities  
D. They perform exactly the same role  

**Answer: C** — They solve different problems and are commonly used together.

### 10. What is the best mental model for a JUnit extension?
A. A database  
B. A plugin or hook into the test process  
C. A production dependency  
D. An assertion  

**Answer: B** — Extensions add behaviour around the test process.

### 11. In `@ExtendWith(MockitoExtension.class)`, what is being registered?
A. The test class  
B. Mockito itself  
C. `MockitoExtension`  
D. `@Mock`  

**Answer: C** — The extension supplied to `@ExtendWith` is registered.

### 12. Which statement is most accurate?
A. An annotation and an extension are always the same thing  
B. An annotation configures/describes; an extension supplies behaviour  
C. An extension is always a mock  
D. A mock is always a JUnit extension  

**Answer: B** — They have different roles.

### 13. What does `@Mock` primarily communicate?
A. Register a JUnit extension  
B. Declare this field as a Mockito mock  
C. Run this method  
D. Assert this value  

**Answer: B** — `@Mock` declares a Mockito mock.

### 14. What does `@InjectMocks` primarily communicate?
A. Create a JUnit extension  
B. Identify the object into which Mockito should inject available mocks  
C. Verify an interaction  
D. Run the test repeatedly  

**Answer: B** — It identifies the object under test for Mockito's injection support.

### 15. Which statement about `MockitoExtension` is FALSE?
A. It is supplied by Mockito  
B. It can be registered with `@ExtendWith`  
C. It integrates Mockito with JUnit 5  
D. It is itself the mock being tested  

**Answer: D** — The extension is integration infrastructure, not the mock.

### 16. Which setup is more declarative?
A. Manually calling `mock()` and constructing every dependency  
B. Using `@Mock`, `@InjectMocks`, and `@ExtendWith`  
C. Writing assertions only  
D. Calling `verify()` before the test  

**Answer: B** — Annotation-based setup describes the desired configuration.

### 17. What is the underlying idea behind annotation-based Mockito setup?
A. Annotations perform all behaviour by themselves  
B. Annotations describe configuration and the framework/extension supplies supporting behaviour  
C. JUnit becomes Mockito  
D. The test no longer needs execution  

**Answer: B** — The annotations configure; the framework supplies the behaviour.

### 18. Which sequence best represents the relationship?
A. Mockito → JUnit → `@ExtendWith` → test  
B. JUnit → `@ExtendWith` → MockitoExtension → Mockito features  
C. `@Mock` → JUnit → MockitoExtension → test  
D. `verify()` → `@ExtendWith` → JUnit → Mockito  

**Answer: B** — JUnit provides the extension mechanism and Mockito supplies the extension.

### 19. A student says, "The `@ExtendWith` annotation creates my mock." What is the best correction?
A. Correct; `@ExtendWith` creates every mock  
B. `@ExtendWith` registers an extension; Mockito provides mock functionality  
C. `@ExtendWith` is only an assertion annotation  
D. `@ExtendWith` executes production code  

**Answer: B** — Registration and mock creation are separate responsibilities.

### 20. A student removes `@ExtendWith(MockitoExtension.class)` but still expects Mockito's annotation-based setup to work automatically. What concept are they missing?
A. Assertions  
B. Extension-based integration  
C. Equivalence partitioning  
D. Test naming  

**Answer: B** — The extension supplies Mockito's JUnit 5 integration.

### 21. Which is a JUnit responsibility?
A. Stubbing a mock's return value  
B. Verifying a mock interaction  
C. Running test methods  
D. Creating Mockito mocks  

**Answer: C** — JUnit executes tests.

### 22. Which is a Mockito responsibility?
A. Discovering JUnit test classes  
B. Providing `@Mock`  
C. Defining `@Test`  
D. Running the entire test suite  

**Answer: B** — `@Mock` is a Mockito annotation.

### 23. Which statement best describes stubbing?
A. Checking that a dependency was called  
B. Controlling what a mock returns/does for a particular call  
C. Registering a JUnit extension  
D. Discovering test methods  

**Answer: B** — Stubbing controls mock behaviour.

### 24. Which statement best describes verification?
A. It controls a mock's return value  
B. It registers Mockito with JUnit  
C. It checks whether an interaction occurred  
D. It creates the object under test  

**Answer: C** — Verification checks interactions.

### 25. Which sequence is most appropriate for the practical Mockito test?
A. Verify → stub → execute → assert  
B. Stub → execute → assert → verify  
C. Assert → verify → stub → execute  
D. Execute → verify → create extension → stub  

**Answer: B** — Control the dependency, execute the unit, assert the result, then verify interaction.

### 26. Which statement is most accurate about a JUnit extension?
A. It is the same thing as a test case  
B. It participates in the framework's test execution/lifecycle  
C. It must always be a Mockito mock  
D. It replaces the class under test  

**Answer: B** — Extensions add behaviour to JUnit's process.

### 27. Which topic is LOWEST priority for this beginner session?
A. What an extension is  
B. `@ExtendWith`  
C. `MockitoExtension`  
D. Implementing custom extension callbacks  

**Answer: D** — Custom extension implementation is beyond this session's Pareto target.

### 28. Why include `@ExtendWith(MockitoExtension.class)`?
A. Turn the test into a production class  
B. Integrate Mockito's JUnit 5 functionality into test execution  
C. Execute the service automatically  
D. Replace `@Mock`  

**Answer: B** — It connects Mockito's functionality with JUnit 5.

### 29. Which statement best distinguishes the extension from the mock?
A. The extension is the dependency; the mock is the test runner  
B. The extension provides integration behaviour; the mock represents a controlled dependency  
C. They are interchangeable  
D. The mock registers the extension  

**Answer: B** — They have different responsibilities.

### 30. Which statement best captures the Pareto target of Session 8?
A. Learn every JUnit extension API  
B. Memorise Mockito's internal extension implementation  
C. Understand extensions, `@ExtendWith`, and Mockito's JUnit 5 integration  
D. Replace all JUnit lifecycle concepts with Mockito  

**Answer: C** — That is the high-value understanding needed at this level.

---

# Quick Retrieval

**What is a JUnit extension?**  
A mechanism for adding behaviour to JUnit's testing process.

**What does `@ExtendWith` do?**  
Registers an extension with JUnit 5.

**What is `MockitoExtension`?**  
Mockito's JUnit 5 extension.

**Does `@ExtendWith` create a mock?**  
No.

**Who provides mocks, stubbing and verification?**  
Mockito.

**Who runs the tests?**  
JUnit.

**Core distinction:**

```text
JUnit → runs tests
Extension → adds/integrates behaviour
Mockito → provides mocking
Mock → controlled dependency
```
