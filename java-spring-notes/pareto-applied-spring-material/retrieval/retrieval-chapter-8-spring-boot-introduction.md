# Retrieval Practice — Chapter 8: Spring Boot Introduction

> Exam-grade, theory-focused. Full chapter coverage. All options are deliberately similar in length and plausibility — there is no shortcut by "spotting the detailed one." You need the concept, not the pattern.

---

## Multiple Choice

**1. What problem does Spring Boot solve, according to this chapter's framing?**
A) It fixes bugs present in Spring's dependency injection engine
B) It removes the pain of manual configuration that existed before Boot, without replacing Spring itself
C) It replaces the Spring container with a completely different underlying technology
D) It removes the need for the Java compiler during development

*Answer: B*

**2. What three things does `@SpringBootApplication` combine into a single annotation?**
A) `@Component`, `@Autowired`, and `@Qualifier`
B) `@Configuration`, `@ComponentScan`, and `@EnableAutoConfiguration`
C) `@Bean`, `@Primary`, and `@Scope`
D) `@Service`, `@Repository`, and `@Controller`

*Answer: B*

**3. In the comparison between the manual rig (Chapters 3–7) and Spring Boot, what specifically does `@SpringBootApplication` replace?**
A) The `Course` and `Student` classes themselves
B) The hand-written `AppConfig` class carrying `@Configuration` and `@ComponentScan`
C) The Maven dependency for `spring-context`
D) The `@Autowired` annotation on injected fields

*Answer: B*

**4. What does `SpringApplication.run(Main.class, args)` do?**
A) It only compiles the project without starting any container
B) It starts the container, scans components, applies auto-configuration, and starts a server if applicable
C) It only starts an embedded server, without touching the Spring container
D) It manually registers each bean one at a time, requiring explicit method calls

*Answer: B*

**5. What is the purpose of `CommandLineRunner` in a Spring Boot application?**
A) It defines the entry point of the application in place of `main`
B) It provides a way to run logic automatically right after the application starts
C) It replaces `@Component` as the way to register beans
D) It is required on every bean class to enable dependency injection

*Answer: B*

**6. How does retrieving and using a bean differ between the manual approach (Chapters 3–7) and the `CommandLineRunner` approach?**
A) Both require calling `context.getBean()` explicitly inside `main`
B) The manual approach pulls beans explicitly in `main`; `CommandLineRunner` has Spring push execution automatically at startup
C) `CommandLineRunner` requires manually creating the `ApplicationContext`, while the manual approach does not
D) There is no meaningful difference between the two approaches

*Answer: B*

**7. What rule governs where the main class (annotated `@SpringBootApplication`) should be placed in the project structure?**
A) It must be placed in the deepest subpackage of the project
B) It should sit in the root package, since component scanning works downward from its location
C) It must be placed in a package named exactly `main`
D) Its location has no effect on component scanning

*Answer: B*

**8. In the build-along where `Main` was deliberately moved into a subpackage while `Course` and `Runner` stayed in the parent package, what was the result?**
A) The application ran identically, since package location has no effect on scanning
B) Component scanning failed to find `Course` and `Runner`, since they were no longer below `Main`'s location
C) The application refused to compile
D) `Runner` was still found, but `Course` was not

*Answer: B*

**9. According to the common student confusions addressed in this chapter, does `ApplicationContext` still exist in a Spring Boot application?**
A) No — Spring Boot removes `ApplicationContext` entirely in favor of a simpler mechanism
B) Yes — it still exists, Spring Boot just hides it from the developer
C) Yes, but only if `CommandLineRunner` is used
D) No — it is replaced entirely by `SpringApplication`

*Answer: B*

**10. Which of the following is one of the three things Spring Boot is described as doing automatically?**
A) Writing unit tests for each bean
B) Providing starter dependencies, so compatible libraries don't need to be manually hunted down
C) Automatically generating documentation for each REST endpoint
D) Converting existing XML configuration files into annotations

*Answer: B*

**11. What does the embedded server capability of Spring Boot remove the need for?**
A) The need for a `main` method in the application
B) The need to manually set up and deploy to an external server like Tomcat
C) The need for any dependency injection in the application
D) The need for a `pom.xml` file

*Answer: B*

**12. Why does an empty Spring Boot application (no components, no `CommandLineRunner`) still print a startup log and then exit?**
A) Because Spring Boot always requires a web dependency to run at all
B) Because the container genuinely starts, scans, and configures itself, but nothing has been given for it to execute afterward
C) Because `@SpringBootApplication` silently fails without at least one `@Component`
D) Because `SpringApplication.run()` only logs errors, never successful starts

*Answer: B*

---

## True or False

**13.** Spring Boot replaces the Spring framework with an entirely separate dependency injection system.
*Answer: False — Spring Boot builds on top of Spring rather than replacing it; `@SpringBootApplication` still relies on `@Configuration` and `@ComponentScan` underneath.*

**14.** The location of the main class within the project's package structure has no effect on which beans component scanning discovers.
*Answer: False — the main class's package is the root that component scanning works downward from; moving it changes what gets discovered.*

---

## Short Theory

**15. Explain what `@SpringBootApplication` actually consists of, and why understanding that matters even though Spring Boot "hides" it.**
A: `@SpringBootApplication` combines `@Configuration`, `@ComponentScan`, and `@EnableAutoConfiguration`. Understanding this matters because nothing new or magical is happening underneath — it's the exact same mechanisms built by hand in earlier chapters, packaged into one annotation. Recognizing this makes debugging easier, since problems can still be traced back to familiar concepts like scanning and bean registration, rather than treated as an unexplainable black box.

**16. Describe what would happen if a Spring Boot project's main class were moved into a subpackage below where its `@Component`-annotated classes live, and explain why.**
A: Moving the main class into a subpackage below its `@Component` classes would break component scanning for those classes, since Spring Boot scans downward starting from the package containing the main class. Any `@Component`-annotated classes now sitting outside that scanning root (above or beside it, rather than below) would no longer be discovered, and anything depending on them being registered as beans would fail.

---

*12 MCQ, 2 True/False, 2 short theory — 16 questions total, full chapter coverage.*
