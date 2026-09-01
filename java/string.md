# Java Strings — Complete Notes

---

## 1. UTF-16 Encoding

**The basic idea:** Computers only understand numbers (bits), so every character (letter, digit, symbol, emoji, etc.) needs to be represented as a number. Unicode is a big universal standard that assigns a unique number to basically every character in every language. UTF-16 is one specific *encoding scheme* for storing Unicode numbers using 16-bit chunks.

**What "16-bit" means:**
Each character takes up 16 bits (2 bytes) of memory. With 16 bits, you can represent numbers from 0 to 65,535 — that's 65,536 possible values. So each character gets mapped to a unique number in that range.

**Example:**
- `'A'` → 65
- `'a'` → 97
- `'€'` → 8364

**UTF meaning:**
UTF = **Unicode Transformation Format**
- **U**nicode → the big table assigning a unique number to every character
- **T**ransformation **F**ormat → the method used to convert those numbers into binary for storage/transmission

| Format | Chunk size | Common use |
|---|---|---|
| UTF-8 | 8-bit (variable, 1-4 bytes) | Most common on the web, files, most languages except Java |
| UTF-16 | 16-bit (variable, 1-2 units) | Used internally by Java, JavaScript, Windows |
| UTF-32 | 32-bit (fixed) | Rarely used, wastes space |

### Java's `char` is a 16-bit type

```java
char c = 'A';
System.out.println((int) c); // prints 65
```

You can do arithmetic with chars since they're numbers under the hood:
```java
char c = 'A';
c = (char)(c + 1);
System.out.println(c); // prints 'B'
```

### `String` = sequence of `char`s (UTF-16 code units)

```java
String s = "Hi";
System.out.println(s.length());        // 2
System.out.println((int) s.charAt(0)); // 72 ('H')
System.out.println((int) s.charAt(1)); // 105 ('i')
```

`length()` and `charAt()` work in terms of 16-bit units, **not** "characters" in the everyday sense — this matters once emojis or rare symbols show up.

### Surrogate pairs

Some Unicode characters (most emoji, rare/old scripts) have code points above 65,535 (0xFFFF), so they can't fit in a single `char`. Java represents these using **two `char`s together** — a surrogate pair.

```java
String s = "😀"; // one emoji
System.out.println(s.length()); // 2, not 1!
```

To count actual Unicode characters (not raw 16-bit units):
```java
s.codePointCount(0, s.length()); // 1 (correct "visual" character count)
```

### Practical implications
- `String.length()` counts UTF-16 code units, not visual characters — trips people up with emoji/rare-script text.
- `charAt(i)` might give you *half* of a surrogate pair, not a full character.
- For real Unicode-safe iteration, use `codePoints()` or `s.codePointAt(i)` instead of `charAt()`.

### Compact Strings (Java 9+)
Internally, Strings use a `byte[]` plus a `coder` flag, instead of always using `char[]`. If all characters fit in Latin-1 (8-bit), Java stores them as 1 byte each instead of 2 — cutting memory in half for typical English text. If any character needs UTF-16, it falls back to 2 bytes per character. This is invisible to the programmer — `charAt()`, `length()`, etc. behave the same either way.

---

## 2. String Basics

A `String` in Java is an object used to store a sequence of characters enclosed in double quotes. It uses UTF-16 encoding and provides a rich API for handling text data.

```java
String name = "Geeks";
String num = "1234";
```

A `String` is a class (`java.lang.String`), not a primitive — but Java gives it special treatment (literals, `+` operator overloading) that makes it feel primitive-like.

### CharSequence interface
`CharSequence` represents a sequence of characters and provides common methods (`length()`, `charAt()`, `subSequence()`, `toString()`). Classes that implement it:

| Class | Mutable? | Thread-safe? | When to use |
|---|---|---|---|
| `String` | No | N/A (safe by nature) | Fixed text, keys, general use |
| `StringBuilder` | Yes | No | Fast in-place editing, single thread |
| `StringBuffer` | Yes | Yes | In-place editing across multiple threads |
| `StringTokenizer` | — | — | Splitting text into tokens by delimiter |

---

## 3. Two Ways of Creating a Java String

### 1. String literal (goes into the String Pool)
```java
String str = "GeeksforGeeks";
```
To make Java more memory efficient, Java stores string literals in the **String Pool**. If the same literal already exists in the pool, Java reuses the existing object instead of creating a new one.

### 2. Using `new` keyword (Heap memory)
```java
String str = new String("GeeksforGeeks");
```
This forces a **new object** in heap memory, even if the same string already exists in the pool:
- One object is created in the heap memory
- The string literal itself is still stored in the string pool (if not already present)
- The reference variable points to the heap object, **not** the pool

So `new String(...)` can create up to two things: one pool entry (from the literal) and one separate heap object (from `new`).

