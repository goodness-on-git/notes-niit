# Chapter 15: Exception Handling

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. If you're a student reading independently, you can skip them.

Right now, if a client asks for a student who doesn't exist, our API doesn't even complain — it quietly returns nothing and calls it a success. This chapter starts by proving that, then replaces it with clean, predictable error responses: the right status code and a message the client can actually use.

## 🎯 Lesson Goal

Students should understand:

- What exceptions are (fast recap — they know this from Core Java)
- Why exception handling matters in an API
- Silent failures vs loud failures
- Custom exceptions
- `@ExceptionHandler`
- `@ControllerAdvice`
- Choosing the right HTTP status code for an error

> 📌 **Scaffolding level: chunked.** This is late-course. Instructions are given 2–3 steps at a time, predictions come before runs, and the Mini Exercise is framed as mostly independent work. Step in only when students stall.

> 📌 **Before class — prerequisites and a 5-minute dry run:**
> 1. Chapter 13's `StudentController` / `StudentService` / `StudentRepository` are working, with `getStudent()` still ending in `.orElse(null)`.
> 2. Chapter 13's `Book` CRUD exists and **`Book` uses an `Integer` id** (not `Long`) — this matches `Student`. If your students built it with `Long`, change it first: the field, its getter/setter, `JpaRepository<Book, Integer>`, and the `id` parameters in `BookService` and `BookController`.
> 3. Chapter 14's `@Valid` validation is on `StudentRequest`.
> 4. Run the three `999` requests in Section 1 yourself once. Two of the results depend on your Spring Boot / Hibernate version (flagged below), and you should know which one your setup shows before students do.

## 🗺️ Lesson Roadmap

1. Start With the Problem (see the silent failures)
2. What Is an Exception? (fast recap)
3. Make the Failure Loud
4. Creating a Custom Exception
5. `@ExceptionHandler`
6. The Problem With That
7. Enter `@ControllerAdvice`
8. Fix Update and Delete Too
9. Validation Errors and the Safety Net
10. Which Status Code? (and Common Exceptions)
11. Visual Flow
12. Mini Exercise
13. Closing Statement

---

## 1. Start With the Problem

> 💬 **Say:** "Our API currently has students in it. Let's find out what happens when a client asks for one that doesn't exist."

### 🔨 Build-Along 1: Ask for a student who doesn't exist

**Setup (2 minutes):** Students `POST /students` with:

```json
{
  "name": "John",
  "age": 20
}
```

Each student **writes down the `id` in their own response**. In these notes we'll call it `X` — use your real number. `999` is a number that definitely doesn't exist.

**🔮 Predict first:**

> ❓ **Ask:** "We're about to send `GET /students/999`. There's no student 999. What comes back — an error, an empty response, or something else? Write your guess down."

Take a show of hands, then:

**⌨️ Students do:** send `GET /students/999`.

**✅ Expected:** `200 OK` with an **empty body**. No error at all.

> 🧠 **Teaching line:** "A failure that returns `200 OK` is a lie. The client thinks it worked."

> 📌 This is why `getStudent()` in Chapter 13 ended with `.orElse(null)`: if the student isn't there, hand back `null`. Spring serializes `null` as an empty body, and the status stays `200`. The client can't tell "student doesn't exist" from "the server has a bug."

**⌨️ Students do (chunked — do both, then compare):**
1. Send `PUT /students/999` with `{"name": "Ghost", "age": 30}`
2. Send `DELETE /students/999`
3. Write down both status codes, then `GET /students` and check whether the table changed

**✅ Expected:**

- `DELETE /students/999` → `200 OK`, empty body. Nothing was deleted, and nobody was told.
- `PUT /students/999` → depends on your version:
  - **Older Hibernate (before 6.6 / Spring Boot before 3.4):** `200 OK`, and `GET /students` now shows a **brand-new student with a fresh id** — not 999. The update silently became an insert.
  - **Newer Hibernate (6.6+ / Spring Boot 3.4+):** a `500` (typically an `ObjectOptimisticLockingFailureException` in the console). Loud, but with the wrong status and a useless message.

