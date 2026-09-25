In Java, annotations are a form of metadata—data about your code—that provide extra information to the compiler or the runtime environment.  
- Compiler Instructions: They help the compiler find errors or suppress warnings (e.g., @Override ensures you actually override a parent method).
- Runtime Processing: Frameworks like Spring use annotations to identify components at runtime (e.g., @RestController tells Spring to treat a class as a web handler).
- Build-time Tools: Tools can use them to generate code, configuration files (like XML), or documentation (Javadoc).

## Project Lombok 
- It is a library that uses annotations to reduce "boilerplate" code. It plugs into your build process and generates standard Java code automatically at compile-time.
- Popular Lombok Annotations:
  - @Getter / @Setter: Generates your getters and setters automatically.
  - @ToString: Creates a toString() method containing all class fields.
  - @AllArgsConstructor: Generates a constructor for all fields.
  - @Data: A "super" annotation that bundles @Getter, @Setter, @ToString, @EqualsAndHashCode, and @RequiredArgsConstructor into one.
  - @Slf4j: Automatically adds a logger field (log) so you can start logging immediately.  

## Custom Annotation
- How to Create Your Own Annotation: You define a custom annotation using the @interface keyword.
- Steps to create one: Define the Interface: Use @interface followed by your annotation name.
- Add Meta-Annotations: These define where and how long the annotation lasts.
  - @Target: Specifies where it can be used (e.g., METHOD, TYPE for classes, FIELD).
  - @Retention: Defines if it is discarded after compiling (SOURCE) or available at runtime (RUNTIME).
- Add Elements: (Optional) These look like methods and can have default values.
- Example:
  
      ```
      import java.lang.annotation.*; 
      
      @Retention(RetentionPolicy.RUNTIME)  
      @Target(ElementType.METHOD) 
      public @interface LogExecution { 
          String value() default "Default Log Message"; 
      } 
      ```
 
