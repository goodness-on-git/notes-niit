# Chapter 9: Building Your First REST API with Spring Boot

> 📌 **Note on this file:** Blockquoted lines like this one are instructor-only teaching cues. This file is your delivery script: you dictate/describe, students write notes and build the code live as you guide them.

> 🆕 **Revision note (not for dictation):** Same project as Chapter 8, one new dependency. Every endpoint below is a build-along — students hit real URLs in a real browser/Postman and see real responses, not just read code. Scaffolding stays chunked and mostly independent, consistent with where the course is by now.

Chapter 8 got a Spring Boot app running internally. Now we expose that app to the outside world: a REST API that other systems — a frontend, a mobile app, Postman — can actually talk to over HTTP.

## 🎯 Lesson Objective

Students should understand — and have built and tested endpoints demonstrating:

- What a REST API is and why it's needed
- How to use `@RestController` and `@GetMapping` to expose endpoints
- How Java objects get automatically serialized to JSON
- How to return lists of objects as JSON arrays
- The main HTTP methods (GET, POST, PUT, DELETE)
- How to test an API with a browser and with Postman

## 🧰 Quick Setup — Build-Along

**Say:** "This chapter needs the **Spring Web** dependency, which Chapter 8 didn't require. Add `spring-boot-starter-web` to your `pom.xml` — or regenerate the project from Spring Initializr with 'Spring Web' checked. Without it, `@RestController` won't exist, and there's no embedded server to hit at `localhost:8080`."

> 💬 **Say, once it's added:** "In Chapter 8, `CommandLineRunner` ran your code once, automatically, at startup. Today's shift: your code runs every time someone sends a request to a URL — could be once, could be a thousand times. Same container, same beans, different trigger."

## 🗺️ Lesson Roadmap

1. Start With This Question
2. What Is a REST API?
3. The Main Annotation (`@RestController`) — **build-along**
4. Your First Endpoint (`@GetMapping`) — **build-along**
5. Returning Objects — **build-along**
6. Return JSON — **build-along, proved**
7. Returning Lists — **build-along**
8. HTTP Methods
9. Intro to POST Requests — **build-along**
10. Testing APIs
11. Mini Exercise — **build-along, independent**
12. Closing Statement

---

## 1. Start With This Question

> 💬 **Ask:** "How does a frontend or mobile app communicate with a backend?"

✅ **Answer:** Through APIs.

> 🧠 **Say:** "An API is a bridge between systems."

---

## 2. What Is a REST API?

*(Representational State Transfer Application Programming Interface)*

> 🧠 **Say:** "A REST API allows applications to communicate using HTTP requests."

**Real example:**

📱 Frontend asks:
```
GET /students
```

🖥️ Backend responds:
```json
[
  {
    "name": "John"
  }
]
```

> 🧠 **Teaching line:** "REST APIs expose data through URLs."

---

## 3. The Main Annotation

✅ `@RestController`

```java
package org.codex;

import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentController {
}
```

**What it means:** "This class handles web requests and returns data."

> 🧠 **Say:** "`@RestController` turns a class into an API controller."

#### 🛠️ Build-Along — Create it (routine at this point)

**Say:** "Create `StudentController`, marked `@RestController`, empty for now."

---

## 4. Your First Endpoint

✅ `@GetMapping`

```java
package org.codex;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Student";
    }
}
```

**What's happening?** When a browser visits `http://localhost:8080/hello`, Spring:
1. Receives the request
2. Finds the matching endpoint
3. Runs the method
4. Returns the response

#### 🛠️ Build-Along — Hit a real URL for the first time

**Say:** "Add the `hello()` method exactly as shown. Run `Main`. Open a browser and go to `http://localhost:8080/hello`."

✅ **Expected result:** the page displays `Hello Student`.

> 💬 **Say:** "That page didn't come from a file — it came from a Java method running live on your machine, right now, because you visited that URL. Refresh it a few times. Every refresh runs `hello()` again."

> 🧠 **Say:** "URLs are mapped to Java methods."

**What is `localhost:8080`?**

| Part | Meaning |
|---|---|
| `localhost` | Your own computer |
| `8080` | The server port |

---

## 5. Returning Objects (Very Important)

```java
package org.codex;

public class Student {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

#### 🛠️ Build-Along — Create the `Student` class

**Say:** "Create `Student` exactly as shown — two fields, a constructor, and getters for both. No `@Component`, no annotations at all — this is a plain data class."

---

## 6. Return JSON

```java
package org.codex;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentController {

    @GetMapping("/student")
    public Student getStudent() {
        return new Student("John", 20);
    }
}
```

✅ **Response:**
```json
{
  "name": "John",
  "age": 20
}
```

### Important Magic

You returned a `Student`. The browser got JSON.

> ❓ **Why?**
> 👉 Spring Boot automatically converts Java objects into JSON.

#### 🛠️ Build-Along — See the conversion happen, and prove it's real JSON

**Say:** "Add the `getStudent()` endpoint. Run it, and visit `http://localhost:8080/student`."

✅ **Expected result:** the browser shows `{"name":"John","age":20}`.

> 💬 **Say, and push a little further:** "Your `Student` class has no `toString()`, no JSON logic, nothing you wrote to make this happen. Add a third field to `Student` — say, `String school` — with a getter, and update the constructor call to pass a value for it. Run it again, no other changes. Watch a third field appear in the response automatically."

