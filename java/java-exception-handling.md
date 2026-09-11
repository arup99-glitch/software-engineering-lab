# Lesson 1 — What actually happens when an exception occurs in Java

Before learning `try-catch`, I want you to understand **what Java actually does when something goes wrong**.

This is the foundation for everything else.

---

## 1. First: what is an exception?

An exception is basically:

> **An event that interrupts the normal flow of your program because something went wrong.**

Example:

```java
int a = 10;
int b = 0;

int result = a / b;

System.out.println("Hello");
```

Java cannot do:

```text
10 / 0
```

So an exception occurs:

```text
ArithmeticException: / by zero
```

And importantly:

```java
System.out.println("Hello");
```

**doesn't execute.**

Why?

Because the normal flow was interrupted.

---

# 2. Think of a method call like a chain

Suppose we have:

```java
public static void main(String[] args) {
    methodA();
}

static void methodA() {
    methodB();
}

static void methodB() {
    methodC();
}

static void methodC() {
    int result = 10 / 0;
}
```

The execution is:

```text
main()
  ↓
methodA()
  ↓
methodB()
  ↓
methodC()
  ↓
10 / 0
  ↓
💥 Exception
```

At this point, Java needs to answer:

> **Who is going to handle this exception?**

---

# 3. Java starts looking backward

Suppose `methodC()` doesn't handle the exception.

Java goes back to `methodB()`.

```java
static void methodB() {
    methodC();
}
```

No handler there.

So Java goes back to `methodA()`.

```java
static void methodA() {
    methodB();
}
```

No handler.

Then back to:

```java
main()
```

If nobody handles it, the JVM handles it and terminates the current thread.

This is called:

## Exception propagation

Think:

```text
methodC()
   ↓
exception happens
   ↓
methodB()       ← can you handle it?
   ↓
methodA()       ← can you handle it?
   ↓
main()          ← can you handle it?
   ↓
JVM
```

This concept is **extremely important**.

---

# 4. Now let's handle it

We can put a `try-catch` in `methodB()`:

```java
public static void main(String[] args) {
    methodA();
}

static void methodA() {
    methodB();
}

static void methodB() {

    try {
        methodC();
    } catch (ArithmeticException e) {
        System.out.println("Something went wrong");
    }
}

static void methodC() {
    int result = 10 / 0;
}
```

Now execution becomes:

```text
main()
 ↓
methodA()
 ↓
methodB()
 ↓
methodC()
 ↓
💥 Exception
 ↓
methodC() can't handle
 ↓
methodB() has catch
 ↓
catch executes
 ↓
"Something went wrong"
```

The program doesn't need to terminate because **someone handled the exception**.

---

# 5. What is `e`?

You often see:

```java
catch (ArithmeticException e)
```

The `e` is a **reference to the exception object**.

Java creates an exception object:

```java
ArithmeticException
```

and gives it to your catch block:

```java
catch (ArithmeticException e)
```

You can inspect it:

```java
catch (ArithmeticException e) {
    System.out.println(e.getMessage());
}
```

Output:

```text
/ by zero
```

Or:

```java
e.printStackTrace();
```

which gives you information about where the exception happened.

---

# 6. What is the stack trace?

This is something you will see **all the time in production**.

For example:

```text
java.lang.ArithmeticException: / by zero
    at com.example.PaymentService.calculate(PaymentService.java:25)
    at com.example.PaymentService.process(PaymentService.java:18)
    at com.example.PaymentController.pay(PaymentController.java:12)
```

Don't think of this as scary output.

Read it like a story:

```text
Exception happened
       ↓
PaymentService.calculate()
       ↓
called from
       ↓
PaymentService.process()
       ↓
called from
       ↓
PaymentController.pay()
```

The first application line is often the most useful clue to where the failure originated.

As a backend engineer, **being comfortable reading stack traces is essential.**

---

# 7. Now an important distinction

Look at this:

```java
static void methodC() {

    int result = 10 / 0;

}
```

The exception was **thrown by Java automatically**.

You didn't write:

```java
throw ...
```

Java detects the illegal operation and throws the exception.

But you can also explicitly throw one yourself.

```java
static void validateAge(int age) {

    if (age < 18) {
        throw new IllegalArgumentException("Age must be 18 or above");
    }
}
```

Now:

```java
validateAge(15);
```

causes:

```text
IllegalArgumentException
        ↓
"Age must be 18 or above"
```

So there are two ideas:

### Java detects a problem

```java
10 / 0;
```

### You explicitly tell Java something is wrong

```java
throw new IllegalArgumentException("Invalid age");
```

We'll study `throw` deeply in a later lesson.

---

# 8. One very important production lesson

Don't do this everywhere:

```java
try {
    // something
} catch (Exception e) {
    System.out.println("Error");
}
```

Why?

Because you may be **hiding the real problem**.

For example:

```java
try {
    paymentService.processPayment();
} catch (Exception e) {
    System.out.println("Payment failed");
}
```

What if the database is down?

What if the payment gateway timed out?

What if there is a programming bug?

You just printed:

```text
Payment failed
```

and lost valuable information.

A better approach depends on the situation, often including proper logging:

```java
catch (Exception e) {
    log.error("Payment processing failed", e);
}
```

Notice:

```java
log.error("Payment processing failed", e);
```

The exception itself is passed to the logger, so the **stack trace is preserved**.

---

# 9. The mental model I want you to remember

Whenever an exception happens, imagine:

```text
              Exception
                  ↓
        Current method checks
        "Can I handle this?"
                  ↓
              No?
                  ↓
        Caller method checks
        "Can I handle this?"
                  ↓
              No?
                  ↓
        Caller checks again
                  ↓
                ...
                  ↓
          Nobody handles it
                  ↓
                 JVM
```

This is **exception propagation**.

---

# 10. Real-life analogy

Imagine you're working in a company.

```text
Developer
   ↓
Team Lead
   ↓
Manager
   ↓
Director
```

A developer encounters a problem.

If the developer can solve it:

```text
Developer → solves problem
```

Otherwise:

```text
Developer
   ↓
Team Lead
   ↓
Manager
```

The exception mechanism works similarly:

```text
methodC()
   ↓
methodB()
   ↓
methodA()
   ↓
main()
```

Each method gets an opportunity to handle the failure.

---

# 11. Your first production-level insight

Here's something I want you to start thinking about:

**The method where an exception occurs is not necessarily the method that should handle it.**

Example:

```java
Repository
    ↓
Service
    ↓
Controller
```

Database exception occurs here:

```text
Repository 💥
```

You might **not** want the repository to decide:

```text
HTTP 500
```

because the repository doesn't know about HTTP.

Instead:

```text
Repository
    ↓
Service
    ↓
Controller / Global Exception Handler
    ↓
HTTP response
```

The higher layer may have more context about what the failure means.

This idea becomes extremely important when we get into **Spring Boot exception architecture**.

---

## What you should know after Lesson 1

If these 5 things are clear, you've understood the foundation:

```text
1. Exception interrupts normal program execution.

2. Exceptions can propagate up through method calls.

3. try-catch allows a method to handle an exception.

4. Stack traces tell you how the program reached the failure.

5. The place where an exception occurs isn't always the
   correct place to handle it.
```

# Lesson 2 — Java Exception Hierarchy

Now let's understand **what kinds of exceptions exist in Java and why**.

This is important because later, when you see something like:

```java
catch (RuntimeException e)
```

you should understand **what you're actually catching**.

---

## 1. The top of the hierarchy: `Throwable`

Almost everything related to Java's exception mechanism starts here:

```text
Throwable
├── Error
└── Exception
```

`Throwable` is the parent class of both `Error` and `Exception`.

But **Error and Exception are not the same thing**.

---

# 2. `Error`

`Error` generally represents a serious problem related to the JVM or runtime environment.

For example:

```text
OutOfMemoryError
StackOverflowError
NoClassDefFoundError
```

Example:

```java
public class Test {

    static void call() {
        call();
    }

    public static void main(String[] args) {
        call();
    }
}
```

This can eventually cause:

```text
StackOverflowError
```

because the call stack keeps growing:

```text
call()
 ↓
call()
 ↓
call()
 ↓
call()
 ↓
...
 ↓
💥 StackOverflowError
```

### Should you normally catch `Error`?

Generally, **no**.

You usually don't write:

```java
catch (Error e) {
}
```

and try to continue normally.

Why?

Because errors such as `OutOfMemoryError` may indicate that the JVM is in a severely unhealthy state.

Think:

