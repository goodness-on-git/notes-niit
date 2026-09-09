# Chapter 13: CRUD Operations with Spring Boot

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** Sections 6–9 (Create, Read, Update, Delete) are now one continuous build-along on the **same student record**, tracked by its actual database-assigned ID — not four separate, disconnected snippets. Students create a real student, read it back, update it, confirm the update, then delete it and confirm it's gone. This is the natural conclusion of the "prove it, don't just show it" approach used since Chapter 1.

This chapter ties together almost everything covered so far. By the end, students will understand how to build a complete API that can create, read, update, and delete data — a **CRUD** API.

## 🎯 Lesson Goal

Students should understand — and have run a full create-to-delete cycle demonstrating:

- What CRUD means
- How CRUD maps to HTTP methods
- How Controller, Service, and Repository work together
- The standard structure used in real applications

## 🗺️ Lesson Roadmap

1. What Is CRUD?
2. CRUD and HTTP Methods
3. The Student Entity — **build-along, proved**
4. Repository
5. Service Layer
6. Create — **build-along (stage 1 of the full cycle)**
7. Read — **build-along (stage 2)**
8. Update — **build-along (stage 3, includes a real bug caught live)**
9. Delete — **build-along (stage 4, closes the loop)**
10. Complete CRUD Flow
11. Common Student Confusions
12. Mini Exercise — **build-along, independent**
13. Closing Statement

---

## 1. What Is CRUD?

CRUD stands for:

| Letter | Meaning |
|---|---|
| C | Create |
| R | Read |
| U | Update |
| D | Delete |

Think about a Student Management System — you need to add students, view students, edit students, and remove students. That's CRUD.

> 🧠 **Teaching line:** "Almost every business application is a CRUD application at its core."

---

## 2. CRUD and HTTP Methods

REST APIs map CRUD to HTTP methods:

| CRUD | HTTP Method |
|---|---|
| Create | POST |
| Read | GET |
| Update | PUT |
| Delete | DELETE |

> 🧠 **Teaching line:** "CRUD describes the operation; HTTP describes how we perform it."

---

## 3. The Student Entity

