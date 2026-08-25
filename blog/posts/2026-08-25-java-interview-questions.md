# Java Interview Questions & Answers

This reference collects 100 of the most commonly asked core Java interview questions, spanning the language, the platform, and the standard library. It is aimed at candidates preparing for backend, full-stack, and platform roles where solid command of the JVM and modern Java is expected. Questions are grouped into themed sections — Fundamentals, OOP, Strings, Collections, Generics, Exceptions, Memory & JVM internals, Concurrency, Java 8+ features, and Best practices — and are numbered continuously so you can track your progress. Each answer aims to be concise but interview-ready, with code snippets and bullet lists where they clarify the point.

---

## Fundamentals

**1. What is the difference between JDK, JRE, and JVM?**

The three form a layered stack:

- **JVM (Java Virtual Machine)** is the abstract runtime that executes Java bytecode. It is platform-specific and provides class loading, bytecode verification, JIT compilation, and garbage collection.
- **JRE (Java Runtime Environment)** is the JVM plus the standard class libraries (`rt.jar`/module runtime) needed to *run* Java applications. It contains no compiler.
- **JDK (Java Development Kit)** is the JRE plus development tools such as `javac`, `jar`, `javadoc`, and `jdb`. You need the JDK to *build* Java code.

In short: JDK ⊃ JRE ⊃ JVM.

**2. What is bytecode and why does it matter?**

Bytecode is the intermediate, platform-neutral instruction set (`.class` files) that `javac` produces from source. The JVM interprets or JIT-compiles this bytecode into native machine code at runtime. Because bytecode targets the JVM rather than a physical CPU, the same compiled artifact runs on any platform that has a compatible JVM — the foundation of Java's portability.

**3. How does Java achieve platform independence?**

Java compiles to bytecode rather than native code. Each operating system/architecture ships its own JVM implementation that understands the same standardized bytecode. This is the "write once, run anywhere" model: the compiled `.class` or `.jar` is portable, and only the JVM itself is platform-specific. Note that platform independence applies to the bytecode, not to the JVM binary.

**4. What is the difference between primitive types and objects (reference types)?**

Primitives (`int`, `long`, `double`, `boolean`, `char`, `byte`, `short`, `float`) hold their value directly, live on the stack (when local), have no methods, and cannot be `null`. Reference types point to objects on the heap, carry behavior, participate in inheritance, and can be `null`. Primitives are faster and more memory-efficient; objects are needed for generics, collections, and polymorphism.

**5. What is autoboxing and unboxing?**

Autoboxing is the automatic conversion of a primitive to its wrapper (`int` → `Integer`); unboxing is the reverse. The compiler inserts `Integer.valueOf(...)` and `intValue()` calls for you.

```java
List<Integer> list = new ArrayList<>();
list.add(5);          // autobox: int -> Integer
int x = list.get(0);  // unbox: Integer -> int
```

Watch out for two pitfalls: unboxing a `null` wrapper throws `NullPointerException`, and boxing in tight loops creates garbage. Also, `Integer` caches values in the range -128..127, so `==` on cached values may surprise you.

**6. Is Java pass-by-value or pass-by-reference?**

Java is strictly **pass-by-value**. For primitives, the value is copied. For objects, the *reference* is copied by value — so the method gets its own copy of the reference pointing at the same object. You can mutate the object's fields through that reference, but reassigning the parameter does not affect the caller's variable.

```java
void f(StringBuilder sb) {
    sb.append("!");   // visible to caller (same object)
    sb = new StringBuilder("new"); // NOT visible to caller
}
```

**7. What is the difference between `==` and `.equals()`?**

`==` compares references (identity) for objects and raw values for primitives. `.equals()` compares logical equality as defined by the class. `String`, wrapper types, and most value classes override `equals()` to compare content. For your own classes, `equals()` defaults to `==` (identity) unless overridden.

**8. What are the default values of instance fields?**

Numeric types default to `0` (`0.0` for floating point), `boolean` to `false`, `char` to `'\u0000'`, and reference types to `null`. Local variables have **no** default and must be explicitly initialized before use — the compiler enforces this.

**9. What is the difference between `int` and `Integer`?**

`int` is a primitive; `Integer` is its wrapper class. `Integer` can be `null`, can be stored in collections and generics, provides utility methods (`parseInt`, `compareTo`), and participates in autoboxing. It carries object overhead and is immutable.

**10. What does the `static` keyword mean?**

`static` associates a member with the class rather than any instance. Static fields are shared across all instances; static methods can be called without an object and cannot access instance state or `this`. Static blocks run once at class initialization. Static nested classes do not hold a reference to the enclosing instance.

**11. What is the `main` method signature and why is it that way?**

```java
public static void main(String[] args) { }
```

