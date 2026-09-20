# Java Generics — Lesson 1

## Why Generics Exist: From `Object` to Type Safety

We’ll learn **Java Generics** step-by-step from Junior → Senior → Production level.

For this first lesson, we will **not** jump into wildcards, bounds, PECS, type erasure internals, generic methods, or advanced patterns.

First, build the correct mental model.

---

# 1. What problem do Generics solve?

Before generics, Java collections commonly looked like this:

```java
List users = new ArrayList();

users.add("Arup");
users.add(100);
users.add(true);
```

Java allows different types because the collection effectively works with `Object`.

Now imagine:

```java
String name = (String) users.get(1);
```

What happens?

```text
100
 ↓
cast to String
 ↓
ClassCastException 💥
```

The compiler didn't protect us.

This is one of the main problems Generics were designed to solve.

---

# 2. The same code with Generics

Now:

```java
List<String> users = new ArrayList<>();

users.add("Arup");
users.add("Rahim");
```

Try this:

```java
users.add(100);
```

The compiler rejects it.

That's the big idea:

> **Generics allow you to tell the compiler what type a class, collection, or method is supposed to work with.**

Instead of:

```java
List
```

we can say:

```java
List<String>
```

Meaning:

> "This List is intended to contain Strings."

---

# 3. Why is this better?

Compare the two.

### Without Generics

```java
List users = new ArrayList();

users.add("Arup");
users.add(100);

String name = (String) users.get(1);
```

Problem:

```text
Runtime
   ↓
ClassCastException
```

### With Generics

```java
List<String> users = new ArrayList<>();

users.add("Arup");
users.add(100);
```

Problem is detected during:

```text
Compilation
    ↓
ERROR
```

This is extremely important.

### Generics move many type errors from runtime to compile time.

That means:

**Earlier failure → easier debugging → safer code**

---

# 4. What does `<String>` actually mean?

This:

```java
List<String>
```

doesn't mean:

> "Create a special StringList class."

It means the `List` is being **parameterized with a type**.

Think of:

```java
List<T>
```

as a general template.

Then:

```java
List<String>
```

means:

```text
T = String
```

And:

```java
List<Integer>
```

means:

```text
T = Integer
```

For example:

```java
List<String> names = new ArrayList<>();

List<Integer> ages = new ArrayList<>();

List<User> users = new ArrayList<>();
```

The same `List` abstraction can work with different types.

That's one of the major benefits of Generics.

---

# 5. The real reason Generics are powerful

Imagine you're building a reusable API.

Without generics:

```java
class Box {

    private Object value;

    public void set(Object value) {
        this.value = value;
    }

    public Object get() {
        return value;
    }
}
```

You can put anything inside:

```java
Box box = new Box();

box.set("Java");
```

But when retrieving:

```java
String value = (String) box.get();
```

You need a cast.

Now:

```java
Box<Integer> box = new Box<>();
box.set(100);
```

And:

```java
Integer value = box.get();
```

No explicit cast.

The compiler knows the type.

---

# 6. Your first generic class

```java
class Box<T> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

Now:

```java
Box<String> stringBox = new Box<>();

stringBox.set("Java");

String value = stringBox.get();
```

And:

```java
Box<Integer> integerBox = new Box<>();

integerBox.set(100);

