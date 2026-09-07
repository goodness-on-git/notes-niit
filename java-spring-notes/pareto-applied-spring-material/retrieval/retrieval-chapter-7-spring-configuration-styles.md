# Retrieval Practice — Chapter 7: Spring Configuration Styles

> Exam-grade, theory-focused. Full chapter coverage. All options are deliberately similar in length and plausibility — there is no shortcut by "spotting the detailed one." You need the concept, not the pattern.

---

## Multiple Choice

**1. Which configuration style is described as verbose, not type-safe, and generally avoided in modern development?**
A) Annotation-based configuration using `@Component`
B) XML configuration
C) Java configuration using `@Configuration` and `@Bean`
D) Auto-configuration via Spring Boot

*Answer: B*

**2. What is the key limitation of annotation-based configuration (`@Component`) that Java configuration solves?**
A) It cannot be used together with `@Autowired`
B) It only works for classes you own and can add annotations to
C) It requires component scanning to be disabled
D) It only supports singleton-scoped beans

*Answer: B*

**3. What does `@Bean`, placed on a method inside a `@Configuration` class, tell Spring?**
A) That the method should be run only once, at application shutdown
B) That the object returned by the method should be registered and managed as a bean
C) That the method's parameters should each become separate beans
D) That the class itself is exempt from component scanning

*Answer: B*

**4. In the full example where `student()` calls `course()` internally, what happens when Spring processes the `@Configuration` class?**
A) The call to `course()` creates a fresh `Course` object each time it's invoked
B) Spring intercepts the call and returns the already-registered bean instead of running the method body again
C) The call is ignored, and `Student` receives a null reference
D) Spring throws an exception, since a `@Bean` method cannot call another `@Bean` method

*Answer: B*

**5. In the build-along that added a constructor print statement to `Course`, referenced by two different `@Bean` methods, how many times did the constructor print?**
A) Twice — once for each `@Bean` method that referenced it
B) Once, regardless of how many `@Bean` methods referenced it
C) Zero times, since constructors are skipped inside `@Configuration` classes
D) Once for the first method, then not at all for any later calls in the same run

*Answer: B*

**6. Which combination allows both automatic scanning and manual bean registration to coexist in one configuration class?**
A) `@Configuration` alone, since it enables both behaviors implicitly
B) `@Configuration` together with `@ComponentScan`, while still including `@Bean` methods in the same class
C) `@Bean` combined with `@Autowired` placed directly on the class
D) `@ComponentScan` used without `@Configuration`, since `@Configuration` blocks scanning

*Answer: B*

**7. When one bean is registered via `@ComponentScan` and another via a `@Bean` method in the same project, how does Spring treat them when wiring dependencies between them?**
A) Beans from different registration styles cannot be injected into each other
B) Spring treats them identically once they're in the container, regardless of which style created them
C) `@Bean`-registered beans always take priority during dependency resolution
D) `@ComponentScan`-registered beans must be passed manually as method arguments

*Answer: B*

**8. Which scenario is given as the clearest reason to use `@Bean` instead of `@Component`?**
A) The class is simple and has no dependencies of its own
B) The class comes from a third-party library and cannot have annotations added to its source
C) The class needs to be scoped as a singleton rather than a prototype
D) The class is expected to be autowired into several other beans

*Answer: B*

**9. According to the classroom analogy in this chapter, what is the core difference between `@Component` and `@Bean`?**
A) `@Component` requires manual object construction; `@Bean` does not
B) With `@Component`, Spring finds and creates the object automatically; with `@Bean`, you build the object yourself and hand it to Spring
C) `@Component` is used only for testing; `@Bean` is used only in production code
D) `@Bean` disables dependency injection for the object it creates

*Answer: B*

**10. In the mixed-style mini exercise (`Library` treated as third-party, `Student` owned), which registration style was used for each class?**
A) Both `Library` and `Student` were registered via `@Bean` methods
B) `Library` was registered via a `@Bean` method; `Student` was picked up via `@ComponentScan`
C) `Library` was picked up via `@ComponentScan`; `Student` was registered via a `@Bean` method
D) Both were registered via `@ComponentScan`, since both live in the same package

*Answer: B*

**11. Why is XML configuration described as "not type-safe" compared to Java configuration?**
A) XML files cannot be version-controlled alongside source code
B) Errors in XML bean definitions are typically only caught at runtime, not by the compiler
C) XML does not support dependency injection at all
D) XML configuration requires a completely separate build tool from Maven

*Answer: B*

**12. What did comparing a `@Bean` method call against a plain, non-`@Bean` method returning `new Course()` demonstrate?**
A) Both approaches produce exactly one object, regardless of how many times they're called
B) The plain method call creates a new object on each call, while the `@Bean` method call reuses the existing bean
C) Neither approach can be used inside a `@Configuration` class
D) The plain method call throws a compilation error inside a `@Configuration` class

*Answer: B*

---

## True or False

**13.** A `@Configuration` class can include both `@ComponentScan` and manually defined `@Bean` methods at the same time.
*Answer: True — this is exactly the mixed-style pattern demonstrated in Section 6.*

**14.** XML configuration is described in this chapter as the modern, recommended approach for new Spring projects.
*Answer: False — XML is described as the old way: verbose and not type-safe. Annotations and Java config are the modern preference.*

---

## Short Theory

**15. Explain why the constructor of `Course` only printed once in the build-along, even though two different `@Bean` methods referenced `course()`.**
A: Spring processes `@Configuration` classes specially, so that calls to `@Bean` methods from within the class don't re-run the method body if the bean already exists — Spring intercepts the call and returns the already-created bean from the container instead. Since the constructor only runs on actual object creation, it fired only once, no matter how many `@Bean` methods referenced the call.

**16. Describe a realistic scenario where mixing `@ComponentScan` and `@Bean` in one configuration class is necessary, and explain why one style alone couldn't cover it.**
A: A realistic scenario is a project where most classes are owned and can be annotated directly with `@Component`, picked up automatically via `@ComponentScan` — but one dependency comes from a third-party library whose source can't be modified to add annotations. `@ComponentScan` alone can't register that library class, since there's no `@Component` annotation for it to detect; a `@Bean` method is needed to manually construct and register it, while `@ComponentScan` continues handling everything else.

---

*12 MCQ, 2 True/False, 2 short theory — 16 questions total, full chapter coverage.*