It is `public` so the JVM can call it from outside the class, `static` so no instance is needed, `void` because it returns nothing to the launcher, and takes `String[] args` for command-line arguments. The name `main` is the fixed entry point the launcher looks for.

**12. What is the difference between `final`, `finally`, and `finalize`?**

- `final` — a modifier that makes variables constant, methods non-overridable, and classes non-subclassable.
- `finally` — a block after `try`/`catch` that always executes (used for cleanup).
- `finalize()` — a deprecated `Object` method the GC *may* call before reclaiming an object; unreliable and removed in recent JDKs. Prefer try-with-resources or `Cleaner`.

---

## Object-Oriented Programming

**13. What are the four pillars of OOP?**

- **Encapsulation** — bundling state and behavior, hiding internals behind access modifiers and exposing a controlled API.
- **Inheritance** — deriving a class from another to reuse and specialize behavior.
- **Polymorphism** — one interface, many implementations; the actual method invoked is resolved at runtime.
- **Abstraction** — modeling essential features while hiding implementation detail, via abstract classes and interfaces.

**14. What is the difference between an interface and an abstract class?**

An abstract class can have state (instance fields), constructors, and a mix of concrete and abstract methods; a class can extend only one. An interface primarily declares a contract, supports multiple inheritance of type, and (since Java 8) can carry `default` and `static` methods, plus `private` methods since Java 9. Use an abstract class for shared implementation and identity; use an interface for capability/contract that unrelated types can implement.

**15. What is the difference between method overloading and overriding?**

Overloading is compile-time (static) polymorphism: same method name, different parameter lists in the same class. Overriding is runtime (dynamic) polymorphism: a subclass provides a new implementation of an inherited method with the same signature. Overriding is resolved by the actual object type at runtime; overloading is resolved by the compiler from the declared argument types.

**16. What are the rules for overriding a method?**

- Same name and parameter list; return type must be the same or a covariant subtype.
- Access modifier cannot be more restrictive than the parent's.
- Cannot throw broader checked exceptions than the overridden method.
- `static`, `final`, and `private` methods cannot be overridden (a `static` with the same signature is *hidden*, not overridden).
- Use `@Override` to let the compiler catch mistakes.

**17. What is the difference between `this` and `super`?**

`this` refers to the current instance and is used to access fields/methods or call another constructor via `this(...)`. `super` refers to the immediate parent — used to call the parent constructor via `super(...)` or to invoke a parent method/field hidden by the subclass. Both constructor calls must be the first statement in the constructor.

**18. What is composition and how does it compare to inheritance?**

Composition models a "has-a" relationship by holding references to other objects, whereas inheritance models "is-a." Composition is more flexible, avoids fragile base-class problems, and is favored by the principle "prefer composition over inheritance." Inheritance couples subclass to superclass implementation; composition delegates and can vary behavior at runtime.

**19. Can you override a `static` method?**

No. Static methods belong to the class and are resolved at compile time by the reference type. Declaring a static method with the same signature in a subclass *hides* the parent's method rather than overriding it — there is no dynamic dispatch.

**20. What is the difference between an abstract method and a concrete method?**

An abstract method has no body and must be declared in an abstract class or interface; subclasses must implement it (unless they are also abstract). A concrete method has a full implementation. A class with any abstract method must itself be abstract.

**21. What are access modifiers in Java?**

- `private` — visible only within the declaring class.
- default (package-private) — visible within the same package.
- `protected` — visible within the package and to subclasses.
- `public` — visible everywhere.

They enforce encapsulation by controlling the surface area of a type.

**22. What is dynamic method dispatch?**

It is the mechanism by which a call to an overridden method is resolved at runtime based on the actual object type, not the reference type. This is how Java implements runtime polymorphism — the JVM uses the object's vtable to pick the correct override.

**23. Can a constructor be inherited or overridden?**

No. Constructors are not members that get inherited, and they cannot be overridden. A subclass constructor implicitly or explicitly calls a superclass constructor via `super(...)`. Constructors can, however, be overloaded within a class.

**24. What is the `Object` class and what key methods does it provide?**

`Object` is the root of every class hierarchy. Its notable methods include `equals()`, `hashCode()`, `toString()`, `getClass()`, `clone()`, `wait()/notify()/notifyAll()`, and (historically) `finalize()`. Overriding `equals`/`hashCode`/`toString` is common; the thread-signaling methods underpin `synchronized` coordination.

---

## Strings

**25. Why are Strings immutable in Java?**

Immutability enables safe sharing in the String pool, makes strings usable as `HashMap` keys (cached hash code), improves thread-safety without synchronization, and enhances security (a string used in a class-loading or network call cannot be mutated after validation). The trade-off is that every "modification" creates a new object.

**26. What is the String pool?**

The String pool (string intern table) is a region where the JVM stores unique string literals so identical literals share one object. String literals are automatically interned; you can force interning of a runtime-built string with `.intern()`.