Integer value = integerBox.get();
```

Same class.

Different type.

That's the fundamental idea behind generic classes.

---

# 7. What is `T`?

This:

```java
class Box<T>
```

contains a **type parameter**.

`T` is just a name.

You could technically write:

```java
class Box<X>
```

or:

```java
class Box<MyType>
```

But Java developers follow conventional names.

Common conventions:

| Symbol | Common meaning |
| ------ | -------------- |
| `T`    | Type           |
| `E`    | Element        |
| `K`    | Key            |
| `V`    | Value          |
| `N`    | Number         |
| `R`    | Result         |

For example:

```java
Map<K, V>
```

becomes:

```java
Map<String, Integer>
```

Here:

```text
K → String
V → Integer
```

---

# 8. Generics aren't only for Collections

This is a common beginner mistake.

People learn:

```java
List<String>
Map<String, User>
Set<Integer>
```

and think:

> "Generics = Collections."

No.

Collections are just where you see Generics frequently.

You can use Generics in:

### Classes

```java
class Box<T> {
}
```

### Interfaces

```java
interface Repository<T> {
}
```

### Methods

```java
public <T> T getValue() {
    ...
}
```

### Records/classes in backend design

```java
class ApiResponse<T> {
    private T data;
}
```

For example:

```java
ApiResponse<User> response;
```

or:

```java
ApiResponse<List<User>> response;
```

This becomes extremely useful in Spring Boot applications.

---

# 9. Real Spring Boot example

Imagine every API in your application returns:

```json
{
    "success": true,
    "message": "User found",
    "data": {
        "id": 10,
        "name": "Arup"
    }
}
```

You might create:

```java
public class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;

    // constructors/getters/setters
}
```

Then:

```java
ApiResponse<User> response;
```

For a list:

```java
ApiResponse<List<User>> response;
```

For a payment:

```java
ApiResponse<Payment> response;
```

For a string:

```java
ApiResponse<String> response;
```

One reusable structure.

Different types.

This is where Generics becomes a **production design tool**, not just an interview topic.

---

# 10. Generic type vs actual type

Look carefully:

```java
class Box<T>
```

`T` is the **type parameter**.

But here:

```java
Box<String>
```

`String` is the **type argument**.

So:

```java
Box<T>
```

```text
T = type parameter
```

while:

```java
Box<String>
```

```text
String = type argument
```

This terminology matters in interviews.

---

# 11. A very important rule

Generics work with **reference types**, not primitive types.

This is invalid:

```java
List<int> numbers;
```

Use:

```java
List<Integer> numbers;
```

Similarly:

```java
List<double>      ❌
List<boolean>     ❌
List<char>        ❌
```

Use:

```java
List<Double>      ✅
List<Boolean>     ✅
List<Character>   ✅
```

Java provides wrapper classes for primitives.

```text
int     → Integer
long    → Long
double  → Double
boolean → Boolean
char    → Character
```

Java can automatically convert between primitives and wrappers in many situations through **autoboxing/unboxing**.

We'll examine that properly later because it has performance implications.

---

# 12. The senior-level perspective

A junior developer often thinks:

> "Generics prevent me from putting the wrong type into a List."

That's true, but incomplete.

At a higher level, Generics provide:

### 1. Type safety

Catch many errors during compilation.

### 2. Reusability

Write one implementation that works with multiple types.

### 3. Better APIs

The type contract becomes visible directly in the API.

Compare:

```java
Object getData();
```

with:

```java
User getData();
```

or:

```java
ApiResponse<User> getData();
```

The second communicates much more information to the compiler and developer.

### 4. Less casting

Instead of:

```java
User user = (User) repository.find();
```

you can have:

```java
User user = repository.find();
```

The type system carries the information for you.

---

# 13. Common beginner misunderstanding

### ❌ "Generics make Java dynamically typed."

No.

It's almost the opposite.

Generics strengthen Java's **static type system**.

---

### ❌ "`T` means String."

No.

`T` is a placeholder for a type.

```java
Box<String>
Box<User>
Box<Integer>
```

The same `T` can represent different types depending on how the generic type is used.

---

### ❌ "Generics create a separate class for every type."

Not in the straightforward source-level sense.

For example:

```java
Box<String>
Box<Integer>
```

do not mean you wrote two different classes:

```java
StringBox
IntegerBox
```

Understanding what actually happens at runtime leads us to **type erasure**, which we'll study later.

---

# 14. Bad vs Good

### ❌ Older/raw style

```java
List users = new ArrayList();

users.add("Arup");
users.add(100);

String name = (String) users.get(1);
```

Problems:

* no compile-time type safety
* explicit casting
* runtime failure possible
* unclear API contract

### ✅ Generic style

```java
List<String> users = new ArrayList<>();

users.add("Arup");
users.add("Rahim");

String name = users.get(0);
```

Benefits:

* compile-time checking
* no manual cast
* clearer intent
* safer refactoring
* better API contract

---

# 15. Production-quality thinking

Don't use Generics simply because:

> "Java developers are supposed to use `<T>`."

Use them when they make the **type contract clearer and safer**.

For example:

```java
ApiResponse<User>
```

communicates something meaningful.

But creating an unnecessarily complicated generic hierarchy such as:

```java
ResponseWrapper<
    ResultContainer<
        DataHolder<
            T
        >
    >
