# General
In Java, async programming has evolved through two main paradigms:
- Explicit API Chains (```CompletableFuture```) — Functionally similar to C#'s Task / promises.
- Virtual Threads (Project Loom) — Java's alternative answer to async/await.
  - When enabled, Spring processes each HTTP request on a dedicated Virtual Thread. Whenever a blocking call occurs (like Thread.sleep or database I/O), the JVM automatically unmounts the virtual thread, freeing the underlying OS thread.

# CompletableFuture (Java) vs async/await (C#) vs Virtual Thread
- Before Java 21, Java relied heavily on CompletableFuture (introduced in Java 8). While C# allows you to write async code sequentially with await, Java's ```CompletableFuture``` requires a functional, callback-based style.
- Java 21 introduced Virtual Threads. Java's creators took a completely different approach from C#'s async/await: Make blocking extremely cheap instead of adding new keywords.
  - In Java with Virtual Threads, you write standard, blocking synchronous code. The Java Virtual Machine (JVM) automatically pauses and resumes the lightweight virtual thread whenever an I/O operation (like a network call or database query) occurs. 

**C#**
```
public async Task<User> GetUserAsync(long id) 
{
    // Thread is NOT blocked while waiting for I/O
    User user = await dbContext.Users.FindAsync(id); 
    string email = await emailService.FetchEmailAsync(user.Id);
    return user;
}
```

**Java (Virtual Thread)**
```
// Looks 100% synchronous, but executes asynchronously on JVM level!
public User getUser(long id) {
    User user = dbContext.findUser(id); // JVM unmounts virtual thread during DB I/O
    String email = emailService.fetchEmail(user.getId());
    user.setEmail(email);
    return user;
}
```

**Java Equivalent (CompletableFuture)**
```
public CompletableFuture<User> getUserAsync(long id) {
    return dbContext.findUserAsync(id)
        .thenCompose(user -> 
            emailService.fetchEmailAsync(user.getId())
                .thenApply(email -> {
                    user.setEmail(email);
                    return user;
                })
        );
}
```

|Feature |C# | Java (CompletableFuture)| Modern Java (Virtual Threads) |
|-|-|-|-|
|Syntax |Language keywords (async/await) | Functional API (thenApply, thenCompose) | Standard sequential Java code|
|Code Structure |Reads sequentially like synchronous code | Method chaining and callbacks | Reads sequentially like synchronous code|
|Error Handling |Standard try / catch| Exception chaining (.exceptionally()) | Standard try / catch|
|Under the Hood | Compiler-generated state machine | ForkJoinPool thread pools | Managed at the JVM runtime layer|

## Fire And Forgot Pattern

**Virtual Thread**
- To fire off a task in the background and immediately continue to the next line of code without waiting for the result, you don't use Virtual Threads directly as a blocking mechanism—instead, you launch a background Virtual Thread using Thread.ofVirtual() or an ExecutorService.

```
public void handleUserSignup(User user) {
    // 1. Fire-and-forget background task using a Virtual Thread
    Thread.ofVirtual().start(() -> {
        emailService.sendWelcomeEmail(user); // Runs in background
    });

    // 2. Execution immediately reaches this line without waiting!
    logger.info("Signup response sent to client.");
}
```

**Spring Boot Way: @Async with Virtual Threads**
- If you are inside Spring Boot, you can combine Spring's @Async with Virtual Threads so you don't even need to write Thread.ofVirtual() manually.
- Behind the scenes, @Async uses the CGLIB dynamic proxy mechanism
- Important Rules to Remember for @Async
  - Must be called from outside the class: Because @Async relies on Spring's CGLIB proxy wrapper, calling an @Async method from within the same class (this.myAsyncMethod()) bypasses the proxy, and it will execute synchronously instead.
  - Method must be public: CGLIB proxy overriding requires methods to be public and non-final.

```
@RestController
public class UserController {

    private final EmailService emailService;

    public UserController(EmailService emailService) {
        this.emailService = emailService;
    }

    @PostMapping("/signup")
    public String signup(@RequestBody User user) {
        emailService.sendWelcomeEmail(user); // Non-blocking: fires and moves to next line immediately

        return "User registered successfully!"; // Responds instantly
    }
}

```

```
@Service
public class EmailService {

    @Async // Runs on a background Virtual Thread automatically
    public void sendWelcomeEmail(User user) {
        // Long-running email call
    }
}
```


## Fetching Results Later Pattern: Structured Concurrency / Futures
- If you want to start a background task, do other work in the main thread, and then collect the result later (equivalent to C# await task), you use CompletableFuture or Java 21's StructuredTaskScope.

Using CompletableFuture with Virtual Threads

```
public void processOrder() {
    // 1. Start background task on a Virtual Thread
    CompletableFuture<String> paymentFuture = CompletableFuture.supplyAsync(
        () -> paymentGateway.chargeCard(), 
        Executors.newVirtualThreadPerTaskExecutor()
    );

    // 2. Do other work immediately in the main thread
    doInventoryCheck();

    // 3. Collect the result when you actually need it (blocks here if not finished)
    String paymentStatus = paymentFuture.join(); 
}
```