```java
String a = "hi";
String b = "hi";
System.out.println(a == b);          // true (same pooled object)
String c = new String("hi");
System.out.println(a == c);          // false (new heap object)
System.out.println(a == c.intern()); // true
```

**27. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?**

- `String` — immutable; each change creates a new object.
- `StringBuilder` — mutable, not synchronized, fast; the default choice for single-threaded string building.
- `StringBuffer` — mutable and synchronized (thread-safe), so slightly slower.

Use `StringBuilder` for local concatenation in loops; use `StringBuffer` only when a builder is shared across threads (rare).

**28. Why is String a popular choice for HashMap keys?**

Because it is immutable and caches its hash code, a `String` key's hash never changes after insertion, so lookups remain correct and efficient. Immutability also guarantees the key cannot be altered to violate the map's invariants.

**29. What does `String.intern()` do?**

It returns the canonical instance from the string pool: if an equal string is already pooled, that reference is returned; otherwise the current string is added and returned. It lets you compare runtime strings by reference, but overuse can pressure the pool's memory.

**30. How do you compare strings correctly?**

Use `.equals()` for content equality and `.equalsIgnoreCase()` for case-insensitive comparison. Never use `==`, which compares references. For ordering, use `.compareTo()`. A common null-safe idiom is `"constant".equals(userInput)`.

**31. What is the difference between `String.concat()` and the `+` operator?**

`+` on strings is compiled (historically) into `StringBuilder` operations or, in modern JDKs, `invokedynamic` via `StringConcatFactory`, and it handles `null` and non-string operands gracefully. `concat()` is a plain method that requires a non-null `String` argument and throws `NullPointerException` on `null`. For repeated concatenation, use an explicit `StringBuilder`.

---

## Collections Framework

**32. What is the Java Collections Framework?**

It is a unified architecture of interfaces (`Collection`, `List`, `Set`, `Queue`, `Map`), implementations (`ArrayList`, `HashSet`, `HashMap`, etc.), and algorithms (`Collections.sort`, `binarySearch`). It provides reusable data structures with consistent APIs, iterators, and interoperability.

**33. What is the difference between `List`, `Set`, and `Map`?**

- `List` — ordered, index-accessible, allows duplicates (`ArrayList`, `LinkedList`).
- `Set` — no duplicates, models a mathematical set (`HashSet`, `TreeSet`, `LinkedHashSet`).
- `Map` — key-value pairs with unique keys (`HashMap`, `TreeMap`, `LinkedHashMap`).

`Map` does not extend `Collection` because its element model (pairs) differs.

**34. What is the difference between `ArrayList` and `LinkedList`?**

`ArrayList` is backed by a resizable array: O(1) random access, O(1) amortized append, but O(n) insert/remove in the middle (due to shifting). `LinkedList` is a doubly linked list: O(1) insert/remove at the ends given a node reference, but O(n) index access. In practice `ArrayList` wins for most workloads due to cache locality; `LinkedList` is rarely the best choice except as a `Deque`.

**35. How does a `HashMap` work internally?**

A `HashMap` stores entries in an array of buckets. On `put`, it computes `hashCode()`, spreads the bits (to reduce collisions), and maps it to a bucket index. Colliding entries form a linked list within the bucket; since Java 8, a bucket converts to a balanced (red-black) tree once it exceeds a threshold (8 entries, with table size ≥ 64), improving worst-case lookup from O(n) to O(log n). When the load factor (default 0.75) is exceeded, the table resizes (doubles) and rehashes.

**36. What is the contract between `equals()` and `hashCode()`?**

- If two objects are equal by `equals()`, they must return the same `hashCode()`.
- Equal hash codes do not require equality (collisions are allowed).
- `hashCode()` must be consistent across calls while the object's equals-relevant state is unchanged.

Violating this breaks hash-based collections: an object may become unfindable in a `HashMap` or `HashSet`.

**37. What is the difference between `HashMap`, `Hashtable`, and `ConcurrentHashMap`?**

- `HashMap` — unsynchronized, allows one `null` key and `null` values, fast for single-threaded use.
- `Hashtable` — legacy, fully synchronized on every method (coarse lock), no `null` keys/values; largely obsolete.
- `ConcurrentHashMap` — thread-safe with fine-grained locking (bucket/bin-level CAS and synchronized bins), high concurrency, no `null` keys/values.

Prefer `ConcurrentHashMap` over `Hashtable` for concurrent access.

**38. How does `TreeMap` order its keys?**

`TreeMap` is a red-black tree that keeps keys sorted by their natural ordering (`Comparable`) or by a `Comparator` supplied at construction. Operations are O(log n). It supports navigation methods (`floorKey`, `ceilingKey`, `firstKey`, `subMap`). Keys must be mutually comparable and effectively immutable with respect to ordering.

