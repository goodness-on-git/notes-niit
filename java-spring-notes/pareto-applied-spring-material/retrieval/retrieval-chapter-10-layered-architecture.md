# Retrieval Practice — Chapter 10: Layered Architecture (Controller → Service → Repository)

> Exam-grade, theory-focused. Full chapter coverage. All options are deliberately similar in length and plausibility — there is no shortcut by "spotting the detailed one." You need the concept, not the pattern.

---

## Multiple Choice

**1. What was the main problem with the "bad example" where all logic sat inside one controller method?**
A) The code was too short to compile correctly
B) Responsibilities like database access, business logic, and formatting were all mixed into one class, making it hard to maintain and test
C) The controller was missing a `@RestController` annotation
D) The endpoint could not be reached from a browser

*Answer: B*

**2. What is the primary responsibility of the Controller layer?**
A) Performing complex business calculations
B) Handling HTTP requests and responses
C) Directly querying the database
D) Validating business rules before saving data

*Answer: B*

**3. What is the primary responsibility of the Service layer?**
A) Mapping URLs directly to Java methods
B) Containing business logic — calculations, rules, and decision-making
C) Directly returning HTTP status codes to the client
D) Storing data permanently on disk

*Answer: B*

**4. What is the primary responsibility of the Repository layer?**
A) Handling data access
B) Formatting JSON responses for the client
C) Mapping incoming HTTP methods to controller methods
D) Making business decisions about which data to return

*Answer: A*

**5. In the build-along where logic moved from Controller directly into a Service, what happened to the response returned by `GET /students`?**
A) It changed slightly, since a new layer was now involved
B) It stayed exactly the same as before the change
C) It became empty, since the controller no longer had direct access to the data
D) It doubled, returning the list twice

*Answer: B*

**6. In the build-along where logic moved from Service into a Repository, what happened to the response, compared to the version before that change?**
A) It stayed exactly the same, for the third structurally different version in a row
B) It changed, since the data now came from a different class
C) It failed, since repositories cannot be injected into services
D) It returned additional fields not present in the earlier versions

*Answer: A*

**7. How are the Controller, Service, and Repository layers wired together in this chapter?**
A) Through static method calls between classes
B) Through constructor injection, the same mechanism used since Chapter 2
C) Through XML configuration files
D) Through field injection exclusively, since constructor injection isn't compatible with `@Service`

*Answer: B*

**8. Which direction does a request travel through the layers, and which direction does the response travel?**
A) Requests travel upward from Repository to Controller; responses travel downward
B) Requests travel downward from Controller to Repository; responses travel back upward
C) Both requests and responses travel in the same direction, downward only
D) Requests and responses both travel upward, starting from the Repository

*Answer: B*

**9. According to the chapter's common student mistakes, what is a mistake specifically related to the Repository layer?**
A) Repositories returning HTTP status codes directly to the browser
B) Repositories containing business logic that should belong to the Service layer
C) Repositories being annotated with `@RestController` instead of `@Repository`
D) Repositories never being allowed to return a `List`

*Answer: B*

**10. According to the chapter's common student mistakes, what is a mistake specifically related to the Service layer?**
A) Services containing calculations or business rules
B) Services returning HTTP responses, which is the Controller's responsibility
C) Services being injected into Controllers via constructor injection
D) Services depending on a Repository via constructor injection

*Answer: B*

**11. In the real-world analogy used in this chapter, which role is compared to a "manager"?**
A) Controller
B) Service
C) Repository
D) Database

*Answer: B*

**12. What is one of the stated benefits of layered architecture, beyond "cleaner code"?**
A) It removes the need for any dependency injection
B) It makes it possible to test each layer independently
C) It eliminates the need for a database entirely
D) It guarantees the application will never throw exceptions

*Answer: B*

---

## True or False

**13.** Splitting logic into Controller, Service, and Repository layers changes what data is returned to the client, compared to putting all logic in one class.
*Answer: False — the response was identical across all three structural versions built in this chapter; only the internal organization of responsibility changed.*

**14.** The Service layer is responsible for directly returning HTTP responses to the client.
*Answer: False — returning HTTP responses is the Controller's responsibility; the Service layer only returns data or results to whatever called it.*

---

## Short Theory

**15. Explain what changed and what stayed the same when the hardcoded student list was moved from the Controller into a Service, and then again into a Repository.**
A: What changed each time was which class was responsible for producing the data, and how the layers were wired together via constructor injection. What stayed exactly the same, across all three versions, was the actual response returned by `GET /students` — proving that layering changes internal responsibility and organization, not application behavior.

**16. Using the responsibilities described in this chapter, explain why placing business logic (such as validation rules) inside a Repository class would be considered a mistake.**
A: A Repository's responsibility is data access — retrieving and storing data, not deciding what that data means or whether it's valid. Business logic, including validation, belongs in the Service layer, which is meant to be the layer responsible for rules and decision-making. Placing that logic in the Repository blurs the separation of concerns the layered structure is meant to provide, making the Repository harder to reuse and the logic harder to find and test independently.

---

*12 MCQ, 2 True/False, 2 short theory — 16 questions total, full chapter coverage.*
