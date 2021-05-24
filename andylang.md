# Andylang

## Version 0.1

This is the initial write-up of the specification of Andylang. There are a number of things that still need to be fleshed out, including some features of classes like static variables and methods; how iterables, ranges/slices, and matrixes work; how to do things like operator overloading or creating custom hashable objects; how imports work; whether global variables are supported; and (eventually) the definition of the standard library. Expect breaking changes as the language develops.

## Goals

Andylang is a general-purpose programming language which encourages programming in a style that is inspired by functional languages, but not fully functional. It is designed to have a clean, readable syntax heavily inspired by Python. It is a garbage-collected language that does not require memory allocation. Unlike Python, it is strongly-typed. It eliminates a number of features in Python that are powerful but dangerous and can result in code that feels unintuitive or "magical" -- for example, Andylang does not have multiple inheritance (except for interfaces). It also encourages patterns that can make code more maintainable, such as function arguments being keyword-only by default. Some of these features will make simple programs a little bit harder to write, but will pay off in the long run when building complicated systems that are maintained for many years.

## Line structure

Andylang's line structure is the same as Python's. Each line is a single statement, and semicolons are optional. Example:

```
a = 10
b = 11
```

Just like in Python, a statement can be spread across multiple lines using parentheses. Example:

```
a = (
  10 + 
  3
)
```

You can optionally use the `\` character as an alternative method to continue across multiple lines:

```
a = 10 \
  + 3
```

And you can use semicolons to combine multiple statements into one line:

```
a = 10; b = 11
```

Just like in Python, indentation is used to indicate code blocks. Example:

```
if True:
  x = 0
  while x < 10:
    x += 1
```

Andylang also includes the `pass` keyword which does nothing, and is useful when you need an empty block of code:

```
while True:
  pass  # This is an infinite loop that does nothing
```

## Comments

Andylang's comment syntax is the same as Python's. The `#` character defines the start of a comment, which continues until the next linebreak.

```
# This is a full line comment
a = 10  # This is a partial-line comment
```

## Intrinsic types