**39. What is the difference between fail-fast and fail-safe iterators?**

Fail-fast iterators (on `ArrayList`, `HashMap`, etc.) throw `ConcurrentModificationException` if the collection is structurally modified during iteration, detected via a `modCount` check. Fail-safe iterators (on `CopyOnWriteArrayList`, `ConcurrentHashMap`) operate on a snapshot or tolerate concurrent modification without throwing, at the cost of possibly not reflecting the latest state.

**40. What is the difference between `Comparable` and `Comparator`?**

`Comparable` defines a type's natural ordering via `compareTo(T)` implemented inside the class. `Comparator` is an external strategy (`compare(T, T)`) that can define multiple orderings without touching the class.

```java
list.sort(Comparator.comparing(Person::getLastName)
                     .thenComparing(Person::getFirstName));
```

Use `Comparable` for the single, intrinsic ordering; use `Comparator` for alternative or ad-hoc orderings.

**41. What is the difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?**

`HashSet` gives O(1) operations with no ordering guarantee. `LinkedHashSet` preserves insertion order via a linked list. `TreeSet` keeps elements sorted (red-black tree), O(log n), and supports navigation. Choose based on whether you need ordering and at what performance cost.

**42. How do you make a collection immutable or read-only?**

Use factory methods `List.of(...)`, `Set.of(...)`, `Map.of(...)` (Java 9+) for truly immutable collections, or `Collections.unmodifiableList(...)` for an unmodifiable view over an existing collection. Note that an unmodifiable *view* still reflects changes to the backing collection; the `of` factories are genuinely immutable and reject `null`.

**43. What is the difference between `Iterator` and `ListIterator`?**

`Iterator` traverses forward only and supports `remove()`. `ListIterator` (for lists) traverses both directions, can `add()`, `set()`, and report indices (`nextIndex`, `previousIndex`). Use `ListIterator` when you need bidirectional traversal or in-place modification.

**44. How do you sort a list of objects by multiple fields?**

Use chained comparators:

```java
people.sort(
    Comparator.comparingInt(Person::getAge)
              .reversed()
              .thenComparing(Person::getName));
```

`Comparator` factory methods (`comparing`, `thenComparing`, `reversed`, `nullsFirst`) compose cleanly and read well.

**45. What is the load factor and initial capacity of a `HashMap`?**

Default initial capacity is 16 buckets and the default load factor is 0.75, meaning the map resizes (doubles) when size exceeds capacity × load factor (12 by default). Sizing the map up front (`new HashMap<>(expectedSize / 0.75 + 1)`) avoids repeated rehashing for large, known datasets.

---

## Generics

**46. What are generics and why are they useful?**

Generics parameterize types over classes, interfaces, and methods, providing compile-time type safety and eliminating explicit casts. `List<String>` guarantees only strings go in and come out, catching type errors at compile time rather than as runtime `ClassCastException`s.

**47. What is type erasure?**

Generics are a compile-time feature: the compiler checks types then *erases* them, replacing type parameters with their bounds (or `Object`) and inserting casts. At runtime `List<String>` and `List<Integer>` are both just `List`. Consequences: you cannot do `new T()`, `instanceof List<String>`, or overload solely by generic type, and arrays of generic types are not directly creatable.

**48. What is the difference between `? extends T` and `? super T`?**

This is the PECS rule — "Producer Extends, Consumer Super":

- `? extends T` — an upper-bounded wildcard; you can *read* items as `T` but cannot add (except `null`). Use when the structure produces values.
- `? super T` — a lower-bounded wildcard; you can *add* `T` (and subtypes) but reads come back as `Object`. Use when the structure consumes values.

```java
void copy(List<? super T> dest, List<? extends T> src) { ... }
```

**49. What is an unbounded wildcard `<?>` and when is it used?**

`List<?>` means a list of some unknown type. It is useful when your code only relies on `Object`-level behavior (e.g., `size()`, `clear()`, printing) and does not need to add typed elements. You cannot add anything except `null` to a `List<?>`.

**50. What is a bounded type parameter?**

A type parameter constrained to a supertype, e.g., `<T extends Number & Comparable<T>>`. It lets the method call the bound's methods on `T` and restricts callers to compatible types. Multiple bounds are separated by `&`, with at most one class bound (listed first).

**51. Can you create a generic array? Why or why not?**

Not directly — `new T[10]` is illegal because of type erasure, which would make the array's runtime type checks unsound. Workarounds include creating an `Object[]` and casting (with `@SuppressWarnings`), or using `Array.newInstance(clazz, n)` with a `Class<T>` token, or simply using collections instead.

**52. What is a generic method?**

A method that declares its own type parameters independent of the class:

```java
public static <T> T firstOrNull(List<T> list) {
    return list.isEmpty() ? null : list.get(0);
}
```

