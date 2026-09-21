# Chapter 14: Validation

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** This chapter opens by claiming Spring "happily saves garbage data" — students now **prove that first**, by saving a student with an empty name and a negative age into their real database, before any fix is introduced. Every "common mistake" in Section 11 is also now a live experiment rather than a prose warning. The chapter's structure becomes: see the problem for real → fix it → verify the fix → verify what happens when you forget a piece of it.

So far, nothing has stopped a client from sending garbage data — an empty name, a negative age, a malformed email — straight into the database. This chapter fixes that with Bean Validation.

## 🎯 Lesson Goal

Students should understand — and have run requests demonstrating:

- Why validation is necessary — proven with real bad data in a real database
- How to validate incoming requests
- Bean Validation annotations
- `@Valid`, and what happens when it's missing
- How validation prevents bad data

## 🧰 Quick Setup — Build-Along

> 📌 Bean Validation annotations (`@NotBlank`, `@Email`, `@Min`, etc.) and `@Valid` need the `spring-boot-starter-validation` dependency — it's **not** included with `spring-boot-starter-web` by default.

**Say:** "Add this to `pom.xml`, then reload your Maven dependencies. Without it, none of today's annotations will even be available to import."

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 🗺️ Lesson Roadmap

1. Start With the Problem — **build-along (save real garbage first)**
2. What Is Validation?
3. The Student Entity (Recap)
4. Bean Validation Annotations
5. Building a Validated DTO — **build-along**
6. Why Use a DTO?
7. The Magic Annotation: `@Valid` — **build-along (forget it first, on purpose)**
8. What Does the Error Response Look Like? — **build-along**
9. Why Validation Matters
10. Common Validation Annotations
11. Common Student Mistakes — **build-along (the `@NotNull` trap)**
12. Mini Exercise — **build-along, independent**
13. Closing Statement

---

## 1. Start With the Problem

Suppose your API allows this:

```json
{
  "name": "",
  "age": -50
}
```

or this:

```json
{
  "name": null,
  "age": 999
}
```

Without validation:

```java
repository.save(student);
```

Spring happily saves garbage data.

#### 🛠️ Build-Along — Don't take my word for it: save the garbage

> 📌 **Instructor Note:** Do this before introducing a single validation annotation. The fix means nothing until the problem is real to them.

**Say:** "Using your working CRUD API from Chapter 13, send `POST /students` with this body: `{\"name\": \"\", \"age\": -50}`. Then run `GET /students` and look at the list. Then open MySQL directly and look at the `student` table."

✅ **Expected result:** the request succeeds with a `200`, a real row is created with a real database-assigned ID, and a student with an empty name and an age of negative fifty is now permanently sitting in the database.

> 💬 **Say, and let it sit:** "Nothing stopped that. No error, no warning — your application cheerfully accepted a person with no name who is minus fifty years old, and wrote it to disk. Now imagine that endpoint is public and gets ten thousand requests a day. **Leave that row there for now** — we're going to come back and try to insert it again at the end of the lesson, after we've built the fix."

> 🧠 **Teaching line:** "Validation protects your application from bad input."

---

## 2. What Is Validation?

**Definition:** Validation is the process of checking whether incoming data satisfies predefined rules before it is processed or stored.

**Real-world analogy:** Think about a university admission form. Rules might be:
- Name cannot be empty
- Age must be positive
- Email must be valid

If the form violates the rules: ❌ rejected. Spring validation works the same way.

---

## 3. The Student Entity (Recap)

> 📌 Simplified here to just the fields that matter for this lesson — the real entity (Chapters 11–13) also has `id` and `@GeneratedValue`.

```java
public class Student {
    private String name;
    private Integer age;
}
```

Currently, this is accepted with no complaints — as you just proved in Section 1.

Let's fix that.

---

## 4. Bean Validation Annotations

These annotations define rules.

### `@NotNull`

```java
@NotNull
private String name;
```

Means: this field cannot be null.

✅ Valid: `{ "name": "John" }`
❌ Invalid: `{ "name": null }`

### `@NotBlank`

```java
@NotBlank
private String name;
```

Means: cannot be null, empty, or only spaces.

❌ Invalid: `{ "name": "" }`
❌ Invalid: `{ "name": "     " }`

> 🧠 **Teaching line:** "`@NotNull` checks existence. `@NotBlank` checks usefulness."
>
> 💬 **Say:** "Flag this distinction now — you'll test it directly in Section 11, and it's the single most common validation mistake beginners make."

### `@Size`

```java
@Size(min = 2, max = 50)
private String name;
```

✅ Valid: `John`
❌ Invalid: `J`

### `@Min`

```java
@Min(1)
private Integer age;
```

Means: `age >= 1`.

❌ Invalid: `{ "age": -5 }`

### `@Max`

```java
@Max(120)
private Integer age;
```