```text
Exception → application-level problem
Error     → serious JVM/runtime problem
```

There are advanced cases where infrastructure/framework code may catch certain `Error`s for cleanup or reporting, but normal application code should not treat `Error` like an ordinary recoverable exception.

---

# 3. `Exception`

Now we get to the part you'll use much more often:

```text
Throwable
    ↓
Exception
```

Examples:

```text
IOException
SQLException
RuntimeException
```

These generally represent problems that application code can potentially deal with.

---

# 4. `RuntimeException`

This is extremely important.

The hierarchy is:

```text
Exception
    ↓
RuntimeException
```

Common examples:

```text
NullPointerException
IllegalArgumentException
IllegalStateException
ArithmeticException
NumberFormatException
IndexOutOfBoundsException
```

Example:

```java
String name = null;

System.out.println(name.length());
```

Java throws:

```text
NullPointerException
```

Another:

```java
int age = Integer.parseInt("abc");
```

throws:

```text
NumberFormatException
```

---

# 5. The big concept: Checked vs Unchecked

This is probably the **most important part of today's lesson**.

Java exceptions are commonly divided into:

```text
Exception
│
├── RuntimeException
│      └── Unchecked exceptions
│
└── Other Exceptions
       └── Checked exceptions
```

So:

### Checked exception

Java forces you to deal with it.

### Unchecked exception

Java does **not** force you to deal with it.

Let's see what that means.

---

# 6. Checked exception

Consider:

```java
import java.io.FileReader;

public class Test {

    public static void main(String[] args) {

        FileReader file = new FileReader("data.txt");

    }
}
```

Java complains because opening a file can fail.

For example:

```text
File doesn't exist
Permission denied
Disk problem
```

So Java requires you to handle the exception.

You could do:

```java
try {
    FileReader file = new FileReader("data.txt");
} catch (Exception e) {
    System.out.println("Could not open file");
}
```

Or declare that the method passes responsibility to its caller:

```java
public static void readFile() throws Exception {
    FileReader file = new FileReader("data.txt");
}
```

That's a **checked exception**.

The compiler basically says:

> "Hey, this operation can fail. You need to make a decision about what to do."

---

# 7. Unchecked exception

Now:

```java
public static void main(String[] args) {

    String name = null;

    System.out.println(name.length());
}
```

Java doesn't force you to write:

```java
try {
    ...
} catch (NullPointerException e) {
    ...
}
```

The program compiles.

But when it runs:

```text
NullPointerException
```

That's an **unchecked exception**.

Why?

Because `NullPointerException` extends `RuntimeException`.

---

# 8. Why did Java design it this way?

This is where you should stop thinking:

> "Checked = this list of exceptions."

Instead think about **why**.

Imagine every possible programming mistake required:

```java
try {
    ...
} catch (...) {
}
```

For example:

```java
try {
    user.getName();
} catch (NullPointerException e) {
}
```

You'd have enormous amounts of meaningless exception handling.

Many `RuntimeException`s represent **programming bugs or invalid state** that should usually be fixed rather than caught locally.

For example:

```java
String name = null;
name.length();
```

The better solution is usually:

```text
Why is name null?
       ↓
Fix the bug
```

rather than:

```java
catch (NullPointerException)
```

and continue pretending everything is okay.

---

# 9. Checked exception example

Suppose you're reading a file:

```java
public String readConfig() throws IOException {
    return Files.readString(Path.of("config.txt"));
}
```

The operation depends on an external resource.

The file might:

```text
not exist
be inaccessible
be corrupted
be unavailable
```

So Java makes you acknowledge that possibility.

That's the philosophy behind checked exceptions.

---

# 10. Unchecked exception example

Now imagine:

```java
public void setAge(int age) {

    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }

    this.age = age;
}
```

This is an unchecked exception.

Why?

Because the caller supplied an invalid argument.

You don't necessarily want every caller forced to write:

```java
try {
    student.setAge(-10);
} catch (IllegalArgumentException e) {
}
```

Instead, the caller should normally **avoid passing invalid data** or let the error propagate to an appropriate boundary.

---

# 11. Very important: `throw` vs `throws`

You will see these everywhere.

### `throw`

Means:

> **I am throwing an exception right now.**

```java
throw new IllegalArgumentException("Invalid age");
```

### `throws`

Means:

> **This method may pass this exception to its caller.**

```java
public void readFile() throws IOException {
}
```

Think:

```text
throw  → actually throws
throws → declares possibility
```

We'll cover these in detail in a dedicated lesson.

---

# 12. One thing beginners often misunderstand

This:

```java
catch (Exception e)
```

does **not** mean:

> "Catch every possible Java problem."

It catches exceptions under `Exception`.

It does **not** catch things like:

```text
OutOfMemoryError
StackOverflowError
```

because those belong under:

```text
Error
```

not:

```text
Exception
```

---

# 13. The hierarchy you should memorize

Don't memorize hundreds of exception names.

Understand this:

```text
                    Throwable
                   /         \
                  /           \
               Error        Exception
                              /     \
                             /       \
                  RuntimeException   Checked
                       |
          ┌────────────┼──────────────┐
          ↓            ↓              ↓
       NPE          IllegalArg     IllegalState
```

More accurately, checked exceptions are the `Exception` subclasses that are **not** `RuntimeException` subclasses.

---

# 14. Production-level thinking

Here's where this becomes useful in backend development.

Imagine:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
PostgreSQL
```

Suppose PostgreSQL is unavailable.

That's fundamentally different from:

```java
user.getName()
```

causing:

```text
NullPointerException
```

The database failure may be:

```text
temporary
recoverable
retryable
infrastructure-related
```

The NPE is often:

```text
a programming bug
```

So experienced developers don't simply ask:

> "Is this an exception?"

They ask:

> **"What kind of failure is this, and what should the system do about it?"**

That's the mindset we'll develop throughout this course.

---

# 15. One more important rule

Don't blindly do this:

```java
catch (Exception e) {
    // handle everything
}
```

Why?

Because you're mixing potentially very different failures:

```text
Validation failure
Database failure
Network failure
Programming bug
Authentication failure
Configuration failure
```

These may require completely different behavior.

---

# Your Lesson 2 takeaway

You should now understand:

```text
Throwable
   │
   ├── Error
   │     └── Serious JVM/runtime problems
   │
   └── Exception
         │
         ├── RuntimeException
         │     └── Unchecked
         │
         └── Other Exception subclasses
               └── Checked
```

And remember these four ideas:

**1. Error ≠ Exception**

**2. RuntimeException = unchecked**

**3. Other direct/indirect Exception subclasses = checked (unless they extend RuntimeException)**

**4. Checked vs unchecked is about whether the compiler forces you to handle/declare it.**

---

### Next lesson: `try`, `catch`, and `finally`

We'll go much deeper than the basic syntax. We'll look at **what exactly executes, what happens when `catch` itself fails, multiple catch blocks, catch ordering, `finally`, and the tricky `return` + `finally` behavior** that shows up in real Java interviews and production debugging.

# Lesson 4 — `throw` vs `throws`

This is one of the most important Java exception concepts.

At first, these two look almost identical:

```java
throw
```

and

```java
throws
```

But they do **completely different jobs**.

---

# 1. `throw` — actually throws an exception

When you write:

```java
throw new IllegalArgumentException("Age cannot be negative");
```

you are saying:

> **"An error has happened right here. Stop normal execution and throw this exception."**

Example:

```java
public void setAge(int age) {

    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }

    System.out.println("Age is valid");
}
```

If you call:

```java
setAge(-5);
```

execution becomes:

```text
setAge(-5)
    ↓
age < 0
    ↓
throw
    ↓
IllegalArgumentException
    ↓
normal execution stops
```

The line:

```java
System.out.println("Age is valid");
```

doesn't execute.

---

# 2. `throws` — declares a possibility

Now look at:

```java
public void readFile() throws IOException {
    // ...
}
```

This means:

> **"This method may result in an IOException, and I'm telling the caller about it."**

It does **not** itself throw the exception.

Think:

```text
throw  → actually throw it
throws → declare that it may happen
```

---

# 3. Simple analogy

Imagine a building.

### `throw`

You press the fire alarm:

> 🔥 **"Fire! Something is wrong right now."**

### `throws`

You put a warning sign on the door:

> ⚠️ **"There may be a fire risk here."**

So:

```text
throw  = action
throws = declaration
```

---

# 4. Example with `throw`

```java
public void withdraw(double amount) {

    if (amount > 10000) {
        throw new IllegalArgumentException(
            "Maximum withdrawal is 10000"
        );
    }

    System.out.println("Withdrawal successful");
}
```

Calling:

```java
withdraw(15000);
```

causes:

```text
IllegalArgumentException
```

because **you explicitly told Java to throw it**.

---

# 5. Example with `throws`

Suppose we're reading a file:

```java
public String readFile() throws IOException {

    return Files.readString(
        Path.of("data.txt")
    );
}
```

Here:

```java
throws IOException
```

means:

> "The caller needs to know that this operation can result in `IOException`."

The caller can handle it:

```java
try {
    String data = readFile();
} catch (IOException e) {
    System.out.println("Could not read file");
}
```

So the flow is:

```text
readFile()
   ↓