>
```

doesn't automatically make your system better.

### Expert rule:

> **Use Generics to express a useful type relationship, not to show off the type system.**

Readable code beats clever code.

---

# 16. Performance: Does Generics make Java slower?

Don't assume:

```java
List<String>
```

is slower than:

```java
List
```

simply because it contains a generic type.

For ordinary reference types, the main purpose of Generics is **compile-time type safety and reusable APIs**.

However, there is an important performance topic:

```java
List<Integer>
```

uses `Integer`, not primitive `int`.

Therefore:

```java
List<Integer>
```

may involve boxing/unboxing.

That can matter in high-performance code.

We'll cover this later when we discuss **Generics + type erasure + primitives + boxing + performance**.

---

# Key Takeaways

1. **Generics provide compile-time type safety.**
2. They reduce runtime `ClassCastException` problems.
3. They reduce unnecessary casting.
4. They make APIs clearer.
5. They allow reusable classes, interfaces, and methods.
6. `T` is a type parameter.
7. `String` in `Box<String>` is a type argument.
8. Generics work with reference types, not primitives.
9. Collections are only one use of Generics.
10. Generics are heavily used in real backend APIs and reusable components.

---

# Expert Insights

### 1. Generics are primarily a type-system feature

Don't think:

> "Generics are a special kind of collection."

Think:

> **"Generics let me express relationships between types."**

That's the deeper concept.

---

### 2. Good generic APIs communicate intent

Compare:

```java
Object process();
```

with:

```java
<T> T process();
```

or:

```java
ApiResponse<User> process();
```

The type information becomes part of the API contract.

---

### 3. Raw types are usually a warning sign

If you see:

```java
List list;
Map map;
```

in modern Java code, investigate why.

You may be dealing with legacy code, interoperability, or an actual design problem.

---

# Common Mistakes

### Mistake 1

```java
List list = new ArrayList();
```

instead of:

```java
List<String> list = new ArrayList<>();
```

### Mistake 2

Thinking:

```java
T = String
```

always.

`T` is just a placeholder.

### Mistake 3

Using raw types unnecessarily.

### Mistake 4

Thinking Generics are only for Collections.

### Mistake 5

Creating unnecessarily complicated generic APIs.

### Mistake 6

Ignoring boxing when using:

```java
List<Integer>
```

in performance-sensitive code.

---

# Real-World Scenario

You're reviewing this Spring Boot code:

```java
public ApiResponse getUser(Long id) {

    User user = userRepository.findById(id)
            .orElse(null);

    return new ApiResponse(user);
}
```

You notice:

```java
ApiResponse
```

is a raw type.

You ask the developer why.

They say:

> "It works."

### Your code-review question:

What problems could this cause?

And how would you improve it?

Think about:

```java
ApiResponse<User>
```

instead of:

```java
ApiResponse
```

---

# Interview Questions

Don't look up the answers yet. Try answering them yourself.

### Junior

**Q1.** What are Generics in Java?

**Q2.** Why were Generics introduced?

**Q3.** What's the difference between:

```java
List list;
```

and:

```java
List<String> list;
```

**Q4.** Why can't we write:

```java
List<int>
```

?

---

### Mid-level

**Q5.** What problem does this solve?

```java
class Box<T> {
    private T value;
}
```

**Q6.** What's the difference between a **type parameter** and a **type argument**?

**Q7.** Why do Generics reduce the need for casting?

---

### Senior follow-up

Imagine you have:

```java
class ApiResponse<T> {
    private T data;
}
```

and:

```java
ApiResponse<User> response;
```

A developer says:

> "Generics are only there to make the code look cleaner."

**Do you agree? Explain why or why not.**

---

# Practical Exercise

Create your own generic class:

```java
class Box<T> {

    private T value;

    public void set(T value) {
        // implement
    }

    public T get() {
        // implement
    }
}
```

Then create:

```java
Box<String>
Box<Integer>
Box<User>
```

Try to understand exactly what changes and what doesn't.

### Bonus

Try intentionally doing:

```java
Box<String> box = new Box<>();

box.set(100);
```

Ask yourself:

**At what stage does Java reject this?**



Absolutely. Since you want **all Java Generics lessons together**, here is the complete roadmap from **Junior → Mid → Senior → Expert → Production**, including internals, coding, pitfalls, performance, Spring Boot, and interview preparation.

````md
# Java Generics — Complete Senior Engineer Learning Path

Goal:
Understand Java Generics deeply enough to design, debug, review, and explain
generic code like an experienced backend engineer.

---

# Lesson 1 — Why Generics Exist

## 1.1 The problem before Generics

Before Generics:

```java
List users = new ArrayList();

users.add("Arup");
users.add(100);

String name = (String) users.get(1);
````

The compiler allows different types.

The problem appears at runtime:

```text
100
 ↓
cast to String
 ↓
ClassCastException
```

Generics move many type errors from runtime to compile time.

---

## 1.2 With Generics

```java
List<String> users = new ArrayList<>();

users.add("Arup");
users.add("Rahim");

// users.add(100); // compile-time error
```

Now the compiler knows:

```text
List<String>
     ↓
only String values
```

---

## 1.3 Main benefits

Generics provide:

* compile-time type safety
* less casting
* reusable code
* clearer APIs
* better refactoring
* better developer tooling

---

## 1.4 Generic class

```java
class Box<T> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

Usage:

```java
Box<String> box1 = new Box<>();
box1.set("Java");

String value = box1.get();
```

Another type:

```java
Box<Integer> box2 = new Box<>();
box2.set(100);

Integer number = box2.get();
```

Same class.

Different type.

---

## 1.5 Important terminology

```java
class Box<T>
```

`T` = type parameter

```java
Box<String>
```

`String` = type argument

---

## Key takeaway

Generics are not simply a Collections feature.

They are a Java type-system feature that allows us to express relationships between types.

---

# Lesson 2 — Generic Classes, Interfaces and Methods

## 2.1 Generic class

```java
class Box<T> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

---

## 2.2 Multiple type parameters

```java
class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }
}
```

Usage:

```java
Pair<String, Integer> pair =
        new Pair<>("age", 25);
```

Here:

