# Chapter 12: Handling Request Data in Spring Boot

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** All three data-arrival styles are now build-alongs, tested with real requests. The two "Common Student Mistakes" that involve a wrong/missing annotation are turned into deliberate build-alongs — one produces a loud error (easy to notice), the other produces silently null fields (much easier to miss, and worth experiencing directly for exactly that reason).

This is one of the most practical chapters in Spring. Up until now, your endpoints have looked like:

```java
@GetMapping("/students")
public List<Student> getStudents() {
    return service.getStudents();
}
```

But real applications need data *from* users — a specific ID, a search filter, or a whole object to create. This chapter covers the three ways that data arrives, and how Spring gets it into your Java method.

## 🎯 Lesson Goal

Students should understand — and have built and tested requests demonstrating:

- `@PathVariable`
- `@RequestParam`
- `@RequestBody`
- When to use each — including what happens when the wrong one is used
- How Spring maps request data to Java objects

## 🗺️ Lesson Roadmap

1. Path Variables — **build-along**
2. Request Parameters — **build-along**
3. Path Variable vs Request Param — **build-along (cause the mistake on purpose)**
4. Request Body — **build-along, plus the DTO proof**
5. Visual Summary
6. Real CRUD Examples
7. Common Student Mistakes — **build-along (the silent one)**
8. Mini Exercise — **build-along, independent**
9. Closing Statement

---

> 💬 **Ask:** "If the URL contains data, how can my Java method access it?"

**Three main ways data arrives:**

```
Client Request
      ↓
Path Variable
      ↓
Request Parameter
      ↓
Request Body
```

---

## 1. Path Variables

**What is a path variable?** A value embedded inside the URL path.

**Example:** `GET /students/1` — here, `1` is part of the URL itself.

**Spring solution:**

```java
@GetMapping("/students/{id}")
public String getStudent(@PathVariable Integer id) {
    return "Student ID = " + id;
}
```

**Request:** `GET /students/5`
**Output:** `Student ID = 5`

**What's happening?** Spring sees `@PathVariable Integer id` and says: "Take the value from the URL and place it into this parameter."

> 🧠 **Teaching line:** "Path variables capture values embedded in the URL."

#### 🛠️ Build-Along — Try a few different IDs

**Say:** "Add this endpoint to `StudentController`. Run it, and visit `/students/1`, then `/students/42`, then `/students/999` — same code, different URLs."

✅ **Expected result:** the number in the response always matches the number in the URL, with no code changes between requests.

---

## 2. Request Parameters

Also called **query parameters**.

**Example:** `GET /students?name=John` — breaks down into `/students` + `?name=John`.

**Spring solution:**

```java
@GetMapping("/students")
public String getStudentByName(@RequestParam String name) {
    return "Searching for " + name;
}
```

**Request:** `GET /students?name=Mary`
**Response:** `Searching for Mary`

> 🧠 **Teaching line:** "Request parameters are optional pieces of information attached to a URL."

**Multiple parameters:**

```
GET /students?name=John&age=20
```

```java
@GetMapping("/students")
public String searchStudent(
        @RequestParam String name,
        @RequestParam Integer age) {
    return name + " " + age;
}
```

#### 🛠️ Build-Along — Add a second parameter

**Say:** "Update the endpoint to take both `name` and `age` as shown. Run it and visit `/students?name=John&age=20`."

✅ **Expected output:** `John 20`

**Then say:** "Now visit `/students?name=John` with no `age` at all."

✅ **Expected result:** an error — by default, `@RequestParam` treats the parameter as required.

> 💬 **Say:** "We'll cover making parameters optional in a later chapter — for now, notice that a missing required parameter fails loudly, which is actually the safer default."

---

## 3. Path Variable vs Request Param

> 📌 Students confuse these constantly.

| | Path Variable | Request Param |
|---|---|---|
| Example | `/students/5` | `/students?name=John` |
| Meaning | "Give me student number 5" | "Search students using this filter" |

> 🧠 **Teaching line:** "Path variables identify resources. Request parameters filter resources."

#### 🛠️ Build-Along — Cause the mismatch on purpose