may cause IOException
   ↓
throws IOException
   ↓
caller decides what to do
```

---

# 6. Very important: `throws` doesn't mean the exception will definitely happen

This:

```java
public void readFile() throws IOException {
}
```

doesn't mean:

> "An IOException will happen."

It means:

> **"An IOException is possible."**

Similar to a method saying:

```text
⚠️ This operation can fail in this particular way.
```

---

# 7. `throw` can throw only one exception at a time

For example:

```java
throw new IllegalArgumentException("Invalid age");
```

That's one exception object.

You don't write:

```java
throw new IllegalArgumentException(), IOException();
```

Instead, you could have different conditions:

```java
if (age < 0) {
    throw new IllegalArgumentException("Invalid age");
}

if (name == null) {
    throw new IllegalArgumentException("Name required");
}
```

---

# 8. `throws` can declare multiple exceptions

You can do:

```java
public void process()
        throws IOException, SQLException {

    // code
}
```

This says the method may pass either:

```text
IOException
```

or:

```text
SQLException
```

to its caller.

---

# 9. The most important example

Consider:

```java
public void process(int age) {

    if (age < 18) {
        throw new IllegalArgumentException("Must be 18+");
    }
}
```

Notice:

```java
throw
```

but no:

```java
throws
```

Why?

Because `IllegalArgumentException` is a **RuntimeException**.

It's unchecked.

You don't have to declare unchecked exceptions.

You *can* technically write:

```java
public void process(int age)
        throws IllegalArgumentException {
```

but normally you don't need to.

---

# 10. Checked exception example

Now:

```java
public void readFile() throws IOException {
    Files.readString(Path.of("data.txt"));
}
```

`IOException` is checked.

Therefore Java requires you to either:

### Handle it

```java
try {
    Files.readString(Path.of("data.txt"));
} catch (IOException e) {
    // handle
}
```

### Or declare it

```java
public void readFile() throws IOException {
    Files.readString(Path.of("data.txt"));
}
```

So for checked exceptions:

```text
                 IOException
                      ↓
             ┌────────┴────────┐
             ↓                 ↓
           catch             throws
        handle here      tell caller
```

---

# 11. A very common misunderstanding

People sometimes think:

```java
throws IOException
```

means:

> "Throw IOException."

No.

Compare:

```java
throw new IOException();
```

versus:

```java
throws IOException
```

The first **actually creates and throws** an exception.

The second **declares that the method may propagate one**.

---

# 12. Real backend example

Now let's move toward Spring Boot.

Imagine:

```text
Controller
    ↓
Service
    ↓
Payment Gateway
```

Suppose the payment gateway fails.

Your service might do:

```java
public PaymentResponse processPayment()
        throws PaymentException {

    // payment logic
}
```

Inside the method:

```java
if (paymentFailed) {
    throw new PaymentException("Payment failed");
}
```

Now notice the relationship:

```text
Method declaration:

throws PaymentException
       ↑
       │
Method can propagate it


Inside method:

throw new PaymentException(...)
       ↑
       │
Actually creates and throws it
```

This distinction is fundamental.

---

# 13. Custom exception

Now let's create our own.

```java
public class UserNotFoundException
        extends RuntimeException {

    public UserNotFoundException(String message) {
        super(message);
    }
}
```

Then:

```java
public User getUser(Long id) {

    User user = repository.findById(id)
            .orElseThrow(() ->
                new UserNotFoundException(
                    "User not found: " + id
                )
            );

    return user;
}
```

Here:

```java
new UserNotFoundException(...)
```

creates the exception.

And:

```java
throw
```

causes it to be thrown.

---

# 14. Why this is useful in production

Imagine:

```text
GET /users/100
```

but user 100 doesn't exist.

Instead of:

```java
return null;
```

you can do:

```java
throw new UserNotFoundException(
    "User not found: " + id
);
```

Then your global exception handler can convert it into:

```http
404 Not Found
```

For example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(
            UserNotFoundException e) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(e.getMessage());
    }
}
```

The architecture becomes:

```text
Client
  ↓
Controller
  ↓
Service
  ↓
User doesn't exist
  ↓
throw UserNotFoundException
  ↓
GlobalExceptionHandler
  ↓
HTTP 404
```

This is a very common production pattern in Spring Boot.

---

# 15. One senior-level concept: don't use exceptions for normal flow

Don't do this:

```java
try {
    User user = findUser(id);
} catch (UserNotFoundException e) {
    // user doesn't exist
}
```

everywhere just to check whether something exists.

If absence is a normal expected result, something like:

```java
Optional<User>
```

can sometimes be more appropriate.

Exceptions are generally better for **exceptional/error conditions**, not ordinary control flow.

---

# 16. Another important concept: preserve the original cause

Suppose:

```java
try {
    paymentGateway.call();
} catch (IOException e) {
    throw new PaymentException(
        "Payment gateway failed",
        e
    );
}
```

The second argument:

```java
e
```

is extremely important.

It preserves the original failure.

You get a chain:

```text
PaymentException
      ↓
caused by
      ↓
IOException
      ↓
original failure
```

Without the cause:

```java
throw new PaymentException("Payment failed");
```

you may lose valuable debugging information.

We'll study this properly when we cover **exception chaining and wrapping**.

---

# 17. `throw` + `throws` together

You can absolutely use both:

```java
public void validateUser(User user)
        throws UserValidationException {

    if (user == null) {
        throw new UserValidationException(
            "User cannot be null"
        );
    }
}
```

Here:

### `throws`

```java
throws UserValidationException
```

says:

> This method may propagate this exception.

### `throw`

```java
throw new UserValidationException(...)
```

says:

> The problem has happened now.

---

# 18. Remember this forever

```text
throw
│
└── "Throw this exception NOW."

throws
│
└── "This method MAY propagate this exception."
```

Example:

```java
public void test() throws IOException {

    if (somethingWrong) {
        throw new IOException("Something failed");
    }
}
```

Read it as:

> "This method can propagate `IOException`, and if `somethingWrong` happens, I actually throw one."

---

# 19. Quick test for you

What happens here?

```java
public void test(int age) {

    if (age < 18) {
        throw new IllegalArgumentException("Too young");
    }

    System.out.println("Accepted");
}
```

If we call:

```java
test(15);
```

Execution is:

```text
test(15)
   ↓
age < 18
   ↓
throw IllegalArgumentException
   ↓
"Accepted" does NOT execute
   ↓
exception propagates to caller
```

That's the core idea.

---

## Lesson 4 summary

| Keyword         | Meaning                                           |
| --------------- | ------------------------------------------------- |
| `throw`         | Actually throws an exception                      |
| `throws`        | Declares that a method may propagate an exception |
| `throw new X()` | Creates and throws exception `X`                  |
| `throws X`      | Tells caller that `X` may escape the method       |

### Next lesson: **Lesson 5 — Checked vs Unchecked Exceptions in depth**

We'll go beyond the definition and answer the questions that matter to a backend engineer:

**Why did Java create checked exceptions? Why does Spring Boot mostly use unchecked exceptions? When should you create checked vs unchecked custom exceptions? What happens to the caller when a checked exception is introduced into a method? And why do many experienced Java developers dislike checked exceptions in large applications?**


# Lesson 5 — Checked vs Unchecked Exceptions

This is one of the **most important topics in Java exception handling**.

Don't just memorize:

> Checked = compiler checks
> Unchecked = compiler doesn't check

That's only the beginning.

To think like an experienced Java developer, you need to understand **why this distinction exists and when to use each one.**

---

# 1. First, remember the hierarchy

```text
Throwable
│
├── Error
│
└── Exception
     │
     ├── RuntimeException
     │      ├── NullPointerException
     │      ├── IllegalArgumentException
     │      ├── IllegalStateException
     │      └── ...
     │
     └── Other Exceptions
            ├── IOException
            ├── SQLException
            └── ...
```

The important rule is:

```text
RuntimeException
      ↓
Unchecked exception

Exception subclasses that aren't RuntimeException
      ↓
Checked exception
```

---

# 2. What does "checked" actually mean?

Let's look at:

```java
import java.io.IOException;

public void readFile() throws IOException {
    // ...
}
```

Java's compiler sees:

```text
IOException
```

and says:

> "This is a checked exception. You must either handle it or declare it."

So this won't compile:

```java
public void readFile() {

    Files.readString(Path.of("data.txt"));

}
```

You need either:

### Option 1 — Handle it

```java
public void readFile() {

    try {
        Files.readString(Path.of("data.txt"));
    } catch (IOException e) {
        System.out.println("File reading failed");
    }

}
```

### Option 2 — Declare it

```java
public void readFile() throws IOException {

    Files.readString(Path.of("data.txt"));

}
```

That's what **checked** means.

The compiler forces you to make a decision.

---

# 3. What does "unchecked" mean?

Consider:

```java
public void test() {

    String name = null;

    System.out.println(name.length());

}
```

This can throw:

```text
NullPointerException
```

But Java doesn't say:

> "You must catch NullPointerException."

You can compile this perfectly.

That's because:

```text
NullPointerException
        ↓
RuntimeException
        ↓
Unchecked
```

---

# 4. Why doesn't Java force us to catch `NullPointerException`?

This is a very important question.

Imagine Java forced you to write:

```java
try {
    user.getName();
}
catch (NullPointerException e) {
    ...
}
```

every time you accessed an object.

That would be terrible.

Because often the correct solution isn't:

```text
catch the NPE
```

The correct solution is:

```text
WHY IS THIS OBJECT NULL?
        ↓
FIX THE BUG
```

For example:

```java
User user = userRepository.findById(id);

user.getName();
```

If `user` is unexpectedly `null`, catching the NPE doesn't necessarily fix the problem.

You should investigate why the object was null.

---

# 5. Checked exceptions usually represent external/expected failure possibilities

Consider:

```java
Files.readString(Path.of("data.txt"));
```

The file might genuinely not exist.

That's not necessarily a programming bug.

The environment can cause it:

```text
File missing
Permission denied
Disk problem
Network filesystem unavailable
```

So Java says:

> "This operation has a known failure possibility. Developer, make a decision."

That's the philosophy behind checked exceptions.

---

# 6. Another example: database

Imagine:

```java
connection.createStatement();
```

Database operations can fail because:

```text
Database unavailable
Connection lost
SQL problem
Network problem
Timeout
```

Historically, Java's JDBC APIs use checked:

```java
SQLException
```

So the developer must acknowledge the possibility.

---

# 7. But here's where real-world Java gets interesting

You might ask:

> "If checked exceptions are useful, why do modern Spring Boot applications often use `RuntimeException`?"

Excellent question.

Because large applications can become extremely verbose with checked exceptions.

Imagine:

```java
public void methodA() throws IOException {
    methodB();
}

public void methodB() throws IOException {
    methodC();
}

public void methodC() throws IOException {
    methodD();
}

public void methodD() throws IOException {
    // actual operation
}
```

Now `IOException` has to travel through every layer.

In a large backend:

```text
Controller
    ↓
Service
    ↓
Service
    ↓
Repository
    ↓
Infrastructure
```

you can end up declaring exceptions everywhere.

---

# 8. This is one reason unchecked exceptions are popular

Instead of:

```java
public User getUser(Long id)
        throws UserNotFoundException
```

you can have:

```java
public User getUser(Long id) {

    return repository.findById(id)
        .orElseThrow(() ->
            new UserNotFoundException(
                "User not found: " + id
            )
        );
}
```

where:

```java
public class UserNotFoundException
        extends RuntimeException {
}
```

No `throws` declaration is required.

The exception can travel:

```text
Repository
   ↓
Service
   ↓
Controller
   ↓
GlobalExceptionHandler
```

without putting `throws` on every method.

---

# 9. Does that mean unchecked exceptions are better?

**No.**

This is an important senior-level point.

Neither is universally better.

The choice depends on:

```text
What kind of failure is this?
Who should handle it?
Can the caller reasonably recover?
Is forcing handling useful?
Does declaring it improve the API?
```

Don't turn:

```text
checked = bad
unchecked = good
```

into a rule.

That's incorrect.

---

# 10. A useful way to think about checked exceptions

Suppose your method is:

```java
public String loadConfiguration()
        throws IOException
```

The caller may reasonably do something different depending on the failure:

```text
Configuration unavailable
        ↓
Use default configuration
```

That can be a legitimate recovery strategy.

So checked exceptions can make sense.

---

# 11. When unchecked exceptions make sense

Suppose:

```java
public void createEmployee(Employee employee)
```

and:

```java
if (employee.getSalary() < 0) {
    throw new IllegalArgumentException(
        "Salary cannot be negative"
    );
}
```

Making every caller write:

```java
try {
    service.createEmployee(employee);
}
catch (IllegalArgumentException e) {
}
```

usually isn't useful.

The application should validate its input appropriately and handle such failures at an appropriate boundary.

---

# 12. Custom checked exception

You can create one:

```java
public class PaymentException
        extends Exception {

    public PaymentException(String message) {
        super(message);
    }
}
```

Because it extends:

```text
Exception
```

and not:

```text
RuntimeException
```

it's checked.

Therefore:

```java
public void processPayment()
        throws PaymentException {
}
```

The caller has to handle or declare it.

---

# 13. Custom unchecked exception

More common in many Spring applications:

```java
public class PaymentException
        extends RuntimeException {

    public PaymentException(String message) {
        super(message);
    }
}
```

Now:

```java
public void processPayment() {
    throw new PaymentException("Payment failed");
}
```

No `throws` declaration is required.

---

# 14. Very important: `throws` does NOT make an exception checked

This is a common misunderstanding.

This:

```java
public void test()
        throws RuntimeException {
}
```

doesn't make `RuntimeException` checked.

`RuntimeException` is still unchecked.

And this:

```java
public void test()
        throws IOException {
}
```

doesn't make it checked either.

`IOException` is inherently a checked exception because of its class hierarchy.

The hierarchy determines it:

```text
IOException
   ↓
Exception
   ↓
NOT RuntimeException
   ↓
Checked
```

---

# 15. Let's see the compiler difference

### Checked

```java
void test() {
    throw new IOException();
}
```

Compilation error.

Why?

Because `IOException` is checked.

You need:

```java
void test() throws IOException {
    throw new IOException();
}
```

---

### Unchecked

```java
void test() {
    throw new RuntimeException();
}
```

Compiles.

Because:

```text
RuntimeException → unchecked
```

---

# 16. Production example

Imagine your attendance application:

```text
Employee
   ↓
Check-in Service
   ↓
Geofence validation
   ↓
Device validation
   ↓
Database
```

Suppose an employee doesn't exist.

You might create:

```java
public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

Then:

```java
Employee employee =
    employeeRepository.findById(employeeId)
        .orElseThrow(() ->
            new EmployeeNotFoundException(
                "Employee not found: " + employeeId
            )
        );
```

The exception travels upward:

```text
EmployeeNotFoundException
          ↓
Service
          ↓
Controller
          ↓
@RestControllerAdvice
          ↓
404 Not Found
```

That's a clean architecture.

---

# 17. But what if the database itself fails?

Suppose PostgreSQL becomes unavailable.

That's different:

```text
PostgreSQL
    ↓
connection timeout
    ↓
database exception
```

The service might translate the low-level infrastructure failure into something meaningful for the application.

For example:

```java
try {
    repository.save(employee);
} catch (DataAccessException e) {
    throw new EmployeePersistenceException(
        "Unable to save employee", e
    );
}
```

Notice:

```java
new EmployeePersistenceException(
    "Unable to save employee", e
)
```

We're preserving the original exception.

We'll study this deeply later.

---

# 18. A critical production rule

Don't use exceptions just because you can.

Bad design:

```java
try {
    User user = repository.findById(id).orElse(null);

    if (user == null) {
        throw new RuntimeException();
    }

} catch (Exception e) {
    return null;
}
```

You're creating an exception and immediately hiding it.

That's pointless.

Good exception handling should answer:

> **What useful decision does catching this exception allow me to make?**

---

# 19. Another senior-level insight

An exception's **type should communicate something useful**.

Compare:

```java
throw new RuntimeException("Something failed");
```

with:

```java
throw new EmployeeNotFoundException(
    "Employee 123 was not found"
);
```

The second tells you:

```text
What happened?
     ↓
Employee not found

Which entity?
     ↓
Employee

Which ID?
     ↓
123
```

This makes exception handling much easier.

---

# 20. The rule I want you to remember

Don't decide:

> "I'll always use RuntimeException."

Instead ask:

### Is this a programming error?

Usually:

```text
RuntimeException
```

Examples:

```text
NullPointerException
IllegalArgumentException
IllegalStateException
```

### Is this an external/operational condition where callers may reasonably be expected to recover?

A checked exception **may** be appropriate.

Examples traditionally include:

```text
IOException
```

But modern application architecture may still choose to translate such low-level checked exceptions into unchecked application-specific exceptions.

---

# 21. Interview-level question

### Why is `NullPointerException` unchecked?

Because it extends:

```text
RuntimeException
```

and Java doesn't require callers to catch or declare it.

Conceptually, NPE often indicates an unexpected programming/state problem rather than a failure that every caller can reasonably recover from.

---

# 22. Another interview question

### Can we create a checked exception?

Yes:

```java
class MyException extends Exception {
}
```

This is checked.

### Can we create an unchecked exception?

Yes:

```java
class MyException extends RuntimeException {
}
```

This is unchecked.

---

# 23. Your mental model

Keep this picture:

```text
                    Throwable
                       │
             ┌─────────┴─────────┐
             │                   │
           Error              Exception
                                 │
                       ┌─────────┴─────────┐
                       │                   │
                RuntimeException       Other
                       │              Exceptions
                       │                   │
                  UNCHECKED             CHECKED
```

And:

```text
CHECKED
   ↓
Compiler says:
"Handle it or declare it."

UNCHECKED
   ↓
Compiler says:
"I won't force you to handle it."
```

---

# 24. The experienced-engineer mindset

When you see an exception, don't immediately think:

```text
"Where should I put try-catch?"
```

Think:

```text
What failed?
     ↓
Is the failure expected?
     ↓
Can someone recover?
     ↓
Who has enough context to recover?
     ↓
Should I translate the exception?
     ↓
Should it become an API error?
     ↓
Should it trigger retry?
     ↓
Should the transaction rollback?
```

That's the difference between **knowing exception syntax** and **designing exception handling**.

---

## Lesson 5 complete ✅

You should now understand:

* Checked vs unchecked
* Why Java has both
* Why `RuntimeException` is unchecked
* Why `IOException` is checked
* How the compiler treats them differently
* How to create custom checked/unchecked exceptions
* Why Spring applications commonly use unchecked exceptions
* Why "unchecked is always better" is wrong
* How exception type communicates meaning

### Next → Lesson 6: Custom Exceptions + Exception Architecture

We'll build a realistic Spring Boot flow:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database

       💥
       ↓
Custom Exception
       ↓
Exception Translation
       ↓
@RestControllerAdvice
       ↓
Proper HTTP response
```

We'll also learn **where to create exceptions, where to throw them, where to catch them, where to log them, and where NOT to catch them**.


# Lesson 6 — Custom Exceptions & Exception Architecture

Now we move from **“how exceptions work”** to **“how experienced backend engineers design exceptions.”**

This is where exception handling becomes really important in **Spring Boot, REST APIs, databases, payments, attendance systems, and microservices**.

---

## 1. Why do we need Custom Exceptions?

Suppose you write:

```java
if (employee == null) {
    throw new RuntimeException("Employee not found");
}
```

This works.

But imagine your application has:

```java
RuntimeException("Employee not found")
RuntimeException("Email already exists")
RuntimeException("Device not registered")
RuntimeException("Payment failed")
RuntimeException("Attendance already marked")
```

Everything is just `RuntimeException`.

That's a problem.

Your application cannot clearly distinguish:

```text
Employee not found
        ↓
Email already exists
        ↓
Payment failed
        ↓
Database failed
```

A better design is:

```java
EmployeeNotFoundException
DuplicateEmailException
DeviceNotRegisteredException
PaymentFailedException
DatabaseOperationException
```

Now the exception itself communicates **what went wrong**.

---

# 2. Creating a Custom Exception

Let's create:

```java
public class EmployeeNotFoundException extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

Now we can do:

```java
if (employee == null) {
    throw new EmployeeNotFoundException("Employee not found");
}
```

Instead of:

```java
throw new RuntimeException("Employee not found");
```

The first version is much more meaningful.

---

# 3. Why `super(message)`?

Remember the parent class:

```java
RuntimeException
```

has constructors that accept a message.

When we write:

```java
super(message);
```

we are sending our message to `RuntimeException`.

So:

```java
throw new EmployeeNotFoundException("Employee 123 not found");
```

stores:

```text
Employee 123 not found
```

inside the exception.

Then:

```java
e.getMessage()
```

returns:

```text
Employee 123 not found
```

---

# 4. Real Application Example

Imagine your service:

```java
@Service
public class EmployeeService {

    public Employee getEmployee(UUID id) {

        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() ->
                        new EmployeeNotFoundException(
                                "Employee not found: " + id
                        )
                );

        return employee;
    }
}
```

This is much better than:

```java
throw new RuntimeException("Something went wrong");
```

because the exception tells us exactly what happened.

---

# 5. Don't Create One Giant Exception

A beginner might create:

```java
ApplicationException
```

and use it everywhere:

```java
throw new ApplicationException("Employee not found");

