## SDK vs Java
- In Java development (especially within IDEs like JetBrains IntelliJ IDEA), the SDK (Software Development Kit / JDK) represents the physical tools and compiler used to build your application, while the Language Level dictates the specific syntax rules and coding assistance available in the code editor.
- The primary reason to separate these two settings is downward compatibility. You can use a newer SDK alongside a lower Language Level.
  -  You install JDK 21 as your SDK to benefit from the latest compiler optimizations and security patches. However, your production servers only run Java 17.

|Feature| Project SDK (JDK)|Language Level |
|-|-|-|
|What it is|The actual installation of the Java Development Kit (e.g., JDK 21, JDK 17).| A configuration setting that limits or allows specific Java syntax features.|
|Purpose|Provides the tools to compile, package, run, and debug the application.|Determines what code patterns the IDE allows you to write without throwing syntax errors.|
|Control Scope|Defines the maximum technical capabilities and standard libraries available.|Controls IDE Intellisense, refactoring suggestions, and target bytecode compatibility.|

## Why Kotlin over Java?
- Less boilerplate
- Null safety
- Coroutines
- Full Java interoperability

## Why Java over Kotlin?
- Larger talent pool
- Mature ecosystem
- Easier onboarding
- Long-term enterprise stability

## Why Groovy?
- Excellent for scripting
- DSL creation
- CI/CD automation
- Jenkins files and Gradle

## Why not Groovy for large backend services?
- Dynamic typing catches more errors at runtime
- Lower maintainability in very large codebases
- Usually slower than Java/Kotlin for application code

# Java vs Kotlin vs Groovy
- Java for stability and massive ecosystems. Its a traditional enterprise JVM language.
- Kotlin for modern application development with null -safety.
- Groovy for scripting, Gradle, and automation

| Feature             | Java                              | Kotlin                    | Groovy                        |
| ------------------- | --------------------------------- | ------------------------- | ----------------------------- |
| Typing              | Static                            | Static                    | Dynamic (optional static)     |
| Syntax              | Verbose                           | Concise                   | Very concise                  |
| Null Safety         | No built-in                       | Built-in                  | No built-in                   |
| Performance         | Excellent                         | Very good (close to Java) | Generally slower              |
| Learning Curve      | Moderate                          | Easy for Java devs        | Easy initially                |
| Interop with Java   | Native                            | Excellent                 | Excellent                     |
| Best Use            | Enterprise apps, backend services | Modern backend, Android   | Scripting, Gradle, automation |
| Compile-time Safety | High                              | High                      | Lower by default              |


# Performance
## Java
- Fastest and most mature.
- Optimized JVM support.
- Preferred for large enterprise systems.
  
## Kotlin
- Very close to Java performance.
- Additional features usually have little overhead.
- Most Spring Boot applications run equally well in Java or Kotlin.
  
## Groovy
- Dynamic dispatch and runtime metaprogramming make it slower for large production workloads.
- Fine for scripting and build automation.

# Ecosystem
## Java

Best ecosystem:
- Spring Boot
- Hibernate
- Kafka
- Enterprise applications

## Kotlin

Popular for:

- Android (Google's preferred language)
- Spring Boot
- Ktor
- Multiplatform projects

## Groovy

Popular for:

- Jenkins Pipelines
- Gradle build scripts
- Spock testing framework
- Automation scripts

Groovy is widely used in Gradle and Jenkins ecosystems.


