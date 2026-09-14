# JUnit Session 8 — JUnit 5 Extensions

## Position
Previous: Session 7 — Mockito in practice  
Today: Understand JUnit 5 extensions, using Mockito as the concrete example  
Next: Spring testing / JUnit with Spring

## 1. Start with the problem

Ask:

> "In Session 7, why did we use `@ExtendWith(MockitoExtension.class)`?"

Let students reason first.

They should discover:
- `@Mock` and `@InjectMocks` need Mockito support.
- JUnit runs the test, but does not automatically provide Mockito's annotation-based setup.
- Something needs to connect Mockito to JUnit 5.

Then introduce:

> **A JUnit 5 extension adds behaviour to the JUnit testing process.**

## 2. Core mental model

```text
JUnit
  ↓
runs the test

Extension
  ↓
adds/integrates extra behaviour

MockitoExtension
  ↓
integrates Mockito with JUnit 5
```

Think of an extension as a **plugin/hook into the test process**.

## 3. `@ExtendWith`

Example:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
}
```

Meaning:

> Register `MockitoExtension` for this test class.

Important distinction:

| Item | Role |
|---|---|
| `@ExtendWith` | JUnit 5 mechanism for registering an extension |
| `MockitoExtension` | Mockito-provided JUnit 5 extension |
| `@Mock` | Declares a Mockito mock |
| `@InjectMocks` | Identifies the object into which Mockito injects mocks |

Do **not** teach `@ExtendWith` as a Mockito annotation.

## 4. Why Mockito uses an extension

Compare:

### Manual setup

```java
@BeforeEach
void setUp() {
    repository = mock(UserRepository.class);
    service = new UserService(repository);
}
```

### Annotation-based setup

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;
}
```

The second is more declarative.

Key idea:

> **Annotations describe what we want; the framework/extension supplies the supporting behaviour.**

Do not let students think annotations are "magic."

## 5. JUnit vs Mockito

| Responsibility | JUnit | Mockito |
|---|---:|---:|
| Discover/run tests | ✓ | |
| Assertions | ✓ | |
| Test lifecycle | ✓ | |
| Create mocks | | ✓ |
| Stub mock behaviour | | ✓ |
| Verify interactions | | ✓ |
| Mockito/JUnit integration | | ✓ |

The extension is the **integration mechanism**.

## 6. Guided practical

Give students:

```java
class PaymentService {

    private PaymentGateway gateway;

    PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    boolean pay(double amount) {
        return gateway.charge(amount);
    }
}
```

Ask them to create a JUnit 5 + Mockito test that:

1. Registers Mockito's extension.
2. Declares a `PaymentGateway` mock.
3. Injects it into `PaymentService`.
4. Stubs `charge(100)` to return `true`.
5. Calls `pay(100)`.
6. Asserts `true`.
7. Verifies `charge(100)` was called.

### Hint ladder

1. Which annotation registers a JUnit extension?
2. Which extension does Mockito provide?
3. Which annotation declares a mock?
4. Which annotation identifies the object under test?
5. What controls the mock's return value?
6. What checks that the dependency was called?

Do not show the complete solution first.

## 7. Modify the test

After the guided version, change:

```java
pay(100)
```

to:

```java
pay(250)
```

Ask:
- What changes in the stub?
- What changes in the assertion?
- What changes in verification?

Then add:

```text
charge(250) → false
```

This reinforces stubbing, execution, assertion, and verification.

## 8. Critical distinctions

### Extension ≠ mock

`MockitoExtension` integrates Mockito with JUnit.

A mock represents a controlled dependency.

### Extension ≠ annotation

`@ExtendWith` is an annotation.

`MockitoExtension` is the extension being registered.

### Stubbing ≠ verification

```java
when(repository.findById(10)).thenReturn(user);
```

controls behaviour.

```java
verify(repository).findById(10);
```

checks interaction.

## 9. What NOT to teach today

Skip:
- custom extension classes
- `BeforeEachCallback`
- `AfterEachCallback`
- parameter resolvers
- extension stores
- extension ordering
- complex composed annotations
- extension internals

Pareto target:

> Students understand what an extension is, why it exists, what `@ExtendWith` does, and how Mockito uses the mechanism.

## 10. Exit test

Students should be able to explain:

```text
JUnit
  ↓
runs tests

@ExtendWith
  ↓
registers an extension

MockitoExtension
  ↓
integrates Mockito with JUnit 5

Mockito
  ↓
provides mocking, stubbing and verification
```

And recognise:

```java
@ExtendWith(MockitoExtension.class)
class SomeServiceTest {

    @Mock
    SomeDependency dependency;

    @InjectMocks
    SomeService service;

    @Test
    void shouldDoSomething() {
        // stub
        // execute
        // assert
        // verify
    }
}
```

## 11. Two-hour structure

| Time | Activity |
|---:|---|
| 0–15 min | Retrieval from Sessions 5–7 |
| 15–30 min | Why Mockito needs JUnit integration |
| 30–50 min | Extension + `@ExtendWith` |
| 50–70 min | Mockito extension example |
| 70–100 min | Guided coding |
| 100–115 min | Modify + second scenario |
| 115–120 min | Exit retrieval |

### Teacher rule

Do **not** turn extensions into an API lecture.

The high-value understanding is:

> **JUnit can be extended. `@ExtendWith` registers an extension. Mockito provides `MockitoExtension` so Mockito can integrate with JUnit 5.**