throw new ApplicationException("Email already exists");

throw new ApplicationException("Payment failed");

throw new ApplicationException("Device not found");
```

This loses useful information.

Instead:

```text
Application
│
├── EmployeeNotFoundException
├── DuplicateEmailException
├── DeviceNotFoundException
├── PaymentFailedException
└── AttendanceAlreadyMarkedException
```

Now different problems can have different handling.

---

# 6. Exception Categories

A production application often has different categories.

For example:

```text
Exceptions
│
├── Domain Exceptions
│
├── Application Exceptions
│
└── Infrastructure Exceptions
```

Let's understand them.

---

## 7. Domain Exception

A **domain exception** means:

> The business rule does not allow this operation.

Example:

```java
public class AttendanceAlreadyMarkedException
        extends RuntimeException {

    public AttendanceAlreadyMarkedException(String message) {
        super(message);
    }
}
```

Then:

```java
if (attendanceAlreadyMarked) {
    throw new AttendanceAlreadyMarkedException(
            "Attendance already marked for today"
    );
}
```

This isn't a database problem.

It isn't a network problem.

It's a **business rule violation**.

---

# 8. Another Domain Example

Imagine an employee cannot check in outside the allowed location.

```java
if (!isInsideGeofence(employee, location)) {
    throw new OutsideAllowedLocationException(
            "Employee is outside the allowed check-in location"
    );
}
```

Again, this is a business/domain rule.

---

# 9. Infrastructure Exception

Now imagine your application calls a payment provider:

```java
paymentClient.processPayment();
```

The external API fails:

```text
Connection timeout
HTTP 503
Connection refused
```

That's an infrastructure/external dependency problem.

You might create:

```java
public class PaymentGatewayException
        extends RuntimeException {

    public PaymentGatewayException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Then:

```java
try {

    paymentClient.processPayment();

} catch (Exception e) {

    throw new PaymentGatewayException(
            "Payment gateway failed",
            e
    );
}
```

Notice something important:

```java
e
```

is passed as the **cause**.

---

# 10. Why Preserve the Cause?

Suppose the original exception is:

```text
SocketTimeoutException
```

If you do this:

```java
catch (Exception e) {
    throw new PaymentGatewayException("Payment failed");
}
```

you lose the original cause.

Bad.

Instead:

```java
catch (Exception e) {
    throw new PaymentGatewayException(
            "Payment gateway failed",
            e
    );
}
```

Now the chain is:

```text
PaymentGatewayException
        ↓
SocketTimeoutException
```

This is called **exception chaining**.

We'll study this deeply later.

---

# 11. Very Important Production Rule

Don't leak infrastructure details to your API users.

Suppose PostgreSQL produces:

```text
ERROR: duplicate key value violates unique constraint
```

You generally don't want your API to respond:

```json
{
  "message": "ERROR: duplicate key value violates unique constraint..."
}
```

That exposes implementation details.

Instead:

```json
{
  "status": 409,
  "code": "EMAIL_ALREADY_EXISTS",
  "message": "Email already exists"
}
```

The client doesn't need to know PostgreSQL's internal error.

---

# 12. Exception Translation

This is a **very important production concept**.

Suppose:

```text
PostgreSQL
    ↓
Hibernate
    ↓
Repository
    ↓
Service
    ↓
Controller
```

The database might throw something like:

```text
DataIntegrityViolationException
```

Your business layer shouldn't necessarily need to understand database-specific details.

You can translate:

```text
Database exception
        ↓
Application-specific exception
```

For example:

```java
try {

    employeeRepository.save(employee);

} catch (DataIntegrityViolationException e) {

    throw new DuplicateEmailException(
            "Employee email already exists",
            e
    );
}
```

Now the service/application understands:

```text
DuplicateEmailException
```

instead of:

```text
PostgreSQL/Hibernate implementation details
```

That's called **exception translation**.

---

# 13. Where Should You Throw Exceptions?

A useful rule:

### Business rule failure → Service/domain layer

Example:

```java
if (employeeAlreadyCheckedIn) {
    throw new AttendanceAlreadyMarkedException(
            "Attendance already marked"
    );
}
```

### Resource not found → Service layer

```java
throw new EmployeeNotFoundException(
        "Employee not found"
);
```

### External service failure → Integration/client layer

```java
throw new PaymentGatewayException(
        "Payment provider unavailable",
        e
);
```

### HTTP response conversion → Web/controller layer

This is where:

```text
Exception
    ↓
HTTP status
    ↓
JSON response
```

should happen.

---

# 14. Enter `@RestControllerAdvice`

Now we reach one of the most important Spring Boot concepts.

Instead of doing this in every controller:

```java
try {
    ...
} catch (EmployeeNotFoundException e) {
    return ResponseEntity.status(404)...
}
```

we can create one global exception handler.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<?> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(e.getMessage());
    }
}
```

Now:

```text
Controller
    ↓
