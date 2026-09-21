# Session: Test Isolation

**Suggested time:** about 70 minutes (adjust per section)
**Works with:** JUnit 5 and JUnit 6 (the `org.junit.jupiter.api` package is the same)
**Running example:** `Calculator`, extended with a small history feature (code in Section 1)

---

## The One Idea

> **A test is a self-contained experiment. It builds its own world, runs, checks, and leaves nothing behind.**

Everything in this session is a consequence of that sentence. If a test is a self-contained experiment, then:

- it can run **alone** and give the same result,
- it can run in **any order** and give the same result,
- it can run **twice in a row** and give the same result,
- it never needs another test to have run first.

**Analogy: the clean whiteboard.** Every test walks into a room with an empty whiteboard. If it finds someone else's writing on it, or leaves writing behind for the next test, isolation is broken. Most of this session is about finding out where the writing on the whiteboard comes from.

*Limit of the analogy:* building a fresh room for every test can be expensive, so sometimes tests share things on purpose. Section 7 covers when that is safe.

## Learning Goals

By the end of this session, students can:

1. Explain what test isolation means and why a suite without it can't be trusted.
2. Explain why tests must not depend on each other, and why test order shouldn't matter.
3. Spot shared mutable state, including the hidden kinds (`static` fields, singletons, "final" collections).
4. Spot dependence on external state (files, databases, time, environment) and remove it.
5. Explain why a test can pass alone but fail when run with others, and diagnose it step by step.

## Session Plan

| # | Section | Time |
|---|---------|------|
| 1 | Hook: passes alone, fails together | 8 min |
| 2 | Why it happens: shared mutable state | 8 min |
| 3 | Tests that depend on each other | 8 min |
| 4 | State outside the JVM | 10 min |
| 5 | Diagnosing "passes alone, fails together" | 8 min |
| 6 | Practice: the leaky test class | 15 min |
| 7 | When sharing is deliberate | 5 min |
| 8 | Recap and self-check | 5 min |

**Conventions used below:** **Ask** = question for students. **Say** = the point to land. **Show** = board or screen. **Predict** = students commit to a guess *before* you run it. **Expected** = the answer you're steering toward.

---

## Setup: The Calculator's History Feature

Add this to the `Calculator` if it isn't there already. It gives us state to leak.

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;

public class Calculator {

    private final List<String> history = new ArrayList<>();

    public int add(int a, int b) {
        int result = a + b;
        history.add(a + " + " + b + " = " + result);
        return result;
    }

    public List<String> getHistory() {
        return List.copyOf(history);
    }

    /** Appends the history to a file, one entry per line. */
    public void saveHistory(Path file) throws IOException {
        Files.write(file, history, StandardOpenOption.CREATE, StandardOpenOption.APPEND);
    }
}
```

*Run every build-along below yourself before class so the output matches what students will see.*

---

## 1. Hook: Passes Alone, Fails Together (8 min)

Start with a failure, not a definition.

**Show:**

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorHistoryTest {

    private static final Calculator calculator = new Calculator();

    @Test
    void addRecordsOneHistoryEntry() {
        calculator.add(2, 3);
        assertEquals(1, calculator.getHistory().size());
    }

    @Test
    void addAnotherPairRecordsOneHistoryEntry() {
        calculator.add(4, 5);
        assertEquals(1, calculator.getHistory().size());
    }
}
```

**Ask:** "Read both tests. Is either one wrong?" Let students argue. Both look correct.

**Predict:** Ask students to write down what happens when you (a) run the first test alone, (b) run the second test alone, (c) run the whole class.

**Run all three.**

**Expected:** Each passes alone. Run together, one fails with `expected: <1> but was: <2>`.

**Ask:** "Which test failed? Is *that* test the broken one?"

**Say:** The failing test is only the messenger. It failed because the *other* test ran first and left something behind. The bug belongs to both tests.

**Now make the order visible.** Add one line to the class and run it 5 times:

```java
@TestMethodOrder(MethodOrderer.Random.class)
class CalculatorHistoryTest { ... }
```

**Expected:** The failing test changes from run to run. Random ordering just makes the problem visible: without it, JUnit's default order is repeatable but intentionally hard to predict, so you can't rely on it either. Remove the line after the demo.

