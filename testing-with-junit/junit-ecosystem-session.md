# Session: Understanding the JUnit Ecosystem

**Suggested time:** about 70 minutes (adjust per section)

---

## Learning Goals

By the end of this session, students can:

1. Tell JUnit 4 code from JUnit 5 code by reading imports, not annotation names.
2. Explain why JUnit 5 is an architectural redesign, not "JUnit 4 plus new annotations".
3. Name the three parts of JUnit 5 (Platform, Jupiter, Vintage) and say what each one does.
4. Explain why JUnit 4 tests can still run in a JUnit 5 project, and what must be on the classpath for that to work.
5. Explain why the Platform matters even though they only write Jupiter tests.

## Session Plan

| # | Section | Time |
|---|---------|------|
| 1 | Hook: same `@Test`, different JUnit | 5 min |
| 2 | Why is there more than one JUnit? | 5 min |
| 3 | JUnit 4 in a nutshell | 5 min |
| 4 | JUnit 5 is three pieces | 3 min |
| 5 | Jupiter: what you write | 10 min |
| 6 | The Platform and test engines | 10 min |
| 7 | Vintage: old tests, new platform | 7 min |
| 8 | Recap: one diagram, five definitions | 5 min |
| 9 | Practice: recognize, build-along, transfer | 20 min |

**Conventions used below:** **Ask** = question for students. **Say** = the point to land. **Show** = board or screen. **Expected** = the answer you're steering toward.

---

## 1. Hook: Same `@Test`, Different JUnit (5 min)

Do **not** start with the history of JUnit 3, 4 and 5. Start with a problem.

**Show:** put both classes on the board.

**Test A**

```java
import org.junit.Test;

public class CalculatorTest {

    @Test
    public void shouldAddNumbers() {
        // test
    }
}
```

**Test B**

```java
import org.junit.jupiter.api.Test;

public class CalculatorTest {

    @Test
    void shouldAddNumbers() {
        // test
    }
}
```

**Ask:** "Both of these have `@Test`. Are they the same JUnit?" Let students answer.

**Ask:** "If I remove the imports, can you tell which is JUnit 4 and which is JUnit 5?"

**Expected:** Not reliably. (`public` is a hint, since JUnit 4 requires it and Jupiter doesn't, but it isn't proof.) The annotation name alone is not enough.

**Say:** The two `@Test` annotations live in different packages:

```java
org.junit.Test                 // JUnit 4
org.junit.jupiter.api.Test     // JUnit 5 (Jupiter)
```

> Never identify a JUnit version by annotation name. Look at where the annotation is imported from.

---

## 2. Why Is There More Than One JUnit? (5 min)

**Show:**

```text
JUnit 3  →  JUnit 4  →  JUnit 5
```

**Say:** This looks like a normal version chain, but JUnit 5 is a redesign, not JUnit 4 with a few extra annotations. Two problems with JUnit 4 drove it:

- **Tools were tied to JUnit itself.** IDEs, Maven and Gradle were coupled to JUnit 4's own classes, so supporting another test framework, or changing JUnit, was hard.
- **Extension was limited.** A test class could use only one `@RunWith` runner, so extensions were hard to combine.

JUnit 5 answers both by splitting responsibilities: one part for **writing** tests, another for **discovering and launching** them. The next sections show that split.

*Don't go deeper into history than this.*

---

## 3. JUnit 4 in a Nutshell (5 min)

**Say:** JUnit 4 is one framework with its own programming model and its own way of running tests.

**Show:** a basic JUnit 4 test.

```java
import org.junit.Test;
import static org.junit.Assert.*;

public class CalculatorTest {

    @Test
    public void shouldAddNumbers() {
        assertEquals(5, 2 + 3);
    }
}
```

Students should recognize these:

```java
// Annotations
@Test  @Before  @After  @BeforeClass  @AfterClass  @Ignore

// Assertions
assertEquals(...)  assertTrue(...)  assertFalse(...)  assertNull(...)
```

**Ask:** "What package does `@Test` come from?"
**Expected:** `org.junit.Test`

**Ask:** "What happens if I change only the `@Test` import to `org.junit.jupiter.api.Test` but leave `@Before` from `org.junit`?"

**Expected:** It may still compile and run, but `@Before` is silently ignored, because Jupiter doesn't recognize JUnit 4 annotations. The two are different programming models and are not interchangeable.

---

## 4. JUnit 5 Is Three Pieces (3 min)

**Say:** "JUnit 5 is not one library. It's an ecosystem of three components."

```text
JUnit 5  =  JUnit Platform  +  JUnit Jupiter  +  JUnit Vintage
```