Service
    ↓
EmployeeNotFoundException
    ↓
GlobalExceptionHandler
    ↓
HTTP 404
```

The controller doesn't need to catch it.

---

# 15. A Better Error Response

Instead of returning only:

```text
Employee not found
```

production APIs often return structured data.

For example:

```java
public record ErrorResponse(
        int status,
        String code,
        String message,
        Instant timestamp
) {
}
```

Then:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        ErrorResponse response = new ErrorResponse(
                404,
                "EMPLOYEE_NOT_FOUND",
                e.getMessage(),
                Instant.now()
        );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(response);
    }
}
```

Response:

```json
{
  "status": 404,
  "code": "EMPLOYEE_NOT_FOUND",
  "message": "Employee not found",
  "timestamp": "2026-09-11T12:30:00Z"
}
```

---

# 16. Why Have Both `status` and `code`?

This is a senior-level API design detail.

Don't make frontend developers depend on:

```text
message = "Employee not found"
```

because the message might change.

Instead:

```json
{
    "code": "EMPLOYEE_NOT_FOUND",
    "message": "Employee not found"
}
```

Frontend can reliably check:

```text
EMPLOYEE_NOT_FOUND
```

The message is for humans.

The code is for machines.

---

# 17. Mapping Exceptions to HTTP Status