```text
K → String
V → Integer
```

---

## 2.3 Generic interface

```java
interface Repository<T> {

    T findById(Long id);

    void save(T entity);
}
```

Implementation:

```java
class UserRepository implements Repository<User> {

    @Override
    public User findById(Long id) {
        // ...
        return null;
    }

    @Override
    public void save(User user) {
        // ...
    }
}
```

This creates a strong type contract.

---

## 2.4 Generic method

A generic method declares its own type parameter.

```java
public <T> T identity(T value) {
    return value;
}
```

Usage:

```java
String name = identity("Arup");

Integer number = identity(100);
```

The `<T>` before `T` is important:

```java
public <T> T identity(T value)
       ^^^
```

It tells Java:

> This method has its own type parameter.

---

## 2.5 Multiple generic method parameters

```java
public <K, V> void printPair(K key, V value) {

    System.out.println(key);
    System.out.println(value);
}
```

Usage:

```java
printPair("age", 25);
printPair(100, "Java");
```

---

## 2.6 Diamond operator

Instead of:

```java
List<String> names =
        new ArrayList<String>();
```

modern Java allows:

```java
List<String> names =
        new ArrayList<>();
```

Java infers the type from the context.

---

## Senior insight

Generic classes and generic methods are different concepts.

```java
class Box<T>
```

The class is generic.

But:

```java
public <T> T process(T value)
```

means the method itself declares `T`.

This distinction appears frequently in interviews.

---

# Lesson 3 — Type Bounds

Sometimes you don't want to accept every possible type.

You want to restrict the type.

---

## 3.1 Upper bound

```java
class Calculator<T extends Number> {

    private T value;
}
```

Now:

```java
Calculator<Integer> a;
Calculator<Double> b;
Calculator<Long> c;
```

are valid.

But:

```java
Calculator<String> d;
```

is invalid.

Because:

```text
T extends Number
```

means:

```text
T must be Number or a subclass of Number
```

---

## 3.2 Generic method with bound

```java
public <T extends Number> double doubleValue(T value) {
    return value.doubleValue() * 2;
}
```

This allows:

```java
doubleValue(10);
doubleValue(10.5);
doubleValue(100L);
```

because all are Numbers.

---

## 3.3 Multiple bounds

Java allows a class plus interfaces:

```java
<T extends Number & Comparable<T>>
```

Meaning:

```text
T must:
    ↓
extend Number
AND
implement Comparable<T>
```

Example:

```java
public <T extends Number & Comparable<T>>
T process(T value) {
    return value;
}
```

---

## 3.4 Why bounds matter

Without a bound:

```java
<T>
```

you cannot assume Number-specific methods exist.

With:

```java
<T extends Number>
```

the compiler knows:

```java
value.intValue();
value.doubleValue();
value.longValue();
```

are available.

---

## Interview trap

People often think:

```java
<T extends Number>
```

means inheritance only.

In generic syntax, `extends` can represent:

* class inheritance
* interface constraints

For example:

```java
<T extends Comparable<T>>
```

---

# Lesson 4 — Wildcards

This is where Generics becomes much more interesting.

There are three major wildcard forms:

```java
?
? extends T
? super T
```

---

# 4.1 Unbounded wildcard

```java
List<?> list
```

Means:

> A List of some unknown type.

It could be:

```java
List<String>
List<Integer>
List<User>
```

---

## What can you read?

```java
Object value = list.get(0);
```

Because the actual type is unknown, the safe common type is `Object`.

---

## What can you add?

Generally, you cannot add a specific value:

```java
list.add("Java"); // compile error
```

except:

```java
list.add(null);
```

because `null` is valid for reference types.

---

# 4.2 Upper bounded wildcard

```java
List<? extends Number>
```

Means:

> List of some unknown type that is Number or a subtype.

Possible:

```java
List<Integer>
List<Double>
List<Long>
```

---

## Reading

```java
Number number = list.get(0);
```

Safe.

Because whatever the actual type is, it is at least a Number.

---

## Adding

You cannot safely do:

```java
list.add(10);
```

Why?

Suppose the actual list is:

```java
List<Double>
```

Then adding an Integer would be unsafe.

This is the core reason.

---

# 4.3 Lower bounded wildcard

```java
List<? super Integer>
```

Means:

> A list whose element type is Integer or one of Integer's supertypes.

Possible:

```java
List<Integer>
List<Number>
List<Object>
```

Now:

```java
list.add(10);
```

is safe.

Because all three lists can accept an Integer.

---

# 4.4 The famous PECS rule

**PECS = Producer Extends, Consumer Super**

If you are reading/producing values:

```java
? extends T
```

If you are writing/consuming values:

```java
? super T
```

Example:

```java
public double sum(List<? extends Number> numbers) {

    double total = 0;

    for (Number number : numbers) {
        total += number.doubleValue();
    }

    return total;
}
```

The list produces Numbers.

Therefore:

```java
? extends Number
```

