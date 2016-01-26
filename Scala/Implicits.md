Implicit Conversions, Parameters and Classes
======

Implicits are a powerful type system feature that allow us to extend other people's code in a more extensive way than ohter languages like C#, Ruby or Smalltalk.

## 1. Implicit conversions
The implicit conversion is just a normal method. The only thing that's special is the implicit modifier at the start. It would let us, for example, to extend the `String` class and make it a subtype of `RandomAccessSeq`.

```scala
implicit def stringWrapper(s: String) = 
    new RandomAccessSeq[Char] {
      def length = s.length
      def apply(i: Int) = s.charAt(i)
    }
```

By doing this, we would be extending its implementation, even if the designers of those libraries had not thought of making their classes extend `RandomAccessSeq`.

This aspect of implicits is similar to extension methods in C#, which also allow you to add new methods to existing classes. However, implicits can be far more concise than extension methods. For instance, we only needed to define the length and apply methods in the `stringWrapper` conversion, and we got all other methods in `RandomAccessSeq` for free.

Implicit conversions are governed by the following general rules:

* **Marking Rule:** Only definitions marked `implicit` are available. The `implicit` keyword is used to mark which declarations the compiler may use as implicits. We can use it to mark any variable, function, or object definition.
* **Scope Rule:** The Scala compiler will only consider implicit conversions that are in scope. To make an implicit conversion available, therefore, we must in some way bring it into scope. Moreover, with one exception, the implicit conversion must be in scope as a single _identifier_. If we want to make it available, therefore, we would need to import it, which would make it available as a single identifier. 
    *  There's one exception to the "single identifier" rule. The compiler will also look for implicit definitions in the companion object of the source or expected target types of the conversion.
*  **Non-Ambiguity Rule:** An implicit conversion is only inserted if there is no other possible conversion to insert. If the compiler has two options, then it will report an error and refuse to choose between them.
*  **One-at-a-time Rule:** Only one implicit is tried.
*  **Explicits-First Rule:** Whenever code type checks as it is written, no implicits are attempted. The compiler will not change code that already works.
*  **Naming an implicit conversion.** Implicit conversions can have arbitrary names. The name of an implicit conversion matters only in two situations: 
    *  If you want to write it explicitly in a method application.
    *  For determining which implicit conversions are available at any place in the program. For example, by using imports: `import MyConversions.stringWrapper`.
* **Where implicits are tried.** There are three places implicits are used in the language: conversions to an expected type, conversions of the receiver of a selection, and implicit parameters.

#### 1.1. Implicit conversion to an expected type
Implicit conversion to an expected type is the first place the compiler will use implicits. The rule is simple. Whenever the compiler sees an X, but needs a Y, it will look for an implicit function that converts X to Y. For example, normally a double cannot be used as an integer, because it loses precision:

```scala
scala> val i: Int = 3.5
  <console>:5: error: type mismatch;
   found   : Double(3.5)
   required: Int
         val i: Int = 3.5
                      ^
```
However, you can define an implicit conversion to smooth this over:

```scala
  scala> implicit def doubleToInt(x: Double) = x.toInt
  doubleToInt: (Double)Int
  
  scala> val i: Int = 3.5
  i: Int = 3
```

#### 1.2. Converting the receiver
Implicit conversions also apply to the receiver of a method call, the object on which the method is invoked. This kind of implicit conversion has two main uses. First, receiver conversions allow smoother integration of a new class into an existing class hierarchy. And second, they support writing domain-specific languages (DSLs) within the language.

To see how it works, suppose you write down a `Rational` class:
```scala
class Rational(n: Int, d: Int) {
    ...
    def + (that: Rational): Rational = ...
    def + (that: Int): Rational = ...
  }
```

`Rational` has two overloaded variants of the + method, which take Rationals and Ints, respectively, as arguments. So you can either add two rational numbers or a rational number and an integer. However, what about an expression like `1 + oneHalf`? This expression is tricky because the receiver, 1, does not have a suitable + method. So it would throw an error.

To allow this kind of mixed arithmetic, you need to define an implicit conversion from Int to Rational:
```scala
  scala> implicit def intToRational(x: Int) = new Rational(x, 1)
  intToRational: (Int)Rational
```

The other major use of implicit conversions is to simulate adding new syntax. Which in essence is the same as before, but it is used differently, like the `->` operator in Map:
```scala
Map(1 -> "one", 2 -> "two", 3 -> "three")
```

## 2. Implicit parameters

The remaining place the compiler inserts implicits is within argument lists. The compiler will sometimes replace someCall(a) with someCall(a)(b), or new SomeClass(a) with new SomeClass(a)(b), thereby adding a missing parameter list to complete a function call.

**_Not finished_**

## 3. Implicit classes
Classes annotated with the `implicit` keyword are refered to as implicit classes. An implicit class must have a primary constructor with exactly one argument in its first parameter list. It may also include an additional implicit parameter list. An implicit class must be defined in a scope where method definitions are allowed (not at the top level). An implicit class is desugared into a class and implicit method pairing, where the implciit method mimics the constructor of the class.

The generated implicit method will have the same name as the implicit class. This allows importing the implicit conversion using the name of the class, as one expects from other implicit definitions. For example, a definition of the form:

```scala
implicit class RichInt(n: Int) extends Ordered[Int] {
  def min(m: Int): Int = if (n <= m) n else m
  ...
}
```
will be transformed by the compiler as follows:

```scala
class RichInt(n: Int) extends Ordered[Int] {
  def min(m: Int): Int = if (n <= m) n else m
  ...
}
implicit final def RichInt(n: Int): RichInt = new RichInt(n)
```
