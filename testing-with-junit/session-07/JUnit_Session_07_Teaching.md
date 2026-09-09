# JUnit — Session 07
## Mockito in Practice: Creating, Injecting, Stubbing, and Verifying Mocks

### Position in the course
Session 6 established why mocking exists: dependencies can make unit tests difficult to isolate; a mock provides a controlled substitute; stubbing controls behavior; verification checks interaction.

Session 7 moves from the idea to the actual JUnit + Mockito workflow:

```text
Unit under test
      ↓
Dependency
      ↓
Replace dependency with mock
      ↓
Stub the mock
      ↓
Execute the real unit
      ↓
Assert result / verify interaction
```

## 1. Lesson Outcomes

By the end of the session, students should be able to:
1. Create a Mockito mock and use it in a unit test.
2. Stub a dependency and distinguish stubbing from verification.
3. Explain how the mock is supplied to the class under test.

## 2. Start With Retrieval

Ask:
- Why replace a real dependency with a mock?
- What does stubbing do?
- What does verification do?
- Does verification check a returned value or an interaction?

Expected ideas:
- Mocking isolates the unit under test.
- Stubbing controls mock behavior.
- Verification checks interactions.
- Assertions check observed results.

## 3. The Problem

```java
class UserRepository {
    User findById(int id) {
        // database access
        return ...;
    }
}

class UserService {
    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }

    String getUserName(int id) {
        User user = repository.findById(id);
        return user.getName();
    }
}
```

Ask:

> Do we want this unit test to access a real database?

No.

We want to test `UserService` while controlling the repository.

## 4. Create a Mock

```java
UserRepository repository = mock(UserRepository.class);
```

This creates a Mockito-controlled substitute.

The key idea:

```text
Real repository  →  replaced by mock  →  UserService
```

## 5. Stubbing

```java
User user = new User(10, "Alice");

when(repository.findById(10))
        .thenReturn(user);
```

Read this as:

> When the mock's `findById(10)` is called, return `user`.

This is **stubbing**.

```text
when(...)       → what should happen?
thenReturn(...) → what response should be supplied?
```

## 6. Execute the Unit Under Test

```java
String result = service.getUserName(10);
```

The flow is:

```text
service.getUserName(10)
        ↓
repository.findById(10)
        ↓
Mockito mock
        ↓
returns User("Alice")
        ↓
service returns "Alice"
```

Then:

```java
assertEquals("Alice", result);
```

## 7. Complete Basic Test

```java
@Test
void shouldReturnUserName() {
    UserRepository repository = mock(UserRepository.class);

    User user = new User(10, "Alice");

    when(repository.findById(10))
            .thenReturn(user);

    UserService service = new UserService(repository);

    String result = service.getUserName(10);

    assertEquals("Alice", result);
}
```

Do not teach this as a block to memorize. Break it into:
1. What is the dependency?
2. What are we replacing?
3. What should the dependency return?
4. What class are we actually testing?
5. What result should we observe?

## 8. Verification

To check whether the service asked the repository for user 10:

```java
verify(repository).findById(10);
```

This is **verification**.

Critical distinction:

```text
STUBBING
when(repository.findById(10))
    .thenReturn(user);

→ controls mock behavior


VERIFICATION
verify(repository).findById(10);

→ checks interaction
```

## 9. Assertion vs Verification

### Assertion

```java
assertEquals("Alice", result);
```

Question:

> Did the system produce the expected result?

### Verification

```java
verify(repository).findById(10);
```

Question:

> Did the system interact with the dependency as expected?

Both can appear in one test.

## 10. Injecting the Mock

```java
UserService service = new UserService(repository);
```

This is ordinary Java dependency injection.

Do not confuse the roles:

```text
Mockito
→ creates/controls the mock

Java constructor
→ supplies the dependency to the service
```

## 11. Mockito + JUnit 5

Mockito can integrate with JUnit 5:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    ...
}
```

Mocks can then be declared:

```java
@Mock
UserRepository repository;
```

and the class under test can use:

```java
@InjectMocks
UserService service;
```

Example:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;

    @Test
    void shouldReturnUserName() {
        User user = new User(10, "Alice");

        when(repository.findById(10))
                .thenReturn(user);

        String result = service.getUserName(10);

        assertEquals("Alice", result);
        verify(repository).findById(10);
    }
}
```

Teach the annotations only after the underlying workflow is clear:

```text
mock → stub → inject → execute → assert/verify
```

## 12. What `@Mock` Means

```java
@Mock
UserRepository repository;
```

Conceptually:

> Mockito should provide a mock instance for this field when the Mockito/JUnit integration is active.

It does not create a real repository or automatically reproduce real database behavior.

## 13. What `@InjectMocks` Means

```java
@InjectMocks
UserService service;
```

Conceptually:

> Mockito should create/configure the object under test and inject available mocks where appropriate.

Keep the beginner mental model simple:

```text
@Mock
   ↓
mock dependency

@InjectMocks
   ↓
object under test receives mock
```

Do not teach the internal injection algorithm yet.

## 14. Practical Progression

### Exercise 1 — Guided

Given:

```java
class PriceRepository {
    int getPrice(int productId) {
        return 100;
    }
}

class PriceService {
    private final PriceRepository repository;

    PriceService(PriceRepository repository) {
        this.repository = repository;
    }

    int getPrice(int productId) {
        return repository.getPrice(productId);
    }
}
```

Task: create a mock repository, make it return `250`, and assert that the service returns `250`.

Ask:
- Which class is the unit?
- Which class is the dependency?
- Why mock the dependency?
- What are we controlling?
- What are we asserting?

### Exercise 2 — Modified

Change the requirement:

> For product `5`, the mocked repository should return `800`.

Students modify the test rather than copying it.

### Exercise 3 — Verification

Add:

```java
verify(repository).getPrice(5);
```

Ask:

> What does this prove?

Expected:

> The service called the repository's `getPrice` method with `5`.

Then ask:

> Does it prove that the returned price was `800`?

No. The assertion proves the returned result.

### Exercise 4 — Independent

Given:

```java
class AccountRepository {
    boolean exists(String accountNumber) {
        return false;
    }
}

class AccountService {
    private final AccountRepository repository;

    AccountService(AccountRepository repository) {
        this.repository = repository;
    }

    boolean accountExists(String accountNumber) {
        return repository.exists(accountNumber);
    }
}
```

Task:

> Write a Mockito test that makes the repository return `true`, asserts that the service returns `true`, and verifies the repository was called with the correct account number.

Do not immediately provide the solution.

## 15. Hint Ladder

1. What is the dependency?
   → `AccountRepository`

2. How do we replace it?
   → Create a mock.

3. What should the mock return?
   → `true`

4. What controls a mock's return value?
   → `when(...).thenReturn(...)`

5. How do we check the dependency was called?
   → `verify(...)`

Only demonstrate the complete solution after students attempt it.

## 16. Common Beginner Errors

### Error 1 — Mocking the class under test

```java
UserService service = mock(UserService.class);
```

This prevents the basic test from exercising the real `UserService` implementation.

### Error 2 — Confusing `when` and `verify`

```java
when(repository.findById(10)).thenReturn(user);
```

controls behavior.

```java
verify(repository).findById(10);
```

checks interaction.

### Error 3 — Forgetting injection

Creating a mock does not automatically mean the service uses it.

### Error 4 — Expecting mocks to behave like real objects

A mock is controlled test behavior, not a miniature database.

## 17. Core Mental Model

```text
             UNIT TEST
                 │
                 ↓
        ┌─────────────────┐
        │  UserService    │
        │  (real object)  │
        └────────┬────────┘
                 │
                 ↓
          mocked dependency
                 │
        ┌────────┴────────┐
        ↓                 ↓
     STUBBING         VERIFICATION
        ↓                 ↓
 control response     check interaction
        │                 │
        └────────┬────────┘
                 ↓
              ASSERT
                 ↓
          expected result
```

Core sentence:

> **Use Mockito to control dependencies; use JUnit to assert the behavior you care about.**

## 18. What NOT to Teach Yet

Defer:
- spies
- argument captors
- custom answers
- deep stubs
- strictness configuration
- advanced verification modes
- complicated injection cases
- Mockito internals

Master the basic workflow first.

## 19. Exit Questions

1. Why do we mock a dependency?
2. Which object should contain the real logic being tested?
3. What does `when(...).thenReturn(...)` do?
4. What does `verify(...)` do?
5. What's the difference between an assertion and verification?
6. Does `@Mock` create a real dependency?
7. Does creating a mock automatically inject it into the service?
8. Why is constructor injection useful?

If students can answer these and independently write the basic test, Session 7 has achieved its objective.

## Preparation Focus

### 0–25 min
Rehearse:

**mock → stub → inject → execute → assert/verify**

### 25–70 min
Build the examples yourself. Deliberately break:
- the stub,
- the injection,
- the argument passed to `verify`,
- the expected assertion.

Observe what fails and why.

### 70–100 min
Build the four practical exercises.

### 100–120 min
Prepare likely misconceptions and MCQs.

Do not spend this preparation time learning advanced Mockito features.

## Session 7 Success Test

Hand students a class with one dependency and ask:

> "Test this class in isolation using Mockito."

A successful beginner should reason:

```text
Which is the unit?
        ↓
Which is the dependency?
        ↓
Mock the dependency
        ↓
Decide what it should return
        ↓
Inject it
        ↓
Run the real unit
        ↓
Assert the result
        ↓
Verify important interaction
```