> 📌 Run these on your own setup before class so you know which result to expect. Either way, the point is the same: **none of the three requests says "not found."** Also, the `DELETE` behavior above is what Spring Data JPA 3.x does; older versions throw `EmptyResultDataAccessException` instead.

> 🧠 **Teaching line:** "Three requests for a student who doesn't exist — and not one of them told the client. That's what we're fixing."

---

## 2. What Is an Exception? (Fast Recap)

> 📌 Students know this from Core Java. Spend two minutes, no more. The new part is the last paragraph: where does the exception go when *we're not the caller*?

An exception is an object that represents an error condition.

```java
int result = 10 / 0;   // throws ArithmeticException
```

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero");
}
```

> ❓ **Ask:** "Checked or unchecked — who remembers the difference?"

- **Checked** exceptions must be declared or caught (`throws` / `try-catch`).
- **Unchecked** exceptions (subclasses of `RuntimeException`) don't have to be. That's why our custom exceptions will extend `RuntimeException`: no `throws` clutter through every controller and service method.

> 🧠 **Teaching line:** "Exceptions allow programs to react to problems instead of crashing."

**The new problem:** `try`/`catch` works when *we* call the method that might fail. But in a Spring app, who calls our controller method? **Spring does.** If an exception escapes our controller, it goes up to Spring — and we haven't told Spring what to do with it.

> 🧠 **Teaching line:** "Exceptions are unexpected situations that prevent normal execution."

---

## 3. Make the Failure Loud

> 💬 **Say:** "A silent failure is more dangerous than a loud one, because nothing warns you anything went wrong. Let's make this one loud first, then make it *good*."

### 🔨 Build-Along 2: Throw instead of returning `null`

**⌨️ Students do (chunked):**
1. In `StudentService.getStudent()`, change `.orElse(null)` to `.orElseThrow()` — **no arguments**
2. Restart the app
3. Send `GET /students/999` again

```java
public Student getStudent(Integer id) {
    return repository.findById(id).orElseThrow();
}
```

**✅ Expected:**

- The client gets a `500 Internal Server Error`. In a browser you'll see the Whitelabel Error Page; in Postman, a small JSON error with `"status": 500`.
- In the **console**, a stack trace: `java.util.NoSuchElementException: No value present`.

> 🧠 **Teaching line:** "Loud is better than silent. But loud isn't the same as good."

> ❓ **Ask:** "The client asked for a student that doesn't exist. Whose mistake is that — the server's or the client's?" (The client's. A `500` says "the server is broken," and it isn't.)

**Two problems to name out loud:**

1. `"No value present"` tells nobody what's wrong.
2. `500` is the wrong status. The right one is `404 Not Found`.

---

## 4. Creating a Custom Exception

Instead of a generic `NoSuchElementException`, we throw something that says exactly what happened.

### 🔨 Build-Along 3: `StudentNotFoundException`

**⌨️ Students do (chunked):**
1. Create a class `StudentNotFoundException` in a new package `org.codex.exception`
2. It extends `RuntimeException` and has one constructor that takes a `String message` and passes it to `super`
3. In `getStudent()`, replace `.orElseThrow()` with `.orElseThrow(...)` that throws it, including the id in the message

```java
package org.codex.exception;

public class StudentNotFoundException extends RuntimeException {

