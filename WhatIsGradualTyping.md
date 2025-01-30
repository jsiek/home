# What is Gradual Typing

(For a Japanese translation, go here)

Gradual typing is a type system I developed with Walid Taha in 2006
that allows parts of a program to be dynamically typed and other parts
to be statically typed. The programmer controls which parts are which
by either leaving out type annotations or by adding them in. This
article gives a gentle introduction to gradual typing.

The following were our motivations for developing gradual typing:

* Large software systems are often developed in multiple languages
  partly because dynamically typed languages are better for some tasks
  and statically typed languages are better for others. With a gradual
  type system, the programmer can choose between static and dynamic
  typing without having to switch to a different language and without
  having to deal with the pain of language interoperability. Hopefully
  this will increase programmer productivity.
* Several languages already have optional type annotations, but
  surprisingly, there had been little formal work on what a type
  checker should do with the optional type annotations and what kind
  of guarantees the type system should provide. Languages with
  optional type annotations include Common LISP, Dylan, Cecil, Visual
  Basic.NET, Bigloo Scheme, Strongtalk. Gradual typing is meant to
  provide a foundation for what these languages do with their optional
  type annotations. There are several new languages in development
  that will also include optional type annotations such as Python 3k,
  the next version of Javascript (ECMAScript 4), and Perl 6. Hopefully
  our work on gradual typing will influence these languages.

Before describing gradual typing, let’s review dynamic and static type checking.
Dynamic type checking

A number of popular languages, especially scripting languages, are
dynamically typed. Examples include Perl, Python, Javascript, Ruby,
and PHP. In general, a type is something that describes a set of
values that have a bunch of operations in common. For example, the
type int describes the set of (usually 32 bit) numbers that support
operations like addition, subtraction, etc. A type error is the
application of an operation to a value of the wrong type. For example,
applying concatenation to an integer would be a type error in a
language where concatenation is an operation only on strings. Another
example of a type error is invoking a method on an object that doesn’t
implement the method, such as `car.fly()`. (Isn’t it a shame that flying
cars have not yet hit the mainstream, and it’s well past the year
2000!) The precise definition of type error is programming language
dependent. For example, one language might choose to allow
concatenation of integers and another language not. In a dynamically
typed language, type checking is performed during program execution,
immediately before the application of each operation, to make sure
that the operand type is suitable for the operation.

The following is an example Python program that results in a type error.

```
def add1(x):
    return x + 1

class A(object):
    pass

a = A()
add1(a)
```

The output from running the above program on the standard Python
interpreter is

```
TypeError: unsupported operand type(s) for +: 'A' and ‘int'
```

## Static type checking

There are also a number of popular statically checked languages, such as Java, C#, C and C++. In a statically checked language, some or even all type errors are caught by a type checker prior to running the program. The type checker is usually part of the compiler and is automatically run during compilation.

Here’s the above example adapted to Java.

```
class A {
    int add1(int x) {
      return x + 1;
    }
    public static void main(String args[]) {
      A a = new A();
      add1(a);
    }
}
```

When you compile this class, the Java compiler prints out
the following message.

```
A.java:9: add1(int) in A cannot be applied to (A)
        add1(a);
        ^
1 error
```

You may wonder, how can a type checker predict that a type error will
occur when a particular program is run? The answer is that it
can’t. It is impossible to build a type checker that can predict in
general which programs will result in type errors and which will
not. (This is equivalent to the well-known halting problem.) Instead,
all type checkers make a conservative approximation of what will
happen during execution and give error messages for anything that
might cause a type error. For example, the Java compiler rejects the
following program even though it would not actually result in a type
error.

```
class A {
    int add1(int x) {
    return x + 1;
    }
    public static void main(String args[]) {
    A a = new A();
        if (false)
        add1(a);
        else
            System.out.println("Hello World!");
    }
}
```

