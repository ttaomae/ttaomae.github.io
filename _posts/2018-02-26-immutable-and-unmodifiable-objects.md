---
layout: post
title: Immutable and Unmodifiable Objects
categories: software-engineering
tags: java
---
In my [last post]({{ site.baseurl }}{% post_url 2017-12-03-immutable-data %}) I
discussed some of the benefits of immutable objects as well as how to implement
them in Java and other languages. In this post I will introduce *unmodifiable*
objects and discuss how to implement them and some of their uses.

# Unmodifiable Objects
Unmodifiable objects are sort of a mix between immutable and mutable objects.
Externally, they look like immutable objects because their public interface does
not allow modifications. However, they may be internally modified through other
means.

I am taking the term "unmodifiable" from the Java Collections Framework[^origin],
specifically the
[`Collections.unmodifiableX()`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-)
methods, so let's start with an example using one of those.

```java
List<String> mutableList = new ArrayList<>();
mutableList.add("foo");

List<String> unmodifiableList = Collections.unmodifiableList(mutableList);
System.out.println(unmodifiableList.get(0)); // "foo"

mutableList.set(0, "bar");
System.out.println(unmodifiableList.get(0)); // "bar"

unmodifiableList.set(0, "baz"); // Runtime error.
```

There are a few things to note about this example. First, this example doesn't
do anything useful, so don't worry if you don't yet see the purpose of
unmodifiable objects. In future sections I will try to provide more practical
examples. Second, the unmodifiable list is simply a read-only view of the
mutable list; it is only *externally* unmodifiable. If the underlying list
changes, so will the unmodifiable view. This is why `get(0)` returns different
values in each of the print statements.

Lastly, and most importantly, this example will always throw a runtime exception
--- specifically `UnsupportedOperationException` --- on the last line. This is
the primary motivation for this post. The unmodifiable collection looks exactly
like a normal collection, which has the advantage of allowing an unmodifiable
collection to be used anywhere that a normal collection can be used, but it
forces the programmer to keep track of whether or not they are allowed to modify
a particular collection or risk a runtime exception.

# Implementing Unmodifiable Objects
## Unmodifiable Wrapper

One of the simplest ways to implement an unmodifiable object is to just hold a
reference to a regular object and delegate to that object for all accessor
methods.

```java
class UnmodifiablePoint {
    private final Point point;

    public UnmodifiablePoint(Point point) {
        this.point = point;
    }

    public int getX() {
        return this.point.getX();
    }

    public int getY() {
        return this.point.getY();
    }
}
```

For mutator methods, you can either simply not provide those methods, as in the
above example; or, if you want to implement a common interface, you can throw an
`UnsupportedOperationException`, which is what the collections produced by the
`Collections.unmodifiableX()` methods do.

```java
class UnmodifiablePoint implements Point {
    private final Point point;

    public UnmodifiablePoint(Point point) {
        this.point = point;
    }

    public int getX() {
        return this.point.getX();
    }

    public void setX(int x) {
        throw new UnsupportedOperationException();
    }

    public int getY() {
        return this.point.getY();
    }

    public void setY(int y) {
        throw new UnsupportedOperationException();
    }
}
```

This approach is convenient because it allows you to use existing types, but it
is limited in that it only provides runtime guarantees of unmodifiability. Both
of these approaches also require the creation of a new object. In the next
section I will describe an alternative that does not require creating a new
object while also providing compile-time guarantees that an object is not
modified. However, it will most likely require creating the type hierarchy from
scratch; you would not be able to extend an existing class or interface.

## Unmodifiable and Immutable Object Type Hierarchy

If you recall back to the Rust example in my previous post, the compiler was
able to prevent calls to setter methods depending on whether or not a variable
was declared as mutable.

```rust
let mut p1 = Point::new(1, 2);
p1.set_x(3);

let p2 = Point::new(4, 5);
p2.set_x(6); // Compiler error.
```

Java doesn't have anything like Rust's `mut` keyword,
but we can take advantage of polymorphism to get similar behavior[^rust].

In Java it might look something like this:
```java
MutablePoint p1 = Point.create(1, 2);
p1.setX(3);

Point p2 = Point.create(4, 5)
// Compiler error. `p2` does not have a `setX()` method.
p2.setX(6);
```

Note that in the Rust example, `p2` would be immutable and the compiler can
ensure that it is not modified. However, in the Java example, `p2` could
potentially be modified if there is another reference to the same object as a
`MutablePoint`. If you are sure that there will only ever be one reference to
the object, you could effectively treat it as immutable, but the compiler will
not be able to help you.