> 🧠 **Say:** "Spring Boot serializes Java objects into JSON automatically — and it's genuinely automatic, not something that only works for `name` and `age` specifically."

---

## 7. Returning Lists

```java
@GetMapping("/students")
public List<Student> getStudents() {
    return List.of(
            new Student("John", 20),
            new Student("Mary", 22)
    );
}
```

✅ **Response:**
```json
[
  {
    "name": "John",
    "age": 20
  },
  {
    "name": "Mary",
    "age": 22
  }
]
```

#### 🛠️ Build-Along — Return a list

**Say:** "Add `getStudents()` to `StudentController`, returning a `List<Student>` with at least two students. Visit `http://localhost:8080/students`."

✅ **Expected result:** a JSON array with one object per student.

---

## 8. HTTP Methods

| Method | Purpose |
|---|---|
| `GET` | Fetch data |
| `POST` | Create data |
| `PUT` | Update data |
| `DELETE` | Remove data |

> 🧠 **Teaching line:** "REST APIs use HTTP verbs to describe actions."

---

## 9. Intro to POST Requests

✅ `@PostMapping`

```java
@PostMapping("/student")
public String addStudent() {
    return "Student Added";
}
```

> 📌 This is just a first look at the annotation — it doesn't actually receive or save any data yet. Accepting data in the request body (`@RequestBody`) is covered in a later chapter.

#### 🛠️ Build-Along — Send a POST request with Postman

> 📌 **Instructor Note:** This is most students' first time sending a non-GET request. Walk through opening Postman together before assuming they know the interface.

**Say:** "Add the `addStudent()` method to `StudentController`. Run the app. In Postman, set the method dropdown to `POST`, enter `http://localhost:8080/student`, and hit Send."

✅ **Expected result:** the response body shows `Student Added`.

> 💬 **Say:** "Try changing the method back to `GET` in Postman and sending it to the same URL — watch it fail. The URL alone isn't enough to reach this endpoint; the HTTP method has to match too."

---

## 10. Testing APIs

✅ **Browser** — for GET requests only

✅ **Postman** (Important) — used for:
- GET
- POST
- PUT
- DELETE

> 🧠 **Say:** "Postman is like a remote control for APIs. You've now used both tools today — browser for GET, Postman for POST."

---

## 11. Mini Exercise

1. Create a `Course` class with `name` and `instructor` fields
2. Create a `CourseController` annotated with `@RestController`
3. Add a `GET /courses` endpoint that returns a list of at least 2 `Course` objects
4. Test it in the browser or Postman

#### 🛠️ Build-Along — Solve it independently

> 📌 **Instructor Note:** Give the spec as-is. This directly repeats the shape of Sections 5–7 with a new domain object — should be routine at this point in the course.

<details>
<summary>💡 Click to reveal a suggested solution</summary>

```java
package org.codex;

public class Course {
    private String name;
    private String instructor;

    public Course(String name, String instructor) {
        this.name = name;
        this.instructor = instructor;
    }

    public String getName() {
        return name;
    }

    public String getInstructor() {
        return instructor;
    }
}
```

```java
package org.codex;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class CourseController {

    @GetMapping("/courses")
    public List<Course> getCourses() {
        return List.of(
                new Course("Spring Boot", "Mr. Codex"),
                new Course("Java Fundamentals", "Mrs. Lane")
        );
    }
}
```

✅ **Response:**
```json
[
  {
    "name": "Spring Boot",
    "instructor": "Mr. Codex"
  },
  {
    "name": "Java Fundamentals",
    "instructor": "Mrs. Lane"
  }
]
```

</details>

---

## 12. Closing Statement

## ✅ Key Takeaways

- A REST API lets systems communicate over HTTP, exposing data through URLs
- `@RestController` marks a class as an API controller; `@GetMapping` maps a URL to a Java method
- Returned Java objects (and lists of them) are automatically serialized to JSON — no manual conversion needed, and it works for any field you add, not just the ones shown in examples
- HTTP verbs describe intent: GET fetches, POST creates, PUT updates, DELETE removes — and the verb has to match, not just the URL
- Browsers can only test GET requests; Postman can test all of them
- Unlike `CommandLineRunner` in Chapter 8 (runs once, at startup), endpoint methods run every time a matching request arrives

---

**← Previous: [Chapter 8 — Spring Boot Introduction](./chapter-8-spring-boot-introduction.md)** | **Next: Chapter 10 →**

---

## 🔧 What Changed in This Revision

> 🆕 **Not for dictation — for your reference only.**

- Every endpoint is now a build-along — students hit real URLs in a browser or Postman and read real responses, rather than reading code and trusting the shown output.
- The JSON serialization "magic" is now provable, not just asserted: students add a third field to `Student` and watch it appear in the response with zero extra code, which makes "it's genuinely automatic" concrete instead of a claim to take on faith.
- Added an explicit bridge back to Chapter 8's push model (`CommandLineRunner` runs once vs. endpoints running per-request) — same instinct as the Chapter 8 revision, tying new behavior to something already understood.
- Added a small POST/GET mismatch demonstration so students see that the HTTP method is part of what routes a request, not just the URL — this heads off a common early confusion before it happens.