Don't rush this line. It is the central idea of the session. Over the next three sections you'll build **one** diagram on the board, adding one piece at a time. Start with the piece students already use.

---

## 5. Jupiter: What You Write (10 min)

**Ask:** "If I'm writing a new JUnit 5 test, which part am I mainly using?"
**Expected:** JUnit Jupiter.

**Say:** Jupiter is two things: the **programming model** you write tests with (annotations, assertions, extension model), and the **engine** that runs Jupiter tests. Students mostly touch the first.

**Show:**

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @Test
    void shouldAddNumbers() {
        int result = 2 + 3;
        assertEquals(5, result);
    }
}
```

**Compare the imports:**

| | JUnit 4 | Jupiter (JUnit 5) |
|---|---|---|
| Test annotation | `org.junit.Test` | `org.junit.jupiter.api.Test` |
| Assertions | `org.junit.Assert.*` | `org.junit.jupiter.api.Assertions.*` |

### Build the migration table with students

Give them the JUnit 4 column and the `org.junit.jupiter.api` package (IDE autocomplete works well). Have them fill in the right column, then reveal:

| JUnit 4 | Jupiter |
|---|---|
| `@Test` | `@Test` (different package) |
| `@Before` | `@BeforeEach` |
| `@After` | `@AfterEach` |
| `@BeforeClass` | `@BeforeAll` |
| `@AfterClass` | `@AfterAll` |
| `@Ignore` | `@Disabled` |
| `@Category` | `@Tag` |
| `@RunWith` | `@ExtendWith` |

Also changed: `Assert` became `Assertions`, and the optional failure message moves from the first argument to the last.

**Ask:** "This isn't just renaming. What changed in the design?"

**Expected:** Names now say *when* they run (`BeforeEach` vs `Before`). More importantly, `@ExtendWith` replaces `@RunWith` and rules with one extension model, and a class can use several extensions at once.

> Don't teach the table as "memorize seven replacements." The point is that JUnit 5 introduced a new programming and extension model.

---

## 6. The Platform and Test Engines (10 min)

This is the concept students find most confusing. Go slowly.

**Ask:** "Jupiter gives you the way to write tests. Who actually discovers and launches them?"
**Expected:** The JUnit Platform.

**Say:** Three terms, kept distinct:

- **Platform:** the infrastructure that tools (Maven, Gradle, your IDE) talk to in order to discover and launch tests. It defines the `TestEngine` API.
- **Engine:** a plugin that understands **one** testing model. It finds tests written in that model, runs them, and reports results back.
- **Test:** the code you wrote that gets discovered and run. Note that `CalculatorTest` is a test; `Calculator` is the code under test.

**Analogy: a media player and codecs.** The Platform is the player app. Each engine is a codec for one file format. The tests are the files. You press play in the player; it uses the right codec for each file.

*Limit of the analogy:* a codec only decodes data. An engine also finds tests, executes them, and reports pass or fail.

**Show (step 1 of the diagram):**

```text
   Maven / Gradle / IDE
            │
      JUnit Platform
            │
      Jupiter Engine
            │
      Jupiter tests
```

**Ask:** "Is Jupiter the only thing that can plug into the Platform?"

**Expected:** No. Any testing framework can provide an engine (Spock and Cucumber do). Don't list frameworks at length. The only idea needed is that **the Platform is built around engines, not around Jupiter.**

---

## 7. Vintage: Old Tests, New Platform (7 min)

**Ask:** "A company has 5,000 existing JUnit 4 tests. Must they rewrite all 5,000 before adopting the JUnit Platform?"
**Expected:** No.

**Say:** JUnit provides the **Vintage engine**. It understands JUnit 3/4 tests and lets them run on the Platform, so old and new tests can coexist in one project. It's a migration path.

**Show (step 2 of the diagram):** add the second engine.

```text
       Maven / Gradle / IDE
                │
          JUnit Platform
                │
       ┌────────┴────────┐
       │                 │
 Jupiter Engine    Vintage Engine
       │                 │
 Jupiter tests     JUnit 3/4 tests
```

**Say:** One catch. Vintage is a **separate dependency** (`junit-vintage-engine`). Without it, JUnit 4 tests are simply not run. In Spring Boot projects this happens in practice: `spring-boot-starter-test` has not included Vintage since Spring Boot 2.4. Students will see this in the build-along.

---

## 8. Recap: One Diagram, Five Definitions (5 min)

**Show:** the full diagram once more, then have students say each definition aloud.

```text
       Maven / Gradle / IDE
                │
          JUnit Platform
                │
       ┌────────┴────────┐
       │                 │
 Jupiter Engine    Vintage Engine
       │                 │
 Jupiter tests     JUnit 3/4 tests
