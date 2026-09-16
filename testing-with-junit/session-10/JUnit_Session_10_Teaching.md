# JUnit Session 10 — Spring + Mockito: Choosing What to Control

## Position in the course

**Previous:** Session 9 — JUnit 5 with Spring  
**Today:** Combine Spring testing and Mockito, and understand when to use a real Spring-managed object versus a mock dependency  
**Next:** JUnit with external systems/frameworks — REST testing

## Pareto target

Students should leave able to answer:

> **What am I testing for real, and which dependency should I replace with a mock?**

Core model:

```text
JUnit → runs the test
Spring → manages Spring objects/context
Mockito → provides controlled test doubles
```

Do not turn this into a full Spring lesson.

---

# 1. Start with retrieval

Ask students:

1. What does an ApplicationContext do?
2. What is a Spring bean?
3. Why might plain JUnit be insufficient for a Spring-managed object?
4. What does `@SpringJUnitConfig` do?
5. What does `@Mock` do?
6. What does `@InjectMocks` do?
7. What is stubbing?
8. What is verification?
9. What does `@ExtendWith` do?

Then ask:

> **Can Spring and Mockito be used in the same test?**

Answer: **Yes.**

---

# 2. The problem: real dependency vs controlled dependency

Use:

```java
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

> If we test `UserService`, do we really want to test `UserRepository` at the same time?

If the repository uses a database, the test now depends on database availability, data, configuration, and behaviour.

That makes it harder to isolate `UserService`.

---

# 3. The key decision

Teach:

> **What is the unit under test?**

If the target is:

```text
UserService
```

then:

```text
UserService → real
UserRepository → controlled mock
```

Mental model:

```text
              test
               ↓
        ┌──────────────┐
        │ UserService  │  ← REAL
        └──────┬───────┘
               │
               ↓
        ┌──────────────┐
        │ UserRepository│ ← MOCK
        └──────────────┘
```

This is the central idea of the session.

---

# 4. Where Spring enters

Suppose `UserService` is a Spring bean:

```java
@Service
class UserService {

    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Do not assume students already know what `@Service` means.

Explain only:

> `@Service` is a Spring stereotype used to identify a class as a component that Spring can manage.

The important question remains:

> **Should the repository be real or controlled?**

---

# 5. Two testing approaches

## Approach A — Plain Mockito unit test

```text
JUnit
  ↓
real UserService
  ↓
mock UserRepository
```

No Spring context is required.

Conceptually:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;
}
```

Useful when the goal is to test `UserService` behaviour in isolation.

## Approach B — Spring-integrated test

```text
JUnit
  ↓
Spring test support
  ↓
Spring context
  ↓
real Spring-managed UserService
  ↓
controlled dependency
```

The key question is:

> **What does Spring need to manage for this test?**

---

# 6. Do not confuse the two goals

### Goal 1: Test business logic in isolation

Use:

```text
JUnit + Mockito
```

You may not need Spring.

### Goal 2: Test Spring configuration/integration

Use:

```text
JUnit + Spring test support
```

You may need the Spring context.

### Core distinction

> **Use the smallest environment that actually tests the behaviour you care about.**

This does not mean every test should avoid Spring.

---

# 7. Spring + Mockito conceptual picture

```text
             JUnit
               │
        runs the test
               │
       ┌───────┴────────┐
       ↓                ↓
    Spring            Mockito
    manages           controls
    real object       dependency
       │                │
       └───────┬────────┘
               ↓
             Test
```

These tools have different responsibilities.

---

# 8. Guided practical

Use:

```java
class User {
    private final String name;

    User(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }
}
```

```java
interface UserRepository {
    User findById(int id);
}
```

```java
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

Ask students to create a test.

Requirements:

1. `UserService` must be the real object under test.
2. `UserRepository` must be mocked.
3. Stub `findById(10)` to return `User("John")`.
4. Call `getUserName(10)`.
5. Assert `"John"`.
6. Verify `findById(10)` was called.

Do not display the complete test first.

---