    public StudentNotFoundException(String message) {
        super(message);
    }
}
```

```java
public Student getStudent(Integer id) {
    return repository.findById(id)
            .orElseThrow(() -> new StudentNotFoundException("Student not found with id: " + id));
}
```

**⌨️ Restart, then send `GET /students/999`.**

**✅ Expected:** *Still* a `500` for the client — but now the console says `StudentNotFoundException: Student not found with id: 999`.

> 🧠 **Teaching line:** "Custom exceptions make errors meaningful."

> 📌 Be clear about what this step did and didn't fix. The **developer** now gets a meaningful message. The **client** still gets a `500`. That gap is exactly what the rest of the chapter closes.

---

## 5. `@ExceptionHandler`

> 💬 **Say:** "We need to tell Spring: 'when this exception comes out of my controller, here's what to send back.' That's what `@ExceptionHandler` does."

### 🔨 Build-Along 4: Handle it inside the controller

**⌨️ Students do:** add this method to `StudentController`.

```java
@ExceptionHandler(StudentNotFoundException.class)
public String handleStudentNotFound(StudentNotFoundException ex) {
    return ex.getMessage();
}
```

**🔮 Predict first:**

> ❓ **Ask:** "We're about to send `GET /students/999` again. The body will be our message. What status code will Postman show?"

Most students say `404`. Let them commit.

**⌨️ Students do:** restart and send `GET /students/999`.

**✅ Expected:** body `Student not found with id: 999`, status **`200 OK`**.

> 📌 We've traded a silent failure for a *slightly less* silent one. The message is right, but the status still says success. A client that checks status codes (and every real client does) would treat this as a working response. The status code is part of the message.

> 🧠 **Teaching line:** "Getting the message right isn't enough. The status code has to match too."

---

## 6. The Problem With That

> 💬 **Say:** "The handler lives inside `StudentController`. Let me show you what that means for `BookController`."

### 🔨 Build-Along 5: Prove the handler is scoped to one controller

> 📌 This is a **temporary** hack, purely as proof. We revert it at the end of the next section.

**⌨️ Students do (chunked):**
1. In `BookService.getBook()`, change `.orElse(null)` to throw `StudentNotFoundException` (yes, the *Student* one, on purpose)
2. Restart
3. Send `GET /books/999`

```java
public Book getBook(Integer id) {
    return repository.findById(id)
            .orElseThrow(() -> new StudentNotFoundException("Book not found with id: " + id)); // TEMPORARY — proof only
}
```

**🔮 Predict first:**

> ❓ **Ask:** "There's a handler for `StudentNotFoundException`, and we're throwing `StudentNotFoundException`. Does that handler catch it?"

**✅ Expected:** a raw `500`. The handler sits in `StudentController`, and this exception came out of `BookController`. Same exception type, handler exists, still not caught.

> 🧠 **Teaching line:** "A controller-level handler only protects that controller."

Now the cost is obvious. Every controller — `StudentController`, `BookController`, `UserController` — would need its own copy of the same handler. That's duplicated code.

---

## 7. Enter `@ControllerAdvice`

> 💬 **Say:** "Spring lets us write the handler **once**, in a class that applies to the whole application."

### 🔨 Build-Along 6: One global handler

**⌨️ Students do (chunked):**
1. **Delete** the `@ExceptionHandler` method from `StudentController`
2. Create `GlobalExceptionHandler` in `org.codex`, annotated with `@ControllerAdvice`
3. Move the handler there, but this time return a `ResponseEntity` with a real `404`

```java
package org.codex;