The compiler usually infers `T` from arguments, so callers rarely specify it explicitly.

---

## Exception Handling

**53. What is the difference between checked and unchecked exceptions?**

Checked exceptions (subclasses of `Exception` excluding `RuntimeException`) must be declared or caught — the compiler enforces handling (e.g., `IOException`, `SQLException`). Unchecked exceptions (`RuntimeException` and subclasses like `NullPointerException`, `IllegalArgumentException`) represent programming errors and need not be declared. `Error` (e.g., `OutOfMemoryError`) signals serious JVM problems you generally should not catch.

**54. What is the exception hierarchy in Java?**

`Throwable` is the root, with two branches: `Error` (unrecoverable JVM conditions) and `Exception`. `Exception` splits into checked exceptions and `RuntimeException` (unchecked). You catch `Throwable`/`Exception` at the top but should catch the most specific type you can handle.

**55. What is the difference between `throw` and `throws`?**

`throw` is a statement that actually raises an exception instance: `throw new IllegalStateException("bad")`. `throws` is a method-signature clause declaring which checked exceptions the method may propagate: `void read() throws IOException`. One throws an object; the other declares a possibility.

**56. What is try-with-resources?**

Introduced in Java 7, it automatically closes resources that implement `AutoCloseable` at the end of the block, in reverse order of creation, even on exception:

```java
try (var in = Files.newInputStream(path);
     var out = Files.newOutputStream(dest)) {
    in.transferTo(out);
}   // both closed automatically
```

It eliminates verbose `finally` blocks and prevents resource leaks. Exceptions from `close()` are added as *suppressed* exceptions.

**57. Does `finally` always execute?**

Almost always — it runs after `try`/`catch` whether or not an exception occurred, and even after a `return` in the try block. It does *not* run if the JVM exits (`System.exit()`), the thread is killed, or the machine loses power. Avoid `return`ing from `finally`, as it silently discards exceptions.

**58. How do you create a custom exception?**

Extend `Exception` (checked) or `RuntimeException` (unchecked) and provide constructors, ideally including one that chains a cause:

```java
public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Choose checked for recoverable conditions callers should handle, unchecked for programming/precondition failures.

**59. What is exception chaining?**

Wrapping a low-level exception inside a higher-level one while preserving the original as the *cause* (`new ServiceException("failed", ioException)`). This retains the full stack trace context while exposing an abstraction-appropriate exception to callers. Retrieve it with `getCause()`.

**60. What is a multi-catch block?**

Since Java 7 you can catch several unrelated exception types in one clause:

```java
try {
    ...
} catch (IOException | SQLException e) {
    log.error("I/O or DB failure", e);
}
```

The caught variable is implicitly `final`, and the types must not be in a subclass relationship with each other.

**61. What is the difference between `final`, `finally` regarding exception handling and resource cleanup best practices?**

Prefer try-with-resources over manual `finally` cleanup because it is less error-prone and correctly handles suppressed exceptions. If you must use `finally`, guard against `NullPointerException` (resource may be null) and never let cleanup code swallow the primary exception. Keep `finally` blocks short and non-throwing.

---

## Memory & JVM Internals

**62. What is the difference between heap and stack memory?**

The stack holds per-thread frames containing local variables, primitives, and object references; it is fast, LIFO, and automatically reclaimed when a method returns. The heap holds all objects and is shared across threads, managed by the garbage collector. A `StackOverflowError` comes from deep recursion; `OutOfMemoryError` from heap (or metaspace) exhaustion.

**63. What is garbage collection and how does it work?**

GC automatically reclaims heap memory occupied by objects that are no longer reachable from GC roots (stack references, static fields, JNI handles). Most collectors use a mark-and-sweep foundation with generational and/or region-based strategies, tracing live objects from roots and freeing the rest. It frees developers from manual `free()` but does not prevent logical memory leaks.

**64. What is the generational hypothesis and the young/old generation split?**

The hypothesis is that most objects die young. The heap is split into a **young generation** (Eden + two survivor spaces) where new objects are allocated and minor GCs run frequently and cheaply, and an **old (tenured) generation** for long-lived objects, collected less often by more expensive major/full GCs. Objects surviving enough minor collections are promoted to old.

**65. Name some garbage collectors in the JVM.**

- **Serial GC** — single-threaded, small heaps/embedded.
- **Parallel GC** — throughput-oriented, multi-threaded.
- **G1 GC** — region-based, low-pause, the modern default.
- **ZGC** and **Shenandoah** — concurrent, ultra-low-pause collectors for large heaps.

You select one with flags like `-XX:+UseG1GC` or `-XX:+UseZGC`.

**66. What causes a memory leak in Java, given garbage collection?**

Leaks occur when objects remain *reachable* but are never used again, so the GC cannot reclaim them. Common causes: unbounded caches/collections, listeners/callbacks never unregistered, `ThreadLocal`s not cleared in pooled threads, static references holding onto objects, and keys with broken `equals`/`hashCode`. Tools like heap dumps and profilers (VisualVM, MAT) help find them.

**67. What is the JIT compiler?**

The Just-In-Time compiler translates frequently executed ("hot") bytecode into optimized native machine code at runtime. HotSpot uses tiered compilation (C1 client + C2 server) and profile-guided optimizations like inlining, escape analysis, and loop unrolling. This lets Java approach native performance after warm-up while retaining portability.

**68. Explain the class loading process.**

Class loading proceeds through **Loading** (reading bytecode), **Linking** — which is **Verification** (bytecode safety checks), **Preparation** (allocating static fields with defaults), and **Resolution** (resolving symbolic references) — and finally **Initialization** (running static initializers and assignments). Classes load lazily on first active use.

**69. What is the class loader hierarchy and delegation model?**

Class loaders form a parent-first hierarchy: Bootstrap → Platform/Extension → Application (system). The **delegation model** means a loader first asks its parent to load a class and only loads it itself if the parent fails. This prevents core classes from being overridden and avoids duplicate class definitions.

**70. What is metaspace?**

Metaspace (Java 8+, replacing PermGen) stores class metadata — class structures, method data, and the runtime constant pool — in native memory rather than the heap. It grows dynamically by default, reducing the classic `PermGen space` errors, though you can cap it with `-XX:MaxMetaspaceSize`.

**71. What are strong, weak, soft, and phantom references?**

- **Strong** — ordinary references; the object is never GC'd while reachable.
- **Soft** — cleared only when memory is low; good for memory-sensitive caches.
- **Weak** — cleared at the next GC once no strong references remain; used in `WeakHashMap`.
- **Phantom** — enqueued after finalization for post-mortem cleanup via `ReferenceQueue`.

**72. What is escape analysis?**

A JIT optimization that determines whether an object "escapes" the method/thread that created it. If it does not, the JVM can allocate it on the stack (scalar replacement) or eliminate synchronization (lock elision), reducing heap pressure and GC work.

---

## Concurrency & Multithreading

**73. What is the difference between extending `Thread` and implementing `Runnable`?**

Implementing `Runnable` (or `Callable`) is preferred: it separates the task from the thread mechanism, allows the class to extend something else, and works naturally with the Executor framework. Extending `Thread` couples your logic to a thread and wastes the single inheritance slot. Prefer submitting `Runnable`/`Callable` to an executor over managing raw threads.

**74. What is the difference between `start()` and `run()`?**

`start()` creates a new thread and schedules it, which then invokes `run()` on that new thread. Calling `run()` directly just executes the code on the *current* thread — no concurrency happens. Calling `start()` twice throws `IllegalThreadStateException`.

**75. What does the `synchronized` keyword do?**

It provides mutual exclusion and visibility: only one thread can hold a given monitor lock at a time, and entering/exiting a synchronized block establishes a happens-before relationship that flushes memory. You can synchronize a method (locks `this` or the class object for static) or a block on a specific object. Keep synchronized regions small to reduce contention.

**76. What is the `volatile` keyword?**

`volatile` guarantees visibility and ordering for a single variable: reads and writes go to main memory, and it prevents certain reorderings (a happens-before edge). It does **not** provide atomicity for compound operations like `count++`. Use it for flags and safe publication; use locks or atomics for compound updates.

**77. How do `wait()`, `notify()`, and `notifyAll()` work?**

These `Object` methods coordinate threads on a monitor and must be called while holding that object's lock (inside `synchronized`). `wait()` releases the lock and suspends the thread until notified; `notify()` wakes one waiting thread and `notifyAll()` wakes all. Always call `wait()` in a loop that rechecks the condition to guard against spurious wakeups.

```java
synchronized (lock) {
    while (!ready) lock.wait();
    // proceed
}
```

**78. What is the Executor framework?**

`java.util.concurrent`'s abstraction for decoupling task submission from execution. You submit `Runnable`/`Callable` tasks to an `ExecutorService` (backed by a thread pool) instead of creating threads manually. It manages pooling, queuing, and lifecycle (`shutdown`, `awaitTermination`), improving resource control and throughput.

**79. What are the main types of thread pools from `Executors`?**

- `newFixedThreadPool(n)` — fixed number of threads, unbounded queue.
- `newCachedThreadPool()` — grows/shrinks on demand, reuses idle threads.
- `newSingleThreadExecutor()` — one worker, serialized tasks.
- `newScheduledThreadPool(n)` — delayed/periodic tasks.
- `newVirtualThreadPerTaskExecutor()` (Java 21) — a virtual thread per task.

For production, configuring a `ThreadPoolExecutor` explicitly gives control over queue and rejection policy.

**80. What is the difference between `Runnable` and `Callable`?**

`Runnable.run()` returns nothing and cannot throw checked exceptions. `Callable.call()` returns a result and may throw checked exceptions. Submit a `Callable` to an executor to get a `Future<T>` for the result.

**81. What is a `Future` and what are its limitations?**

A `Future<T>` represents the pending result of an asynchronous task; `get()` blocks until completion, and you can `cancel()` or poll `isDone()`. Its limitations — no chaining, no non-blocking callbacks, no easy composition — motivated `CompletableFuture`, which supports `thenApply`, `thenCompose`, `thenCombine`, and exception handling.

**82. What is a deadlock and how do you prevent it?**

A deadlock occurs when two or more threads each hold a lock the other needs, waiting forever. Its four Coffman conditions are mutual exclusion, hold-and-wait, no preemption, and circular wait. Prevent it by acquiring locks in a consistent global order, using `tryLock` with timeouts, minimizing lock scope, and avoiding nested locks where possible.

**83. What is the difference between `synchronized` and `ReentrantLock`?**

`ReentrantLock` (in `java.util.concurrent.locks`) offers more than `synchronized`: `tryLock` (with timeout), interruptible locking, fairness policies, and multiple `Condition` objects. It requires explicit `lock()`/`unlock()` (in a `finally`), whereas `synchronized` is simpler and auto-released. Use `synchronized` by default; reach for `ReentrantLock` when you need its extra capabilities.

**84. What are atomic classes?**

Classes in `java.util.concurrent.atomic` (`AtomicInteger`, `AtomicLong`, `AtomicReference`) provide lock-free, thread-safe operations using CAS (compare-and-swap) hardware instructions. They make compound updates like `incrementAndGet()` atomic without locks, offering high performance under contention. `LongAdder` scales even better for hot counters.

**85. How does `ConcurrentHashMap` achieve thread safety?**

Modern `ConcurrentHashMap` uses fine-grained concurrency: reads are largely lock-free, and writes lock only the individual bin (bucket), using CAS for empty bins and `synchronized` on the bin head for collisions. This allows many threads to update different bins concurrently, far outperforming a globally synchronized `Hashtable`. It rejects `null` keys/values.

**86. What is `CompletableFuture`?**

`CompletableFuture<T>` (Java 8) is a composable, non-blocking future supporting a fluent pipeline of async stages:

```java
CompletableFuture
    .supplyAsync(() -> fetchUser(id))
    .thenApply(User::name)
    .thenAccept(System.out::println)
    .exceptionally(ex -> { log(ex); return null; });
