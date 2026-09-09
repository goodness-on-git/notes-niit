# Chapter 11: Spring Data JPA & Database Integration

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** This chapter opens with a claim — "databases outlive the application" — that every prior chapter's hardcoded `List.of(...)` data violates. That claim is now proven, not just stated: students restart the app partway through and watch their data survive. The Repository section also now explicitly calls back to the hand-written `StudentRepository` from Chapter 10, which only had one method — that's the motivation for everything `JpaRepository` gives for free.

Every example so far has used hardcoded `List.of(...)` data that resets every time the app restarts. This chapter replaces that with a real database, using Spring Data JPA to avoid writing SQL by hand.

## 🎯 Lesson Objective

Students should understand — and have run code demonstrating:

- Why databases are needed — proven, not just claimed
- What JPA is
- What an Entity is
- How Spring talks to a database
- How repositories become powerful — and how much manual work that replaces

## 🧰 Quick Setup (MySQL) — Build-Along

> 📌 **Instructor Note:** This requires MySQL actually installed and running — confirm that in advance of class, or walk through installation together if needed. This is a genuinely new piece of infrastructure, unlike most setups since Chapter 3.

**Chunk 1 — say:** "Add these two dependencies to `pom.xml`: `spring-boot-starter-data-jpa`, and the MySQL connector, `mysql-connector-j`."

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

**Chunk 2 — say:** "In MySQL Workbench, the CLI, or any client you have, create a database."

```sql
CREATE DATABASE codex_school;
```

**Chunk 3 — say:** "In `src/main/resources/application.properties`, add your connection details."

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/codex_school
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

> 📌 `ddl-auto=update` lets Hibernate create and update tables automatically based on your `@Entity` classes. It's convenient for learning — in a real production app, you'd usually manage schema changes more carefully (e.g. with migration tools like Flyway).
>
> 💬 **Say:** "`show-sql=true` matters — leave it on all chapter. You're about to see something you've never seen before: the actual SQL Hibernate writes on your behalf."

## 🗺️ Lesson Roadmap

1. Why Databases
2. What Is a Database?
3. What Is JPA?
4. What Is Hibernate?
5. Entity — **build-along**
6. Repository Revolution — **build-along, tied to Chapter 10**
7. What Do We Get For Free? — **build-along, SQL made visible**
8. Prove It: Restart the App — **build-along (the payoff)**
9. Understanding the Generics
10. Mini Exercise — **build-along, independent**
11. Closing Statement

---

## 1. Why Databases

Databases exist because data stored in variables is lost the moment the application stops running.

> 🧠 **Say:** "Variables live in memory. Databases live beyond the application's lifetime. Every list you've hardcoded since Chapter 9 has reset every single time you restarted the app — you just haven't had a reason to notice, because you always restarted right before checking. Today we fix that, and I'll prove it to you directly."

---

## 2. What Is a Database?

A database is a system designed to store, organize, and retrieve data.

**Popular databases:**
- MySQL
- PostgreSQL
- Oracle Database
- MongoDB

---

## 3. What Is JPA?

❌ JPA is **NOT** a database
❌ JPA is **NOT** Spring
✅ JPA — **Java Persistence API** — is a specification (a set of rules) for mapping Java objects to database tables

> 🧠 **Say:** "JPA defines the rules. Hibernate does the work."

---

## 4. What Is Hibernate?

Hibernate is an intermediary between Spring and the database.

**Where does Hibernate fit?**

```
Application
     ↓
Spring Data JPA
     ↓
Hibernate
     ↓
Database
```

> 🧠 **Teaching line:** "Spring talks to Hibernate. Hibernate talks to the database."

---

## 5. Entity

An Entity represents a database table.