A common pattern:

| Exception                 |    HTTP |
| ------------------------- | ------: |
| EmployeeNotFoundException |     404 |
| DuplicateEmailException   |     409 |
| InvalidRequestException   |     400 |
| UnauthorizedException     |     401 |
| ForbiddenException        |     403 |
| PaymentGatewayException   | 502/503 |
| UnexpectedException       |     500 |

For example:

```text
Employee doesn't exist
        ↓
404 Not Found
```

```text
Email already exists
        ↓
409 Conflict
```

```text
Invalid request
        ↓
400 Bad Request
```

```text
Payment provider unavailable
        ↓
502/503
```

---

# 18. Real Attendance System Architecture

Let's apply everything to your attendance backend.

Imagine:

```text
POST /attendance/check-in
```

Request:

```json
{
    "employeeId": "...",
    "deviceId": "...",
    "latitude": 23.8103,
    "longitude": 90.4125
}
```

Flow:

```text
Controller
    ↓
AttendanceService
    ↓
EmployeeRepository
    ↓
DeviceService
    ↓
Geofence validation
    ↓
AttendanceRepository
```

Different things can fail.

### Employee doesn't exist

```java
throw new EmployeeNotFoundException(
        "Employee not found"
);
```

→ `404`

### Device doesn't exist

```java
throw new DeviceNotFoundException(
        "Device not registered"
);
```

→ `404`

### Already checked in

```java
throw new AttendanceAlreadyMarkedException(
        "Attendance already marked"
);
```

→ `409`

### Outside geofence

```java
throw new OutsideAllowedLocationException(
        "Employee is outside allowed location"
);
```

→ `422` or your chosen validation/business-rule status

### Database unavailable

```text
Database exception
        ↓
DatabaseOperationException
        ↓
Global handler
        ↓
503
```

This is how a real application starts becoming maintainable.

---

# 19. Don't Catch Everything

This is bad:

```java
try {

    employeeService.create(employee);

} catch (Exception e) {

    System.out.println("Something went wrong");

}
```

Why?

You just swallowed the problem.

The request might appear successful even though the operation failed.

---

# 20. This Is Also Bad

```java
try {
    employeeService.create(employee);
} catch (Exception e) {
    throw e;
}
```

Why catch it?

You didn't add anything.

Just allow it to propagate.

Better:

```java
employeeService.create(employee);
```

---

# 21. When Should You Catch?

Catch an exception when you can **do something meaningful**.

For example:

### Translate it

```java
catch (SQLException e) {
    throw new DatabaseOperationException(
            "Unable to save employee",
            e
    );
}
```

### Recover

```java
catch (TemporaryServiceException e) {
    return cachedValue;
}
```

### Retry

```text
Temporary network failure
        ↓
Retry
        ↓
Success
```

### Add meaningful context

```java
catch (IOException e) {
    throw new EmployeeImportException(
            "Failed while importing employee CSV",
            e
    );
}
```

Otherwise, let it propagate.

---

# 22. Don't Log the Same Exception Everywhere

Suppose:

```text
Repository
    ↓ logs exception

Service
    ↓ logs exception

Controller
    ↓ logs exception

Global Handler
    ↓ logs exception
```

Your logs may contain the same stack trace four times.

Instead, establish a clear logging boundary.

For example:

```java
log.error(
    "Unexpected error while processing check-in",
    exception
);
```

at the appropriate application boundary.

---

# 23. A Good Custom Exception Structure

For important exceptions, you can support both message-only and cause constructors:

```java
public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }

    public EmployeeNotFoundException(
            String message,
            Throwable cause) {

        super(message, cause);
    }
}
```

Now you can use:

```java
throw new EmployeeNotFoundException(
        "Employee not found"
);
```

or:

```java
throw new EmployeeNotFoundException(
        "Failed to retrieve employee",
        e
);
```

---

# 24. A Production-Level Mental Model

When an exception occurs, ask these questions:

```text
1. What actually failed?

2. Is this expected or unexpected?

3. Is it a business-rule failure?

4. Is it an infrastructure failure?

5. Can I recover?

6. Should I retry?

7. Should I translate the exception?

8. Who has enough context to handle it?

9. What should the API client see?

10. What should the logs contain?
```

This way of thinking is more valuable than memorizing `try/catch`.

---

# 25. The Architecture You Should Remember

A good Spring Boot exception flow looks like:

```text
                    CLIENT
                       │
                       ▼
                 CONTROLLER
                       │
                       ▼
                   SERVICE
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        REPOSITORY          EXTERNAL API
             │                   │
             ▼                   ▼
          DATABASE          PAYMENT SERVICE
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
                Exception occurs
                       │
                       ▼
             Exception Translation
                       │
                       ▼
             Custom Application
                  Exception
                       │
                       ▼
             @RestControllerAdvice
                       │
                       ▼
                  HTTP Response
                       │
                       ▼
                     CLIENT
```

That's the architecture you should start thinking about.

---

# 26. One Important Rule to Remember

Don't think:

> **"Where can I catch this exception?"**

Instead think:

> **"Which layer has enough information to make the correct decision?"**

That's a much better engineering question.

For example:

```text
Repository
```

knows about database errors.

But it may not know:

```text
What should the user see?
```

The service knows the business meaning.

The global handler knows:

```text
What HTTP response should be returned?
```

So responsibilities stay separated.

---

# 27. Lesson 6 Summary

Today you learned:

```text
Custom Exceptions
        ↓
Why RuntimeException isn't enough
        ↓
Domain Exceptions
        ↓
Infrastructure Exceptions
        ↓
Exception Chaining
        ↓
Exception Translation
        ↓
Where to throw
        ↓
Where to catch
        ↓
@RestControllerAdvice
        ↓
HTTP error mapping
        ↓
Structured ErrorResponse
        ↓
Error codes
        ↓
Production exception architecture
```