> Whenever a test passes alone and fails with others, or fails in a different place each run, suspect **leftover state from another test**.

---

## 2. Why It Happens: Shared Mutable State (8 min)

**Ask:** "In the setup/teardown session we saw JUnit create a fresh `Calculator` for every test with `@BeforeEach`. So why did these two tests share one?"

**Expected:** Because the calculator here is `static`.

**Say:** JUnit creates a **new instance of the test class for every test method**, so ordinary instance fields start fresh each time. The JUnit docs describe this as deliberate, to avoid side effects from mutable instance state. But a `static` field belongs to the class, not the instance, so one copy lives for the whole run and every test sees it.

**Show:** the fix. Remove `static final` and build the calculator in `@BeforeEach`:

```java
class CalculatorHistoryTest {

    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    // same two tests as before
}
```

**Predict, then run the whole class.** Both pass, in any order.

**Say:** Same two tests, one word removed. That word was the only thing connecting them.

**Define:** *Shared mutable state* is anything that (1) more than one test can reach and (2) can change. Both conditions matter. Remove either and the problem disappears.

**Where it hides:**

```java
private static Calculator calculator = new Calculator();      // static object
private static int counter = 0;                               // static counter
private static final List<String> log = new ArrayList<>();   // "final" but mutable
System.setProperty("mode", "test");                           // global JVM setting
```

**Ask:** "Is `private static final List<String> log` safe to use across tests?"
**Expected:** No. `final` protects the *reference*, not the *contents*. Tests can still add to the list.

*Exception:* `@TestInstance(Lifecycle.PER_CLASS)` makes JUnit reuse one instance for all tests in the class. With it on, instance fields are shared too, so you must reset them yourself in `@BeforeEach`.

---

## 3. Tests That Depend on Each Other (8 min)

The mirror image of Section 1: tests that pass **together** but fail **alone**.

**Show:**

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CalculatorOrderTest {

    private static final Calculator calculator = new Calculator();

    @Test
    @Order(1)
    void step1_addPerformsCalculation() {
        calculator.add(2, 3);
    }

    @Test
    @Order(2)
    void step2_historyHasOneEntry() {
        assertEquals(1, calculator.getHistory().size());
    }
}
```

**Predict:** "What happens if I run the class? What if I run only `step2`?"

**Run.** The class passes. `step2` alone fails with `expected: <1> but was: <0>`.

**Ask:** "Why does `step2` pass in the class?"
**Expected:** Because `step1` ran first and put an entry there. `step2` depends on a test it doesn't mention.

**Ask:** "A teammate says: 'Just keep the `@Order` annotations.' What's wrong with that?"

**Expected:** `@Order` didn't fix the dependency, it *hid* it. Now one test can't be run, moved, skipped, or deleted without breaking another. And a failure in `step1` produces a confusing second failure in `step2`.

**Say:** JUnit lets you control order, but its own guide says true unit tests shouldn't need it. Order control is for cases like integration or workflow tests where the sequence *is* the thing being tested.

**Fix (build it with students):** each test creates what it needs.

```java
@Test
void historyHasOneEntryAfterOneAdd() {
    calculator.add(2, 3);                                  // set up its own world
    assertEquals(1, calculator.getHistory().size());
}
```

> **If test B needs something, test B creates it.**

---

## 4. State Outside the JVM (10 min)

Fresh objects fix state inside the JVM. Some state lives outside it.

**Show:**

```java
class CalculatorFileTest {

    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();   // fresh, so no shared state in memory
    }

    @Test
    void savedFileHasOneEntry() throws IOException {
        calculator.add(2, 3);
        Path file = Path.of("history.txt");

        calculator.saveHistory(file);

        assertEquals(1, Files.readAllLines(file).size());
    }
}
```

**Ask:** "The calculator is fresh every time. Is this test isolated?"

**Predict:** "What happens the first time I run it? The second time?"

**Run it twice.** (Delete any existing `history.txt` first.) First run passes. Second run fails: the file now has 2 lines.

**Say:** The first run left `history.txt` on disk. This test is not isolated from *its own previous run*. The whiteboard isn't in memory, it's on the file system.

**Fix:** let JUnit provide a fresh directory for each test.

```java
@Test
void savedFileHasOneEntry(@TempDir Path tempDir) throws IOException {
    calculator.add(2, 3);
    Path file = tempDir.resolve("history.txt");

    calculator.saveHistory(file);

    assertEquals(1, Files.readAllLines(file).size());
}
```

**Run it as many times as you like.** It passes every time. `@TempDir` creates a temporary directory and deletes it when the test finishes. (Declared as a non-private, non-final *instance* field instead, it gives each test its own directory too.)

### Other things that live outside your code

| Hidden dependency | What goes wrong | Way out |
|---|---|---|
| Files on disk | Passes the first run, fails the second | `@TempDir` |
| Database rows | Fails because another test left data behind | Each test creates its own data and cleans up (or rolls back) |
| Current time | Fails only after midnight, on weekends, at month end | Pass the time in; use a fixed value in tests |
| Random numbers | Fails 1 run in 20 | Pass in a seeded or fixed source |
| Environment variables, system properties | Works on your machine, fails on CI | Set what the test needs; don't assume it |
| Network and other services | Fails when offline or slow | Replace with a fake (mocking is a later topic) |

**Show one, briefly (time):**

```java
class SessionTimer {
    private final Clock clock;