---

## 4. String Pool Mechanics

### Example — literal reuse
```java
String str1 = "Hello";
String str2 = "Hello";
```
Both `str1` and `str2` refer to the *same* pooled object.

### Example — new keyword
```java
String str1 = new String("John");
String str2 = new String("Deo");
```
Each creates a separate heap object; the literals inside are still pool-backed.

### `intern()`
The `intern()` method returns the canonical String reference from the String Pool. If an equal String isn't already present in the pool, it's added; otherwise, the existing pooled reference is returned.

```java
String internedString = demoString.intern(); // adds to / retrieves from pool
```

### String Pool migration from PermGen to Heap
- Before Java 7: the String Pool lived in **PermGen**.
- From Java 7 onward: the String Pool moved into the **regular heap**.
  - Pooled strings are now properly garbage-collected.
  - Removed limitations tied to PermGen's small fixed size.

### Full example
```java
class Geeks {
    public static void main(String args[]) {
        // Declaring Strings using String literals
        String s1 = "TAT";
        String s2 = "TAT";

        // Declaring Strings using new keyword
        String s3 = new String("TAT");
        String s4 = new String("TAT");

        System.out.println(s1); // TAT
        System.out.println(s2); // TAT
        System.out.println(s3); // TAT
        System.out.println(s4); // TAT
    }
}
```
`s1` and `s2` use the same string literal, so they refer to the same String Pool object. `s3` and `s4` are created using `new`, so each represents a separate String object on the heap. All four contain the same text, `"TAT"`.

### Memory diagram (described)
- **Heap memory** (outer container)
  - **String pool** (inner region): holds one copy per literal, e.g. `"Hello"`, `"TAT"`
  - **Heap objects created via `new`** (separate inner region): e.g. `"John"`, `"Deo"` — each `new String(...)` call is its own object here, while the literal passed inside is still pool-backed.
- `s1 = "TAT"; s2 = "TAT";` → both point to the **same** pool object.
- `s3 = new String("TAT"); s4 = new String("TAT");` → **separate** heap objects each time.

---

## 5. `==` vs `.equals()`

`==` checks if two references point to the *exact same object* in memory. `.equals()` checks if the *content* is the same.

```java
String a = "TAT";
String b = "TAT";
System.out.println(a == b);        // true — same pool object
System.out.println(a.equals(b));   // true — same content

String c = new String("TAT");
System.out.println(a == c);        // false — different heap objects!
System.out.println(a.equals(c));   // true — content still matches
```

Using `intern()` to force pool membership:
```java
String c = new String("TAT").intern();
System.out.println(a == c); // now true — c now points to the pooled "TAT"
```

### Interview gotcha — full chain
```java
String a = "Java";
String b = "Java";
String c = new String("Java");
String d = c.intern();

a == b        // true  (same pool object)
a == c        // false (c is a separate heap object)
a == d        // true  (intern() returns the pooled reference)
a.equals(c)   // true  (content is identical)
```

### Interview gotcha — compile-time constant folding
```java
final String x = "Hel" + "lo";     // compiler folds this into "Hello" at compile time — pooled
String y = "Hel";
String z = y + "lo";               // NOT compile-time constant — built at runtime, NOT pooled
x == "Hello"   // true
z == "Hello"   // false
```
String concatenation is only pool-eligible if the compiler can fully resolve it at compile time (all operands are literals or `final` compile-time constants).

**Rule:** Always use `.equals()` to compare string *values*, never `==`.

---

## 6. Immutability

**In Java, string objects are immutable.** Once a string object is created, its data or state can't be changed — but a new string object is created instead.

### Example — `concat()` "does nothing" if unassigned
```java
public class GFG {
    public static void main(String[] args) {
        String str = "Hello";
        str.concat(" World");
        System.out.println(str);
    }
}
```
Output: `Hello`

**Explanation:** `String.concat()` does not modify the original String object. When `str.concat(" World")` executes:
1. A new String object `"Hello World"` is created.
2. The original String `"Hello"` remains unchanged.
3. Since the new object is not assigned to any variable, it is discarded (eligible for garbage collection).

### A confusing example, step by step
```java
String s = "Hello";
s = s + " World";
System.out.println(s); // "Hello World"
```
This *looks* like `s` changed, but really:
1. `"Hello"` is created as a String object.
2. `s + " World"` creates a **completely new** String object `"Hello World"` elsewhere in memory.
3. `s` is reassigned to point to this new object.
4. The original `"Hello"` object still exists, unchanged — just orphaned, with nothing pointing to it.

### Proof strings never change in place
```java
String s = "Hello";
String original = s;

s = s + " World";

System.out.println(original); // still "Hello"
System.out.println(s);        // "Hello World"
```
If Strings were mutable, `original` would've changed too. It didn't, because `s + " World"` never modified the original string — it created a new one.

