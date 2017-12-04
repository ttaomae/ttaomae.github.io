---
layout: post
title: Immutable Data (in Java, Kotlin, and Rust)
categories: software-engineering
tags: java kotlin rust
---
Immutable objects are ones which cannot be changed once they have been created. While this is a
simple idea, it is a useful tool for making your program easier to reason about.
This post will discuss some of the benefits of immutable objects --- or more generally, immutable
*data* --- as well as how several different languages can help to make it easier to work with such
data. In particular, I will look at Java as well as [Kotlin](http://kotlinlang.org/) and
[Rust](https://www.rust-lang.org/) which are two relatively new languages that I have been
experimenting with recently.

# Benefits of Immutable Data
## Simplified Reasoning
Consider the following example (in Java) where `Point` is a class representing a coordinate on a 2-d
grid.
```java
public int computeResult(Point p) {
    int result = 0;
    for (int i = 0; i < 100; i++) {
        result += i * p.computeSomething();
    }
    return result;
}
```
If, after some benchmarking, you find that the repeated calls to `computeSomething()` takes up a
large portion of the program execution time, you may be tempted to extract the computation out of
the loop.

```java
public int computeResult(Point p) {
    int something = p.computeSomething()
    int result = 0;
    for (int i = 0; i < 100; i++) {
        result += i * something;
    }
    return result;
}
```
Assuming that the `Point` class is immutable and that the result of `computeSomething()` is only
dependent on the value of `p`, that is a perfectly valid optimization.

However, if `Point` is mutable and `computeSomething()` modifies `p`, the final result may be
different since `computeSomething()` might return a different value each time. In this simple
example, this error would probably be caught by even a basic test suite. However, if you are using a
more complicated object that gets passed between many methods, it may be difficult to write unit
tests that will cover all the ways that the object might mutate. Furthermore, even a simple method
may have unexpected results in a multi-threaded environment, such as the example below.

```java
Point p = new Point(1, 2);

new Thread(() -> {
    computeResult(p);
}).start();

new Thread(() -> {
    p.setX(3);
}).start();
```
In this example `computeResult()` might return different results depending on which version of the
method we use even if `computeSomething()` does not modify the point, since it is modified
externally in a different thread. In addition, the result may change each time this snippet is run
depending on how the threads are scheduled and when `setX()` occurs.

Using immutable objects eliminates these issues and makes it easier to reason about the expected
outputs and behavior of your program. In the first example, it allows you to confidently assert
that the output of `p.computeSomething()` will be consistent which makes the optimization possible.
In the second example, the fact that the object cannot be modified means that it will not change
unexpectedly on a separate thread.


## Optimization
Immutability allows you to make additional assumptions about the object which can enable performance
or memory optimizations. Extending from the `Point` example, let's make it an interface instead.
```java
public interface Point {
    int getX();
    int getY();
    // This represents any method which performs a potentially expensive
    // computation.
    double getDistanceFromOrigin();
}
```
A simple implementation might look like this.
```java
public final class DefaultPoint implements Point {
    private final int x;
    private final int y;

    public DefaultPoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public int getX() {
        return this.x;
    }

    @Override public int getY() {
        return this.y;
    }

    @Override public double getDistanceFromOrigin() {
        double dx = this.x * this.x;
        double dy = this.y * this.y;
        return Math.sqrt(dx + dy);
    }
}
```
This object uses 8 bytes (4 bytes for each `int`, `x` and `y`; ignoring overhead which is present
for all objects) and requires multiple floating-point operations to compute the distance from the
origin.

If you make many calls to `getDistanceFromOrigin()` you may decide to add the following
implementation.
```java
public class ComputationOptimizedPoint implements Point {
    private final int x;
    private final int y;
    private final double distanceFromOrigin;

    public ComputationOptimizedPoint(int x, int y) {
        this.x = x;
        this.y = y;

        double dx = this.x * this.x;
        double dy = this.y * this.y;
        this.distanceFromOrigin = Math.sqrt(dx + dy);
    }

    @Override public int getX() {
        return this.x;
    }

    @Override public int getY() {
        return this.y;
    }

    @Override public double getDistanceFromOrigin() {
        return this.distanceFromOrigin;
    }
}
```

This implementation uses an additional 8 bytes (the size of a `double`) and adds cost to the
constructor for computing the distance from the origin, but it does not require any floating-point
operations when calling `getDistanceFromOrigin()`.

We can also create optimized implementations for specific cases. For example, consider the following
implementation of the origin point. It uses 0 bytes and requires no floating-point operations.
```java
public class OriginPoint implements Point {
    @Override public int getX() {
        return 0;
    }

    @Override public int getY() {
        return 0;
    }

    @Override public double getDistanceFromOrigin() {
        return 0.0;
    }
}
```

8 bytes is not much and in this particular example you might not see much, if any, difference in
practice. However, for more complex classes, this can result in much more drastic memory and
performance improvements. A good example of this are the new
[immutable collections](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/65464a307408/src/java.base/share/classes/java/util/ImmutableCollections.java)
introduced in Java 9, accessible via the new `List.of()`, `Set.of()`, and `Map.of()` methods.

The most drastic difference is probably for a set containing a single element.
[`ImmutableCollections.Set1`](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/65464a307408/src/java.base/share/classes/java/util/ImmutableCollections.java#l334)
requires only enough memory for a single reference (again, I am ignoring standard object overhead,
as well as the object contained in the set since that will be common between implementations). In
contrast, a `HashSet` with a single element requires a `HashMap`, which in turn requires three
`int`s, a `float`, a reference to a cached `entrySet()`, and most importantly a 16-element array,
one element of which will be a `HashSet.Node`, which contains an `int` and three object
references[^hashset]. I won't compare performance because it is significantly more complex to
analyze and would not provide much value to this post. However, given the fact that `Set1` is
implemented in only ~40 lines of code as opposed to `HashSet` and `HashMap` which are nearly 4000
lines combined, it is reasonable to assume that it `Set1` performs fewer operations and would likely
perform better.


## Caching
Another optimization that you can perform is to cache and reuse objects. Continuing with the `Point`
example again, if you have a large number of `Point`s and multiple instances which represent the
same coordinate, you could instead create a cache and re-use the same instance, as in the example
below. This reduces memory usage (there is some overhead in order to maintain the cache, so you
should consider whether or not this will actually benefit your use case) and can improve
performance by eliminating potentially expensive constructor calls.

```java
public class Point {
    private static final Map<Long, Point> CACHE = new HashSet<>();

    private final int x;
    private final int y;

    // Make the constructor private, forcing the use of the factory method.
    private Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public static Point createPoint(int x, int y) {
        // Compute a unique key for retrieving Points from the cache.
        Long key = (long) x << 32 | (long) y;
        return Point.CACHE.computeIfAbsent(key, k -> new Point(x, y));
    }
}
```


# Limitations of Immutable Data
Although I would recommend using immutable objects in most cases, they are not a universal solution
and as with nearly all things in software engineering there are tradeoffs that you should consider.

In particular, immutable objects can be restricting. Immutability can limit the behavior of objects
and it may be difficult or impossible to do certain things efficiently with immutable objects. For
example, suppose we use a `Point` to represent the position of a character in a game. You'll
probably want your character to move, which means the `Point` will need to change. If `Point` is
immutable, that means creating a new instance every time the character moves. Furthermore, if the
character is also immutable, that means you must also create a new character instance each time. In
addition to the overhead of repeatedly creating objects, this can create a lot of garbage and put
pressure on the garbage collector, which will have a negative impact on performance.


# Implementing Immutable Objects ...
Now that we've gone over some of the benefits and use cases of immutable data, let's discuss how to
implement them in Java, Kotlin, and Rust. This is not meant to be a tutorial for any of these
languages, so I will not be going into much detail. The point is simply to compare how a few
different languages treat immutability.


## ... In Java
First we'll start with a fairly standard Java (7+) implementation of an immutable `Point`.
```java
// The `final` keyword is important here. If the class were not final, there
// could be a subclass that breaks immutability.
public final class Point {
    // Making the fields `final` is not strictly necessary as long as they are
    // private and you are careful not to modify the fields. However, doing so
    // allows the compiler help to ensure that you do not change primitives or
    // change object references. However, if one of the fields is an object,
    // the compiler cannot prevent you from mutating it.
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj instanceof Point) {
            Point other = (Point) obj;

            // I am using Objects.equals() to show a more general approach which
            // handles object fields which may be null. If the constructor(s)
            // disallow nulls, then Object.equals(Object) can be used instead.
            // For primitives, '==' can be used instead.
            return Objects.equals(this.x, other.x)
                    && Objects.equals(this.y, other.y);
        }
        return false;
    }

    @Override
    public int hashCode() {
        Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }
}
```

Java is often criticized for being too verbose, although some argue that its verbosity and
explicitness are actually advantages. That is not the point of this post, so I won't say much on the
topic. However, I do think that for classes which are essentially just data containers with little
or no behavior, the above example is quite tedious. Particularly when considering that before Java 7
there was no `Objects` class which would make the implementation of `equals()` and `hashCode()` even
more verbose.

The following examples will show how [AutoValue](https://github.com/google/auto/tree/master/value),
[Immutables](http://immutables.github.io/), and [Project Lombok](https://projectlombok.org/) can be
used to simplify the implementation of such objects in Java. Each example will try to approximate
the plain Java example. One thing to note is that although each tool can generate a `toString()`
methods, the format will differ, so in each case I override the default implementation to match the
above example.

This is not meant to be a comparison of these tools, so I will only cover the most basic use case.
Each of these tools have a number of other features, and while there is a lot of overlap, they each
do things a little differently.


### ... With AutoValue
AutoValue is an
[annotation processor](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Processor.html)
that automatically generates implementations of immutable objects. Below is the AutoValue version of
a `Point` implementation.
```java
@AutoValue
public abstract class Point {
    static Point create(int x, int y) {
        return new AutoValue_Point(x, y);
    }

    public abstract int getX();
    public abstract int getY();

    @Override
    public String toString() {
        return "(" + getX() + ", " + getY() + ")";
    }
}
```

This will automatically generate a concrete subclass (named `AutoValue_Point` in this case) of the
abstract `Point` class using annotation processing; the generated class will be similar to the plain
Java example.

In the `create()` factory method, you call the constructor of the generated class and
you create instances through this factory method.
```java
Point p = Point.create(1, 2);
```


### ... With Immutables
Immutables is another annotation processor similar to AutoValue. Below is the Immutables version of
a `Point`.
```java
@Value.Immutable
public abstract class Point
{
    @Value.Parameter
    public abstract int getX();
    @Value.Parameter
    public abstract int getY();

    @Override
    public String toString() {
        return "(" + getX() + ", " + getY() + ")";
    }
}
```

Like AutoValue, it automatically generates an immutable implementation (named `ImmutablePoint` in
this case) and we are able to override the default implementation if we choose. Unlike AutoValue, it
creates a builder by default (you must opt in to a builder implementation with AutoValue) and only
generates the `of()` factory method if you use the `@Value.Parameter` annotation.
```java
Point p1 = ImmutablePoint.of(1, 2);
Point p2 = ImmutablePoint.builder()
        .x(1)
        .y(2)
        .build();
```


### ... With Project Lombok
On the surface Project Lombok looks very similar to AutoValue and Immutables. However, behind the
scenes, it is quite different --- more on this after the example. Below is a `Point` implementation
using Project Lombok.
```java
@Value
public class Point {
    private final int x;
    private final int y;

    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }
}
```

In contrast to AutoValue and Immutables, which create subclasses, Project Lombok will enhance the
`Point` class bytecode with the appropriate methods. This has the advantage of making its usage
identical to that of the plain Java class.
```java
Point p = new Point(1, 2);
```

However, it is not possible to do this with standard Java annotation processing, so Lombok relies on
internal compiler APIs. Since it is using non-public APIs, it could technically break at any time if
the compiler internals changed and in fact this is essentially what happened in Java 9. In Java 9,
[internal APIs were encapsulated](http://openjdk.java.net/jeps/260) which means that Lombok cannot
run without specifying a number of compiler flags to explicitly allow access to those APIs. Refer to
[this issue](https://github.com/rzwitserloot/lombok/issues/985) tracking Java 9 support for more
information.


## ... In Kotlin
Kotlin is a relatively new language which originally targeted the JVM, but now support compilation
to JavaScript and native platforms. I like to think of Kotlin as what Java would become if the
language designers were able to redesign Java from scratch using everything learned over the past
20+ years and without worrying about backwards compatibility. Obviously this is not really how
Kotlin was designed, but along those lines, one thing they might add is the ability to more easily
create data container classes without relying on annotation processing or compiler hacks. This is
exactly what Kotlin does with its "data classes."

```kotlin
// Classes in Kotlin are final by default and must be explicitly declared
// `open`. Data classes cannot be `open`.
// In Kotlin, you define a primary constructor in the class header.
data class Point(val x: Int, val y: Int) {
    // Like the Java annotation processors, Kotlin provides a `toString()`
    // implementation, but we override it.
    // (If we did not need this, the entire class definition would just be the
    // first line)
    override fun toString(): String {
        // Kotlin supports string interpolation.
        return "($x, $y)"
    }
}
```
Without looking at the generated bytecode (assuming you are targeting the JVM), which I have not
done, it is hard to make exact comparisons to Java. However, this should be very similar to the
original plain Java example, with a few other additional methods required for some other Kotlin
features.

The generated class can be seamlessly used from either Java or Kotlin (and probably other JVM
languages as well, but I don't have any experience with that).
```java
Point p = new Point(1, 2);
```
{:.titled-code-block title="Java"}

```kotlin
// Kotlin does not use the `new` keyword for constructors.
val p1: Point = Point(1, 2)
// The type of `p` can be omitted since it can be inferred by the compiler.
val p2: Point(1, 2)
```
{:.titled-code-block title="Kotlin"}


## ... In Rust
Rust is another relatively new language. There are not many similarities to Java or Kotlin so I
won't try to make any comparisons. However, I think it is still very relevant to this discussion
because of its very different approach to immutability. I am still very much a novice at Rust, so in
order to avoid getting too much wrong, I will not be going into too much detail.

First, let's start with a Rust `Point` implementation. The comments should briefly explain what is
going on.

```rust
// Create a structure of related data.
// Rust does not automatically generate hash code and equality methods, but
// they can often be automatically derived.
#[derive(PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}

// Start an implementation block where we define methods on the struct.
impl Point {
    // Rust does not have constructors, but a common pattern is to provide a
    // `new` function to initialize struct instances.
    fn new(x: i32, y: i32) -> Point {
        Point { x: x, y: y }
    }

    // A `self` parameter indicates that it is a method that operates on an
    // instance of the struct. If it does not have a `self` parameter (as with
    // the `new` function above) then it is an "associated function" and is
    // approximately analogous to a static method in Java.
    fn get_x(&self) -> i32 {
        self.x
    }

    fn get_y(&self) -> i32 {
        self.y
    }

    // The `mut` keyword means that we can mutate the struct.
    // More on this later.
    fn set_x(&mut self, x: i32) {
        self.x = x
    }

    fn set_y(&mut self, y: i32) {
        self.y = y
    }
}

// This implements the std::fmt::Display "trait", which defines the "fmt"
// function. This is approximately equivalent to implementing toString() in
// Java/Kotlin.
impl std::fmt::Display for Point {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```
The first thing to note is that I've included methods to change the `Point`'s data. In most
languages this would mean that the type is not immutable. However, Rust determines mutability per
instance rather than per type. This means that we can have two instances of the same type, one of
which is mutable while the other is immutable.

```rust
// Variables are immutable by default.
let p1 = Point::new(1, 2);

// You must explicitly declare a variable as mutable using the `mut` keyword.
let mut p2 = Point::new(3, 4);

// Compile error because `p1` is not mutable.
p1.set_x(5);

// This is fine since `p2` is mutable.
p2.set_x(6);
```

This is only a small portion of the guarantees that Rust provides about mutability, but I don't want
this to turn into a Rust tutorial so won't go into any more detail[^rust-mutability]. The point of
this section was simply to demonstrate that Rust treats mutability very differently than many other
languages. It treats it as a fundamental part of the language and makes immutability the default
rather than requiring the programmer to opt in to immutability.


# Conclusion
Immutability is a simple yet powerful tool allowing you to write simpler and more efficient code. We
briefly explored three different languages and showed how some more modern languages such as Kotlin
and Rust embrace and encourage immutability. Kotlin does this by providing features such as data
classes to make it easier to create immutable classes. Rust does this by making mutability a
fundamental part of the language and requiring you to explicitly declare that a value is mutable.

Although Java does not make it as easy to work with immutable data as Kotlin or Rust, there are a
number of third-party tools that can help. Even if you were not convinced by my arguments, the fact
that there are at least three such tools in popular usage should give an impression of the value of
immutable data.


[^hashset]: This is based on the current OpenJDK implementation of
    [`HashSet`](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/65464a307408/src/java.base/share/classes/java/util/HashSet.java) and
    [`HashMap`](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/65464a307408/src/java.base/share/classes/java/util/HashMap.java).
    The exact memory usage may differ in different or future implementations and I may have gotten
    some of the details wrong about the current implementation, but the fact remains that there is a
    drastic difference between a general set implementation and a specialized implementation that
    only needs to handle one specific case.

[^rust-mutability]: For more information, check out the
    [rust book](https://doc.rust-lang.org/book/second-edition/), particularly the chapter on
    [ownership](https://doc.rust-lang.org/stable/book/second-edition/ch04-00-understanding-ownership.html)
    which is fundamental part of the language that determines how data can be accessed and mutated.