- `NoneType`: This is a type that has one value, `None`. It is the same as in Python; however, because the language is strongly-typed, you can't use `None` interchangeably with other types.
- `bool`: This is a type that is either `True` or `False`. This is the same as Python, except this isn't magically treated as an integer.
- `int`: This is an integer type of any length (same as Python3, except can't use interchangeably with float)
- `float`: This is a floating-point type implemented as a 64-bit double (same as Python, except can't use interchangeably with int)
- `str`: This is a sequence of unicode code points (same as Python3)
- `bytes`: This is a sequence of 8-bit characters (same as Python3)
- `list<P>`: This is a sequence of type `<P>` (similar to Python, except you must explicitly define the type it contains)
- `set<P>`: This is an unordered collection of type `<P>` (similar to Python, except you must explicitly define the type it contains)
- `dict<P, Q>`: This is a mapping of hashtable type `<P>` to values of type `<Q>`

You can define additional types by creating a `type`, `union`, `class`, or `enum` (more on these in later sections)

All types are immutable by default. For types that support it, you can make the type mutable by adding an exclamation point. For example, a `list` is equivalent to Python's concept of a tuple; whereas a `list!` is equivalent to Python's list. Similarly, a `set` is equivalent to Python's concept of a frozenset, whereas a `set!` is equivalent to Python's set.

A common pattern is to create a union of a type with `None`, to make a nullable type. As convenient syntax, you can do that by adding a question-mark. For example, `int?` can be either an integer or `None`. More on this when unions are discussed.

## Constants and variables

Assignment creates a constant which exist until the end of the code block. Example:

```
a = 10
```

You can create a variable by using the `mutable` keyword:

```
mutable a = 10
a = 11
```

The type of a variable is determined when it is declared and cannot be changed in later statements. This is an error:

```
mutable a = 10
a = 'a string'  # This is an error
```

The type is automatically determined based on the value after the `=`. However, you can optionally make the type explicit by using a cast:

```
mutable a = (int)10
```

You need to explicitly define the type when creating a union. Here's an example:

```
mutable a = (union<int, string>)10
a = 'a string'  # This is ok!
```

This also applies when using the shorthand for a union with `None`. Here's an example that shows the difference:

```
mutable a = 10
a = None  # This is an error

mutable a = (int?)10
a = None  # This is ok!
```

You also need to explicitly declare types if you want the type to be a base class or interface of the object being used in the initialization. More on that later.

## Arithmetic operators

In addition to assignment, Andylang supports the standard Python operators:

- `+` (addition)
- `-` (subtraction)
- `*` (multiplication)
- `/` (division): this is Python2 division, not Python3. `5 / 2` is `2`
- `%` (modulus)
- `**` (exponentation)

Andylang does not support bitwise operations; however, there are standard-language functions that implement these when needed.

You can combine any operator with assignment:

```
mutable a = 10
a += 1
a -= 1
a *= 2
a /= 2
a %= 2
a **= 3
```

## Boolean logic

Andylang supports the same logic operators as Python:
- `not`
- `and`
- `or`

And the same comparison operators:
- `==`  
- `!=`
- `>`
- `>=`
- `<`
- `<=`

## Control flow and loops

Andylang has `if/elif/else`, `while`, and `for` constructs that are nearly identical to Python. The only difference is that logic expressions for `if` and `while` need to be of type `bool` and are not explicitly casted. This example shows the difference:

```
# This works in Python but not in Andylang because mylist.length is an int, not a bool
if mylist.length:
  mylist.append(5)

# In Andylang, you must make the comparison explicit
if mylist.length > 0:
  mylist.append(5)
```

Otherwise, these constructs are identical to Python. The `break` and `continue` statements are the same as well. Example:

```
y = 0
for x in range(10):
  y += x
  if x > 5:
    break
```

Andylang does not have a `switch` statement. You can accomplish similar goals using `match` or `if/elif/else`.

## Functions

Functions are a type that can be "called" using the parenthesis operator. Here is an example of a function definition:

```
return_zero = function():
  return 0

x = return_zero()  # x is now 0
```

Single-statement functions can be written on a single line and can omit the `return` keyword:

```
return_zero = function(): 0

x = return_zero()  # x is now 0
```

Function arguments can either be passed via keyword, or positionally, but not both. The default is keyword, as these can encourage more readable code. This example shows the difference:

```
add = function(first, second):
  return first + second

# This does not work. first and second are keyword arguments
ten = add(4, 6)

# This does work
ten = add(first=4, second=6)

# This works too -- the order does not matter for keyword arguments
ten = add(second=6, first=4)
```

You can use the `pos` keyword to declare that an argument is positional:

```
add = function(pos first, pos second):
  return first + second

# This works now
ten = add(4, 6)

# But this no longer works.
ten = add(first=4, second=6)
```

Argument types and return types are automatically determined. Therefore, a function can accept multiple types as long as everything in the function consistently supports those types. For example, in the above `add` function, you could alternatively call it with strings, and it will still work:

```
helloworld = add('hello', 'world')
```

You can optionally choose to declare (and thus, restrict) argument types. For example:

```
add_integers = function(pos first: int, pos second: int):
  return first + second
```

You can also declare a return type, too, and the compiler will enforce this for you. Example:

```
add_integers = function(pos first: int, pos second: int) -> int:
  return first + second
```

If you have multiple `return` statements, the compiler will automatically determine the return type. If all statements use the same type, that will be the return type. Otherwise, the return type will be a union of all possible return types:

```
# Return type will be union<str, int>
return_five = function(use_string: bool):
  if use_string:
    return '5'
  else:
    return 5
```

If you don't return anything at the bottom of the function, and you don't raise an exception at the bottom of the function, then the return type will be `None`. Therefore, this function's return type will be `union<int, NoneType>`:

```
my_function = function(return_something: bool):
  if return_something:
    return 100
```

Function arguments are constant by default. If you want them to be variables, you must add the `mutable` keyword. Example:

```
my_function = function(input_value: int) -> int:
  input_value += 5  # This is invalid -- cannot overwrite input_value
  return input_value

my_function = function(mutable input_value: int) -> int:
  input_value += 5  # This works because input_value is mutable
  return input_value
```

Function arguments can have a default value.

```
my_function = function(input_value=0):
  return input_value + 5
```

The expression for the default value is re-evaluated every time it is called, which avoids a common Python pitfall. Example:

```
my_function = function(input_value=![]):
  input_value.append(5)
  return input_value

list1 = my_function()
list2 = my_function()
# In Python, list1 and list2 would be the same object and would have length 2.
# In Andylang, they are separate objects and have length 1.
```

As in Python, you can declare functions that accept a variable number of positional arguments, although they must be of a consistent type. This is done by declaring a single parameter with name starting with an asterisk. This parameter will become a `list` in your function of the type used as input. Here's an example:

```
add = function(*params):
  sum_value = 0  # This forces params to be of type list<int>, although we could make that explicit if desired
  for param in params:
    sum_value += param
  return sum_value

twenty_four = add(2, 8, 14)
```

You can also declare functions that accept a variable number of keyword arguments. You can do this by declaring a single parameter with name starting with two asterisks. This parameter will become a `dict` in your function with keys of type `str` and values of the types being passed in. Example:

```
add = function(**params)
  return params['first_value'] + params['second_value'] + params['third_value']

twenty_four = add(first_value=2, second_value=8, third_value=14)
```

## Enums

You can use the `enum` keyword to create a special type that has finite choices. Unlike enums in other languages, these choices do not correspond to actual values. Here's an example:

```
enum Color:
  RED
  GREEN
  BLUE


get_rgb = function(color):
  if color == Color.RED:
    return rgb(255, 0, 0)
  elif color == Color.GREEN:
    return rgb(0, 255, 0)
  elif color == Color.BLUE:
    return rgb(0, 0, 255)
  else:
    raise ValueError('Unknown color')
```

This pattern is especially useful when you want to keep track of a finite number of modes or states. Without enums, this is often simulated (poorly) using multiple booleans, but a single enum can be a much better way of doing it. Enums can pair nicely with the `match` statement (more on that later).

## Unions

You can use the `union` keyword to create a type that can be one of several possible types. Example:

```
x = (union<int, string>)'Hello world'  # Right now it's a string
x = 10  # Now it's an integer
x = 'Hello again'  # Back to a string
```

An important feature of unions is that you cannot read from a union until you have determined its type by using the `match` statement (more on the later). Therefore, this code does not work:

```
x = (union<int, string>)'Hello world'  # Right now it's a string
x = 10  # Now it's an integer
x = x + 5  # Can't read from 'x' to add 5 because x could be an integer or a string
```

However, this works, as long as `my_function()` accepts both integers and strings:

```
x = (union<int, string>)'Hello world'  # Right now it's a string
x = 10  # Now it's an integer
my_function(x)
```

A common pattern is to create a union of a type with `NoneType`. As syntatic sugar, you can put a `?` character after a type name to represent a union of that type with `NoneType`. For example, `int?` is a shortcut for `union<int, None>`.

## Custom types

You can use the `type` keyword to create a new type that is a clone of an existing type. This can be used to create a shorter name for a long type. Here is an example (the name isn't actually shorter here, but you get the idea):

```
type StringList from list!<str>

my_list = StringList()
my_list.append('1')
```

Because Andylang is strongly-typed, type conversions must be explicit, even when a type is a clone of another type. Example:

```
type IntClone from int

a = (int)10
b = (IntClone)20
c = a + b  # This is invalid
c = a + int(b)  # This works
```

Custom types are also useful when you intentionally want a circular reference within a type. Here's how you can define JSON within Andylang:

```
type Json from union<
  NoneType,
  bool,
  float,
  str,
  list<Json>,
  dict<str, Json>,
>
```

## Matching

The `match` keyword allows you to write code that acts differently depending on the value or type of an input expression. When using a type within a matcher, the variable will be treated as that type within the matcher. This is critical for working with `union` types, and also can be useful for `enum` types and some other situations. Here's an example:

```
do_something = function(input_value: union<int, str>):
  match input_value:
    case int:
      # Within this block, input_value is an int
      return input_value + 5
    case str:
      # Within this block, input_value is a str
      return input_value + 'hello'
```

You must either include cases for every type; or, include a `default` block which will be called if no type is matched.

For short blocks, you can condense them into one line and omit the `return` keyword. Here's an alternative version of the above function:

```
do_something = function(input_value: union<int, str>):
  new_value = match input_value:
    case int: input_value + 5
    case str: input_value + 'hello'

  return new_value
```

You can optionally include matchers for specific values. The matchers are evaluated in the order they are specified. Example:

```
do_something = function(input_value: union<int, str>):
  new_value = match input_value:
    case 5: 100
    case 2: 40
    case int: input_value + 5
    case str: input_value + 'hello'

  return new_value
```

## Classes

Classes in Andylang are conceptually more like C# classes than Python classes, although parts of the syntax still borrow from Python.

All class members must be explicitly declared in the class, including types. You can't just ad-hoc create and remove class members like Python allows. Example:

```
class Point3d:
  public x = float
  public y = float
  public z = float
```

You can optionally include default values when declaring the class members:

```
class Point3d:
  public x = float(1.0)
  public y = float(2.0)
  public z = float(-3.0)
```

Classes can contain a constructor:

```
class Line:
  private x1 = float
  private x2 = float
  private y1 = float
  private y2 = float

  constructor(x1, x2, y1, y2):
    self.x1 = x1
    self.x2 = x2
    self.y1 = y1
    self.y2 = y2
```

In all class functions, you can use the `self` keyword to refer to the current object. However, unlike in Python, `self` is not explicitly defined as a function parameter.

Everything inside a class is defined as `public`, `protected`, or `private`. These mirror how they are used in languages like C++. Anything declared `private` cannot be accessed outside the class and cannot be accessed from derived classes. Anything declared `protected` cannot be accessed outside the class, but can be accessed from derived classes. Anything declared `public` can be accessed anywhere.

You can also use the `final` keyword to create functions that cannot be overridden by a derived class.

This example demonstrates how to add a function to a class:

```
class Line:
  private x1 = float
  private x2 = float
  private y1 = float
  private y2 = float

  constructor(x1, x2, y1, y2):
    self.x1 = x1
    self.x2 = x2
    self.y1 = y1
    self.y2 = y2

  public length():
    return sqrt((self.x2 - self.x1) ** 2 + (self.y2 - self.y1) ** 2)
```

By default, all functions are immutable (except the `constructor`), and can only read from class data (but not modify it). If you want to make a function that mutates class data, declare it with the `mutable` keyword:

```
class Line:
  private x1 = float
  private x2 = float
  private y1 = float
  private y2 = float

  constructor(x1, x2, y1, y2):
    self.x1 = x1
    self.x2 = x2
    self.y1 = y1
    self.y2 = y2

  public mutable translate(x_shift, y_shift):
    self.x1 += x_shift
    self.x2 += x_shift
    self.y1 += y_shift
    self.y2 += y_shift
```

When you construct an object of a class, by default, the object is immutable. This means that you can only call immutable functions. You need to use an exclamation-point to construct a mutable object for which you can call mutable functions. Example:

```
immutable_line = Line(1.0, 1.0, 5.0, 5.0)
immutable_line.translate(2.5, 3.5)  # compiler error -- cannot call mutable function on Line

mutable_line = Line!(1.0, 1.0, 5.0, 5.0)
mutable_line.translate(2.5, 3.5)  # this works
```

## Class inheritance

Classes can either not have a base class, or can have one base class. Multiple base classes are not allowed (except for interfaces -- more on that soon). Use the `extends` keyword to demonstrate inheritance. Example:

```
class Point2d:
  protected x = float
  protected y = float
```

```
class Point3d extends Point2d:
  # x and y are inherited from Point2d
  protected z = float
```

You can cast an object to its base class, which will allow you to use it for code that only expects the base class and not the derived class. Example:

```
a = Point3d(1.0, 2.0, 3.0)
b = (Point2d)a
# b is still a Point3d, but can be passed into functions that accept Point2d
```

## Interfaces

You can define interfaces, which are a collection of functions that a class can choose to implement. Here's an example:

```
interface IIterable
  next()

class IterablePoints implements IIterable
  private points = list<Point3d>
  private position = 0

  public mutable next():
    if self.position >= self.points.length():
      raise StopIteration

    self.position += 1
    return self.points[self.position - 1]
```

Classes can implement as many interfaces as they want. Similar to inheritance, you can cast a class to any interface it implements.

## Generic classes

Using syntax similar to C++, you can create classes that can handle one or more types as inputs, and use those types in definitions:

```
class MyList<T>:
  public value = T
  public next = MyList<T>?
```

For classes with multiple generic types, seperate them with commas:

```
class Dict<KeyType, ValueType>:
  # ... definition goes here ...
```

## Exception handling

Exception handling is the same as in Python. There is a special class called `Exception`, and a number of built-in derivations of `Exception` such as `ValueError` and `TypeError`. You can create custom classes that derive from `Exception`.

Exceptions are raised with the `raise` keyword. When an exception is raised, execution of the current function immediately ends, and the function bubbles upward until a matching `try` block is found. If none is found, the program aborts.

You can use `except` blocks to define matchers. If the exception matches the class in the `except` block (or is derived from that class), it will be matched.

You can also include a `finally` block which will always be run, regardless of whether or not an exception is raised. This is useful for cleanup code.

## What features of Python are not included?

Andylang intentionally leaves out a lot of complexity from Python. Here are some examples of things that were intentionally ommitted:
- Assignment expressions
- Multiple inheritance -- use interfaces instead
- Decorators
- Class properties (`@property` in Python) -- use a function instead
- Class methods (`@classmethod` in Python)
- `locals()`
- `eval()`
- Runtime type info / inspection
