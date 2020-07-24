---
layout: page
title: "Data and Type Classes"
permalink: /haskell2/
---


## Learning Outcomes

- Define data structures using Haskell's [Algebraic Data Types](/haskell2#algebraic-data-types) and use [pattern matching](/haskell2#pattern-matching) to define functions that handle each of the possible instances
- Use the alternate [record syntax](/haskell2#record-syntax) to define data structures with named fields
- Understand that Haskell [type classes](/haskell2#typeclasses) are similar to TypeScript interfaces in provide a definition for the set of functions that must be available for instances of those type classes and that typeclasses can extend upon one another to create rich hierarchies
- Understand that the [Maybe](/haskell2#maybe) typeclass provides an elegant way to handle *partial functions*

## Algebraic Data Types

We can declare custom types for data in Haskell using the data keyword.  Consider the following declaration of our familiar cons list:

```haskell
data ConsList = Nil | Cons Int ConsList 
```

The `|` operator looks rather like the union type operator in TypeScript, and indeed it serves a similar purpose.  Here, a `ConsList` is defined as being a composite type, composed of either Null or a Cons of an Int value and another `ConsList`.  This is called an “algebraic data type” because `|` is like an “or”, or algebraic “sum” operation for combining elements of the type while separating them with a space is akin to “and” or a “product” operation.  

Note that neither `Nil` or `Cons` are built in.  They are simply labels for constructor functions for the different versions of a `ConsList` node.  You could equally well call them `EndOfList` and `MakeList` or anything else that’s meaningful to you. `Nil` is a function with no parameters, `Cons` is a function with two parameters.  `Int` is a built-in primitive type for limited-precision integers.

Now we can create a small list like so:

```haskell
l = Cons 1 $ Cons 2 $ Cons 3 Nil  
```

## Pattern Matching

In Haskell, we can define multiple versions of a function to handle the instances of an algebraic data types.  This is done by providing a *pattern* in the variable list of the function definition, in the form of an expression beginning with the constructor of the data instance (e.g. `Cons` or `Nil`) and variable names which will be bound to the different fields of the data instance.  

For example, we can create a function to determine a `ConsList`’s length using *pattern matching*; to not only create different definitions of the function for each of the possible instances of a `ConsList`, but also to destructure the non-empty `Cons`:

```haskell
consLength :: ConsList -> Int
consLength Nil = 0
consLength (Cons _ rest) = 1 + consLength rest 
```

Since we don’t care about the head value in this function, we match it with `_`, an unnamed variable, which effectively ignores it.  Note that another way to conditionally destructure with pattern matching is using a [case statement](/haskell1/conditional-code-constructs-cheatsheet).

-----
 Note that such a definition for lists is made completely redundant by Haskell’s wonderful built-in lists, where [] is the empty list, and : is an infix cons operator.  We can pattern match the empty list or destructure (head:rest), e.g.:

```haskell
length :: [a] -> Int
length [] = 0
length (_:rest) = 1 + length rest 
```
-----

### Record Syntax
Consider the following simple record data type: 

```haskell
data Student = Student Int String Int 
```

A `Student` has three fields, mysteriously typed `Int`, `String` and `Int`.  Let’s say my intention in creating the above data type was to store a student’s id, name and mark.  I would create a record like so:

```haskell
> t = Student 123 "Tim" 95
```

Here’s how one would search for the student with the best mark:

```haskell
best :: [Student] -> Student -> Student
best [] b = b
best (a@(Student _ _ am):rest) b@(Student _ _ bm) =
  if am > bm
  then best rest a
  else best rest b 
```

The `@` notation, as in `b@(Student _ _ bm)` stores the record itself in the variable b but also allows you to unpack its elements, e.g. `bm` is bound to mark.

To get the data out of a record I would need to either destructure using pattern matching, as above, every time, or create some accessor functions:

```haskell
id (Student n _ _) = n
name (Student _ n _) = n
mark (Student _ _ n) = n
> name t
"Tim" 
```

It’s starting to look a bit like annoying boilerplate code.  Luckily, Haskell has another way to define such record types, called record syntax:

```haskell
data Student = Student { id::Integer, name::String, mark::Int } 
```

This creates a record type in every way the same as the above, but the accessor functions id, name and mark are created automatically.

## Typeclasses
Haskell uses “type classes” as a way to associate functions with types.  A type class is like a promise that a certain type will have specific operations and functions available.  You can think of it as being similar to a TypeScript interface.  Despite the name however, it is not like a TypeScript class, since a type class is does not actually define the functions themselves.  The functions are defined in “instances” of the type class.  A good starting point for gaining familiarity with type classes is seeing how they are used in the standard Haskell prelude.  From GHCi we can ask for information about a specific typeclass with the :i command, for example, Num is a typeclass common to numeric types:

```haskell
GHCi> :i Num
class Num a where
  (+) :: a -> a -> a
  (-) :: a -> a -> a
  (*) :: a -> a -> a
  negate :: a -> a
  abs :: a -> a
  signum :: a -> a
  fromInteger :: Integer -> a
  {-# MINIMAL (+), (*), abs, signum, fromInteger, (negate | (-)) #-}
        -- Defined in `GHC.Num'
instance Num Word -- Defined in `GHC.Num'
instance Num Integer -- Defined in `GHC.Num'
instance Num Int -- Defined in `GHC.Num'
instance Num Float -- Defined in `GHC.Float'
instance Num Double -- Defined in `GHC.Float' 
```

This is telling us that for a type to be an instance of the `Num` typeclass, it must provide the operators `+`, `*` and the functions `abs`, `signum` and `fromInteger`, and either `(-)` or `negate`.  The last is an option because a default definition exists for each in terms of the other.  The last five lines (beginning with “`instance`”) tell us which types have been declared as instances of `Num` and hence have definitions of the necessary functions.  These are `Word`, `Integer`, `Int`, `Float` and `Double`.  Obviously this is a much more finely grained set of types than JavaScript’s universal “`number`” type.  This granularity allows the type system to guard against improper use of numbers that might result in loss in precision or division by zero.

The main numeric type we will use in this course is `Int`, i.e. fixed-precision integers.

Note some obvious operations we would likely need to perform on numbers that are missing from the `Num` typeclass.  For example, equality checking.  This is defined in a separate type class `Eq`, that is also instanced by concrete numeric types like `Int`:
```haskell
> :i Eq
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  {-# MINIMAL (==) | (/=) #-}
...
instance Eq Int
... 
```

Note again that instances need implement only `==` or `/=` (not equal to), since each can be easily defined in terms of the other.  Still we are missing some obviously important operations, e.g., what about inequalities?  These are defined in the Ord type class:
```haskell
> :i Ord
class Eq a => Ord a where
  compare :: a -> a -> Ordering
  (<) :: a -> a -> Bool
  (<=) :: a -> a -> Bool
  (>) :: a -> a -> Bool
  (>=) :: a -> a -> Bool
  max :: a -> a -> a
  min :: a -> a -> a
  {-# MINIMAL compare | (<=) #-} 
```

```haskell
The compare function returns an Ordering:
> :i Ordering
data Ordering = LT | EQ | GT 
```

A custom data type can be made an instance of Ord by implementing either `compare` or `<=`.  The definition `Eq a => Ord a `means that anything is an instance of `Ord` must also be an instance of `Eq`.   Thus, typeclasses can build upon each other into rich hierarchies:

![Numeric Typeclasses](/haskell2/numerictypeclasses.png)

## Creating custom instances of type classes

If we have our own data types, how can we make standard operations like equality and inequality testing work with them?  Luckily, the most common type classes can easily be instanced automatically through the deriving keyword. example, if we want to define a `Suit` type for a card game.

```haskell
data Suit = Spade|Club|Diamond|Heart
 deriving (Eq,Ord,Enum,Show)

> Spade < Heart
True
```

The `Show` typeclass allows the data to be converted to strings with the `show` function (e.g. so that GHCi can display it).  The Enum typeclass allows enumeration, e.g.:

```haskell
> [Spade .. Heart]
[Spade,Club,Diamond,Heart]
```

We can also create custom instances of typeclasses by providing our own implementation of the necessary functions, e.g.:

```haskell
instance Show Suit where
 show Spade = "^"
 show Club = "&"
 show Diamond = "O"
 show Heart = "V"

> [Spade .. Heart]
[^,&,O,V] 
```

## Maybe
Another important built-in type is Maybe:

```haskell
> :i Maybe
data Maybe a = Nothing | Just a 
```

All the functions we have considered so far are assumed to be total.  That is, the function provides a mapping for every element in the input type to an element in the output type.  Maybe allows us to have a sensible return-type for partial functions.  Functions which do not have a mapping for every input:

![Total and Partial Functions](/haskell2/partialfunctions.png)

For example, the built-in lookup function can be used to search a list of key-value pairs, and fail gracefully by returning Nothing if there is no matching key.

```haskell
phonebook :: [(String, String)]
phonebook = [ ("Bob",   "01788 665242"), ("Fred",  "01624 556442"), ("Alice", "01889 985333") ]

> :t lookup
lookup :: Eq a => a -> [(a, b)] -> Maybe b

> lookup "Fred" phonebook
Just "01624 556442"

> lookup "Tim" phonebook
Nothing 
```

We can use pattern matching to extract values from Maybe (when we have Just a value), or to perform some sensible default behaviour when we have Nothing. 
```haskell
printNumber name = msg $ lookup name phonebook
where
   msg (Just number)  = print number
   msg Nothing        = print $ name ++ " not found in database"

*GHCi> printNumber "Fred"
"01624 556442"
*GHCi> printNumber "Tim"
"Tim not found in database" 
```