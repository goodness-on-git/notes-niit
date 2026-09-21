# JUnit Session 11 — From Requirements to Test Cases

## Session objective

By the end of this session, students should be able to:

1. Read a simple requirement and identify meaningful test cases.
2. Use **equivalence partitioning** to avoid testing every possible input.
3. Use **boundary analysis** to deliberately test values where behaviour changes.
4. Turn those decisions into JUnit tests, preferably parameterized when the test structure is the same.

---

# 1. The real problem

So far, students have learned how to write JUnit tests.

The next question is more important:

> **How do I decide what to test?**

A test can be perfectly written and still be a poor test if the chosen input does not tell us much.

Consider:

```java
boolean isValidAge(int age)
```

Suppose the requirement is:

> A customer must be between 18 and 60 years old, inclusive.

A beginner might write:

```text
25 → true
30 → true
40 → true
50 → true
```

These are all valid inputs, but they tell us relatively little.

The more useful question is:

> **Where can the behaviour change?**

For this requirement:

```text
age < 18       → invalid
18–60          → valid
age > 60       → invalid
```

That gives us much more useful test cases.

---

# 2. Equivalence partitioning

## Core idea

**Equivalence partitioning** divides possible inputs into groups where we expect the software to behave in the same way.

Instead of testing every value:

```text
18, 19, 20, 21, ... 60
```

we identify representative groups:

```text
Below 18       → invalid
18–60          → valid
Above 60       → invalid
```

We can then choose representative values:

```text
17 → false
30 → true
61 → false
```

### Mental model

```text
Many possible inputs
        ↓
Group inputs with similar expected behaviour
        ↓
Choose representative values
        ↓
Write tests
```

The purpose is not to prove every value works. It is to get useful coverage with a manageable number of tests.

---

# 3. Boundary analysis

Equivalence partitioning tells us about the groups.

Boundary analysis asks:

> **Where do those groups meet?**

For:

```text
18–60 inclusive
```

the important boundaries are:

```text
17 | 18 ........ 60 | 61
```

So the high-value cases are:

```text
17 → just below lower boundary
18 → lower boundary
60 → upper boundary
61 → just above upper boundary
```

### Practical rule

When you see a requirement such as:

```text
minimum = 18
maximum = 60
```

immediately think:

```text
minimum - 1
minimum
maximum
maximum + 1
```

Then also consider one ordinary value inside the valid range.

---

# 4. Equivalence partitioning + boundary analysis

These are related, but they are not the same thing.

### Equivalence partitioning

Asks:

> **What groups of inputs behave differently?**

### Boundary analysis

Asks:

> **Where are the edges of those groups?**

Use them together.

| Technique | Useful cases |
|---|---|
| Equivalence partitions | 17, 30, 61 |
| Boundary analysis | 17, 18, 60, 61 |

Together they give us a strong small test set.

---

# 5. Practice Example — Build the tests in class

We will test:

```java
public class AgeValidator {

    public boolean isValidAge(int age) {
        return age >= 18 && age <= 60;
    }
}
```

## Step 1 — Do NOT write JUnit yet

Ask students:

> What are the different behaviours this method can have?

Let them identify:

```text
Below 18
18–60
Above 60
```

Then ask:

> What values would represent those behaviours?

Expected discussion:

```text
17
30
61
```

Then ask:

> Where are the boundaries?

Students should identify:

```text
17 / 18
60 / 61
```

---

# 6. Turn the decisions into JUnit

Because the test structure is the same and only the data changes, parameterized testing is a good fit.

```java
@ParameterizedTest
@CsvSource({
    "17, false",
    "18, true",
    "30, true",
    "60, true",
    "61, false"
})
void shouldValidateAge(int age, boolean expected) {
    AgeValidator validator = new AgeValidator();

    assertEquals(expected, validator.isValidAge(age));
}
```

The important part is **not the annotation**.

The important part happened before the annotation:

```text
Requirement
    ↓
Identify behaviour groups
    ↓
Identify boundaries
    ↓
Choose meaningful values
    ↓
Write tests
```

---

# 7. Questions to ask during the example

Do not immediately give them the cases.

1. If I test only age 30, what behaviour have I tested?
2. What happens below 18?
3. What happens at exactly 18?
4. What happens at exactly 60?
5. Why test 17 and 61?
6. Do we need to test every age from 18 to 60?

Desired reasoning:

> We do not need every value. We want representative cases for different expected behaviours and important transition points.

---

# 8. Their own practice — Exercise 1

Give students this requirement:

> A password is considered valid if it contains between **8 and 20 characters, inclusive**.

Do **not** give them the test values.

### Their task

Before writing code:

1. Identify the equivalence partitions.
2. Identify the boundaries.
3. Choose representative test values.
4. Write the JUnit tests.
5. Use a parameterized test if appropriate.

They should eventually discover something similar to:

```text
Below 8
8–20
Above 20
```

and investigate:

```text
7
8
20
21
```

plus an ordinary valid value such as:

```text
12
```

### Teacher behaviour

If they ask:

> "What values should I use?"

Do not give the answer.

Ask:

> **"Where does the behaviour change?"**

---

# 9. Their own practice — Exercise 2

Now change the problem.

> A bank account withdrawal amount must be **greater than 0 and less than or equal to the current balance**.

Given:

```java
boolean canWithdraw(int balance, int amount)
```

Students must:

1. Identify meaningful input situations.
2. Identify boundaries.
3. Choose test cases.
4. Write JUnit tests.
5. Decide whether parameterization is useful.

Encourage them to consider:

```text
amount = 0
amount < 0
amount = balance
amount < balance
amount > balance
```

The important part is that they **derive the cases from the requirement**.

---

# 10. Small challenge

Ask:

> Which is more valuable?

### A

```text
25
26
27
28
29
```

### B

```text
17
18
30
60
61
```

Do not accept merely:

> "B because those are the boundaries."

Ask:

> **Why does B give us more information?**

Desired reasoning:

> The selected values represent different expected behaviours and the points where behaviour changes.

---

# 11. Common mistakes

### Mistake 1 — Random inputs

Students choose values because they are easy.

**Correction:**

> Choose inputs because they represent a behaviour or boundary.

### Mistake 2 — Only testing happy paths

Example:

```text
30 → true
```

**Correction:**

Ask:

> What could make this method behave differently?

### Mistake 3 — Testing only the exact boundaries

Students may test:

```text
18
60
```

but omit:

```text
17
61
```

Ask:

> What happens immediately outside the allowed range?

### Mistake 4 — Thinking equivalence partitioning means "test one value"

The goal is to identify meaningful classes and select representative cases.

### Mistake 5 — Thinking parameterized testing is test design

```text
Test design
→ decides WHAT to test

Parameterized testing
→ provides a convenient way to run similar test logic with different data
```

---

# 12. Reusable workflow

Students should leave with this process:

```text
READ THE REQUIREMENT
        ↓
What behaviours can occur?
        ↓
Identify equivalence partitions
        ↓
Where do behaviours change?
        ↓
Identify boundaries
        ↓
Choose representative + boundary values
        ↓
Choose ordinary or parameterized JUnit tests
        ↓
Run the tests
        ↓
Ask whether the test set gives useful coverage
```

---

# 13. Independent assignment — Discount Calculator

You are given:

```java
public class DiscountCalculator {

    public double calculateDiscount(double amount) {
        if (amount < 100) {
            return 0;
        }

        if (amount <= 500) {
            return amount * 0.10;
        }

        return amount * 0.20;
    }
}
```

### Requirement

```text
Less than 100       → 0% discount
100–500             → 10% discount
Above 500           → 20% discount
```

### Your task

Before writing the JUnit tests:

1. Identify the equivalence partitions.
2. Identify all important boundaries.
3. Choose representative values.
4. Choose boundary values.
5. Decide which cases can be expressed as a parameterized test.
6. Write the JUnit tests.
7. Run them.
8. Deliberately introduce one defect into `DiscountCalculator` and see whether your tests detect it.

### Final question

Write **3–5 sentences** answering:

> Why did you choose these particular test values instead of simply choosing random amounts?

This explanation is part of the assignment.

---

# 14. Teacher preparation

Before class:

- Implement the example yourself.
- Run the tests.
- Deliberately introduce boundary defects, e.g. change `< 100` to `<= 100`.
- See which test detects the defect.
- Try removing one boundary test and observe what changes.
- Prepare to ask questions rather than immediately supplying test cases.

---

# 15. Exit test

Students should be able to answer these without notes:

1. What is equivalence partitioning?
2. What is boundary analysis?
3. How are they related?
4. Why is `minimum - 1` often useful?
5. Why is `minimum` itself useful?
6. Why might we not test every value in a valid range?
7. What is the difference between test design and parameterized testing?
8. Given a requirement, can you derive test cases before writing JUnit code?

## Session takeaway

> **Good testing starts before we write the test code: we first decide which inputs and behaviours are worth testing.**