---

Consumer:

```java
public void addUsers(
        List<? super User> users) {

    users.add(new User());
}
```

The list consumes Users.

Therefore:

```java
? super User
```

---

# Lesson 5 — Invariance

This is one of the most important concepts.

Many beginners assume:

```java
Integer extends Number
```

therefore:

```java
List<Integer> extends List<Number>
```

But this is FALSE.

Java Generics are generally **invariant**.

---

## Example

```java
List<Integer> integers =
        new ArrayList<>();
```

You cannot do:

```java
List<Number> numbers = integers;
```

Why?

Imagine Java allowed it.

Then:

```java
numbers.add(3.14);
```

Now the original:

```java
List<Integer>
```

would contain a Double.

That would break type safety.

---

## But arrays behave differently

Java arrays are covariant:

```java
Integer[] integers = new Integer[10];

Number[] numbers = integers;
```

This compiles.

But:

```java
numbers[0] = 3.14;
```

causes:

```text
ArrayStoreException
```

Generics avoid this kind of runtime problem by enforcing stronger compile-time rules.

---

# Lesson 6 — Type Erasure

Now we reach the important internal concept.

You write:

```java
List<String>
```

and:

```java
List<Integer>
```

But Java's generics were designed primarily around **compile-time type checking**.

Java uses **type erasure**.

Conceptually, generic type information is removed/reduced in the generated bytecode representation according to erasure rules.

For example:

```java
class Box<T> {

    T value;

    T get() {
        return value;
    }
}
```

is conceptually erased toward:

```java
class Box {

    Object value;

    Object get() {
        return value;
    }
}
```

Then the compiler inserts appropriate casts where necessary.

---

## Why does Java do this?

Generics were introduced while Java needed compatibility with existing code.

This design allowed generic source code to work with the existing JVM type system without creating a completely separate runtime generic type system.

---

## Important consequence

You cannot normally do:

```java
new T();
```

because at runtime the JVM doesn't have enough information about what `T` actually is.

---

## You also cannot normally do:

```java
T[] array = new T[10];
```

for the same fundamental reason.

---

## Generic type information isn't simply available through:

```java
obj.getClass()
```

For example:

```java
List<String> a = new ArrayList<>();
List<Integer> b = new ArrayList<>();
```

At runtime, both are fundamentally instances of:

```text
ArrayList
```

not two separate runtime classes:

```text
StringArrayList
IntegerArrayList
```

---

# Lesson 7 — Generics + Inheritance

Suppose:

```java
class Animal {
}

class Dog extends Animal {
}
```

This is valid:

```java
Dog dog = new Dog();

Animal animal = dog;
```

But:

```java
List<Dog> dogs =
        new ArrayList<>();

List<Animal> animals = dogs;
```

is invalid.

Again:

```text
Dog → Animal
```

doesn't imply:

```text
List<Dog> → List<Animal>
```

---

## Use wildcard when appropriate

```java
List<? extends Animal> animals = dogs;
```

Now the relationship is expressed correctly.

You can safely read:

```java
Animal animal = animals.get(0);
```

---

# Lesson 8 — Generic Methods + Type Inference

Java can often infer generic types.

Example:

```java
public static <T> T first(T a, T b) {
    return a;
}
```

Then:

```java
String value =
        first("Java", "Spring");
```

Java infers:

```text
T = String
```

Another:

```java
Integer value =
        first(10, 20);
```

Java infers:

```text
T = Integer
```

---

## Explicit type witness

Sometimes you can explicitly specify:

```java
String value =
        MyClass.<String>first(
                "Java",
                "Spring"
        );
```

Usually inference is cleaner.

---

# Lesson 9 — Generic Constructors and Static Context

## Static fields cannot use a class's type parameter

This is invalid:

```java
class Box<T> {

    static T value;
}
```

Why?

`T` belongs to the instance/type parameterization:

```java
Box<String>
Box<Integer>
```

But static members belong to the class itself.

There isn't a single class-level `T` that can simultaneously mean:

```text
String
Integer
User
```

---

## Static generic method

But this is valid:

```java
class Utils {

    public static <T> T identity(T value) {
        return value;
    }
}
```

Because the method declares its own `T`.

---

# Lesson 10 — Generics + Primitive Types

Generics don't directly accept primitives:

```java
List<int> numbers; // invalid
```

Use:

```java
List<Integer> numbers;
```

This introduces boxing:

```java
int
 ↓
Integer
```

and unboxing:

```java
Integer
 ↓
int
```

---

## Why this matters

Consider:

```java
List<Integer> numbers =
        new ArrayList<>();

for (int i = 0; i < 1_000_000; i++) {
    numbers.add(i);
}
```

There can be significant object/allocation overhead compared with primitive-oriented data structures.

This can matter in:

* high-throughput systems
* numerical processing
* huge collections
* latency-sensitive applications

---

## Senior decision

Don't blindly replace everything with primitive collections.

First measure.