```java
package org.codex.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;
    private Integer age;

    public Student() {
    }

    public Student(Integer id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
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

> 📌 We're adding `@GeneratedValue` here, refining the entity from Chapter 11 — this lets the database assign IDs automatically, which is exactly what makes the `StudentRequest` DTO from Chapter 12 work cleanly for Create.

#### 🛠️ Build-Along — Add `@GeneratedValue`, prove what it changes

**Say:** "Update your `Student` entity to add `@GeneratedValue(strategy = GenerationType.IDENTITY)` above `@Id`, exactly as shown. In Chapter 11, you always passed an explicit ID yourself, like `new Student(1, \"John\", 20)`. Keep that in mind — we're about to remove that requirement entirely."

> 💬 **Say:** "Hold that thought — you'll prove this changed something in the very next section, not just read about it."

---

## 4. Repository

```java
package org.codex.repository;

import org.codex.model.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Integer> {
}
```

No implementation — Spring generates it automatically.

---

## 5. Service Layer

```java
package org.codex;

import org.codex.repository.StudentRepository;
import org.springframework.stereotype.Service;

@Service
public class StudentService {

    private final StudentRepository repository;

    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }
}
```

We'll now build the CRUD methods one at a time — and test each one before moving to the next, on the **same record**.

---

## 6. Create 🟢

**Service:**
```java
public Student createStudent(StudentRequest request) {
    Student student = new Student();
    student.setName(request.getName());
    student.setAge(request.getAge());

    return repository.save(student);
}
```

**Controller:**
```java
@PostMapping("/students")
public Student createStudent(@RequestBody StudentRequest student) {
    return service.createStudent(student);
}
```

**What happens?**
```
Controller
    ↓
Service
    ↓
Repository.save()
    ↓
Database
```

> 🧠 **Teaching line:** "`save()` inserts a new record when the ID is null — Hibernate knows there's nothing to update yet."

#### 🛠️ Build-Along — Stage 1: Create, and prove `@GeneratedValue` worked

**Say:** "Add `createStudent` to both `StudentService` and `StudentController`. In Postman, send `POST /students` with body `{\"name\": \"John\", \"age\": 20}` — notice: no `id` anywhere in that body."

**Request:**
```json
{
  "name": "John",
  "age": 20
}
```

✅ **Expected response** — note the database-assigned `id`:
```json
{
  "id": 1,
  "name": "John",
  "age": 20
}
```

> 💬 **Say, and make it explicit:** "You never sent an `id`. The database assigned one, and Hibernate handed it straight back to you in the response. That's the direct payoff of `@GeneratedValue` from Section 3 — without it, this would have failed or required you to invent an ID yourself, the way you had to in Chapter 11. **Write down the `id` you got back** — every remaining stage of this chapter reuses it."

---

## 7. Read 🔵

**Get all students:**

```java
// Service
public List<Student> getStudents() {
    return repository.findAll();
}
```

```java
// Controller
@GetMapping("/students")
public List<Student> getStudents() {
    return service.getStudents();
}
```

**Get one student:**

```java
// Service
public Student getStudent(Integer id) {
    return repository.findById(id).orElse(null);
}
```

```java
// Controller
@GetMapping("/students/{id}")
public Student getStudent(@PathVariable Integer id) {
    return service.getStudent(id);
}
```

> 🧠 **Teaching line:** "`findById()` returns an `Optional` because the record may not exist."

#### 🛠️ Build-Along — Stage 2: Read back the student you just created

**Say:** "Add both endpoints. First, `GET /students` — confirm the student from Stage 1 is in the list. Then, `GET /students/{the id you wrote down}` — confirm you get that exact same student back, individually."

> 💬 **Say:** "Same record, same data, from a completely different endpoint. This is the same object living in the database, not something regenerated each time."

**Then say:** "Try `GET /students/9999` — an ID that definitely doesn't exist."

✅ **Expected result:** an empty/null response body, not an error — because `orElse(null)` quietly returns nothing rather than failing loudly.

> 💬 **Say:** "Notice this doesn't crash — it just returns nothing, silently. Proper 'not found' error handling is a later chapter; for now, just be aware that a missing ID doesn't currently produce a clear error."

---

## 8. Update 🟡

Students usually find this surprising.

> 💬 **Say:** "Spring uses `save()` again. Wait... didn't we use `save()` for Create? Yes. Why?"

Hibernate checks: does this ID already exist?
- If **NO** → `INSERT`
- If **YES** → `UPDATE`

**Service:**
```java
public Student updateStudent(Integer id, StudentRequest request) {
    Student updatedStudent = new Student();
    updatedStudent.setId(id);
    updatedStudent.setName(request.getName());
    updatedStudent.setAge(request.getAge());

    return repository.save(updatedStudent);
}
```

**Controller:**
```java
@PutMapping("/students/{id}")
public Student updateStudent(
        @PathVariable Integer id,
        @RequestBody StudentRequest student) {

    return service.updateStudent(id, student);
}
```

> 🧠 **Teaching line:** "`save()` can both insert and update — the presence of an ID is what decides which one happens. Create lets the database assign the ID (so it's null going in); Update explicitly sets the existing ID first."

#### 🛠️ Build-Along — Stage 3: Update your student, then catch a real bug on purpose

**Say:** "Add `updateStudent` to both layers, exactly as shown. In Postman, send `PUT /students/{your id}` with body `{\"name\": \"Michael\", \"age\": 25}`. Then `GET /students/{your id}` again to confirm."

✅ **Expected result:** the student's `name` and `age` changed, but the `id` stayed the same.

> 💬 **Say:** "Same record, same ID — just updated in place. Now let's break it on purpose."

**Now say:** "In `updateStudent()`, comment out the line `updatedStudent.setId(id);` — leave everything else the same. Send the exact same `PUT /students/{your id}` request again."

**Then say:** "Run `GET /students` (the full list, not one record) and count how many students are in it."

✅ **Expected result:** a **new** student appears in the list, with a brand-new, different `id` — your original record is untouched, unedited.

> 💬 **Say, and make sure this lands:** "This is a real, easy-to-make bug: without setting the ID first, Hibernate sees a `Student` with a null ID and treats it as brand new — so `save()` does an `INSERT`, not an `UPDATE`. You didn't get an error. You got silently wrong behavior — a duplicate record instead of an edit. Put `updatedStudent.setId(id)` back and confirm updates work correctly again."

---

## 9. Delete 🔴

**Service:**
```java
public void deleteStudent(Integer id) {
    repository.deleteById(id);
}
```

**Controller:**
```java
@DeleteMapping("/students/{id}")
public void deleteStudent(@PathVariable Integer id) {
    service.deleteStudent(id);
}
```

**What happens?**
```sql
DELETE FROM student WHERE id = 1;
```

> 🧠 **Teaching line:** "`deleteById()` removes a row using its primary key."

#### 🛠️ Build-Along — Stage 4: Close the loop

**Say:** "Add `deleteStudent`. Send `DELETE /students/{your id}`. Then `GET /students/{your id}` one final time."

✅ **Expected result:** the record from Stage 1 is gone — the GET now returns empty/null.

> 💬 **Say:** "You just took one record through its entire life: created it, read it, updated it, watched a bug corrupt it, fixed the bug, and finally deleted it — all through the same four endpoints a real application uses, on the same piece of data the whole way through."

---

## 10. Complete CRUD Flow

| Operation | Request |
|---|---|
| Create | `POST /students` |
| Read All | `GET /students` |
| Read One | `GET /students/1` |
| Update | `PUT /students/1` |
| Delete | `DELETE /students/1` |

**Visual flow** — every CRUD operation follows this same path:

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

---

## 11. Common Student Confusions

> ❓ "Why does `save()` do both Create and Update?"
> 👉 Because JPA checks whether the entity's ID already exists in the database.

> ❓ "Why use PUT for Update?"
> 👉 Because REST conventions define PUT as the standard update operation.

> ❓ "Why use `@PathVariable` for Update/Delete?"
> 👉 Because you need to identify exactly which resource you're modifying.

> ❓ "Why does Create's response include an `id` the request never sent?"
> 👉 Because `@GeneratedValue` lets the database assign it. Hibernate fills it in during `save()`, and the now-complete entity — including its new `id` — is what gets returned. You proved this directly in Stage 1.

---

## 12. Mini Exercise

Build a full CRUD API for **Books**.

Entity fields: `id`, `title`, `author`

Create endpoints:
- `POST /books`
- `GET /books`
- `GET /books/{id}`
- `PUT /books/{id}`
- `DELETE /books/{id}`

#### 🛠️ Build-Along — Solve it independently, full cycle

> 📌 **Instructor Note:** Give the spec as-is. Have students run the same create → read → update → delete cycle on their own `Book` data that you just walked them through for `Student` — no staged prompts needed at this point.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String author;

    public Book() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
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
package org.codex.repository;

import org.codex.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
}
```

```java
package org.codex;