```java
package org.codex.model;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
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

> 📌 This `Student` supersedes the plain POJO version from Chapter 9 — from here on, this JPA-backed version is what your repository, service, and controller layers work with.

#### 🛠️ Build-Along — Create the Entity

**Say:** "Replace your Chapter 9 `Student` class with this version — same name, but now `@Entity` and `@Id` are added, and there's a no-args constructor alongside the existing one. Don't run anything yet."

**What does this create?**

| id | name | age |
|---|---|---|
| 1 | John | 20 |
| 2 | Mary | 22 |

> 🧠 **Say:** "An Entity is a Java representation of a database table."

---

## 6. Repository Revolution

A Repository is a component responsible for accessing and managing data — retrieving, storing, updating, and deleting it from a data source. It acts as a bridge between the application's business logic and the underlying storage system.

The data source could be:
- A database
- A file
- An API
- In-memory data

**Before** — we manually wrote every method:
```java
@Repository
public class StudentRepository {
}
```

**Now:**
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository
        extends JpaRepository<Student, Integer> {
}
```

No implementation. No SQL. No code.

> 🧠 **Say:** "Yes. Spring generates the implementation automatically."

#### 🛠️ Build-Along — Feel the contrast with Chapter 10

> 📌 **Instructor Note:** This is the motivation moment — make it concrete before showing the fix.

**Say:** "Think back to `StudentRepository` in Chapter 10. It had exactly one method — `findAll()` — returning a hardcoded list, and you wrote every line of it by hand. If I asked you to also hand-write `save()`, `findById()`, `deleteById()`, `count()`, and `existsById()` — each one talking to a real database with real SQL — how much code do you think that would take?"

> 🧠 **Let them guess, then reveal:** "Dozens of lines, minimum, per method, done correctly. Now delete your Chapter 10 `StudentRepository` class entirely, and replace it with this one-line interface."

```java
public interface StudentRepository extends JpaRepository<Student, Integer> {
}
```

> 💬 **Say:** "That single line just gave you everything the manual version would have taken hours to write — and more."

---

## 7. What Do We Get For Free?

By extending `JpaRepository<Student, Integer>`, we automatically get:
- `save()`
- `findAll()`
- `findById()`
- `deleteById()`
- `count()`
- `existsById()`

**Example:**
```java
studentRepository.findAll();
studentRepository.save(student);
studentRepository.deleteById(1);
```

> 🧠 **Say:** "Spring writes the CRUD code so you don't have to."

#### 🛠️ Build-Along — Save real data, and watch the real SQL

**Say:** "Create a `CommandLineRunner` (or reuse an existing one) that injects `StudentRepository` via constructor, then calls `save()` twice with two different students, then calls `findAll()` and prints each one. Run the app and watch the console closely, not just the printed names."

```java
@Component
public class StudentRunner implements CommandLineRunner {

    private final StudentRepository studentRepository;

    public StudentRunner(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    @Override
    public void run(String... args) {
        studentRepository.save(new Student(1, "John", 20));
        studentRepository.save(new Student(2, "Mary", 22));

        studentRepository.findAll().forEach(s ->
                System.out.println(s.getName() + " - " + s.getAge())
        );
    }
}
```

✅ **Expected console output includes:** real `insert into student (...)` and `select ... from student` SQL statements, printed above your `John - 20` / `Mary - 22` lines — this is `show-sql=true` at work.

> 💬 **Say:** "You never wrote a single line of SQL. That's Hibernate, generating and running real SQL on your behalf, from nothing but the method calls you made and the `@Entity` you defined."

---

## 8. Prove It: Restart the App

> 📌 **Instructor Note:** This is the single most important moment in the chapter — it's the direct payoff of Section 1's claim. Don't skip it, even if time is tight.

#### 🛠️ Build-Along — Watch the data survive a restart

**Say:** "Stop the application completely. Now comment out — don't delete, just comment out — both `save()` calls in your runner, leaving only the `findAll()` and the print statements. Run the app again."

✅ **Expected output:** the exact same two students print — `John - 20` and `Mary - 22` — even though this run of the app never called `save()` at all.

> 💬 **Say, and let it land:** "Go back to Chapter 9 or 10 in your head. If you'd done this with the hardcoded `List.of(...)` version, and I told you to comment out the list itself, what would `findAll()` return? Nothing — because that data only ever existed in memory, for the lifetime of that one run. This time, the data is sitting in MySQL, completely independent of whether your Java application is even running. That's the entire point of this chapter, and you just proved it yourself instead of taking my word for it."

