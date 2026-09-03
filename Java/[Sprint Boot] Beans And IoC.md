# What is Bean
- A Spring Bean is simply a Java object that is instantiated, assembled, and managed by the Spring Inversion of Control (IoC) container rather than being created manually using ```new MyObject()```.  
- Spring handles the lifecycle of these beans—creating them, passing necessary dependencies into them (Dependency Injection), and destroying them when the application shuts down.
- Instead of you manually creating objects and stitching them together using new MyService(), control is inverted to the framework(Spring Boot). The IoC container takes over the responsibility of instantiating, configuring, injecting dependencies into, and destroying objects (called Beans).

## Example
To create a Java file (a service class) and inject it into a REST controller via Dependency Injection (DI), follow these steps:

- Mark your class with ```@Service``` (or ```@Component```). This tells Spring's component scanner to register it as a managed bean.
```
  package com.example.demo.service;

  import org.springframework.stereotype.Service;

  @Service // Tells Spring to manage this class as a Bean
  public class MessageService {

      public String getGreeting() {
          return "Hello from the injected MessageService!";
      }
  }
```
- Create your ```@RestController``` and inject the ```MessageService``` using Constructor Injection (the recommended approach in Spring Boot).
```
  package com.example.demo.controller;

  import com.example.demo.service.MessageService;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class HelloController {
  
      private final MessageService messageService;
  
      // Spring automatically injects the MessageService bean here
      public HelloController(MessageService messageService) {
          this.messageService = messageService;
      }
  
      @GetMapping("/hello")
      public String sayHello() {
          return messageService.getGreeting();
      }
  }
```
## Basic Annotations for Creating Beans
- ```@Component```: General-purpose annotation for any Spring-managed component.  
- ```@Service```: Used for classes containing business logic (specialized @Component).
- ```@Repository```: Used for data access layers/DAOs (handles database exceptions).  
- ```@Bean```: Used inside a @Configuration class to manually define a bean (useful for third-party library classes you don't own).
- ```@RestController```: Combines ```@Controller``` and ```@ResponseBody```. Automatically serializes returned objects directly into JSON/XML HTTP responses.
- ```@Configuration```: Contains methods annotated with ```@Bean```. Spring enhances this class with CGLIB proxies to ensure methods return singleton bean instances.

# IoC Container
- The Inversion of Control (IoC) Container is the core engine inside Spring Boot that acts as a central factory and manager for all application components.
- Think of the IoC container as an automated assembly line that works in four phases:
  - **Scanning & Reading Definitions**: At startup, Spring scans your project packages for annotations (@Component, @Service, @Repository, @RestController, @Configuration).
  - **Instantiation**: It creates instances of these classes.
  - **Dependency Injection (DI)**: It looks at what each object needs. If OrderController needs PaymentService, the container injects the PaymentService instance directly into the controller's constructor.
  - **Lifecycle Management**: It manages the object from initialization (@PostConstruct) through its destruction (@PreDestroy) when the application stops.
- Representation in Spring Code
  - ApplicationContext (Recommended & Modern): The primary IoC container used in Spring Boot applications. It adds enterprise-level capabilities like event handling, internationalization, and web application support on top of object management.

## Traditional Code vs. IoC Container

|Traditional Java (You in Control)| Spring Boot IoC (Container in Control)|
|-|-|
|Object Creation: You use new PaymentService() inside every class that needs it.| Object Creation: The container creates a single managed instance (Singleton) at startup. |
| Coupling: High. If PaymentService's constructor changes, you must update every place it's instantiated. |Coupling: Low. Spring automatically supplies the dependency where requested.|
|Testing: Difficult to write unit tests because dependencies are hardcoded. |Testing: Easy. You can easily inject mock implementations into classes.|

## How to Access the Container Directly (Optional)
In standard Spring Boot development, dependency injection handles object retrieval automatically. However, you can interact with the IoC container directly by fetching beans from ApplicationContext:
```
  @SpringBootApplication
  public class DemoApplication {
  
      public static void main(String[] args) {
          // SpringApplication.run() initializes and returns the IoC container
          ApplicationContext container = SpringApplication.run(DemoApplication.class, args);
  
          // Explicitly asking the container for a bean
          PaymentService paymentService = container.getBean(PaymentService.class);
      }
  }
```