### Same applies to other "modifying" methods
```java
String s = "hello";
s.toUpperCase();
System.out.println(s); // still "hello", NOT "HELLO"!
```
`toUpperCase()` returns a **new** string — it doesn't change `s` itself. You must capture the result:
```java
String s = "hello";
s = s.toUpperCase(); // now s points to the new "HELLO" string
System.out.println(s); // "HELLO"
```
Same applies to `replace()`, `substring()`, `concat()`, `trim()`, etc. — **all** return new strings rather than modifying the original.

### Why Java's designers made String immutable

1. **Security** — Strings are used everywhere (file paths, network URLs, passwords, class names, reflection e.g. `Class.forName(name)`). If they could change unexpectedly, it could create security holes (e.g. a filename changed after validation but before use). Immutability guarantees a validated string stays validated forever.

2. **String pool / memory efficiency** — Java keeps the String pool where identical string literals are shared:
   ```java
   String a = "cat";
   String b = "cat";
   ```
   `a` and `b` point to the *same* object — safe only because strings can't change. If one could be mutated, it would silently corrupt the other.

3. **Hashcode caching** — Since a String never changes, Java computes its hash code **once** and caches it. Every future `.hashCode()` call returns the cached value instantly instead of recomputing. This is why Strings are ideal `HashMap`/`HashSet` keys.

4. **Thread safety** — Multiple threads can share the same String without locks or synchronization, since nothing can modify it.

5. **Safe as HashMap keys** — Since a String's value never changes, its hashcode never changes, so it's reliably safe to use as a key in a `HashMap` or element in a `HashSet`.

### If you need a "mutable string" — use `StringBuilder`

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // modifies sb directly, no new object created
System.out.println(sb); // "Hello World"
```

Use `StringBuilder` (or `StringBuffer` for thread-safe version) when doing lots of string modifications, e.g. in a loop:

```java
// Bad: creates many intermediate String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;   // each += silently does: new StringBuilder(result).append(i).toString()
}