The Java type checker does not try to figure out which branch of an if
statement will be taken at runtime. Instead it conservatively assumes
that either branch could be taken and therefore checks both branches.

## Comparing dynamic and static type checking

There is a religious war between people who think dynamic checking is
better and people who think static type checking is better. I believe
that one of the reasons why this war has gone on for so long is that
both groups have good points. (They also have some not-so-good
points.) Unfortunately the two groups typically don’t acknowledge the
good points made by the other group as being good points. My
evaluation of the points, given below, will probably annoy both the
static typing fans and the dynamic typing fans. There are of course
arguments to be made for or against each of the points, and the
evaluation below shows where I land after considering the arguments.

* Static type checking catches bugs earlier, thereby removing the
  greater cost of fixing bugs later in the development cycle or the
  even greater cost of a bug that occurs in the field. Good point!
  Fans of dynamic typing will argue that you catch even more bugs by
  creating a thorough test suite for your programs. Nevertheless, I
  believe static type checking provides a convenient and low-cost way
  to catch type errors.
* Dynamic type checking doesn’t get in your way: you can immediately
  run your program without first having to change your program into a
  form that the type checker will accept.<br> Good point! Fans of
  static typing will argue that either 1) you don’t really need to
  change your program very much, or 2) by changing your program to fit
  the type checker, your program will become better structured. The
  reason why 1) feels true to some programmers is that the language
  you use changes how you think about programming and implicitly
  steers you towards writing programs that will type check in whatever
  language you are using. Also, you get so use to working around the
  minor annoyances of the type system that you forget that they are
  annoyances and instead become proud of your ability to workaround
  the type system. As for 2), there are situations in which the type
  system gets in the way of expressing code in its most clear and
  reusable form. The well-known Expression Problem is a good example
  of this. The reason why research on type systems continues to
  flourish is that it is difficult to design and implement a type
  system that is expressive enough to enable the straightforward
  expression of all programs that we would like to write.
* Static type checking enables faster execution because type checking
  need not be performed at runtime and because values can be stored in
  more efficient representations. Good point!
* Dynamic type checking makes it easy to deal with situations where
  the type of a value depends on runtime information. Good point!
* Static typing improves modularity. Good point! For example, in a
  dynamic language, you can call a library subroutine incorrectly but
  then get a type error deep inside that routine. Static checking
  catches the type errors up-front, at the point where you called the
  subroutine.
* Static type checking makes you think more seriously about your
  program which helps to further reduce bugs. Bad point. Type checkers
  only check fairly simple properties of your program. Most of the
  work in making sure that your program is correct, whether written in
  a statically or dynamically checked language, goes into developing
  comprehensive tests.
* With dynamic type checking, you don’t have to spend time writing
  type annotations. Bad point. The time it takes to write down a type
  annotation is rather trivial and there are programs called type
  inferencers that can do type checking without requiring type
  annotations.

Because neither static or dynamic type checking is universally better
than the other, it makes sense to provide the programmer a choice,
without forcing them to switch programming languages. This brings us
to gradual typing.

## Gradual type checking

A gradual type checker is a type checker that checks, at compile-time,
for type errors in some parts of a program, but not others, as
directed by which parts of the program have been annotated with
types. For example, our prototype gradual type checker for Python does
not give an error for the above program, reproduced again below.

```
def add1(x):
    return x + 1

class A:
    pass

a = A()
add1(a)
```

However, if the programmer adds a type annotation for the parameter x,
as follows, then the type checker signals an error because the type
of variable a is A, which is inconsistent with
the type of parameter x of the add1 function,
which is int.

```
def add1(x : int):
    return x + 1

class A:
    pass

a = A()
add1(a)
```

(Our rules for assigning static types to local variables such as a are
somewhat complicated because Python does not have local variable
declarations but in most cases we give the variable the same type as
the expression on the right-hand side of the assignment.)