```

It supports combining multiple futures (`allOf`, `thenCombine`), custom executors, and centralized error handling.

**87. What is the difference between a process and a thread?**

A process is an independent program with its own memory space; a thread is a lightweight unit of execution within a process that shares the process's heap and resources. Threads are cheaper to create and communicate faster (shared memory) but require synchronization; processes are isolated and more robust against each other's failures.

**88. What is thread starvation and a livelock?**

Starvation occurs when a thread is perpetually denied CPU or a lock because others monopolize it (e.g., unfair locks, low priority). A livelock is when threads keep responding to each other and changing state but make no progress (e.g., two people repeatedly stepping aside in a corridor). Both are liveness failures distinct from deadlock.

**89. What is the `happens-before` relationship?**

It is the Java Memory Model's ordering guarantee: if action A happens-before action B, A's memory effects are visible to B. Examples: a `volatile` write happens-before subsequent reads of it; unlocking a monitor happens-before a later lock; `Thread.start()` happens-before the thread's actions; a thread's actions happen-before another's return from `join()`.

---

## Java 8+ Features

**90. What is a lambda expression and a functional interface?**

A lambda is a concise anonymous implementation of a single abstract method. A functional interface has exactly one abstract method (optionally annotated `@FunctionalInterface`), which the lambda targets.

```java
Runnable r = () -> System.out.println("run");
Comparator<String> byLen = (a, b) -> a.length() - b.length();
```

Common built-in functional interfaces include `Function`, `Predicate`, `Consumer`, `Supplier`, and `BiFunction`.

**91. What is the Streams API?**

Streams (`java.util.stream`) provide a declarative, pipeline-based way to process sequences of elements with operations like `filter`, `map`, `reduce`, and `collect`. Pipelines are lazy (intermediate ops build up, a terminal op triggers execution) and can run in parallel with `.parallelStream()`.

```java
List<String> names = people.stream()
    .filter(p -> p.getAge() > 18)
    .map(Person::getName)
    .sorted()
    .collect(Collectors.toList());