import org.codex.exception.StudentNotFoundException;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(StudentNotFoundException.class)
    public ResponseEntity<String> handleStudentNotFound(StudentNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

**⌨️ Restart and send both:** `GET /students/999` and `GET /books/999`.

**✅ Expected:** both return **`404 Not Found`**, body as plain text (`Student not found with id: 999` and `Book not found with id: 999`).

> 🧠 **Teaching line:** "`@ControllerAdvice` centralizes error handling."

Now the `BookController` hack has served its purpose:

**⌨️ Students do:** revert `BookService.getBook()` back to `.orElse(null)`. (The Mini Exercise will build the *proper* Book exception.)

> 📌 `@ControllerAdvice` is Spring's "use this class for handling exceptions across the entire application." You'll also see `@RestControllerAdvice` in real projects — it's `@ControllerAdvice` plus `@ResponseBody`, so plain return values are written straight into the response body. We return `ResponseEntity` here, which works with either.

### 🔮 Predict: is that JSON?

> ❓ **Ask:** "Look at the response body in Postman. Is it JSON like `{"message": "..."}` or something else? Check the `Content-Type` header too."

**✅ Expected:** plain text (`text/plain`), not JSON. Our handler returns a `String`.

> 💬 **Say:** "Real APIs return JSON errors, so clients can parse them. Let's fix that."

**⌨️ Students do:** change the return type to `ResponseEntity<Map<String, String>>` and wrap the message in a map. Use `HttpStatus.NOT_FOUND` instead of the bare `404`.

```java
@ExceptionHandler(StudentNotFoundException.class)
public ResponseEntity<Map<String, String>> handleStudentNotFound(StudentNotFoundException ex) {
    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(Map.of("message", ex.getMessage()));
}
```

**✅ Expected:** `GET /students/999` → `404 Not Found`

```json
{
  "message": "Student not found with id: 999"
}
```

> 📌 A real project usually defines a dedicated `ErrorResponse` class (status, message, timestamp). A `Map` keeps today's build-along small; mention the upgrade path and move on.

### 💥 Break it on purpose (2 minutes, optional but worth it)

> 💬 **Say:** "Remember the project-structure rule from Spring Boot Introduction? Let's see it bite again."

**⌨️ Students do:**
1. Move `GlobalExceptionHandler` into a package **outside** `org.codex` (for example `org.example.web`)
2. Restart — notice **no error at startup**
3. Send `GET /students/999`

**✅ Expected:** back to a raw `500`. The app started fine, and Spring never warned you the handler was being ignored. That makes it a **silent failure**: nothing tells you the safety net isn't attached.

**⌨️ Move it back to `org.codex`.**

> 🧠 **Teaching line:** "Component scanning applies to advice classes too. If it's outside the base package, Spring never sees it."

---

## 8. Fix Update and Delete Too

> 💬 **Say:** "Remember Section 1? Three requests for student 999 and none of them said 'not found.' We've fixed `GET`. Now the other two."

### 🔨 Build-Along 7: Not-found for `PUT` and `DELETE`

**⌨️ Students do (chunked):**
1. Change `updateStudent()` so it first loads the student with `getStudent(id)` (which throws if missing), copies `name` and `age` onto it, and saves it
2. Change `deleteStudent()` so it throws `StudentNotFoundException` if `existsById(id)` is false, and only then calls `deleteById(id)`
3. Restart

```java
public Student updateStudent(Integer id, StudentRequest request) {
    Student student = getStudent(id);          // throws if missing
    student.setName(request.getName());
    student.setAge(request.getAge());

    return repository.save(student);
}
```

```java
public void deleteStudent(Integer id) {
    if (!repository.existsById(id)) {
        throw new StudentNotFoundException("Student not found with id: " + id);
    }
    repository.deleteById(id);
}
```

**⌨️ Re-run the three 999 requests from Section 1** (`GET`, `PUT`, `DELETE`).

**✅ Expected:** all three → `404 Not Found` with the JSON `message`.

> 📌 **Tie-back to Chapter 13:** the old update built a `new Student()` and called `setId(id)` by hand, and forgetting that line silently created a duplicate row. Now the entity is *loaded* from the database, so it already carries its id. The Chapter 13 bug can't happen here. Also note that `existsById()` came free from `JpaRepository`, like `findAll()` and `findById()`.

### Finish the thread: your tracked student

**⌨️ Students do (chunked, using their real `X` from Section 1):**
1. `PUT /students/X` with `{"name": "Michael", "age": 25}` → updated student, `200`
2. `DELETE /students/X` → `200`
3. `GET /students/X` again

**✅ Expected:** the last request now returns **`404`** with `"Student not found with id: X"`. The student was real a moment ago and is now correctly reported as gone.

> 🧠 **Teaching line:** "Same record, whole lifecycle — and now every step tells the truth."

---

## 9. Validation Errors and the Safety Net

### 9a. Chapter 14 tie-back: make validation errors useful

> 💬 **Say:** "In Chapter 14, `@Valid` started rejecting bad input. But what does the client actually *see*?"

**⌨️ Students do:** `POST /students` with a body that breaks one of your Chapter 14 rules (for example, a blank `name`).

**✅ Expected:** `400 Bad Request` — the right status, but the body is Spring's generic error JSON with **no hint about which field was wrong**.

> 📌 This uses whatever rules your students added in Chapter 14. Adjust the invalid request to match your annotations.

**⌨️ Students do:** add a second handler to `GlobalExceptionHandler` for `MethodArgumentNotValidException`, which is what `@Valid` throws.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors()
            .forEach(error -> errors.put(error.getField(), error.getDefaultMessage()));

    return ResponseEntity.badRequest().body(errors);
}
```

**✅ Expected:** `400` with a body like `{"name": "<your Chapter 14 message>"}`. The client now knows exactly which field to fix.

> 🧠 **Teaching line:** "One `@ControllerAdvice` class, many exception types — it becomes your API's error policy."

### 9b. Optional stretch: the catch-all handler

> 📌 **Skip if short on time.** It's useful, but it isn't part of the core 20%.

> 💬 **Say:** "What about the exceptions we didn't plan for? A handler for `Exception` catches everything else."

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<Map<String, String>> handleUnexpected(Exception ex) {
    // a real app would log ex here, otherwise you've hidden the bug from yourself
    return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(Map.of("message", "Something went wrong"));
}
```