// Good: modifies the same StringBuilder object
StringBuilder result = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    result.append(i);
}
String finalResult = result.toString();
```

Rule of thumb: use `+` for a handful of one-off concatenations (compiler optimizes it), but always use `StringBuilder` for loops or heavy building.

---

## 7. Common String Methods (Reference)

**Length & access**
```java
"Hello".length();          // 5
"Hello".charAt(1);         // 'e'
"Hello".isEmpty();         // false
"   ".isBlank();           // true (Java 11+)
```

**Comparison**
```java
"abc".equals("abc");            // true — content check
"abc".equalsIgnoreCase("ABC");  // true
"abc".compareTo("abd");         // negative (lexicographic diff)
"abc" == "abc";                 // true (pool), risky for new String()
```

**Searching**
```java
"Hello World".contains("World");     // true
"Hello World".indexOf("o");          // 4 (first occurrence)
"Hello World".lastIndexOf("o");      // 7
"Hello World".startsWith("Hello");   // true
"Hello World".endsWith("World");     // true
```

**Extracting & transforming**
```java
"Hello".substring(1);       // "ello"
"Hello".substring(1, 3);    // "el"
"Hello".toUpperCase();      // "HELLO"
"Hello".toLowerCase();      // "hello"
"  Hi  ".trim();            // "Hi" (removes leading/trailing whitespace)
"  Hi  ".strip();           // "Hi" (Java 11+, Unicode-aware trim)
"Hello".replace('l', 'L');  // "HeLLo"
```

**Splitting & joining**
```java
"a,b,c".split(",");              // ["a","b","c"]
String.join("-", "a", "b", "c"); // "a-b-c"
```

**Building**
```java
"Hello".concat(" World");   // "Hello World" (new object)
"Hello" + " " + "World";    // compiler converts this to StringBuilder internally
```

**Formatting**
```java
String.format("Name: %s, Age: %d", "Tom", 25);
```

**Strings in `switch` (Java 7+)**
Internally uses `.hashCode()` + `.equals()` checks:
```java
switch (day) {
    case "MON": System.out.println("Monday"); break;
    case "TUE": System.out.println("Tuesday"); break;
    default: System.out.println("Unknown");
}
```

---

## 8. `StringBuilder` vs `StringBuffer` — Thread Safety

### Single-threaded application
A program where only **one sequence of instructions** runs at a time — like one cook following a recipe top to bottom, one step at a time. No risk of two pieces of code touching the same object at the exact same moment, because there's only one "worker" (thread).

```java
public static void main(String[] args) {
    StringBuilder sb = new StringBuilder();
    sb.append("Hello");
    sb.append(" World");
    System.out.println(sb);
}
```

### Multi-threaded application
A program where **multiple threads** (independent sequences of execution) run *concurrently*, potentially at the exact same time, often sharing the same objects/memory. Like hiring a second cook who works at the same time as the first, in the same kitchen (same shared objects/memory).

```java
Thread t1 = new Thread(() -> sb.append("Hello"));
Thread t2 = new Thread(() -> sb.append("World"));
t1.start();
t2.start();
```

**Line-by-line breakdown:**
- `Thread t1 = new Thread(() -> sb.append("Hello"));` — does NOT run anything yet. Just prepares a "cook" (`t1`) with an instruction card: *"when you start, run `sb.append("Hello")`"*.
- `Thread t2 = new Thread(() -> sb.append("World"));` — same thing, prepares a second cook with a different instruction.
- At this point, nothing has executed — both cooks are standing by.
- `t1.start();` — cook `t1` actually starts running `sb.append("Hello")`.
- `t2.start();` — cook `t2` also starts running `sb.append("World")`.

**Key point:** `start()` does NOT wait. Calling `t1.start()` tells `t1` "go!" and immediately continues to the next line — so `t2.start()` fires almost immediately after, while `t1` might still be running. Both threads end up working **at the same time**, in parallel, not one after another.

**Why this is dangerous:** Both threads work on the same shared `sb` object. If two threads try to write into the same notebook at the same instant, they can collide — one's write gets partially overwritten, or the object ends up in a broken half-written state. You don't know if the output will be `"HelloWorld"`, `"WorldHello"`, or something corrupted.

```
Time →
Thread t1:  [ start ] → running sb.append("Hello") ......→ done
Thread t2:      [ start ] → running sb.append("World") ..→ done
```
Notice `t1` and `t2` overlap in time — that overlap is exactly what "concurrent" means, and why shared objects need protection if multiple threads touch them.

### What "thread-safe" means
A class or piece of code is **thread-safe** if it behaves correctly even when multiple threads use it *simultaneously*, without external synchronization from the caller. "Correctly" means no corrupted data, no lost updates, no crashes — even under concurrent access.

### Why non-thread-safe code breaks
`StringBuilder.append()` internally does roughly:
1. Check current buffer size
2. If not enough space, grow the internal array
3. Copy characters in
4. Update the length counter

If two threads run these steps at the same time, they can interleave in a broken way — e.g. both threads check the size *before* either grows the array, then both try to write into the same slots. Result: corrupted text, lost characters, or even a crash (`ArrayIndexOutOfBoundsException`).

### How `StringBuffer` avoids this
`StringBuffer`'s methods are `synchronized` — only **one thread at a time** is allowed to execute `append()` (or any other method) on that object. If thread A is inside `append()`, thread B must **wait** until A finishes before it can start. This guarantees correctness, but adds locking overhead — which is why `StringBuffer` is slower than `StringBuilder`.

**Simulated timeline (unsafe vs safe):**
- **Unsafe (`StringBuilder`)**: both threads start at the same instant and overlap → buffer ends up scrambled/unpredictable (e.g. `"HeWorlldo"`), because nothing stops them writing at the same moment.
- **Safe (`StringBuffer`)**: `t1` acquires the lock first; `t2` waits. Only once `t1` fully finishes does `t2` get its turn → always correctly `"HelloWorld"`, never garbled. This waiting is exactly what "thread-safe" buys — correctness, at the cost of one thread sometimes sitting idle while it waits its turn.

### Side-by-side comparison

| | `StringBuilder` | `StringBuffer` |
|---|---|---|
| Thread-safe? | No | Yes (synchronized methods) |
| Speed | Faster | Slower (locking overhead) |
| Use case | Single-threaded code (most common case) | Multiple threads modifying the same buffer |

### The practical rule
- **90% of the time**, your code (loops, single-method string building, normal app logic) runs on **one thread** → use `StringBuilder`.
- Only reach for `StringBuffer` if you *know* multiple threads will genuinely share and modify the **same** buffer object concurrently. This is relatively rare — most multi-threaded programs give each thread its own local `StringBuilder`, avoiding the problem entirely rather than paying the locking cost.

---

## 9. Quick Summary Table

| Concept | Key fact |
|---|---|
| Storage | UTF-16 code units (or compact byte[] since Java 9) |
| Mutability | Immutable — every "change" makes a new object |
| Literal creation | Uses/reuses the String Pool |
| `new String()` creation | Always makes a new heap object |
| `==` | Compares references (memory address) |
| `.equals()` | Compares actual content |
| `intern()` | Forces a heap string into the pool |
| Fast mutation | Use `StringBuilder` (or `StringBuffer` for thread safety) |
| Pool location | Heap (since Java 7); was PermGen before |
| Thread-safe class | Behaves correctly under simultaneous multi-thread access |
| `StringBuilder` | Fast, NOT thread-safe |
| `StringBuffer` | Slower, synchronized/thread-safe |