```

```text
JUnit 4        = older testing framework, with its own model and runner
JUnit 5        = newer ecosystem: Platform + Jupiter + Vintage
JUnit Jupiter  = JUnit 5 programming and extension model + the Jupiter engine
JUnit Platform = infrastructure that discovers and runs tests through engines
JUnit Vintage  = engine that runs JUnit 3/4 tests on the Platform
```

Students should **not** leave with "JUnit 5 is the newer version of JUnit 4." That is too shallow.

If they remember only one sentence:

> **Jupiter is the modern JUnit 5 programming model; the Platform is the infrastructure that runs test engines; Vintage lets old JUnit 3/4 tests run on that Platform.**

---

## 9. Practice (20 min)

### A. Recognize the JUnit (5 min)

For each example ask: (1) Which JUnit style? (2) How did you know? (3) Which package does `@Test` come from? (4) Which package does the setup annotation come from?

**Example A**

```java
import org.junit.Before;
import org.junit.Test;

public class CalculatorTest {

    @Before
    public void setup() {
    }

    @Test
    public void shouldAddNumbers() {
    }
}
```

**Expected:** JUnit 4 (`org.junit.Test`, `org.junit.Before`).

**Example B**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class CalculatorTest {

    @BeforeEach
    void setup() {
    }

    @Test
    void shouldAddNumbers() {
    }
}
```

**Expected:** JUnit 5 / Jupiter (`org.junit.jupiter.api.*`).

### B. Build-along: predict, then run (10 min)

Use a Calculator project with two test classes: one JUnit 4 test (`org.junit.Test`) and one Jupiter test (`org.junit.jupiter.api.Test`). The project has Jupiter and JUnit 4 (`junit:junit`) on the classpath, but **not** the Vintage engine.

1. **Predict:** "If I run all tests, how many run: 1 or 2? Which one?" Students write their prediction down first.
2. **Run.** Only the Jupiter test runs. The JUnit 4 test is silently skipped.
3. **Ask:** "The JUnit 4 test compiled fine. Why didn't it run?" Steer toward: no engine on the Platform understands it.
4. **Add the Vintage engine:**

   ```xml
   <dependency>
       <groupId>org.junit.vintage</groupId>
       <artifactId>junit-vintage-engine</artifactId>
       <scope>test</scope>
   </dependency>
   ```

   (In a Spring Boot project the version is managed for you.)
5. **Predict again, then run.** Both tests run.
6. **Ask:** "Which engine ran which test?"
7. **Convert:** have students migrate the JUnit 4 test to Jupiter (imports, lifecycle annotations, assertions, drop `public`), then run it. It no longer needs Vintage.

*Run this build-along yourself before class so the output matches what students will see.*

### C. Transfer (5 min)

> You join an existing Java project. You find:
>
> - 30 test classes using `org.junit.Test`
> - 15 test classes using `org.junit.jupiter.api.Test`
> - All tests run successfully.
>
> **Explain how this is possible.**

**Expected:** The 15 Jupiter tests run through the Jupiter engine; the 30 JUnit 4 tests run through the Vintage engine (so it must be on the classpath); both engines plug into the same Platform. The goal is not to memorize the diagram but to understand why two testing styles can coexist.

### Closing question (spend the most time here)

> **"If Jupiter is the JUnit 5 programming model, why do we need the JUnit Platform?"**

**Expected:** Maven, Gradle and IDEs need one stable way to discover and launch tests without knowing which framework wrote them. The Platform is that single entry point, and Jupiter and Vintage are engines plugged into it. Without it, every tool would need to know every framework, and JUnit 4 and JUnit 5 tests couldn't run in the same build.

---

## Teacher Notes

### Extra questions if students are quiet

- Why isn't the Platform simply called "JUnit 5"?
- What is a `TestEngine`'s job?
- You find a JUnit 4 test in a project that otherwise uses JUnit 5. Should you rewrite it immediately? (No. Vintage lets it run; migrate gradually.)
- Which component understands Jupiter tests? Which understands JUnit 4 tests?
- Where does the Platform sit between your tools and your tests?

### What NOT to teach deeply yet

Don't turn this session into a dependency-management lecture. Skip, or mention only briefly:

- every JUnit Platform module
- every Maven artifact (beyond the Vintage one in the build-along)
- internal Platform APIs
- writing custom `TestEngine`s
- detailed migration support
- every historical JUnit release
- every alternative JVM testing framework

The high-value knowledge here is **architecture and recognition**.