### The biggest takeaway:

> **An exception is not just an error message. It is part of your application's architecture and communicates what failed, where it belongs, and what the next layer should do about it.**

---

## Next: Lesson 7

**Lesson 7 will be very important for real Spring Boot development:**

### `@RestControllerAdvice` + `@ExceptionHandler` Deep Dive

We'll build a **real global exception handling system** from scratch, including:

```text
EmployeeNotFoundException
DuplicateEmailException
ValidationException
DatabaseException
AuthenticationException
UnexpectedException
        ↓
GlobalExceptionHandler
        ↓
Consistent JSON API response
```

We'll also cover **exception handling for `@Valid`, request validation, database errors, 401/403/404/409/500, and how production Spring Boot APIs should structure error responses.**

# Lesson 7 — `@RestControllerAdvice` & Global Exception Handling

This is one of the **most important exception-handling topics for a Spring Boot backend developer**.

In the previous lesson, you learned:

```text
Exception
   ↓
Custom Exception
   ↓
Global Handler
   ↓
HTTP Response
```

Now we're going to build that system properly.

---

# 1. The Problem Without Global Exception Handling

Imagine you have three controllers:

```java
EmployeeController
AttendanceController
PaymentController
```

Without a global handler, you might write:

```java
@RestController
public class EmployeeController {

    @GetMapping("/employees/{id}")
    public ResponseEntity<?> getEmployee(@PathVariable UUID id) {

        try {
            return ResponseEntity.ok(
                    employeeService.getEmployee(id)
            );

        } catch (EmployeeNotFoundException e) {

            return ResponseEntity
                    .status(404)
                    .body(e.getMessage());
        }
    }
}
```

Then another controller:

```java
@RestController
public class AttendanceController {

    @PostMapping("/attendance")
    public ResponseEntity<?> checkIn() {

        try {
            ...
        } catch (EmployeeNotFoundException e) {
            ...
        }
    }
}
```

And another:

```text
PaymentController
    ↓
try/catch

DeviceController
    ↓
try/catch

PayrollController
    ↓
try/catch
```

This becomes messy very quickly.

---

# 2. The Solution

Spring provides:

```java
@RestControllerAdvice
```

It allows you to create **one centralized place** for handling exceptions from your controllers.

Think of it as:

```text
                 Controllers
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
   Employee      Attendance     Payment
   Controller    Controller     Controller
       │             │             │
       └─────────────┼─────────────┘
                     ↓
                Exception
                     ↓
          GlobalExceptionHandler
                     ↓
               HTTP Response
```

---

# 3. `@ExceptionHandler`

Inside the global handler, we use:

```java
@ExceptionHandler
```

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<?> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(e.getMessage());
    }
}
```

Now whenever:

```java
throw new EmployeeNotFoundException("Employee not found");
```

happens during a controller request, Spring can route it to:

```java
handleEmployeeNotFound()
```

---

# 4. Complete Flow

Suppose the client sends:

```http
GET /employees/123
```

The flow is:

```text
Client
  ↓
EmployeeController
  ↓
EmployeeService
  ↓
EmployeeRepository
  ↓
Employee doesn't exist
  ↓
EmployeeNotFoundException
  ↓
GlobalExceptionHandler
  ↓
HTTP 404
  ↓
Client
```

The controller doesn't need:

```java
try {
   ...
} catch (...) {
   ...
}
```

That's the major benefit.

---

# 5. Let's Build It Properly

First, our custom exception:

```java
public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

Service:

```java
@Service
public class EmployeeService {

    public Employee getEmployee(UUID id) {

        return employeeRepository.findById(id)
                .orElseThrow(() ->
                        new EmployeeNotFoundException(
                                "Employee not found: " + id
                        )
                );
    }
}
```

Controller:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(
            @PathVariable UUID id) {

        return employeeService.getEmployee(id);
    }
}
```

Notice something.

There is **no try/catch**.

That's intentional.

---

# 6. Global Handler

Now:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<?> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(e.getMessage());
    }
}
```

Now Spring handles it globally.

---

# 7. But Don't Return Just a String

This:

```json
"Employee not found"
```

isn't ideal for a production API.

Instead create an error DTO.

```java
public record ErrorResponse(
        int status,
        String code,
        String message,
        Instant timestamp
) {
}
```

Now:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        ErrorResponse response = new ErrorResponse(
                404,
                "EMPLOYEE_NOT_FOUND",
                e.getMessage(),
                Instant.now()
        );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(response);
    }
}
```

Response:

```json
{
  "status": 404,
  "code": "EMPLOYEE_NOT_FOUND",
  "message": "Employee not found: 123",
  "timestamp": "2026-09-11T12:30:00Z"
}
```

This is much better.

---

# 8. Why `code` Is Important

Consider this:

```json
{
    "message": "Employee not found"
}
```

Tomorrow you change the message:

```json
{
    "message": "No employee exists with this ID"
}
```

Frontend code that depends on the message can break.

Instead:

```json
{
    "code": "EMPLOYEE_NOT_FOUND",
    "message": "No employee exists with this ID"
}
```

The frontend can always depend on:

```text
EMPLOYEE_NOT_FOUND
```

So:

> **Message = for humans**

> **Code = for machines**

This is a very useful production pattern.

---

# 9. Multiple Exceptions

A real application has many exceptions.

For example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFound(
            EmployeeNotFoundException e) {

        return buildResponse(
                HttpStatus.NOT_FOUND,
                "EMPLOYEE_NOT_FOUND",
                e.getMessage()
        );
    }

    @ExceptionHandler(DuplicateEmailException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateEmail(
            DuplicateEmailException e) {

        return buildResponse(
                HttpStatus.CONFLICT,
                "EMAIL_ALREADY_EXISTS",
                e.getMessage()
        );
    }
}
```

We can make a helper:

```java
private ResponseEntity<ErrorResponse> buildResponse(
        HttpStatus status,
        String code,
        String message) {

    ErrorResponse response = new ErrorResponse(
            status.value(),
            code,
            message,
            Instant.now()
    );

    return ResponseEntity
            .status(status)
            .body(response);
}
```

Now the handler stays clean.

---

# 10. HTTP Status Mapping

Let's understand common mappings.

### 404 — Not Found

```text
EmployeeNotFoundException
```

Example:

```text
Employee ID doesn't exist
```

---

### 409 — Conflict

```text
DuplicateEmailException
AttendanceAlreadyMarkedException
```

Example:

```text
Email already exists
```

---

### 400 — Bad Request

The client sent invalid input.

```text
InvalidRequestException
```

---

### 401 — Unauthorized

The client isn't authenticated.

```text
AuthenticationException
```

---

### 403 — Forbidden

The client is authenticated but doesn't have permission.

```text
AccessDeniedException
```

---

### 500 — Internal Server Error

Something unexpected happened.

```text
NullPointerException
Unexpected database problem
Programming bug
```

But be careful.

You generally **don't want to expose**:

```text
NullPointerException
at EmployeeService.java:87
```

to the client.

---

# 11. Handling Validation Errors

This is another very important Spring Boot case.

Suppose:

```java
public class EmployeeRequest {

    @NotBlank
    private String name;

    @Email
    private String email;

}
```

Controller:

```java
@PostMapping
public Employee create(
        @Valid @RequestBody EmployeeRequest request) {

    return employeeService.create(request);
}
```

The client sends:

```json
{
    "name": "",
    "email": "hello"
}
```

Spring detects validation errors.

You can handle them globally.

Depending on your Spring version and validation setup, common exceptions include:

```text
MethodArgumentNotValidException
HandlerMethodValidationException
ConstraintViolationException
```

For request-body validation:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException e) {

    String message = e.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error ->
                    error.getField() + ": " + error.getDefaultMessage()
            )
            .findFirst()
            .orElse("Invalid request");

    return buildResponse(
            HttpStatus.BAD_REQUEST,
            "VALIDATION_ERROR",
            message
    );
}
```

Now instead of a confusing Spring error, your API can return:

```json
{
    "status": 400,
    "code": "VALIDATION_ERROR",
    "message": "email: must be a well-formed email address",
    "timestamp": "2026-09-11T12:30:00Z"
}
```

---

# 12. The Dangerous Catch-All Handler

You will often see:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handleException(Exception e) {
    ...
}
```

This can be useful.