import org.codex.dto.BookRequest;
import org.codex.model.Book;
import org.codex.repository.BookRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class BookService {

    private final BookRepository repository;

    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public Book createBook(BookRequest request) {
        Book book = new Book();
        book.setTitle(request.getTitle());
        book.setAuthor(request.getAuthor());
        return repository.save(book);
    }

    public List<Book> getBooks() {
        return repository.findAll();
    }

    public Book getBook(Long id) {
        return repository.findById(id).orElse(null);
    }

    public Book updateBook(Long id, BookRequest request) {
        Book book = new Book();
        book.setId(id);
        book.setTitle(request.getTitle());
        book.setAuthor(request.getAuthor());
        return repository.save(book);
    }

    public void deleteBook(Long id) {
        repository.deleteById(id);
    }
}
```

```java
package org.codex;

import org.codex.dto.BookRequest;
import org.codex.model.Book;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class BookController {

    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @PostMapping("/books")
    public Book createBook(@RequestBody BookRequest book) {
        return service.createBook(book);
    }

    @GetMapping("/books")
    public List<Book> getBooks() {
        return service.getBooks();
    }

    @GetMapping("/books/{id}")
    public Book getBook(@PathVariable Long id) {
        return service.getBook(id);
    }

    @PutMapping("/books/{id}")
    public Book updateBook(@PathVariable Long id, @RequestBody BookRequest book) {
        return service.updateBook(id, book);
    }

    @DeleteMapping("/books/{id}")
    public void deleteBook(@PathVariable Long id) {
        service.deleteBook(id);
    }
}
```

</details>

---

## 13. Closing Statement

## ✅ Key Takeaways

✅ CRUD = Create, Read, Update, Delete — maps to POST, GET, PUT, DELETE
✅ `save()` handles both insert and update — the entity's ID is what decides which one happens
✅ Forgetting to set the ID before an update doesn't error — it silently creates a duplicate record instead. You caused and caught this bug directly.
✅ `findAll()`, `findById()`, and `deleteById()` come free from `JpaRepository`
✅ A clean request DTO (no `id`) plus `@GeneratedValue` on the entity keeps clients from setting server-controlled fields
✅ Every CRUD operation follows the same flow: Controller → Service → Repository → Database

> 🧠 "CRUD APIs allow clients to create, retrieve, update, and delete resources through standardized HTTP operations."

---

**← Previous: [Chapter 12 — Handling Request Data in Spring Boot](./chapter-12-handling-request-data.md)**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- Restructured Create/Read/Update/Delete from four independent, disconnected snippets into one continuous build-along on a single tracked record — students carry one real `id` through the entire chapter, which makes the "same data, different operations" structure of a CRUD API tangible rather than something read four separate times.
- `@GeneratedValue` is now proven, not just mentioned: students explicitly omit `id` from the Create request and watch the database assign one, directly contrasted against the Chapter 11 requirement to invent an ID manually.
- Added a live, caused-on-purpose version of a genuinely common real bug: forgetting to set the ID before calling `save()` on Update. This is arguably the most valuable moment in the chapter — it's a silent failure (duplicate record, no error) rather than a crash, which is exactly the kind of mistake worth experiencing directly rather than reading about.
- The `orElse(null)` "not found" behavior is now flagged explicitly during the Read build-along, with a forward pointer to proper error handling, rather than left as an unremarked detail.