**🔮 Predict twice, then run:**

1. `GET /students/999` — does the `Exception` handler catch it, or the more specific `StudentNotFoundException` one? (**Most specific wins** → still `404`.)
2. `GET /students/abc` (a letter where an `Integer` is expected) — Spring normally answers `400`. What now? (**`500` "Something went wrong"**: the catch-all swallowed Spring's own `400`.)

> 🧠 **Teaching line:** "A catch-all hides internals from clients, which is good. But it can also steal status codes Spring would have chosen correctly, so treat it as a last resort."

---

## 10. Which Status Code?

| Situation | Status | Whose fault? |
|---|---|---|
| Resource doesn't exist | `404 Not Found` | Client asked for something missing |
| Client sent invalid data | `400 Bad Request` | Client |
| Unexpected bug or crash | `500 Internal Server Error` | Server |

> 🧠 **Teaching line:** "`4xx` means *you* got it wrong. `5xx` means *we* did."

**Common built-in exceptions** (plain Java, nothing special to Spring):

| Exception | Meaning |
|---|---|
| `NullPointerException` | Object is null |
| `ArithmeticException` | Illegal arithmetic |
| `IllegalArgumentException` | Bad argument |
| `NoSuchElementException` | Nothing there (what `orElseThrow()` gave us) |
| `RuntimeException` | Generic runtime error |

---

## 11. Visual Flow

```
Request
   ↓
Controller
   ↓
Service
   ↓
Exception Thrown
   ↓
@ControllerAdvice
   ↓
Friendly Response
```

> 📌 **Save this diagram for Chapter 16.** It only covers failures that happen *inside* the Controller → Service path. The next chapter adds a security layer that rejects requests *before* they reach any controller. Ask students then: "Will our `GlobalExceptionHandler` catch that?"

---

## 12. Mini Exercise

> 📌 **Scaffolding: mostly independent.** Students work from the checklist below. Intervene only when they're stuck.

Do for **Books** what we just did for Students. `GET /books/999`, `PUT /books/999`, and `DELETE /books/999` should all return a clean `404` with a JSON `message`.

**Checklist:**
1. Create `BookNotFoundException`
2. Make `BookService.getBook()`, `updateBook()`, and `deleteBook()` throw it when the book doesn't exist
3. Add a handler in `GlobalExceptionHandler` returning `404` with a JSON `message`

**Verify (all three must return `404` + JSON):**
- `GET /books/999`
- `PUT /books/999`
- `DELETE /books/999`

**Then verify it still works for a real book:** create one, `GET` it, update it, delete it, and confirm the final `GET` gives `404`.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex.exception;

public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException(String message) {
        super(message);
    }
}
```

```java
// BookService.java — updated methods
public Book getBook(Integer id) {
    return repository.findById(id)
            .orElseThrow(() -> new BookNotFoundException("Book not found with id: " + id));
}

public Book updateBook(Integer id, BookRequest request) {
    Book book = getBook(id);                  // throws if missing
    book.setTitle(request.getTitle());
    book.setAuthor(request.getAuthor());
    return repository.save(book);
}