    SessionTimer(Clock clock) { this.clock = clock; }

    boolean isExpired(Instant startedAt) {
        return Duration.between(startedAt, clock.instant()).toMinutes() > 30;
    }
}

// In the test, time is fixed, so the result never changes:
Clock fixed = Clock.fixed(Instant.parse("2026-01-01T10:00:00Z"), ZoneOffset.UTC);
```

**Say:** The pattern is always the same: don't reach out to the world for something the test can't control. Let the test supply it.

---

## 5. Diagnosing "Passes Alone, Fails Together" (8 min)

Give students a method, not just fixes. Write it on the board.

1. **Reproduce both ways.** Run the failing test alone, then with the whole class or suite. Note the direction: passes alone and fails together (a leak *into* this test), or passes together and fails alone (this test relies on someone else).
2. **Find the polluter.** Run the failing test paired with each other test, one at a time, until it breaks. Or run the class several times with `MethodOrderer.Random` and see which combinations fail.
3. **Look at what the two share.** Check for `static` fields, singletons, files, database rows, system properties, caches, the clock.
4. **Fix it at the source.** Make the test own its state.

**Ask:** "Which of these would you *not* do?" Then name the tempting non-fixes:

- Add `@Order` so the tests run in the "right" sequence.
- Re-run until it goes green.
- Add a `Thread.sleep`.
- Delete or `@Disabled` the annoying test.

**Say:** A test that passes sometimes and fails sometimes teaches people to ignore red builds. Once a failing test is shrugged off, the whole suite loses its value. Fowler's classic article on non-deterministic tests lists lack of isolation first among the common causes, alongside asynchronous behavior, remote services, time, and resource leaks.

---

## 6. Practice: The Leaky Test Class (15 min)

**Show:** one class with three different leaks.

```java
class CalculatorLeakyTest {

    private static Calculator calculator = new Calculator();
    private static final Path LOG = Path.of("history.txt");

    @Test
    void addReturnsSum() {
        assertEquals(5, calculator.add(2, 3));
    }

    @Test
    void historyContainsFirstCalculation() {
        assertEquals("2 + 3 = 5", calculator.getHistory().get(0));
    }

    @Test
    void savedFileHasAllEntries() throws IOException {
        calculator.saveHistory(LOG);
        assertEquals(calculator.getHistory().size(), Files.readAllLines(LOG).size());
    }
}
```

**Tasks (students work in pairs):**

1. **Predict:** which tests can fail, and why? Mark each suspicious line.
2. **Run** each test alone, then the whole class, then the whole class again. Record what you see.
3. **Name the leak** behind each failure: shared state, order dependence, or external state.
4. **Fix the class** so that every test passes alone, together, in random order, and on repeated runs.
5. **Prove it:** add `@TestMethodOrder(MethodOrderer.Random.class)`, run the class 5 times, and run each test alone.

**Expected (answer key):**

| Line | Leak | Fix |
|---|---|---|
| `private static Calculator calculator` | Shared mutable state | Instance field, created in `@BeforeEach` |
| `historyContainsFirstCalculation` uses a result of `addReturnsSum` | Order dependence | Call `calculator.add(2, 3)` inside the test |
| `LOG = Path.of("history.txt")` | External state (file survives between runs) | `@TempDir` |

A clean version:

```java
class CalculatorTest {