But don't use it as a replacement for proper exception design.

Think of it as the **last safety net**.

For example:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleUnexpectedException(
        Exception e) {

    log.error("Unexpected application error", e);

    return buildResponse(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "INTERNAL_SERVER_ERROR",
            "An unexpected error occurred"
    );
}
```

Notice what the user receives:

```text
An unexpected error occurred
```

But your server logs contain:

```text
NullPointerException
    at EmployeeService...
    at EmployeeController...
```

That's exactly what we want.

---

# 13. Never Do This

Bad:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handle(Exception e) {

    return ResponseEntity
            .status(500)
            .body(e.getMessage());
}
```

Why?

Because `e.getMessage()` might contain:

```text
SQL connection failed: password authentication failed...
```

or:

```text
SELECT * FROM employees WHERE...
```

or other internal information.

The client doesn't need that.

---

# 14. Production Rule

A very important separation:

```text
                    SERVER
                      │
                      │ detailed
                      ▼
                    LOGS
                      │
                      │
                      │ safe
                      ▼
                    CLIENT
```

### Logs

Can contain:

```text
Exception type
Stack trace
Correlation ID
Request information
Internal cause
```

### Client response

Should contain:

```text
HTTP status
Stable error code
Safe human-readable message
Timestamp
Possibly request/correlation ID
```

Don't expose your internal architecture to the client.

---

# 15. Add a Request ID

In production systems, a very useful field is:

```text
traceId
```

or:

```text
requestId
```

Example:

```json
{
    "status": 500,
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred",
    "requestId": "a83f91c2",
    "timestamp": "2026-09-11T12:30:00Z"
}
```

Then the user can tell support:

> "I received error `a83f91c2`."

You search your logs:

```text
a83f91c2
```

and find the exact failure.

This becomes extremely valuable in production.

We'll cover **logging, correlation IDs, tracing, and observability** later.

---

# 16. Should Every Exception Have a Custom Handler?

No.

You don't necessarily need:

```text
One exception
     ↓
One handler
```

for everything.

For example, you might group related exceptions.

```java
@ExceptionHandler({
        EmployeeNotFoundException.class,
        DeviceNotFoundException.class
})
public ResponseEntity<ErrorResponse> handleNotFound(
        RuntimeException e) {

    return buildResponse(
            HttpStatus.NOT_FOUND,
            "RESOURCE_NOT_FOUND",
            e.getMessage()
    );
}
```

But there's a trade-off.

If your API needs different error codes:

```text
EMPLOYEE_NOT_FOUND
DEVICE_NOT_FOUND
```

then separate handlers or a more structured exception model may be better.

---

# 17. A Better Exception Design

You can introduce an application base exception:

```java
public abstract class ApplicationException
        extends RuntimeException {

    private final String code;

    protected ApplicationException(
            String code,
            String message) {

        super(message);
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```

Then:

```java
public class EmployeeNotFoundException
        extends ApplicationException {

    public EmployeeNotFoundException(String message) {
        super("EMPLOYEE_NOT_FOUND", message);
    }
}
```

And:

```java
public class DuplicateEmailException
        extends ApplicationException {

    public DuplicateEmailException(String message) {
        super("EMAIL_ALREADY_EXISTS", message);
    }
}
```

Now the exception itself knows its API/business error code.

Then the handler can be simpler:

```java
@ExceptionHandler(ApplicationException.class)
public ResponseEntity<ErrorResponse> handleApplicationException(
        ApplicationException e) {

    ErrorResponse response = new ErrorResponse(
            400,
            e.getCode(),
            e.getMessage(),
            Instant.now()
    );

    return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(response);
}
```

However, don't blindly map **all** application exceptions to `400`.

The status may depend on the exception.

We'll improve this architecture in later lessons.

---

# 18. Attendance System Example

Let's design your attendance API.

Suppose:

```http
POST /api/attendance/check-in
```

Potential failures:

### Employee doesn't exist

```java
throw new EmployeeNotFoundException(
    "Employee not found"
);
```

Response:

```json
{
    "status": 404,
    "code": "EMPLOYEE_NOT_FOUND",
    "message": "Employee not found"
}
```

---

### Device isn't registered

```java
throw new DeviceNotFoundException(
    "Device is not registered"
);
```

Response:

```json
{
    "status": 404,
    "code": "DEVICE_NOT_FOUND",
    "message": "Device is not registered"
}
```

---

### Attendance already exists

```java
throw new AttendanceAlreadyMarkedException(
    "Attendance already marked for today"
);
```

Response:

```json
{
    "status": 409,
    "code": "ATTENDANCE_ALREADY_MARKED",
    "message": "Attendance already marked for today"
}
```

---

### Outside allowed location

```java
throw new OutsideAllowedLocationException(
    "Employee is outside the allowed location"
);
```

Response:

```json
{
    "status": 422,
    "code": "OUTSIDE_ALLOWED_LOCATION",
    "message": "Employee is outside the allowed location"
}
```

---

# 19. Notice What We Didn't Do

We didn't write:

```java
try {
    employeeRepository.findById(...)
} catch (...) {
    ...
}
```

inside every method.

We didn't write:

```java
try {
    attendanceService.checkIn(...)
} catch (...) {
    ...
}
```

inside every controller.

Instead:

```text
Service
   ↓
throws meaningful exception
   ↓
Global Handler
   ↓
converts it to HTTP response
```

This keeps the business logic clean.

---

# 20. One of the Biggest Mistakes Beginners Make

They think:

> "Exception handling means putting try/catch everywhere."

No.

That's not good exception handling.

Good exception handling means:

```text
Detect failure
     ↓
Classify failure
     ↓
Decide who can handle it
     ↓
Recover / translate / propagate
     ↓
Return appropriate response
     ↓
Log useful information
```

Sometimes the correct action is:

```text
DO NOTHING
```

and simply allow the exception to propagate to the appropriate layer.

---

# 21. The Full Production Flow

You should now visualize your Spring Boot application like this:

```text
                 HTTP REQUEST
                      │
                      ▼
                 Controller
                      │
                      ▼
                   Service
                      │
              ┌───────┴────────┐
              ▼                ▼
          Database         External API
              │                │
              └───────┬────────┘
                      │
                  Failure
                      │
                      ▼
              Custom Exception
                      │
                      ▼
              Propagation
                      │
                      ▼
          @RestControllerAdvice
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Logging          ErrorResponse
                               │
                               ▼
                         HTTP Response
                               │
                               ▼
                            Client
```

This is the core architecture.

---

# 22. Your Lesson 7 Mental Model

Remember these four concepts:

### 1. `@RestControllerAdvice`

> Central place for controller exception handling.

### 2. `@ExceptionHandler`

> Tells Spring which method handles which exception.

### 3. `ErrorResponse`

> Defines a consistent API error format.

### 4. Exception mapping

```text
Exception
    ↓
Meaning
    ↓
HTTP status
    ↓
Error code
    ↓
Safe response
```

---

# 23. What You Should Be Able to Build Now

At this point, you should be able to create:

```text
EmployeeNotFoundException
DuplicateEmailException
DeviceNotFoundException
AttendanceAlreadyMarkedException
PaymentFailedException
```

and then:

```text
                    Exceptions
                        ↓
              GlobalExceptionHandler
                        ↓
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
       404             409             500
     Not Found       Conflict       Internal Error
```

without putting `try/catch` in every controller.

---

# 24. Lesson 7 Summary

You learned:

* What `@RestControllerAdvice` does
* What `@ExceptionHandler` does
* Why global exception handling is useful
* How exceptions propagate from service to controller boundary
* How to create a consistent `ErrorResponse`
* Why error codes are better than relying on messages
* How to handle validation errors
* How to handle unexpected exceptions
* Why stack traces should stay on the server
* Why database/internal details shouldn't be exposed
* HTTP status mapping
* Request/correlation IDs
* How this applies to your attendance system
* Why `try/catch` everywhere is **not** good exception handling

### The key principle:

> **Business layers should focus on business logic. The global exception handler should focus on converting failures into safe, consistent API responses.**

---

## Next — Lesson 8

We'll go deeper into something that separates **basic Java developers from experienced backend engineers**:

### **Exception Chaining, Wrapping & Translation**

We'll study:

```text
SQLException
     ↓
DataAccessException
     ↓
EmployeePersistenceException
     ↓
GlobalExceptionHandler
     ↓
HTTP 500/503
```

and, more importantly, **when you should wrap an exception, when you should NOT wrap it, how `cause` works, and how to preserve the original stack trace while changing the abstraction.**
