Scala
======
## Introduction
Scala is a pure object-oriented language in the sense that every value is an object. Types and behavior of objects are described by classes and traits. Scala is also a functional language in the sense that every function is a value. Scala provides a lightweight syntax for defining anonymous functions, it supports higher-order functions, it allows functions to be nested, and supports currying. Scala’s case classes and its built-in support for pattern matching model algebraic types used in many functional programming languages. Singleton objects provide a convenient way to group functions that aren’t members of a class.

Furthermore, Scala’s notion of pattern matching naturally extends to the processing of XML data with the help of right-ignoring sequence patterns, by way of general extension via extractor objects. In this context, sequence comprehensions are useful for formulating queries. These features make Scala ideal for developing applications like web services.

## Type System
Scala is equipped with an expressive type system that enforces statically that abstractions are used in a safe and coherent manner. In particular, the type system supports:

* Generic classes
* Variance annotations
* Upper and lower type bounds,
* Inner classes and abstract types as object members
* Compound types
* Explicitly typed self references
* Implicit parameters and conversions
* Polymorphic methods

A local type inference mechanism takes care that the user is not required to annotate the program with redundant type information. In combination, these features provide a powerful basis for the safe reuse of programming abstractions and for the type-safe extension of software.