Means: `age <= 120`.

### `@Email`

```java
@Email
private String email;
```

✅ Valid: `john@gmail.com`
❌ Invalid: `john.com`

---

## 5. Building a Validated DTO

```java
package org.codex.dto;

import jakarta.validation.constraints.*;

public class StudentRequest {

    @NotBlank
    @Size(min = 2, max = 50)
    private String name;

    @Min(1)
    @Max(120)
    private Integer age;

    @Email
    private String email;

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

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

> 📌 This refines the `StudentRequest` from Chapters 12–13, adding an `email` field. The `Student` entity gains a matching field too, so this is wired all the way through to persistence:
>
> ```java
> // Student.java — add alongside the existing id, name, age fields
> private String email;
>
> public String getEmail() {
>     return email;
> }
>
> public void setEmail(String email) {
>     this.email = email;
> }
> ```
>
> And in `StudentService`, both `createStudent()` and `updateStudent()` now also call `student.setEmail(request.getEmail())` when building the entity.

#### 🛠️ Build-Along — Add the rules (chunked)

**Chunk 1 — say:** "Update `StudentRequest` with the validation annotations exactly as shown, and add the `email` field."

**Chunk 2 — say:** "Add the matching `email` field, getter, and setter to your `Student` entity, and update both `createStudent()` and `updateStudent()` in `StudentService` to call `student.setEmail(request.getEmail())`."

> 💬 **Say:** "Don't test it yet — there's one piece still missing, and I want you to discover what it is."

---

## 6. Why Use a DTO?

Instead of validating the entity directly, we validate the incoming request shape.

> 🧠 **Teaching line:** "DTOs represent incoming request data."

*(We'll discuss DTOs vs entities more later.)*

---

## 7. The Magic Annotation: `@Valid`

This is where students usually get confused.

```java
@PostMapping("/students")
public Student createStudent(@RequestBody StudentRequest request) {
    return service.create(request);
}
```

**Will validation happen?** ❌ No.

We need:

```java
@PostMapping("/students")
public Student createStudent(@Valid @RequestBody StudentRequest request) {
    return service.create(request);
}
```

Now Spring checks `@NotBlank`, `@Email`, `@Min`, etc. **before** the method body even runs.

> 🧠 **Teaching line:** "`@Valid` tells Spring to enforce the validation rules."

**Visual flow:**

```
Request
   ↓
Validation
   ↓
 Valid?
 /    \
Yes    No
 ↓      ↓