public void deleteBook(Integer id) {
    if (!repository.existsById(id)) {
        throw new BookNotFoundException("Book not found with id: " + id);
    }
    repository.deleteById(id);
}
```

```java
// GlobalExceptionHandler.java — add alongside the existing handlers
@ExceptionHandler(BookNotFoundException.class)
public ResponseEntity<Map<String, String>> handleBookNotFound(BookNotFoundException ex) {
    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(Map.of("message", ex.getMessage()));
}
```

`GET`, `PUT`, and `DELETE` on `/books/999` now all return a clean `404` with `{"message": "Book not found with id: 999"}`.

</details>

---

## 13. Closing Statement

## ✅ Key Takeaways

- An exception is an object representing an error condition — it stops normal execution
- A silent failure (a `200` for something that failed) is more dangerous than a loud one; returning `null` for "not found" hid the problem
- Custom exceptions (extending `RuntimeException`) make errors meaningful instead of generic
- `@ExceptionHandler` handles exceptions per-controller; `@ControllerAdvice` handles them globally, avoiding duplicate code
- `ResponseEntity` lets you control both the status code and the body of an error response
- The status code is part of the message: `404` for missing resources, `400` for bad input, `500` for server bugs
- The most specific matching handler wins; a catch-all `Exception` handler is a last resort, not a first choice

> 🧠 "Exception handling converts application errors into meaningful responses."

---

## 📝 What Changed in This Revision

*(Not for dictation — so you can see the reasoning without re-deriving it.)*

**The bottleneck.** The original opened with a claim ("a huge, ugly error page") that describes a state students aren't actually in. Chapter 13's `getStudent()` returns `null`, so the real behavior is a silent `200` with an empty body. The chapter now starts there: students see the silent failure first, then the loud one, then fix it.

**Techniques applied:**
- **Silent vs loud failure (technique 4):** `GET`/`PUT`/`DELETE` on a missing id are all shown failing quietly before any fix exists.
- **Predict before you run (5):** the `200` status on the controller-level handler; whether a handler in one controller protects another; whether the response is really JSON; most-specific-wins; the catch-all overreach.
- **Proving claims the material only asserted (6):** (a) the controller-level `@ExceptionHandler` returns `200` (the original only noted it), (b) `@ExceptionHandler` is scoped to one controller (the original just said "duplicate code"), (c) the original's "Real API response" showed JSON that its own code didn't produce. Students now see plain text first, then fix it.
- **Tie-backs (7):** Chapter 8's component-scanning rule (advice class outside the base package is silently ignored), Chapter 13's `setId` update bug (now impossible because the entity is loaded), Chapter 14's `@Valid` failures (now handled with field-level messages), `existsById()` from `JpaRepository`.
- **One continuous thread (8):** the tracked student `X` runs through create → missing-id failures → fixed `PUT` → fixed `DELETE` → `404` on the deleted record.
- **Redundancy compressed:** Sections 2–3 and the built-in exceptions table re-taught Core Java; now a two-minute recap, with the new content being *where an exception goes when Spring is the caller*.
- **Scaffolding fade:** chunked 2–3 steps per instruction; the exercise is a checklist with a verify step and no hand-holding.

**Cross-chapter issues to act on:**
1. **Chapter 13 (`Book` id type).** `Student` uses `Integer`, but the Chapter 13 mini-exercise `Book` uses `Long`. We agreed to standardize on `Integer`. In Chapter 13 this changes: the `Book` id field, its getter/setter, `JpaRepository<Book, Long>` → `Integer`, and the `id` parameters in `BookService` and `BookController`. Chapter 15 assumes `Integer` throughout.
2. **Chapter 13 (`getStudent` returning `null`).** Now explicitly superseded here. Chapter 13 should be left as-is, since the silent-failure lesson depends on students having built it that way.

**Things I could not verify by running code, so please check before class:**
- The `PUT /students/999` result differs between Hibernate versions (silent duplicate vs a loud `500`); the `DELETE` result also varies by Spring Data JPA version. Both are flagged in Section 1.
- Section 9a assumes your Chapter 14 rules, since I didn't have the Chapter 14 file in this conversation.

**Added beyond the original (Pareto call):** the "Which Status Code?" table, `PUT`/`DELETE` handling, validation-error handling, and the optional catch-all. Everything else in the chapter is the original's 20%: the custom exception, `@ControllerAdvice`, and `ResponseEntity`.

---

**← Previous: [Chapter 14 — Validation](./chapter-14-validation.md)** | **Next: [Chapter 16 — Spring Security Fundamentals](./chapter-16-spring-security-fundamentals.md)**