In normal business applications:

```java
List<Integer>
```

is perfectly reasonable.

In performance-critical workloads, specialized primitive collections or arrays may be more appropriate.

---

# Lesson 11 — Generic APIs in Spring Boot

Generics are everywhere in Spring applications.

---

## Repository

Conceptually:

```java
interface Repository<T, ID> {

    T findById(ID id);

    T save(T entity);
}
```

Then:

```java
interface UserRepository
        extends Repository<User, Long> {
}
```

The type contract becomes:

```text
Entity = User
ID     = Long
```

---

# ApiResponse<T>

A common backend pattern:

```java
class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;

    public ApiResponse(
            boolean success,
            String message,
            T data) {

        this.success = success;
        this.message = message;
        this.data = data;
    }
}
```

Usage:

```java
ApiResponse<User>
```

or:

```java
ApiResponse<List<User>>
```

or:

```java
ApiResponse<Payment>
```

---

## But don't overuse wrapper generics

This:

```java
ApiResponse<
    Result<
        Data<
            User
        >
    >
>
```

doesn't automatically mean good architecture.

Generics should make relationships clearer.

They shouldn't turn APIs into puzzles.

---

# Lesson 12 — Bad vs Good vs Production Code

## Bad

```java
public Object findUser(Long id) {
    return repository.findById(id);
}
```

Caller:

```java
User user =
        (User) service.findUser(10L);
```

Problems:

* weak contract
* casting
* runtime failure risk
* less readable

---

## Better

```java
public User findUser(Long id) {
    return repository.findById(id);
}
```

Strong type.

---

## Generic reusable abstraction

If the abstraction actually needs multiple entity types:

```java
interface Repository<T, ID> {

    T findById(ID id);

    T save(T entity);
}
```

This is where Generics provides real architectural value.

---

# Lesson 13 — Generics and API Design

Good generic API:

```java
public <T> T convert(
        String value,
        Class<T> type) {
    // ...
}
```

The relationship is meaningful:

```text
input + requested type
        ↓
     T result
```

Bad generic API:

```java
<T, K, V, R, X, Y>
```

when the relationships aren't actually useful.

Senior engineering isn't:

> "Use more Generics."

It's:

> "Use the type system to communicate meaningful contracts."

---

# Lesson 14 — Generic Exception / Throwable Restrictions

Java does not allow generic subclasses of `Throwable` in the normal way.

For example:

```java
class MyException<T>
        extends Exception {
}
```

is not allowed.

This is related to Java's generic type system and exception checking rules.

You should know this as an interview edge case rather than something to use in normal application code.

---

# Lesson 15 — Heap, Runtime and Generics

A common misconception:

```java
List<String>
```

means the JVM creates:

```text
StringList
```

and:

```java
List<Integer>
```

means:

```text
IntegerList
```

Not in the straightforward sense.

Because of type erasure, generic type arguments primarily provide compile-time information.

At runtime:

```java
new ArrayList<String>()
```

and:

```java
new ArrayList<Integer>()
```

are both instances of:

```text
ArrayList
```

This is one of the most important connections between:

**Java Generics + JVM + type erasure.**

---

# Lesson 16 — Advanced Wildcard Design

Consider:

```java
void copy(
    List<? extends Number> source,
    List<? super Number> destination
) {
    for (Number n : source) {
        destination.add(n);
    }
}
```

Why does this work?

Source:

```text
? extends Number
```

means:

> I can safely read Numbers.

Destination:

```text
? super Number
```

means:

> I can safely put Numbers here.

This is PECS in real code.

---

# Lesson 17 — Recursive Generic Bounds

Advanced example:

```java
<T extends Comparable<T>>
```

This means:

```text
T can compare itself with T
```

Example:

```java
public <T extends Comparable<T>>
T max(T a, T b) {

    return a.compareTo(b) >= 0
            ? a
            : b;
}
```

Usage:

```java
Integer result = max(10, 20);
```

This pattern appears in the JDK and advanced APIs.

---

# Lesson 18 — Generic Code and Type Safety

Imagine:

```java
List<String> names =
        new ArrayList<>();
```

This provides a compile-time contract.

But Generics don't magically make every operation safe.

For example, unchecked operations can bypass the type system:

```java
List raw = names;

raw.add(100);
```

Now:

```java
String name = names.get(0);
```

could eventually encounter a runtime cast problem depending on what was inserted.

This is why the compiler warns about raw types and unchecked operations.

Never casually ignore:

```text
unchecked conversion
unchecked cast
raw type
```

in production code.

---

# Lesson 19 — Unchecked Casts

You may see:

```java
Object value = "Java";

String text = (String) value;
```

That's an explicit runtime cast.

With Generics:

```java
List<String> list;
```

the compiler carries more type information.

But sometimes frameworks or legacy APIs force:

```java
@SuppressWarnings("unchecked")
```

This should not be used as:

> "Make the compiler shut up."