Controller   Error Response
```

#### 🛠️ Build-Along — Discover the missing piece yourself

> 📌 **Instructor Note:** This is the chapter's key "predict, then find out" moment. Do NOT tell them about `@Valid` before running this.

**Say:** "You've just added six validation annotations to `StudentRequest`. Your controller is unchanged from Chapter 13. Predict: if you send `{\"name\": \"\", \"age\": -5, \"email\": \"wrong-email\"}` right now, will it be rejected?"

> 🧠 **Let them predict — most will say yes.**

**Then say:** "Send it."

✅ **Expected result:** it saves successfully. Another garbage row lands in your database, exactly like Section 1 — despite every one of those annotations being present.

> 💬 **Say:** "Six annotations, completely ignored. Annotations by themselves are just labels — something has to actually *run* the check. That something is `@Valid`."

**Now say:** "Add `@Valid` before `@RequestBody` in your controller method. Send the identical request again."

✅ **Expected result:** a `400 Bad Request` — and the controller method never runs.

> 💬 **Say:** "One annotation was the entire difference between 'garbage saved silently' and 'rejected before your code even executed.' This is why 'forgetting `@Valid`' is on the common mistakes list — it fails silently, in the most dangerous possible direction."

---

## 8. What Does the Error Response Look Like?

For the rejected request above, Spring returns a `400 Bad Request` automatically, with a body describing what failed:

```json
{
  "status": 400,
  "errors": [
    "name: must not be blank",
    "age: must be greater than or equal to 1",
    "email: must be a well-formed email address"
  ]
}
```

> 📌 The exact shape of this response varies depending on Spring Boot version and whether you've customized error handling — the important part is that it's a `400`, and the controller method body never executes.

#### 🛠️ Build-Along — Read your actual error response

**Say:** "Look closely at the response body Postman showed you for that rejected request, and at your application console. Find the part that names *which* fields failed and *why*."

> 💬 **Say:** "Notice it reports all three failures at once, not just the first one it hit. Your response shape may not exactly match the example above — that depends on your Spring Boot version — but the `400` status and the field-by-field reasons will be there."

**Then say:** "Now send a request that fixes only the email, leaving name and age broken. Watch the error list shrink."

---

## 9. Why Validation Matters

Imagine, without validation:

```json
{
  "name": "",
  "age": -500
}
```

...saved straight into the database. Now reports, analytics, and business logic all become unreliable.

> 💬 **Say:** "You don't have to imagine it — go look at your `student` table right now. That row from Section 1 is still there, and validation can't retroactively remove it. That's the real lesson: validation stops bad data at the door, but it does nothing about bad data already inside."

> 🧠 **Teaching line:** "Bad data is harder to fix than bad code."

---

## 10. Common Validation Annotations

| Annotation | Purpose |
|---|---|
| `@NotNull` | Must exist |
| `@NotBlank` | Must contain text |
| `@Size` | Length limits |
| `@Min` | Minimum value |
| `@Max` | Maximum value |
| `@Email` | Valid email format |

---

## 11. Common Student Mistakes

❌ **Mistake 1 — Using `@NotNull` when they really need `@NotBlank`**
An empty string `""` passes `@NotNull` — it's not null, just useless.

❌ **Mistake 2 — Forgetting `@Valid`**
Without it, all the validation annotations on the DTO are completely ignored.

> 📌 **Instructor Note:** You already proved Mistake 2 live in Section 7 — just point back to it.

❌ **Mistake 3 — Thinking validation happens in the database**
Validation occurs *before* data ever reaches the business logic — the database never even sees invalid data.

#### 🛠️ Build-Along — Prove the `@NotNull` trap

**Say:** "Temporarily change `@NotBlank` on `name` to `@NotNull`, and remove the `@Size` annotation from it too. Send `{\"name\": \"\", \"age\": 20, \"email\": \"john@gmail.com\"}`."

✅ **Expected result:** it **passes** validation and saves — an empty name gets through, because an empty string is not null.

> 💬 **Say:** "`@NotNull` was satisfied. The field existed. It just contained nothing useful. This is exactly why the distinction matters — 'the field is present' and 'the field has a real value' are two completely different questions. Change it back to `@NotBlank` and confirm the same request now gets rejected."

---

## 12. Mini Exercise

Create a `BookRequest` DTO with rules:
- Title required
- Author required
- Pages: minimum 1, maximum 5000

Then build `POST /books` using `@Valid`.

#### 🛠️ Build-Along — Solve it independently, then try to break it

> 📌 **Instructor Note:** Give the spec as-is. Once it works, have students deliberately send invalid data (blank title, 10000 pages) and confirm the `400` — the fix isn't verified until they've tried to break it.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex.dto;

import jakarta.validation.constraints.*;

public class BookRequest {

    @NotBlank
    private String title;

    @NotBlank
    private String author;

    @Min(1)
    @Max(5000)
    private Integer pages;

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

    public Integer getPages() {
        return pages;
    }

    public void setPages(Integer pages) {
        this.pages = pages;
    }
}
```

```java
@PostMapping("/books")
public Book createBook(@Valid @RequestBody BookRequest book) {
    return service.createBook(book);
}
```

A request with `{ "title": "", "author": "Joshua Bloch", "pages": 10000 }` gets rejected with a `400` before `createBook()` ever runs — both the blank title and the out-of-range page count fail validation.

</details>

---

## 13. Closing Statement

## ✅ Key Takeaways

- Validation checks incoming data against rules *before* it's processed or stored
- `@NotNull`, `@NotBlank`, `@Size`, `@Min`, `@Max`, and `@Email` define those rules on a DTO's fields
- `@Valid` is what actually activates the checks — without it, the annotations do nothing at all, silently. You proved this by sending the same bad request twice, with and without it.
- `@NotNull` only checks that a field exists; `@NotBlank` checks it contains something useful — an empty string passes the first and fails the second
- Failed validation returns a `400 Bad Request` automatically, listing every failed field at once; the controller method body never runs
- Validating a DTO instead of the entity keeps request-shape rules separate from your persistence model
- Validation protects against *incoming* bad data — it can't clean up bad data already in the database

---

**← Previous: [Chapter 13 — CRUD Operations with Spring Boot](./chapter-13-crud-operations.md)**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- The chapter now opens by having students actually save garbage data into their real database (Section 1) before any fix is introduced — the original asserted "Spring happily saves garbage data" but never demonstrated it. The row is deliberately left in place and referenced again in Section 9, which turns "bad data is harder to fix than bad code" from a slogan into something visible in their own MySQL table.
- Restructured Section 7 into a predict-then-discover moment: students add all six validation annotations, predict that bad data will now be rejected, watch it save anyway, and only then learn about `@Valid`. This is far stronger than presenting `@Valid` as one more annotation in a list.
- Added a live demonstration of the `@NotNull` vs `@NotBlank` trap (Section 11) — students swap the annotation, watch an empty name sail through, then swap it back. This is the most common validation mistake and previously existed only as a one-line prose warning.
- Added a build-along around reading the actual `400` response, including fixing one field at a time to watch the error list shrink — makes the error structure concrete rather than a sample JSON block they never see for real.
