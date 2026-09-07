# Chapter 10: Layered Architecture (Controller → Service → Repository)

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** The three layers are now built **incrementally, live**, with the same endpoint tested after each layer is added. Students watch the response stay byte-for-byte identical while the code underneath is restructured three times — this is the exact same "the output didn't change, so what actually changed?" moment from Chapter 1, Build-Along 2, now applied to architecture instead of DI. Point that out explicitly when you get there.

Chapter 9 put all the logic for an endpoint directly inside the controller. That's fine for a tiny example, but it breaks down fast in a real app. This chapter introduces the standard 3-layer structure most Spring Boot applications use to keep things organized. No new dependencies needed — this builds directly on the project from Chapters 8–9.

## 🎯 Lesson Objective

Students should understand — and have built and run code demonstrating:

- Why putting everything in one class becomes a problem
- The responsibility of each layer: Controller, Service, Repository
- How to wire the three layers together using constructor injection
- The request flow from browser to database and back
- Common beginner mistakes when splitting responsibilities

## 🗺️ Lesson Roadmap

1. Start With the Bad Example — **build-along**
2. What Is Layered Architecture?
3. Controller Layer — **build-along (stage 1)**
4. Service Layer — **build-along (stage 2, same output)**
5. Repository Layer — **build-along (stage 3, same output again)**
6. Complete Flow
7. Mini Exercise — **build-along, independent**
8. Closing Statement

---

## 1. Start With the Bad Example

> 💬 **Ask:** "What if we put ALL logic inside one class?"

```java
@RestController
public class StudentController {

    @GetMapping("/students")
    public List<String> getStudents() {

        // Database logic
        // Business logic
        // Validation
        // Formatting
        // Everything here 😭

        return List.of("John", "Mary");
    }
}
```

❌ **The problems:**
- Hard to maintain
- Hard to test
- Hard to scale
- Confusing responsibilities

> 🧠 **Say:** "Just because code works doesn't mean the design is good."

#### 🛠️ Build-Along — Start here, on purpose

**Say:** "In your project from Chapters 8–9, create exactly this: a `StudentController` with one `GET /students` endpoint returning a hardcoded `List.of(\"John\", \"Mary\")`. Run it, hit the endpoint, confirm it works."

✅ **Expected result:** `["John","Mary"]` in the browser or Postman.

> 💬 **Say:** "This works completely fine right now, with two names. Hold onto that thought — we're about to restructure this three times, and I want you to notice something at the end of each restructure."

---

## 2. What Is Layered Architecture?

> 🧠 **Say:** "Layered architecture separates application responsibilities into different layers."

**The 3 main layers:**

```
Client
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

> 🧠 **Say:** "Each layer has one responsibility."

---

## 3. Controller Layer 🌐

✅ **Responsibility:** Handles HTTP requests and responses.

❌ **What the controller should NOT do:**
- Database logic
- Complex business logic

> 🧠 **Say:** "Controllers should coordinate, not think."

> 📌 **Instructor Note:** The build-along for this stage is deferred to Section 4 below, since a controller alone with no service to delegate to is just the bad example from Section 1. The real teaching moment is watching it change *into* a coordinator, not building it in isolation.

---

## 4. Service Layer 🧠

✅ **Responsibility:** Contains business logic.

```java
import org.springframework.stereotype.Service;

@Service
public class StudentService {

    public List<String> getStudents() {
        return List.of("John", "Mary");
    }
}
```

**What belongs here:**
- Calculations
- Rules
- Validation
- Decision-making

> 🧠 **Say:** "The service layer is the brain of the application."

#### 🛠️ Build-Along — Stage 1: move logic into a Service (predict first)

**Say:** "Before you touch anything: predict — if we move the hardcoded list out of `StudentController` and into a new `@Service` class, then have the controller call it, will the response from `GET /students` change at all?"

> 🧠 **Let them answer, then proceed.**

**Now say:** "Create `StudentService`, marked `@Service`, with a `getStudents()` method returning the same hardcoded list. Then update `StudentController` to take a `StudentService` via constructor injection, and have the endpoint call `service.getStudents()` instead of returning the list directly."

```java
@RestController
public class StudentController {

    private final StudentService service;

    public StudentController(StudentService service) {
        this.service = service;
    }

    @GetMapping("/students")
    public List<String> getStudents() {
        return service.getStudents();
    }
}
```

**Say:** "Run it. Hit `GET /students` again."

✅ **Expected result:** `["John","Mary"]` — **identical to Section 1.**

> 💬 **Say, and make this land:** "Same response, to the byte. This is the exact same thing that happened back in Chapter 1 when you converted tight coupling into dependency injection — the *output* didn't change, but *who's responsible for what* did. `StudentController` no longer knows or cares where the list comes from. That's the whole point of a service layer."

**What just happened?**
- **Controller:** receives the request, delegates the work
- **Service:** handles the logic

> 🧠 **Say:** "Controllers delegate responsibility to services."

---

## 5. Repository Layer 🗄️

✅ **Responsibility:** Handles data access.

```java
import org.springframework.stereotype.Repository;