You should understand why the cast is safe before suppressing the warning.

---

# Lesson 20 — Generics and Reflection

Because of type erasure:

```java
List<String>
```

doesn't simply expose `String` as the runtime class of the List.

But Java reflection can sometimes recover generic metadata from declarations.

For example:

```java
Field field = MyClass.class
        .getDeclaredField("users");

Type type = field.getGenericType();
```

You may encounter:

```text
ParameterizedType
TypeVariable
WildcardType
GenericArrayType
```

This becomes important in:

* Spring
* Jackson
* Hibernate
* dependency injection
* serialization frameworks

Frameworks frequently inspect generic declarations.

---

# Lesson 21 — Real Production Scenario

Imagine this service:

```java
public List getUsers() {

    return userRepository.findAll();
}
```

A code reviewer asks:

> Why is this a raw List?

Developer:

> It works.

You change it to:

```java
public List<User> getUsers() {

    return userRepository.findAll();
}
```

Now the compiler and IDE understand the contract.

Then downstream:

```java
List<User> users =
        userService.getUsers();
```

No casting.

Better refactoring.

Better readability.

Better compile-time safety.

This is a small example of how Generics improve production code.

---

# Lesson 22 — Performance Considerations

Generics themselves are not automatically a performance problem.

The important areas are:

### 1. Boxing

```java
List<Integer>
```

instead of primitive `int`.

### 2. Allocation

Wrapper objects may be created.

### 3. Memory footprint

Millions of wrapper objects can matter.

### 4. Type erasure

Generics don't automatically mean runtime specialization for every type.

### 5. Collections

Choose the appropriate data structure.

Don't optimize Generics while ignoring:

```text
algorithm
collection choice
allocation rate
memory locality
I/O
database
network
```

Senior engineers optimize the actual bottleneck.

---

# Lesson 23 — Security Considerations

Generics are not a security mechanism.

They provide compile-time type safety.

Don't confuse:

```java
List<User>
```

with:

> "Only authorized users can access these Users."

Authorization still requires:

* authentication
* authorization
* tenant isolation
* validation
* access control
* secure serialization

Generics help prevent type mistakes.

They do not enforce business security.

---

# Lesson 24 — Generics and Multitenant Backend Systems

Suppose you have:

```java
class TenantResponse<T> {

    private UUID tenantId;
    private T data;
}
```

You might use:

```java
TenantResponse<User>
```

or:

```java
TenantResponse<List<User>>
```

But Generics doesn't guarantee tenant isolation.

This:

```java
TenantResponse<User>
```

only describes the Java type.

You still need:

```text
authenticated user
        ↓
tenant authorization
        ↓
database query filtering
        ↓
correct tenant data
        ↓
typed response
```

This distinction is important in production.

---

# Lesson 25 — Senior Code Review Checklist

When reviewing generic code, ask:

### 1. Is the type contract clear?

```java
Object
```

or:

```java
User
```

Which communicates more?

---

### 2. Is a raw type being used?

```java
List
Map
Repository
```

Investigate why.

---

### 3. Are wildcards necessary?

Don't add:

```java
? extends
? super
```

just because you know them.

Use them when they express variance correctly.

---

### 4. Is PECS being applied correctly?

Producer:

```java
? extends
```

Consumer:

```java
? super
```

---

### 5. Is the generic abstraction actually useful?

If a class has:

```java
<T, K, V, R, X>
```

ask whether all of those relationships are necessary.

---

### 6. Are unchecked warnings being ignored?

Investigate:

```text
unchecked cast
unchecked conversion
raw type
```

---

### 7. Is boxing creating a performance problem?

Especially in:

* large collections
* numerical processing
* high-throughput systems

---

# Lesson 26 — Interview Question Bank

## Junior

1. What are Generics?
2. Why were Generics introduced?
3. What is type safety?
4. What is a generic class?
5. What is a generic method?
6. What does `<T>` mean?
7. What is the difference between `T` and `Object`?
8. Why can't `List<int>` be used?
9. What is the diamond operator?
10. What is a raw type?

---

## Mid-level

11. What is the difference between `?` and `T`?
12. What is `? extends`?
13. What is `? super`?
14. Explain PECS.
15. Why is `List<Integer>` not a subtype of `List<Number>`?
16. Why are Java Generics invariant?
17. What is type erasure?
18. Why can't we instantiate `T` directly?
19. Why can't we create `new T[10]`?
20. Can static members use a class's type parameter?

---

## Senior

21. Why did Java use type erasure?
22. What are the consequences of type erasure?
23. How do Generics interact with inheritance?
24. Why are arrays covariant but Generics invariant?
25. Explain:

```java
List<? extends Number>
```

26. Explain:

```java
List<? super Integer>
```

27. Why can you read from `? extends` but not safely add?
28. Why can you add to `? super`?
29. Explain:

```java
<T extends Comparable<T>>
```

30. When would you choose a generic method instead of a generic class?

---