The type hierarchy that would allow this would look like this:
```java
interface Point {
    int getX();
    int getY();
}

interface MutablePoint extends Point {
    void setX(int x);
    void setY(int y);
}

interface ImmutablePoint extends Point {}
```

You then only need to implement a single mutable point and you get an
unmodifiable point for free. There is no need to create wrappers or subclasses.
```java
class DefaultPoint implements MutablePoint {
    private int x;
    private int y;

    public DefaultPoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int getX() {
        return x;
    }

    @Override
    public int getY() {
        return y;
    }

    @Override
    public void setX(int x) {
        this.x = x;
    }

    @Override
    public void setY(int y) {
        this.y = y;
    }
}
```
Implementations of `ImmutablePoint` would be the same as I described in my previous post.

In the next section I will show an example of how this can be used. But first,
there are a few variations to this design that you may want to use depending on
your exact use-case. I will explain my reasoning for choosing this
particular design as well as some alternatives that you may want to consider.

First, I chose `Point` as the top of the hierarchy to encourage non-mutability
by default, in the same way that variables in Rust are immutable by default and
only mutable if you explicitly declare them as such, using the `mut` keyword.
However, it is also reasonable to make `ReadablePoint`, or something similiarly
named, the top of the hierarchy and make `Point` the type which allows mutation.

Second, since `ImmutablePoint` has the same methods as `Point`, you may be
tempted to simplify the hierarchy by making `ImmutablePoint` the top of the
hierarchy and have `MutablePoint extends ImmutablePoint`. This may seem
convenient at first, but it violates the
[Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle),
which states that if **S** is a subtype of **T** (in this case `MutablePoint` is
a subtype of `ImmutablePoint`) then any object of type **T** can be replaced by
an object of type **S**[^liskov]. This does not hold in this case because we
cannot safely replace any immutable point with a mutable point. If the
programmer is relying on the immutability of the point for the correctness of
the code, then replacing it with a mutable point may result in incorrect
behavior. On the other hand, the `Point` type makes no guarantees about the
mutability of the data. If it is being used correctly, the programmer should
treat it as if it may be modified, so it is safe to replace it with a mutable
point.

Along those same lines, it may seem strange to make `ImmutablePoint` its own
interface when it doesn't define any additional methods. You can certainly do
without it by just treating immutable points as `Point`s, but I think it can be
useful to explicitly differentiate between unmodifiable and immutable objects.

Consider the example below.
```java
Point p1 = Point.createImmutable(1, 2);
ImmutablePoint p2 = Point.createImmutable(3, 4);
```
Both `p1` and `p2` provide exactly the same methods, and neither allows direct
modification, but `p2` provides an extra hint that the point will not be
modified by some other means. That being said, it should be noted that a poorly
or maliciously written implementation of `ImmutablePoint` could still be
modifiable. The type declaration is only a hint to the programmer; the compiler
doesn't know that the value should not change.

Another variation is to replace one or more of the interfaces with abstract or
concrete classes. For a simple type like `Point`, where there is probably not
much benefit to having multiple implementations, it might make sense for
`ImmutablePoint` and `MutablePoint` to be classes rather than interfaces.
However, using an interface provides more flexibility, allowing you to provide
multiple implementations which optimize for different cases.

Lastly, it is important to note is that this approach is "vulnerable" to
casting. By that I mean that a user can simply cast a unmodifiable object to its
mutable variant to gain access to its mutator methods.

```java
Point p = Point.create(1, 2);
((MutablePoint) p).setX(3);
```

This may be acceptable if this class is only used internally, in which case it
is probably enough to prevent accidental modification. However, if the API is
being used by an untrusted third-party, you should make sure that you are not
exposing important internals. If this is a concern, you can use the wrapper
approach and implement the `Point` interface.

```java
class UnmodifiablePoint implements Point {
    private final Point point;

    public UnmodifiablePoint(Point point) {
        this.point = point;
    }

    public int getX() {
        return this.point.getX();
    }

    public int getY() {
        return this.point.getY();
    }
}
```

# Using Unmodifiable Objects
At this point you may still be wondering what exactly is the point of
unmodifiable objects in general, and our proposed type hierarchy in particular,
if the object is in fact still mutable. Because of their underlying mutability,
you cannot use the same assumptions and optimizations as with immutable objects.
However, they do share the advantage of making it easier to reason about your
code by helping you to control when and where objects can be modified.

Consider the following example, which uses the `Point` type hierarchy that I
described in the previous section.