> 📌 **Instructor Note:** Directly matches "Mistake 2" in Section 7 — build it wrong here, live, before students read the correction as prose later.

**Say:** "Take your `/students/{id}` endpoint and deliberately change `@PathVariable Integer id` to `@RequestParam Integer id` — keep the URL template `/students/{id}` unchanged. Run it and visit `/students/5`."

✅ **Expected result:** an error — something like `Required request parameter 'id' is not present`. Spring is looking for `?id=5` in the query string, not `5` in the path, because the annotation told it to.

> 💬 **Say, once they see it:** "The URL still had `5` sitting right there — Spring just wasn't told to look for it in the right place. Change it back to `@PathVariable` and confirm it works again."

---

## 4. Request Body

This is **the big one** — most business applications use this constantly.

Imagine creating a student. The client sends:

```json
{
  "name": "John",
  "age": 20
}
```

**How do we receive it?**

```java
package org.codex.dto;

public class StudentRequest {

    private String name;
    private Integer age;

    public StudentRequest() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

> 📌 Notice this is a **separate class** from the `Student` entity in Chapter 11 — it has no `id`. The client shouldn't be able to set an `id`; the database assigns that. Keeping the request shape separate from the persistence model like this is a common real-world pattern, usually called a **DTO** (Data Transfer Object).

**Controller:**

```java
@PostMapping("/students")
public StudentRequest createStudent(@RequestBody StudentRequest student) {
    return student;
}
```

**Request:** `POST /students` with body:
```json
{
  "name": "John",
  "age": 20
}
```

**Response:**
```json
{
  "name": "John",
  "age": 20
}
```

**What happened?** Spring automatically:
1. Read the JSON
2. Created a `StudentRequest` object
3. Filled in its fields
4. Passed the object into the method

> 🧠 **Teaching line:** "`@RequestBody` converts incoming JSON into Java objects."

#### 🛠️ Build-Along — Send it, then try to sneak in an `id`

**Say:** "Add `StudentRequest` and the `createStudent` endpoint exactly as shown. In Postman, send a POST to `/students` with body `{\"name\": \"John\", \"age\": 20}`. Confirm the response matches."

**Then say:** "Now change the request body to include an `id`: `{\"id\": 999, \"name\": \"John\", \"age\": 20}`. Send it again."

✅ **Expected result:** the response still only shows `name` and `age` — `id` is silently ignored, since `StudentRequest` has no field for it.

> 💬 **Say:** "This is the DTO claim proven directly: it doesn't matter what the client tries to send — if the class receiving it has no `id` field, there is no way for the client to set one. The database, later, decides the real `id`."

---

## 5. Visual Summary

| Style | URL / Body | Annotation |
|---|---|---|
| Path Variable | `/students/5` | `@PathVariable Integer id` |
| Request Param | `/students?name=John` | `@RequestParam String name` |
| Request Body | `{ "name": "John" }` | `@RequestBody StudentRequest student` |

---

## 6. Real CRUD Examples

**Read one student:**
```java
@GetMapping("/students/{id}")
public Student getStudent(@PathVariable Integer id) {
    return service.getStudent(id);
}
```

**Search students:**
```java
@GetMapping("/students")
public List<Student> searchStudents(@RequestParam String name) {
    return service.searchByName(name);
}
```

**Create a student:**
```java
@PostMapping("/students")
public Student createStudent(@RequestBody StudentRequest student) {
    return service.create(student);
}
```

> 📌 Here, `service.create(student)` would convert the incoming `StudentRequest` into a `Student` entity before saving it. We'll look at exactly how that conversion works in a later chapter — for now, focus on how the data arrives.

---

## 7. Common Student Mistakes

❌ **Mistake 1 — Missing `@PathVariable`:**
```java
@GetMapping("/students/{id}")
public Student getStudent(Integer id) { // ❌ missing @PathVariable
    ...
}
```

❌ **Mistake 2 — Using `@RequestParam` for a path segment:**
```java
@GetMapping("/students/{id}")
public Student getStudent(@RequestParam Integer id) { // ❌ should be @PathVariable
    ...
}
```

> 📌 **Instructor Note:** You already built and fixed this exact mistake live in Section 3 — no need to repeat it here, just point back to it.

❌ **Mistake 3 — Forgetting `@RequestBody` on POST requests:**
```java
@PostMapping("/students")
public Student createStudent(StudentRequest student) { // ❌ missing @RequestBody
    ...
}
```

#### 🛠️ Build-Along — The silent mistake (more dangerous than an error)

> 📌 **Instructor Note:** This is worth building deliberately, because unlike Section 3's mistake, this one doesn't crash — it fails quietly, which is exactly why it's worth experiencing.

**Say:** "Make a copy of your `createStudent` endpoint — call it `createStudentBroken` — but remove `@RequestBody` from the parameter, leaving `public StudentRequest createStudentBroken(StudentRequest student)`. Map it to `POST /students/broken`. Send the same JSON body you used before: `{\"name\": \"John\", \"age\": 20}`."

✅ **Expected result:** the app does **not** crash — it responds with `{"name":null,"age":null}` (or similar), silently discarding your data.

> 💬 **Say, and make sure this lands:** "No error. No stack trace. Just wrong data, quietly. Without `@RequestBody`, Spring doesn't know to read the JSON body at all — it looks for the fields elsewhere, finds nothing, and moves on. This is more dangerous than Section 3's mistake precisely because nothing tells you it's broken. Always double-check `@RequestBody` is present on any endpoint meant to receive JSON."

---

## 8. Mini Exercise

Create these endpoints for a `Book` resource:

1. `GET /books/10` — using `@PathVariable`
2. `GET /books?author=James` — using `@RequestParam`
3. `POST /books` — using `@RequestBody`

#### 🛠️ Build-Along — Solve it independently

> 📌 **Instructor Note:** Give the spec as-is — this repeats the exact shape of Sections 1, 2, and 4 with a new resource. Step in only where genuinely needed.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex.dto;

public class BookRequest {

    private String title;
    private String author;

    public BookRequest() {
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }
}
```

```java
package org.codex;

import org.codex.dto.BookRequest;
import org.springframework.web.bind.annotation.*;

@RestController
public class BookController {

    @GetMapping("/books/{id}")
    public String getBook(@PathVariable Integer id) {
        return "Book ID = " + id;
    }

    @GetMapping("/books")
    public String searchBooks(@RequestParam String author) {
        return "Searching books by " + author;
    }

    @PostMapping("/books")
    public BookRequest createBook(@RequestBody BookRequest book) {
        return book;
    }
}
```

</details>

---

## 9. Closing Statement

## ✅ Key Takeaways

✅ `@PathVariable` — extracts a value embedded in the URL path
✅ `@RequestParam` — extracts an optional query string value (though required by default — missing one fails loudly)
✅ `@RequestBody` — converts incoming JSON into a Java object
✅ Path variables identify a specific resource; request parameters filter a collection
✅ Using the wrong annotation for path data fails loudly (`@RequestParam` instead of `@PathVariable`); forgetting `@RequestBody` fails silently, with null fields and no error — the second is more dangerous precisely because nothing warns you
✅ Keeping a separate request DTO (like `StudentRequest`) apart from your JPA entity avoids letting clients set server-controlled fields like `id` — proven directly by attempting to sneak an `id` into the request body and watching it get ignored

---

**← Previous: [Chapter 11 — Spring Data JPA & Database Integration](./chapter-11-spring-data-jpa.md)** | **Next: Chapter 13 →**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- All three data-arrival styles are now tested with real requests (browser for path variables/params, Postman for the body), not just read as code.
- Turned "Common Student Mistake 2" (wrong annotation for path data) into a live build-along in Section 3, before students ever read it as a listed mistake — they build it wrong, see a loud error, then fix it.
- Added a new build-along for "Common Student Mistake 3" (missing `@RequestBody`) that deliberately contrasts with Mistake 2: this one fails silently with null fields instead of an error, which is flagged explicitly as the more dangerous case for exactly that reason.
- Added a direct proof of the DTO security claim: students attempt to smuggle an `id` into the request body and watch it get silently dropped, since `StudentRequest` has no field for it — turns an assertion about design safety into something verified.
