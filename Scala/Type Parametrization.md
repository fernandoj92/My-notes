Type Parametrization
======

Type parameterization allows you to write generic classes and traits. For example, sets are generic and take a type parameter: they are defined as `Set[T]`. As a result, any particular set instance might be a `Set[String]`, a `Set[Int]`, etc., but it must be a set of _something_. Unlike Java, which allows raw types, Scala requires that you specify type parameters. Variance defines inheritance relationships of parameterized types, such as whether a `Set[String]`, for example, is a subtype of `Set[AnyRef]`.



```scala
trait Queue[T] {
    def head: T
    def tail: Queue[T]
    def enqueue(x: T): Queue[T]
}
object Queue {
    // constructs a queue with initial elements ‘xs’
    // T* is the notation for repeated parameters
    def apply[T](xs: T*): Queue[T] = new QueueImpl[T](xs.toList, Nil)
    // Using a private class to hide implementation
    private class QueueImpl[T]( private val leading: List[T], private val trailing: List[T] ) extends Queue[T] {
            def mirror =
            if (leading.isEmpty)
                new QueueImpl(trailing.reverse, Nil)
            else this
            
            def head: T = mirror.leading.head

            def tail: QueueImpl[T] = {
                val q = mirror
                new QueueImpl(q.leading.tail, q.trailing)
            }

            def enqueue(x: T) = new QueueImpl(leading, x :: trailing)
    }
}
```
There’s a trait `Queue`, which declares the methods head, tail, and enqueue. All three methods are implemented in a subclass `QueueImpl`, which is itself a private inner class of object `Queue`. This exposes to clients the definition of 3 methods and also defines a default implementation, which is hidden, only accesible by using the apply method in the companion object. Another possibility for hiding information to clients is to use private constructors and methods.

`Queue` is atrait, but not a type. `Queue` is not a type because it takes a type parameter. As a result, you cannot create variables of type `Queue`. Instead, trait `Queue` enables you to specify parameterized types, such as `Queue[String]`, `Queue[Int]`, or `Queue[AnyRef]`. `Queue` is also called a type constructor, because with it you can construct a type by specifying a type parameter.

The combination of type parameters and subtyping poses some interesting questions. For example, if `S` is a subtype of type `T`, then should `Queue[S]` be considered a subtype of `Queue[T]`? If so, you could say that trait `Queue` is **covariant** (or _flexible_) in its type parameter `T`.

In Scala, however, generic types have by default nonvariant (or, _rigid_) subtyping. That is, with Queue defined as in the previous code, queues with different element types would never be in a subtype relationship. A `Queue[String]` would not be usable as a Queue[AnyRef]. However, you can demand covariant (flexible) subtyping of queues by changing the first line of the definition of class Queue like this:

```scala
trait Que[+T]{ ... }
```
Prefixing a formal type parameter with a + indicates that subtyping is **covariant** (flexible) in that parameter. By adding this single character, you are telling Scala that you want `Queue[String]`, for example, to be considered a subtype of `Queue[AnyRef]`.

Besides +, there is also a prefix, which indicates **contravariant** subtyping. If Queue were defined like this:

```scala
trait Que[-T]{ ... }
```

then if T is a subtype of type S, this would imply that `Queue[S]` is a subtype of `Queue[T]` (which in the case of queues would be rather surprising!). Whether a type parameter is covariant, contravariant, or nonvariant is called the parameter’s variance . The + and symbols you can place next to type parameters are called _variance annotations_.

#### Checking variance annotiations

However, there are some situations that make covariance unsound. For instance, if we define the class Cell like this:

```scala
class Cell[T](init: T) {
    private[this] var current = init
    def get = current
    def set(x: T) { current = x }
}
```
Then we could construct the following problematic statement sequence:

```scala
val c1 = new Cell[String]("abc")
val c2: Cell[Any] = c1
c2.set(1)
val s: String = c1.get
```
which in the end it would be assigning an integer to a String. Thankfully, the scala compiler doesn't allow this kind of implementation, so we would be covered for possible runtime errors. It’s a welcome relief that the Scala compiler keeps track of variance positions. Once the variances are computed, the compiler checks that each type parameter is only used in positions that are classified appropriately.

Clearly it’s not just mutable fields that make covariant types unsound. The problem is more general. It turns out that **as soon as a generic parameter type appears as the type of a method parameter, the containing class or trait may not be covariant in that type parameter**. For queues, the _enqueue_ method violates this condition:

```scala
class Queue[+T] {
    def enqueue(x: T) =
    ...
}
```
Running a modified queue class like the one above through a Scala compiler would yield:
```
Queues.scala:11: error: covariant type T occurs in
contravariant position in type T of value x
def enqueue(x: T) =
            ˆ
```

**Note:** Reassignable fields are a special case of the rule that disallows type parameters annotated with + from being used as method parameter types. A reassignable field, `var x: T`, is treated in Scala as a getter method, `def x: T`, and a setter method, `def x_=(y: T)`. As we can see, the setter method has a parameter of the field’s type `T`. So that type may not be covariant. For more information, Odersky's 18.2 chapter.

## Lower Bounds

Lower type bounds declare a type to be a supertype of another type. The term `U >: T` expresses that the type parameter `U` or the abstract type `U` is required to be a supertype of `T`. As an example, we could define `Queue` the next way:

```scala
class Queue[+T] (private val leading: List[T], private val trailing: List[T] ) {
    def enqueue[U >: T](x: U) = new Queue[U](leading, x :: trailing) 
    ...
}
```

The new definition of enqueue is arguably better than the old, because it is more general. Unlike the old version, the new definition allows you to append an arbitrary supertype U of the queue element type `T`. The result is then a `Queue[U]`.

#### Comparison to Java
This is the main reason that Scala prefers declaration-site variance over use-site variance as it is found in Java’s wildcards. With use-site variance, you are on your own designing a class. It will be the clients of the class that need to put in the wildcards, and if they get it wrong, some important instance methods will no longer be applicable. Variance being a tricky business, users usually get it wrong, and they come away thinking that wildcards and generics are overly complicated. With definition-side variance, you express your intent to the compiler, and the compiler will double check that the methods you want available will indeed be available.

## Upper Bounds

An upper type bound `U <: T` declares that type variable `U` refers to a subtype of type `T`.

## References

* Chapter 19: Martin Odersky's _Programming in Scala_.
