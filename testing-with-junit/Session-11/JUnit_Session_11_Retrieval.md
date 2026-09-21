# JUnit Retrieval — Session 11
## Test Design: Equivalence Partitioning & Boundary Analysis

### 1. What is the primary purpose of equivalence partitioning?

A. To make every possible input a separate test  
B. To group inputs expected to produce similar behaviour and select representative cases  
C. To replace assertions  
D. To execute tests repeatedly

**Answer: B**

**Explanation:** Equivalence partitioning reduces a large input space into meaningful groups with similar expected behaviour.

### 2. A username must contain 5–12 characters, inclusive. Which set best represents the key boundary values?

A. 6, 7, 8  
B. 5, 8, 12  
C. 4, 5, 12, 13  
D. 1, 10, 20

**Answer: C**

**Explanation:** The important edges are just below and at the lower boundary, and at and just above the upper boundary.

### 3. What question does boundary analysis primarily ask?

A. Which framework should run the test?  
B. Where does expected behaviour change?  
C. How many assertions should a test contain?  
D. Which IDE should be used?

**Answer: B**

### 4. A value must be between 10 and 100 inclusive. Which value is most clearly a boundary case?

A. 55  
B. 72  
C. 10  
D. 48

**Answer: C**

### 5. Why is testing 9 useful when the valid range is 10–100 inclusive?

A. 9 is a random value  
B. 9 is immediately outside the lower boundary  
C. 9 is the middle of the valid range  
D. 9 is always a valid input

**Answer: B**

### 6. Which statement best distinguishes equivalence partitioning from boundary analysis?

A. They are two names for exactly the same technique.  
B. Equivalence partitioning identifies meaningful input groups; boundary analysis focuses on the edges between groups.  
C. Boundary analysis is only used for strings.  
D. Equivalence partitioning is a JUnit annotation.

**Answer: B**

### 7. A system accepts ages from 18 through 60 inclusive. Which value represents an ordinary valid case rather than a boundary?

A. 17  
B. 18  
C. 30  
D. 61

**Answer: C**

### 8. Which is the strongest reason not to test every possible value in a large valid range?

A. JUnit cannot execute many tests  
B. Every value has a completely different meaning  
C. Representative equivalence classes can provide useful coverage with fewer tests  
D. Assertions only work on boundary values

**Answer: C**

### 9. Which statement about parameterized testing is correct?

A. It determines which inputs should be tested.  
B. It is a test-design technique that replaces equivalence partitioning.  
C. It allows similar test logic to be executed with different data.  
D. It is required whenever boundary analysis is used.

**Answer: C**

### 10. Consider:

```java
boolean isValidAge(int age) {
    return age >= 18 && age <= 60;
}
```

Which set gives a strong small test suite for normal, boundary, and outside-boundary behaviour?

A. 25, 30, 40  
B. 17, 18, 30, 60, 61  
C. 18, 19, 20  
D. 30 only

**Answer: B**

### 11. A student says:

> "I used parameterized testing, so I have done my test design."

What is the best response?

A. Correct; parameterized testing automatically chooses meaningful cases.  
B. Correct; parameterized tests replace requirements analysis.  
C. Incorrect; test design determines what cases are meaningful, while parameterized testing is a way to execute similar test logic with different data.  
D. Incorrect; parameterized tests should never be used for test design.

**Answer: C**

### 12. Which test value is most likely to reveal an off-by-one defect in a minimum-value rule?

A. A value far inside the valid range  
B. The minimum value and the value immediately below it  
C. Only a random value  
D. The maximum value only

**Answer: B**

### 13. A requirement says:

> Amount must be greater than 0.

Which pair is especially important?

A. 500 and 600  
B. 1 and 2  
C. 0 and 1  
D. 100 and 200

**Answer: C**

**Explanation:** The transition between invalid and valid behaviour occurs at 0/1.

### 14. Which sequence best represents the testing thought process?

A. Write annotations → choose random values → read the requirement  
B. Read requirement → identify behaviours/classes → identify boundaries → choose cases → write tests  
C. Write code → choose framework → invent requirement  
D. Choose assertion → choose values → decide what the method does

**Answer: B**

### 15. Why might 17 and 18 both be important when the minimum valid age is 18?

A. They are both valid  
B. They test two sides of the transition between invalid and valid behaviour  
C. They are both invalid  
D. They test unrelated behaviours

**Answer: B**

---

## Short application questions

### 16. Requirement

> A score from 0 to 100 inclusive is valid.

Give:
- the equivalence partitions
- the key boundary values
- one ordinary valid representative

**Expected answer:**

```text
Partitions:
Below 0
0–100
Above 100

Boundary values:
-1, 0, 100, 101

Ordinary valid representative:
50 (or another value inside the range)
```

### 17. Explain the difference

Complete:

```text
Equivalence partitioning asks:
"________________________________________?"

Boundary analysis asks:
"________________________________________?"
```

**Expected:**

```text
"What groups of inputs are expected to behave similarly?"

"Where are the edges/transitions between those groups?"
```

### 18. Retrieval from earlier sessions

A parameterized test contains:

```java
@CsvSource({
    "17, false",
    "18, true",
    "60, true",
    "61, false"
})
```

What question should come **before** writing this test?

A. Why are these values meaningful for the requirement?  
B. Which IDE is installed?  
C. Which JUnit version has the longest name?  
D. How many assertions can Java compile?

**Answer: A**

**Explanation:** The test data should come from test-design reasoning, not from the JUnit syntax.

---

## Teacher note

The key retrieval target is not memorizing the terms.

Students should be able to look at a requirement such as:

> "8–20 characters"

and immediately begin thinking:

```text
What are the partitions?
Where are the boundaries?
What happens just below/at/just above them?
What representative value should I include?
```