```java
class Circle {
    private final MutablePoint center;

    Point getCenter() {
        return center;
    }

    // ... other methods to modify the center ...
}
```

Here, the `Circle` class exposes its internal `MutablePoint`, but it does so as
an unmodifiable `Point`. This prevents the consumer of this API from directly
manipulating the location of the circle, which could lead to an invalid state,
such as moving tthe center to a location that is outside some boundary. By
limiting the ways that the center can be manipulated, you can guarantee that
the circle will always be in a valid state. Again, this comes with the caveat
that the user could cast the returned `Point` to a `MutablePoint`.

Of course there are other ways to accomplish something similar. You could
instead return a (possibly immutable) copy of the object being exposed. Using
the proposed `Point` type hierarchy, you can do this without changing the
`Circle` interface since an `ImmutablePoint` would still satisfy the `Point`
return type. This would still prevent unwanted modification, but the returned
object now represents the value at exactly the time the method is called,
whereas returning the object itself means that the user will have a constantly
updated view of the value. Both are valid options, so you will have to decide
which makes more sense for your given use case. One thing to keep in mind is
that if you return a copy each time, this can put pressure on the garbage
collector if the user makes many calls to the method. It may also be costly to
create a copy if it is a complex object.

# More Complex Unmodifiable Objects

We can extend the ideas demonstrated by the `Point` type hierarchy to create
more complex types and while still making it easy to expose read-only views of
internal data.

First, let's extend our `Circle` example from the previous section to provide
mutable and non-mutable variants.

```java
interface Circle {
    Point getCenter();
}

interface MutableCircle extends Circle {
    @Override MutablePoint getCenter();
}

class DefaultCircle implements MutableCircle {
    private final MutablePoint center;

    @Override
    public MutablePoint getCenter() {
        return center;
    }
}
```

We can now use these like this:
```java
MutableCircle c1 = Cirlce.create()
// `getCenter()` returns a `MutablePoint`, so `setX()` is available.
c1.getCenter().setX(1);

Circle c2 = Circle.create()
// Compiler error.
// `getCenter()` returns a `Point`, which does not have a `setX()` method.
c2.getCenter().setX(2);
```

The first thing to note is that I've omitted the methods for modifying the
center from `MutableCircle`. However it is still able to provide mutability by
returning a `MutablePoint`. This is a nice way to simplify your interface, but
you should keep in mind that the consumer of this API can now modify the center
without restriction. If this is a concern you should continue to return a
`Point` and provide separate methods for modifying the center.

You may also have noticed that the implementation is essentially exactly the
same as our earlier `Circle` class. However, because of the way that we've
defined our interfaces, we are able to treat it as either unmodifiable or
mutable depending on which interface we use.

We can extend this idea even further to create more complex types. For example,
if you are creating a game you might have something like this[^game]:
```java
interface GameWorld {
    Player getPlayer();
}

interface MutableGameWorld extends GameWorld {
    @Override MutablePlayer getPlayer();
}

interface Player {
    Point getLocation();
    Equipment getEquipment();
}

interface MutablePlayer extends Player {
    @Override MutablePoint getLocation();
    @Override MutableEquipment getEquipment();
    void move(int dx, int dy);
    void setEquipment(Equipment equipment);
}

interface Equipment {
    Weapon getWeapon();
    Armor getArmor();
}

interface MutableEquipment extends Equipment {
    @Override MutableWeapon getWeapon();
    @Override MutableArmor getArmor();
    void setWeapon(Weapon weapon);
    void setArmor(Armor armor);
}

interface Weapon {
    double getBaseDamage();
    double getActualDamage();
}

interface MutableWeapon extends Weapon {
    void setBaseDamage(double damage);
    void setDamageModifier(double modifier);
}

interface Armor {
    double getBaseDefense();
    double getActualDefense();
}

interface MutableArmor extends Armor {
    void setBaseDefense(double defense);
    void setDefenseModifier(double modifier);
}
```

You can then provide different views of the game world to different parts of
your system. For example, the game engine would need a mutable view in order to
make updates. However, the renderer only needs access to the current state of
the world but should not be able to change it.
```java
class GameEngine {
    private MutableGameWorld world;

    private void initialize() {
        world.getPlayer().getLocation().setX(0);
        world.getPlayer().getLocation().setY(0);
        world.getPlayer().getEquipment().getWeapon().setBaseDamage(15.0);
        world.getPlayer().getEquipment().getArmor().setBaseDefense(10.0);
    }
}

class Renderer {
    private GameWorld world;

    private void render() {
        drawPlayerAt(world.getPlayer().getLocation());

        // Compiler error.
        // `GameWorld.getPlayer()` returns a `Player`.
        // `Player.getLocation()` returns a `Point`.
        // `Point` does not have a `setX()` method.
        world.getPlayer().getLocation().setX(1);
    }
}
```