# Lesson 27 — Tricky Interview Questions

## Question 1

What is the difference between:

```java
List<Object>
```

and:

```java
List<?>
```

They are NOT the same.

---

## Question 2

Why doesn't this compile?

```java
List<Integer> ints =
        new ArrayList<>();

List<Number> nums = ints;
```

Because Generics are invariant.

---

## Question 3

Why does this work?

```java
List<Integer> ints =
        new ArrayList<>();

List<? extends Number> nums = ints;
```

Because the wildcard expresses covariance for reading.

---

## Question 4

Why can you do:

```java
List<? super Integer> list;

list.add(10);
```

Because the actual list type must be Integer or a supertype of Integer.

---

## Question 5

What happens after type erasure?

You should understand that generic type information is primarily a compile-time construct and the generated/runtime representation follows erasure rules.

---

# Lesson 28 — Interview Simulation

Now imagine I'm interviewing you for a Senior Java Backend Engineer position.

I give you this:

```java
public void process(List<Number> numbers) {
    // ...
}
```

A developer calls:

```java
List<Integer> integers =
        new ArrayList<>();

process(integers);
```

### Interviewer:

**Why doesn't this compile?**

Don't answer:

> "Because Java doesn't allow it."

Explain the actual type-system reasoning.

---

### Follow-up

I change the method to:

```java
public void process(
        List<? extends Number> numbers) {
}
```

Now it works.

### Why?

---

### Follow-up 2

Now I write:

```java
public void process(
        List<? super Integer> numbers) {

    numbers.add(10);
}
```

### Why is adding `10` safe?

---

### Follow-up 3

I ask:

> "Explain PECS without memorizing the acronym."

Your answer should explain the **data flow**.

---

# Lesson 29 — Production Incident

Your Spring Boot service processes:

```text
20,000 requests/sec
```

A developer changes:

```java
List<Integer>
```

to:

```java
List<Number>
```

because they believe it will make the code more generic.

The application now experiences:

* higher memory usage
* higher CPU
* increased latency

### Interviewer:

Would you immediately blame Generics?

**No.**

Investigate:

```text
allocation
boxing
collection size
object lifetime
GC activity
CPU profile
request volume
data structure
actual workload
```

Generics themselves are not automatically the performance problem.

---

# Lesson 30 — Final Mental Model

You should eventually be able to think about Java Generics at five levels.

## Junior

```text
Generics = type safety
```

---

## Mid-level

```text
Generic classes
Generic methods
Bounds
Wildcards
PECS
```

---

## Senior

```text
Variance
Invariance
Type erasure
API design
Unchecked operations
Performance
```

---

## Expert

```text
Type inference
Recursive bounds
Generic reflection
Framework interaction
Compiler-generated casts
Bridge methods
Erasure implications
Generic API design
```

---

## Production

```text
Readable type contracts
Safe abstractions
Spring/JPA/Jackson interaction
Performance-sensitive boxing
Legacy raw types
Unchecked warnings
API compatibility
Maintainability
Debugging framework generic types
```

---

# Final Expert Mental Model

Don't memorize:

```text
<T>
?
extends
super
PECS
```

as isolated syntax.

Think about the problem they solve:

```text
                    Java Type System
                          │
             ┌────────────┴────────────┐
             │                         │
       Type relationships          Type safety
             │                         │
      ┌──────┴──────┐                  │
      │             │                  │
   Generic       Wildcards             │
   Types            │                 │
      │       ┌─────┴─────┐           │
      │       │           │           │
      │    extends       super        │
      │       │           │           │
      └───────┴───────────┴───────────┘
                      │
                Reusable APIs
                      │
                Production Code
```

The deeper idea is:

> **Generics allow you to express relationships between types so that the compiler can help you build safer, reusable, and clearer software.**

And the senior-level question is not:

> "Where can I use `<T>`?"

It is:

> **"What type relationship does my API need to communicate, and can Java's type system enforce it for me?"**

---

# Final Practical Project

Build a small generic repository abstraction:

```java
interface Repository<T, ID> {

    T findById(ID id);

    List<T> findAll();

    T save(T entity);

    void deleteById(ID id);
}
```

Implement:

```java
class UserRepository
        implements Repository<User, Long> {
}
```

and:

```java
class ProductRepository
        implements Repository<Product, Long> {
}
```

Then create:

```java
class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;
}
```

Use:

```java
ApiResponse<User>
ApiResponse<List<User>>
ApiResponse<Product>
ApiResponse<List<Product>>
```

Then add a utility method:

```java
public static <T> T first(List<T> values) {
    return values.get(0);
}
```

Then improve it using appropriate wildcards where useful.

Finally, explain why:

```java
List<Integer>
```

cannot be passed where:

```java
List<Number>
```

is expected, but:

```java
List<? extends Number>
```

can accept it.

If you can explain all of that **without memorizing the answers**, you have moved from basic Generic syntax into real Java type-system understanding.

```
```