    private Calculator calculator;

    @TempDir
    Path tempDir;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    @Test
    void addReturnsSum() {
        assertEquals(5, calculator.add(2, 3));
    }

    @Test
    void historyContainsCalculation() {
        calculator.add(2, 3);
        assertEquals("2 + 3 = 5", calculator.getHistory().get(0));
    }

    @Test
    void savedFileHasAllEntries() throws IOException {
        calculator.add(2, 3);
        Path file = tempDir.resolve("history.txt");

        calculator.saveHistory(file);

        assertEquals(1, Files.readAllLines(file).size());
    }
}
```

**Debrief question:** "Which of the three leaks would have been hardest to notice in a real project? Why?" (Steer toward the file: it passes on the first run and only breaks later or on someone else's machine.)

---

## 7. When Sharing Is Deliberate (5 min)

Isolation is a default, not a religion. Keep this short.

- **Sharing read-only or immutable things is fine.** Constants, fixed test data, an immutable config. Isolation breaks when the shared thing can *change*.
- **Expensive setup can be shared.** Starting a database or loading a big file for every test may be too slow. Use `@BeforeAll` and treat the shared thing as read-only, or reset it in `@BeforeEach`. The trade-off is real: a fresh setup per test is easier to reason about and to debug, but slower.
- **Parallel execution raises the stakes.** JUnit can run tests concurrently when you enable it. Shared mutable state then becomes a race condition, and failures appear randomly.
- **Preview only:** in Spring tests, singleton beans and the database are shared across tests unless you deal with them, and Spring provides tools for that (rolling back test transactions, marking a context as dirty). The Spring testing sessions cover these properly.

---

## 8. Recap and Self-Check (5 min)

**Say:** Say the one idea again together.

> **A test is a self-contained experiment. It builds its own world, runs, checks, and leaves nothing behind.**

Before committing a test, run it four ways: **alone**, **in random order**, **twice in a row**, and ask **"did it build everything it needs and leave nothing behind?"**

**Ask (open discussion, no multiple choice):**

1. "A test fails only when the whole class runs. Name two things you'd look for."
   **Expected:** `static` fields or singletons; leftover files or database rows; another test changing a system property or setting.
2. "Why doesn't `@Order` fix an order-dependent test?"
   **Expected:** It hides the dependency instead of removing it. The tests still can't run alone.
3. "A test passes on the first run and fails on the second. What kind of leak is that?"
   **Expected:** External state left behind by the first run, such as a file or database row.
4. "JUnit makes a new test class instance for every test. Why isn't that enough to guarantee isolation?"
   **Expected:** It only protects instance fields. `static` fields, files, databases, the clock, and system properties live outside the instance.

---

## Teacher Notes

### Common misconceptions to listen for

- "`static final` means it's safe." (It protects the reference, not the contents.)
- "JUnit creates a new instance per test, so everything is fresh." (Only instance fields.)
- "The failing test is the buggy one." (It's often the second victim of a leak from another test.)
- "If it goes green on re-run, it's fine." (That's a flaky test, not a passing one.)

### What NOT to teach deeply yet

- Mocking and fakes (only name them as the way out for network and service dependencies)
- Database isolation strategies and test containers
- Parallel execution configuration
- How Spring's test framework caches contexts (preview only, per Section 7)
- Every `MethodOrderer` (only `Random` and `OrderAnnotation` appear here)

### Sources

- JUnit User Guide, *Test Instance Lifecycle*: https://docs.junit.org/current/writing-tests/test-instance-lifecycle
- JUnit User Guide, *Test Execution Order*: https://docs.junit.org/current/writing-tests/test-execution-order
- JUnit API, `@TempDir`: https://docs.junit.org/current/api/org.junit.jupiter.api/org/junit/jupiter/api/io/TempDir.html
- Martin Fowler, *Eradicating Non-Determinism in Tests*: https://www.martinfowler.com/articles/nonDeterminism.html