```

Streams do not mutate the source and are consumed once.

**92. What is the difference between intermediate and terminal stream operations?**

Intermediate operations (`filter`, `map`, `sorted`, `distinct`) are lazy and return a new stream, chaining into a pipeline. Terminal operations (`collect`, `forEach`, `reduce`, `count`, `findFirst`) trigger evaluation and produce a result or side effect, after which the stream is consumed. Without a terminal op, nothing executes.

**93. What is `Optional` and how should it be used?**

`Optional<T>` is a container that may or may not hold a non-null value, designed to make the absence of a result explicit and reduce `NullPointerException`s. Use `map`, `filter`, `orElse`, `orElseThrow`, and `ifPresent` rather than `isPresent()`/`get()`.

```java
String name = findUser(id)
    .map(User::getName)
    .orElse("unknown");
```

Best practice: use it as a return type, not for fields or method parameters.

**94. What are default methods and why were they added?**

Default methods let an interface provide a method implementation using the `default` keyword. They were introduced in Java 8 primarily to evolve interfaces (like adding `stream()` to `Collection`) without breaking existing implementers. If a class inherits conflicting defaults from two interfaces, it must override and can disambiguate with `Interface.super.method()`.

**95. What are method references?**

Shorthand for lambdas that call an existing method, using `::`. Four kinds:

- Static: `Integer::parseInt`
- Instance of a particular object: `System.out::println`
- Instance of an arbitrary object of a type: `String::toLowerCase`
- Constructor: `ArrayList::new`

They improve readability when a lambda merely delegates to one method.

**96. What are records?**

Records (Java 16) are immutable, transparent data carriers. A single declaration generates a canonical constructor, private final fields, accessors, and value-based `equals`, `hashCode`, and `toString`.

```java
public record Point(int x, int y) {}
```

You can add compact constructors for validation and extra methods, but records cannot extend classes and their fields are final. Ideal for DTOs and value objects.

**97. What are sealed classes?**

Sealed classes/interfaces (Java 17) restrict which types may extend or implement them via a `permits` clause, giving you a closed, exhaustive hierarchy:

```java
public sealed interface Shape permits Circle, Square, Triangle {}
```

Permitted subclasses must be `final`, `sealed`, or `non-sealed`. They pair well with pattern matching for `switch`, enabling exhaustiveness checks.

**98. What is `var` and what are its limitations?**

`var` (Java 10) enables local variable type inference — the compiler infers the type from the initializer. It reduces boilerplate for obvious types but is still statically typed (not dynamic). Limitations: only for local variables with an initializer; not for fields, method parameters, or return types; cannot be `null`-initialized alone or used with lambda targets without an explicit type.

**99. What are virtual threads?**

Virtual threads (Java 21) are lightweight threads managed by the JVM rather than the OS, allowing millions of concurrent threads. They make blocking I/O cheap by parking the virtual thread and freeing the underlying carrier (platform) thread. They let you write simple synchronous, thread-per-request code that scales like async, via `Thread.ofVirtual()` or `Executors.newVirtualThreadPerTaskExecutor()`.

**100. What is the difference between `map()` and `flatMap()` in streams?**

`map()` transforms each element one-to-one, producing a stream of the same cardinality. `flatMap()` transforms each element into a stream and flattens the results into one stream — one-to-many. Use `flatMap` to flatten nested structures such as `List<List<T>>` into `Stream<T>`:

```java
orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());
```

---

## Quick-fire round

- **Is `String` a primitive?** No — it is a class, a reference type.
- **Can `main` be overloaded?** Yes, but the JVM only calls the `String[]` version as the entry point.
- **Default access modifier?** Package-private (no keyword).
- **Can an interface have a constructor?** No.
- **Is `null` an instance of anything?** No — `null instanceof X` is always `false`.
- **What does `hashCode()` return by default?** An identity-based int (often derived from the object's memory address).
- **Can you override a `private` method?** No — it is not visible to subclasses.
- **Is `char` signed?** No — `char` is a 16-bit unsigned Unicode code unit.
- **Default size of a `HashMap`?** 16 buckets.
- **Can a `try` exist without a `catch`?** Yes — with a `finally` or as try-with-resources.
- **Is `StringBuilder` thread-safe?** No — use `StringBuffer` for that.
- **Are arrays covariant?** Yes (`String[]` is an `Object[]`), unlike generics.
- **What is the parent of all exceptions?** `Throwable`.
- **Can lambdas capture local variables?** Yes, if they are effectively final.
- **Does `finally` run after `return`?** Yes, before the method actually returns.

Practical advice: interviewers care less about memorized definitions and more about whether you understand *why* — why Strings are immutable, why `equals`/`hashCode` must agree, why `volatile` is not a substitute for a lock. When you answer, state the concept crisply, then give a one-line example or trade-off. Keep a mental catalog of the collection and concurrency choices and when to reach for each, and always be ready to talk about how modern features (records, streams, virtual threads) change the code you would actually write today. Above all, be honest about what you do not know and reason out loud — demonstrating a sound thought process often matters more than a perfect recall of the API.