The gradual type checker deals with unannotated variables by giving
them the unknown type (also called the dynamic type in the
literature), which we abbreviate as `?` and by allowing implicit
conversions from any type to `?` and also from `?` to any other type. For
simplicity, suppose the + operator expects its arguments to be
integers. The following version of add1 is accepted by the gradual
type checker because we allow an implicit conversion from `?` (the type
of x) to int (the type expected by +).

```
def add1(x):
    return x + 1
```

Allowing the implicit converson from `?` to int is unsafe, and is what
gives gradual typing the flavor of dynamic typing. Just as with
dynamic typing, the argument bound to x will be checked at run-time to
make sure it is an integer before the addition is performed.

As mentioned above, the gradual type checker also allows implicit
conversions from any type to type `?`. For example, the gradual type
checker accepts the following call to add1 because it allows the
implicit conversion from int (the type of 3) to `?` (the implied type
annotation for parameter x).

```
add1(3)
```

The gradual type checker also allows implicit conversions between more
complicated types. For example, in the following program we have a
conversion between different tuple types, from `? * int` to `int * int`.

```
def g(p : int * int):
  return p[0]

def f(x, y : int):
  p = (x,y)
  g(p)
```

In general, the gradual type checker allows an implicit conversion
between two types if they are consistent with each other. We use the
shorthand S ~ T to express that type S is consistent with type T. Here
are some of the rules that define when two types are consistent:

* For any type T, we have both `? ~ T` and `T ~ ?`.
* For any basic type B, such as int, we have `B ~ B`.
* A tuple type `T1 * T2 is` consistent with another tuple type `S1 * S2`
  if `T1 ~ S1` and `T2 ~ S2`. This rule generalizes in a straightforward
  way to tuples of arbitrary size.
* A function type `fun(T1,...,Tn,R)` (the `T1…Tn` are the parameter types
  and `R` is the return type) is consistent with another function type
  `fun(S1,...,Sn,U)` if `T1 ~ S1`…`Tn ~ Sn` and `R ~ U`.

We write `S !~ T` when S is not consistent with T.

So, for example

```
int ~ int
int !~ bool
? ~ int
bool ~ ?
int * int ~ ?
fun(?,?) ~ fun(int,int)
? ~ fun(int,int)
int * int !~ ? * bool
```

## Why subtyping alone does not work

Gradual typing allows an implicit cast from any type to `?`, similar to
object-oriented type systems where Object is the top of the subtype
lattice. However, gradual typing differs in that it also allows
implicit casts from `?` to any other type. This is the distinguishing
feature of gradual typing and is what gives it the flavor of dynamic
typing. Previous attempts at mixing static and dynamic typing, such as
Thatte’s Quasi-static Typing, tried to use subtyping but had to deal
with the following problem. If the dynamic type is treated as both the
top and the bottom of the subtype lattice (allowing both implicit
up-casts and down-casts), then the lattice collapses to one point
because subtyping is transitive. In other words, every type is a
subtype of every other type and the type system no longer rejects any
program, even ones with obvious type errors.

Consider the following program.

```
def add1(x : int) -> int:
   return x + 1
add1(true)
```

Using true as an argument to the function add1 is an obvious type
error but we have `bool <: ?` and `? <: int`, so `bool <: int`. Thus the
subtype-based type system would accept this program. Thatte partially
addressed this problem by adding a post-pass, called plausibility
checking, after the type checker but this still did not result in a
system that catches all type errors within fully annotated code, as
pointed out by Oliart. I won’t go into the details of Thatte’s
plausibility checking, as it is rather complicated, but I will discuss
an example. The following program has an obvious static type error
which is not detected by plausibility checking.

```
def inc(x: number):
   return x + 1

inc(True)
```

In the application of inc to True, both inc and True can be implicitly
up-cast to the dynamic type `?`. Then inc is implicitly down-cast to 
`? -> ?`. The plausibility checker looks for a greatest lower bound of
`number -> number` and `? -> ?`, which is `? -> number`, so it lets this
program pass without warning.