# 9. Hint ladder

**Hint 1:** Which framework gives you a mock?

**Hint 2:** Which annotation declares the mock?

**Hint 3:** Which annotation can identify the object receiving the mock?

**Hint 4:** What do you need to do before executing `getUserName(10)`?

**Hint 5:** Which Mockito operation controls the repository's return value?

**Hint 6:** Which operation checks the repository interaction?

---

# 10. Modify the test

Change:

```java
findById(10)
```

to:

```java
findById(25)
```

Ask students to change only what is necessary.

Then create:

```text
id = 25
repository returns User("Mary")
expected result = "Mary"
```

Then ask:

> **What happens if the repository returns `null`?**

Do not immediately solve it. Ask:

> **What behaviour should the application have when no user exists?**

This transitions from syntax to test-case design.

---

# 11. Critical distinction: assertion vs verification

```java
assertEquals("John", result);
```

asks:

> **Did the system produce the expected result?**

While:

```java
verify(repository).findById(10);
```

asks:

> **Did the system interact with its dependency as expected?**

Therefore:

```text
Assertion
→ outcome/state

Verification
→ interaction
```

---

# 12. Critical distinction: unit test vs integration test

### Unit-testing focus

> Is this unit's behaviour correct when its dependencies are controlled?

### Integration-testing focus

> Do multiple components/framework-managed parts work together correctly?

A Spring context can be involved in an integration-oriented test, but **using Spring alone does not define the test's purpose.**

---

# 13. Common beginner mistakes

### Mistake 1

> If Spring is involved, everything must be real.

No. Spring and Mockito can be combined.

### Mistake 2

> The mock is the class being tested.

Usually no. The mock normally represents a dependency.

### Mistake 3

> Verification checks the returned value.

No. Verification checks interactions.

### Mistake 4

> Assertions tell us whether a dependency was called.

No. Assertions normally check an outcome/value/state.

### Mistake 5

> Every Spring test needs Mockito.

No.

### Mistake 6

> Every test should start the Spring context.

No. Use plain JUnit/Mockito when Spring itself is not relevant.

---

# 14. What NOT to teach today

Do not spend the session on:

- `@SpringBootTest`
- `@MockBean`
- `@WebMvcTest`
- MockMvc
- transactions
- database integration
- Spring profiles
- test slices
- advanced bean configuration
- service/repository architecture
- Spring Boot internals

---

# 15. Retrieval during class

Ask without notes:

1. What is the unit under test?
2. Why might a repository be mocked when testing a service?
3. What does Spring manage?
4. What does Mockito control?
5. When might Spring not be needed?
6. What does a mock represent?
7. What does an assertion check?
8. What does verification check?
9. Does using Spring automatically make a test an integration test?
10. Can a test use Spring and Mockito together?

---

# 16. Exit test

Students should be able to explain:

```text
JUnit
→ runs the test

Spring
→ manages Spring objects/context

Mockito
→ controls selected dependencies

Real object under test
→ behaviour we actually want to test

Mock
→ controlled replacement for a dependency
```

And explain why:

```text
UserService = real
UserRepository = mock
```

can be useful.

Also distinguish:

```text
assertEquals(...)
→ checks outcome

verify(...)
→ checks interaction
```

---

# 17. Two-hour structure

| Time | Activity |
|---:|---|
| 0–15 min | Retrieval from Sessions 7–9 |
| 15–30 min | Real dependency vs controlled dependency |
| 30–45 min | Unit under test |
| 45–60 min | Spring + Mockito relationship |
| 60–90 min | Guided coding |
| 90–105 min | Modify + second scenario |
| 105–115 min | Assertion vs verification / unit vs integration |
| 115–120 min | Exit retrieval |

## Teacher rule

Keep asking:

> **What are we actually trying to test?**

That question should drive the choice of real object, Spring context, and mock.

The Pareto target is not memorising combinations of annotations.

> **Test the target for real; control dependencies when you need isolation; use Spring when Spring-managed behaviour/configuration is part of what you need to test.**