---

## 9. Understanding the Generics

`JpaRepository<Student, Integer>` — what does this mean?

- **First parameter** — `Student` → the entity type
- **Second parameter** — `Integer` → the primary key type

**Example:**
```java
@Entity
public class Book {
    @Id
    private Long id;
}
```

A repository for this would be `JpaRepository<Book, Long>`.

> 🧠 **Say:** "The first generic is the table. The second generic is the ID type."
>
> "Spring Data JPA lets us work with Java objects while Hibernate handles the SQL behind the scenes."

---

## 10. Mini Exercise

1. Create a `Book` entity (`@Entity`) with `id`, `title`, and `author` fields
2. Create a `BookRepository` interface extending `JpaRepository<Book, Long>`
3. Use a `CommandLineRunner` to save two books and print all books from the repository
4. Run it, then check MySQL — a `book` table should now exist with your data

#### 🛠️ Build-Along — Solve it independently, then repeat the restart proof

> 📌 **Instructor Note:** Give the spec as-is. Once it's working, have students repeat the Section 8 restart test on their own `Book` data — a second, independent confirmation of the same lesson, this time without you walking them through it.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex.model;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Book {

    @Id
    private Long id;
    private String title;
    private String author;

    public Book() {
    }

    public Book(Long id, String title, String author) {
        this.id = id;
        this.title = title;
        this.author = author;
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
package org.codex.repository;

import org.codex.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
}
```

```java
package org.codex;

import org.codex.model.Book;
import org.codex.repository.BookRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class BookRunner implements CommandLineRunner {

    private final BookRepository bookRepository;

    public BookRunner(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    @Override
    public void run(String... args) {
        bookRepository.save(new Book(1L, "Clean Code", "Robert C. Martin"));
        bookRepository.save(new Book(2L, "Effective Java", "Joshua Bloch"));

        bookRepository.findAll().forEach(book ->
                System.out.println(book.getTitle() + " by " + book.getAuthor())
        );
    }
}
```

✅ **Output:**
```
Clean Code by Robert C. Martin
Effective Java by Joshua Bloch
```

Check MySQL afterward — Hibernate will have created a `book` table for you automatically, thanks to `ddl-auto=update`.

</details>

---

## 11. Closing Statement

## ✅ Key Takeaways

- Databases outlive the application — variables in memory don't, and you proved this directly by restarting the app and watching the data survive
- JPA is a specification, not a database or a framework — Hibernate is what actually implements it
- An `@Entity` class maps directly to a database table; each field becomes a column
- Extending `JpaRepository<Entity, IdType>` gives you `save()`, `findAll()`, `findById()`, `deleteById()`, and more — with zero implementation code, replacing what would otherwise be a large amount of hand-written CRUD logic like the single-method `StudentRepository` from Chapter 10
- The repository's generic parameters are the entity type and its primary key type, in that order
- `show-sql=true` reveals the real SQL Hibernate generates — the abstraction is real, but it's not a black box

---

**← Previous: [Chapter 10 — Layered Architecture](./chapter-10-layered-architecture.md)** | **Next: Chapter 12 →**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- Added a dedicated "Prove It: Restart the App" section (new Section 8) — the chapter opened with a claim about data outliving the application but never demonstrated it. This is now the structural centerpiece of the chapter, directly paying off Section 1.
- Tied the Repository Revolution section explicitly back to Chapter 10's hand-written `StudentRepository`, which only had one method — this reframes `JpaRepository` from "here's a new interface" into "here's what that manual effort would have scaled to, avoided."
- Made `show-sql=true` an active teaching tool rather than a config line mentioned once — students are told to watch for real generated SQL as proof that Hibernate is doing real work, not hiding it.
- Mini Exercise now explicitly asks students to repeat the restart proof independently on their own `Book` data, reinforcing the chapter's central lesson a second time without instructor guidance.
