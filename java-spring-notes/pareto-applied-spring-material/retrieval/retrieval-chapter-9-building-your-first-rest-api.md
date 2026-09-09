# Retrieval Practice — Chapter 9: Building Your First REST API with Spring Boot

> Exam-grade, theory-focused. Full chapter coverage. All options are deliberately similar in length and plausibility — there is no shortcut by "spotting the detailed one." You need the concept, not the pattern.

---

## Multiple Choice

**1. What is the primary purpose of a REST API, as described in this chapter?**
A) To allow a Java class to inherit behavior from a parent class
B) To allow different applications to communicate over HTTP
C) To convert Java source code into bytecode for faster execution
D) To store data permanently inside the Spring container

*Answer: B*

**2. What does the `@RestController` annotation do to a class?**
A) It marks the class as a background job scheduler
B) It marks the class as one that handles web requests and returns data
C) It marks the class as a database access layer
D) It marks the class as exempt from dependency injection

*Answer: B*

**3. What does `@GetMapping("/hello")` do?**
A) It creates a new HTTP server on a random port each time it's called
B) It maps incoming GET requests at the `/hello` URL to the annotated method
C) It automatically generates a `Student` object named "hello"
D) It disables all other endpoints in the same controller

*Answer: B*

**4. In `http://localhost:8080/hello`, what does `8080` represent?**
A) The number of endpoints registered in the application
B) The port the embedded server is listening on
C) The version number of Spring Boot being used
D) The number of beans currently in the container

*Answer: B*

**5. When a controller method returns a `Student` object, what does Spring Boot do with it by default?**
A) It stores the object in the database automatically
B) It automatically serializes the object into JSON in the response
C) It throws an exception, since only Strings can be returned from endpoints
D) It converts the object into an XML document instead of JSON

*Answer: B*

**6. In the build-along where a third field was added to `Student`, what happened to the JSON response with no extra conversion code written?**
A) The response stayed the same, since only the original two fields are ever serialized
B) The new field appeared automatically in the JSON response
C) The application failed to start until a custom serializer was written
D) The new field appeared only after restarting Postman

*Answer: B*

**7. What happens when a `List<Student>` is returned from an endpoint?**
A) Only the first `Student` in the list is serialized into the response
B) The response becomes a JSON array, with each `Student` serialized as an object inside it
C) Spring Boot throws an exception, since lists cannot be returned directly
D) Each `Student` is returned as a separate, individual HTTP response

*Answer: B*

**8. Which HTTP method is associated with creating new data, according to the table in this chapter?**
A) GET
B) POST
C) PUT
D) DELETE

*Answer: B*

**9. In the chapter's POST build-along, what happened when the same URL was requested using GET instead of POST?**
A) The request succeeded identically, since the URL was unchanged
B) The request failed, since the HTTP method has to match the mapping, not just the URL
C) Spring Boot automatically converted the GET request into a POST request
D) The server crashed and needed to be restarted

*Answer: B*

**10. Why does the `addStudent()` example in this chapter not actually save any data yet?**
A) Because `@PostMapping` is incapable of receiving data under any circumstances
B) Because accepting data in the request body requires `@RequestBody`, covered in a later chapter
C) Because `StudentController` has not been marked `@RestController`
D) Because only `@GetMapping` endpoints are allowed to modify application state

*Answer: B*

**11. Which testing tool is capable of sending GET, POST, PUT, and DELETE requests, according to this chapter?**
A) A web browser, for all four methods equally
B) Postman
C) The Spring Initializr website
D) The embedded Tomcat server's admin console

*Answer: B*

**12. Why can a browser test `GET /students` by typing the URL directly, but not `POST /student` the same way?**
A) Typing a URL into a browser's address bar always sends a GET request, regardless of the endpoint's actual method
B) POST requests are disabled by default in all Spring Boot applications
C) `/student` and `/students` cannot both exist in the same controller
D) Browsers require a separate plugin to send any HTTP request at all

*Answer: A*

---

## True or False

**13.** Returning a `List<Student>` from an endpoint results in a JSON array in the response.
*Answer: True — each `Student` in the list is serialized as an object inside the array.*

**14.** A browser can fully test POST, PUT, and DELETE endpoints by typing their URLs directly into the address bar.
*Answer: False — typing a URL into a browser's address bar always sends a GET request; browsers cannot send POST/PUT/DELETE that way.*

---

## Short Theory

**15. Explain how Spring Boot converts a returned Java object into JSON without any conversion code written by the developer.**
A: Spring Boot includes automatic JSON serialization as part of its web support. When a method annotated with `@GetMapping` (or similar) returns an object, Spring Boot inspects its fields via their getters and converts the object into a JSON representation automatically — no manual conversion code is required from the developer, and this applies to any fields present, not a fixed set.

**16. Explain why the HTTP method (GET, POST, etc.) matters just as much as the URL when a request reaches a Spring Boot application.**
A: Spring Boot maps a specific combination of URL and HTTP method to each endpoint method — a `@GetMapping("/student")` and a `@PostMapping("/student")` can both exist at the same URL but are treated as entirely separate endpoints. A request only reaches a given method if both the URL and the HTTP method match; matching the URL alone is not sufficient.

---

*12 MCQ, 2 True/False, 2 short theory — 16 questions total, full chapter coverage.*