## Limitation on Generics

This pattern can more or less be extended indefinitely, but things get a little
messy when generics are involved.

First let's define a simple generic `List` type.

```java
interface List<E> {
    E get(int i);
}

interface MutableList<E> extends List<E> {
    void add(E e);
}
```

This is fine so far, but let's see what happens when we try to use this.

```java
interface Polygon {
    List<Point> getVertices();
}

interface MutablePolygon extends Polygon {
    @Override MutableList<Point> getVertices();
}
```
In order to provide mutability, we override `getVertices()` and return a mutable
list of unmodifiable points. This only allows you to add or remove vertices, but
you cannot modify existing vertices. If you want to be able to modify existing
verices, you would need to return an unmodifiable or mutable list of mutable
points. However, the compiler won't actually let us do this.

```java
interface MutablePolygon extends Polygon {
    @Override List<MutablePoint> getVertices();
}
// OR
interface MutablePolygon extends Polygon {
    @Override MutableList<MutablePoint> getVertices();
}
```
This will fail to compile because the new return types are not subtypes of
`List<Point>`. The reason for this is a bit complicated, so I won't go into any
more detail other than to say that it has to do with
[covariance and contravariance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)).

If you prefer one of these options, you can modify the `Polygon` interface.
```java
interface Polygon {
    List<? extends Point> getVertices();
}

interface MutablePolygon extends Polygon {
    @Override List<MutablePoint> getVertices();
}
// OR
interface MutablePolygon extends Polygon {
    @Override MutableList<MutablePoint> getVertices();
}
```

This will now compile, but the usage can get a little ugly.

```java
Polygon p = Polygon.create();

// So far so good ...
Point v0 = p.getVertices().get(0);

// ... but, if we try to assign the list to a variable, we get a compiler
// error because List<Point> does not satisfy List<? extends Point>
List<Point> vs = p.getVertices(); // Compiler error.

// If we specify <? extends Point>, it compiles but looks a little messy.
List<? extends Point> vs = p.getVertices();
Point v1 = p.getVertices().get(1)
```

This pattern should still work for even more complicated generic types, such as
those with multiple or nested type parameters. However, it will get even more
messy as your type becomes more complex. For example, let's say you have a
method that returns `Map<Point, List<Point>>`. In order to use this pattern, you
would actually need to specify this as
`Map<? extends Point, ? extends List<? extends Point>>`.

# Conclusion

On the surface, immutable and unmodifiable objects look quite similar. Their
interfaces are the same and they both disallow modification making it easier to
reason about your code. Immutable objects accomplish this by disallowing *all*
modifications which ensures that the data cannot be changed unexpectedly. In
contrast, unmodifiable objects only provide a read-only *view* of the data. The
data cannot be modified through this view, but it can be changed if there are
other references to the data. This doesn't give you as much guarantees about
your data as immutable objects, but it still allows you to control when your
data can be modified by either providing direct access to the mutable data or
providing an unmodifiable view of the data.

In this post I showed how you can create an unmodifiable object by creating a
simple wrapper around a mutable object. I then showed how you can use
polymorphism as another way to provide mutable and read-only views of the same
object without relying on wrapper classes. I then showed how this idea can be
extended to more complex data type as well as limitations of this approach when
dealing with generics.


[^origin]: I don't know where the term originated, but based on a quick search,
    most of the references to unmodifiable objects seem to be related to Java.
    They are sometimes also referred to as "read-only", such as in the .NET
    Framework, which provides a `ReadOnlyCollection` class.

[^rust]: To be clear, I am only claiming to emulate the ability for the compiler
    to prevent calls to certain methods, which is a very small portion of the
    guarantees that Rust provides regarding mutability.

[^liskov]: A less formal, but much simpler way to think of this is to consider
    whether the phrase "**S** is a type of **T**" makes sense. For example, it
    makes sense to say that a "mutable point is a type of point," but it does
    not make sense to say that a "mutable point is a type of immutable point."
    However, keep in mind that the simplification means that it may not always
    be an accurate substitution for the formal definition.

[^game]: Note that I am not a game developer and this is soley intended as a
    demonstration of the type hierarchy. Please do not take this as game design
    advice.