@Repository
public class StudentRepository {

    public List<String> findAll() {
        return List.of("John", "Mary");
    }
}
```

> 🧠 **Say:** "Repositories talk to the database."

#### 🛠️ Build-Along — Stage 2: move the list one layer deeper (predict again)

**Say:** "Same question as before: if we move the hardcoded list out of `StudentService` and into a new `@Repository`, and have the service call the repository, will `GET /students` return anything different?"

**Now say:** "Create `StudentRepository`, marked `@Repository`, with a `findAll()` method returning the hardcoded list. Update `StudentService` to take a `StudentRepository` via constructor injection, and have `getStudents()` call `repository.findAll()` instead of returning the list directly."

```java
@Service
public class StudentService {

    private final StudentRepository repository;

    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }

    public List<String> getStudents() {
        return repository.findAll();
    }
}
```

**Say:** "Run it. Hit `GET /students` one more time."

✅ **Expected result:** `["John","Mary"]` — **identical again, for the third time.**

> 💬 **Say:** "Three completely different internal structures. Same exact response every time. That's the proof that architecture is about organizing *how* the work gets done, not changing *what* gets returned — and it's the same lesson from Chapter 1, now scaled up to a whole application instead of one class."

---

## 6. Complete Flow

**Request flow, end to end:**

```
Browser Request
      ↓
Controller
      ↓
Service
      ↓
Repository
      ↓
Data Returned
```

> 🧠 **Say:** "Requests move downward, responses move upward. You just built and ran all three stages of that flow yourself."

### Why This Design Is Powerful

✅ **1. Easier testing** — you can test each layer independently
✅ **2. Easier maintenance** — logic is organized
✅ **3. Easier scaling** — large teams can work separately
✅ **4. Cleaner code** — each class has one purpose

> 🧠 **Say:** "Good architecture reduces chaos."

### Common Student Mistakes

❌ **Putting everything in the Controller** — a very common beginner problem; you started the lesson exactly there in Section 1
❌ **Repository containing business logic** — repositories should focus on data only
❌ **Services returning HTTP responses** — that belongs to the controller layer

### Real-World Analogy

| Layer | Real-World Role |
|---|---|
| Controller | Receptionist |
| Service | Manager |
| Repository | File clerk |
| Database | Storage room |

> 🧠 **Say:** "The controller receives requests, the service decides, the repository retrieves."

---

## 7. Mini Exercise

Build the full 3-layer chain for a new resource: **books**.

1. Create a `BookRepository` (`@Repository`) with a `findAll()` method returning a hardcoded list of book titles
2. Create a `BookService` (`@Service`) that depends on `BookRepository` and exposes a `getBooks()` method
3. Create a `BookController` (`@RestController`) with a `GET /books` endpoint that delegates to `BookService`
4. Run it and confirm the endpoint returns your list

#### 🛠️ Build-Along — Solve it independently

> 📌 **Instructor Note:** Give the spec as-is, no staged predict-and-build this time — students just built that exact process twice in this chapter and should be able to go straight to the full three-layer structure in one pass.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex;

import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class BookRepository {

    public List<String> findAll() {
        return List.of("Clean Code", "Effective Java");
    }
}
```

```java
package org.codex;

import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class BookService {

    private final BookRepository repository;

    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public List<String> getBooks() {
        return repository.findAll();
    }
}
```

```java
package org.codex;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class BookController {

    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @GetMapping("/books")
    public List<String> getBooks() {
        return service.getBooks();
    }
}
```

✅ **Response:**
```json
["Clean Code", "Effective Java"]
```

</details>

---

## 8. Closing Statement

## ✅ Key Takeaways

- **Controller** — handles requests, doesn't think
- **Service** — business logic, the brain of the app
- **Repository** — data access, doesn't reason about business rules
- Each layer is wired to the next via constructor injection — requests flow down, responses flow up
- Splitting responsibilities this way makes code easier to test, maintain, and scale
- Restructuring into layers doesn't change what a request returns — you proved this by getting the identical response three times, across three different internal structures

> 🧠 "Layered architecture separates concerns to keep applications clean and scalable."

---

**← Previous: [Chapter 9 — Building Your First REST API with Spring Boot](./chapter-9-building-your-first-rest-api.md)** | **Next: Chapter 11 →**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- Restructured the chapter from "show three separate finished versions" into a single continuous build-along: students start with the bad example, then migrate the same logic through Service and then Repository, testing the identical endpoint after each move.
- Added a "predict before you build" moment before each migration, matching the pattern used since Chapter 5 — most students will hesitate or half-expect something to change, which makes the identical output land harder.
- Explicitly named the connection to Chapter 1, Build-Along 2 ("the output didn't change, so what actually changed?") — this is the same underlying lesson (restructuring responsibility vs. changing behavior) recurring at a larger scale, and naming that connection for the instructor makes it easy to say out loud in class.
- The Mini Exercise is now explicitly independent with no staged scaffolding, since students just performed the full staged version twice in the chapter itself.
