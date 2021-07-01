# Week 4-Exceptions
[TOC]

## Start Example
What does this produce?
If we input a number greater than 3, it will break and throw an out-of-range error.
```c++
#include <iostream>
#include <vector>

auto main() -> int {
  std::cout << "Enter -1 to quit\n";
  std::vector<int> items{97, 84, 72, 65};
  std::cout << "Enter an index: ";
  for (int print_index; std::cin >> print_index; ) {
    if (print_index == -1) break;
    std::cout << items.at(print_index) << '\n';
    std::cout << "Enter an index: ";
  }
}
```
What does this produce?
This will catch the errors and print what we want rather than crash the program. And then we could input next number.
```c++
#include <iostream>
#include <vector>

auto main() -> int {
  std::cout << "Enter -1 to quit\n";
  std::vector<int> items{97, 84, 72, 65};
  std::cout << "Enter an index: ";
  for (int print_index; std::cin >> print_index; ) {
    if (print_index == -1) break;
    try {
      std::cout << items.at(print_index) << '\n';
      items.resize(items.size() + 10);
    } catch (const std::out_of_range& e) {
      std::cout << "Index out of bounds\n";
    } catch (...) {
      std::cout << "Something else happened";
    }
    std::cout << "Enter an index: ";
  }
}
```

## Basic
- What?
    - Exceptions: Are for exceptional circumstances
        - Happen during run-time anomalies (things not going to plan A!).
    - Exception handling:
        - Run-time mechanism.
        - C++ detects a run-time error and raises an appropriate exception.
        - Another unrelated part of code catches the exception, handles it, and potentially rethrows it.
- Why?
    - Allows us to gracefully and programmatically deal with anomalies, as opposed to our program crashing.

## Exception Objects
Any type we derive from std::exception.
- throw std::out_of_range("Exception!");
- throw std::bad_alloc("Exception!");

Why std::exception? Why classes?
- #include \<exception\> for std::exception object
- #include \<stdexcept\> for objects that inherit std::exception

Recommend to read [this page](https://en.cppreference.com/w/cpp/error/exception) for more exceptions!

![exceptions](/Images/week4_2_exceptions.png)

stdexcept vs exception Headers in c++?

- **\<stdexcept\>**: Defines a set of standard exceptions that both the library and programs can use to report common errors.

- **\<exception\>**: Defines the base class (i.e., std::exception) for all exceptions thrown by the elements of the standard library, along with several types and utilities to assist handling exceptions.

So, \<exception\> only defines the class std::exception, while \<stdexcept\> defines several classes that inherit from std::exception (e.g., std::logic_error, std::out_of_range). That is why \<stdexcept\> includes \<exception\>.

They are in separate headers because if you want to define your own exception class inheriting std::exception (and not use the classes from \<stdexcept\>), you can avoid unnecessary definitions.

## Conceptual Structure

Exceptions are treated like lvalues.
Limited type conversions exist (pay attention to them):
- nonconst to const
- other conversions we will not cover in the course
```c++
try {
  // Code that may throw an exception
} catch (/* exception type */) {
  // Do something with the exception
} catch (...) { // any exception
  // Do something with the exception
}
```

## Rethrow
When an exception is caught, by default the catch will be the only part of the code to use/action the exception.
What if other catches (lower in the precedence order) want to do something with the thrown exception?
```c++
try {
  try {
    try {
      throw T{};
    } catch (T& e1) {
      std::cout << "Caught\n";
      throw;
    }
  } catch (T& e2) {
    std::cout << "Caught too!\n";
    throw;
  }
} catch (...) {
  std::cout << "Caught too!!\n";
}
```

## Multiple catch options
This does not mean multiple catches will happen, but rather that multiple options are possible for a single catch(one catch happens per try-catch block).

```c++
#include <iostream>
#include <vector> 

auto main() -> int {
  auto items = std::vector<int>{};
  try {
    items.resize(items.max_size() + 1);
  } catch (std::bad_alloc& e) {
    std::cout << "Out of bounds.\n";
  } catch (std::exception&) {
    std::cout << "General exception.\n";
  }
}
```

## Catching the right way
Throw by value, catch by const reference
Ways to catch exceptions:
- By value (no!)
- By pointer (no!)
- By reference (yes)

References are preferred because:
- more efficient, less copying (exploring today)
- no slicing problem (related to polymorphism, exploring later)

Recommend to read [this page](https://blog.knatten.org/2010/04/02/always-catch-exceptions-by-reference/).

The following example is copy-by-value, ```Giraffe g```it's actually copy constructor.
```c++
#include <iostream>

class Giraffe {
 public:
  Giraffe() { std::cout << "Giraffe constructed" << '\n'; }
  Giraffe(const Giraffe &g) { std::cout << "Giraffe copy-constructed" << '\n'; }
  ~Giraffe() { std::cout << "Giraffe destructed" << '\n'; }
};

void zebra() {
  throw Giraffe{};
}

void llama() {
  try {
    zebra();
  } catch (Giraffe g) {
    std::cout << "caught in llama; rethrow" << '\n';
    throw;  // throw the exception by caught.
  }
}

int main() {
  try {
    llama();
  } catch (Giraffe g) {
    std::cout << "caught in main" << '\n';
  }
}
```
The output is
```shell
Giraffe constructed
Giraffe copy-constructed
caught in llama; rethrow
Giraffe destructed
Giraffe copy-constructed
caught in main
Giraffe destructed
Giraffe destructed
```

The following example is catch-by-reference. It will not call copy constructor and less memory will be wasted.
```c++
#include <iostream>

class Giraffe {
 public:
  Giraffe() { std::cout << "Giraffe constructed" << '\n'; }
  Giraffe(const Giraffe &g) { std::cout << "Giraffe copy-constructed" << '\n'; }
  ~Giraffe() { std::cout << "Giraffe destructed" << '\n'; }
};

void zebra() {
  throw Giraffe{};
}

void llama() {
  try {
    zebra();
  } catch (const Giraffe& g) {
    std::cout << "caught in llama; rethrow" << '\n';
    throw;
  }
}

int main() {
  try {
    llama();
  } catch (const Giraffe& g) {
    std::cout << "caught in main" << '\n';
  }
}
```
The output is
```c++
Giraffe constructed
caught in llama; rethrow
caught in main
iraffe destructed
```

## Exception safety levels
This part is not specific to C++
Operations performed have various levels of safety
- No-throw (failure transparency)
- Strong exception safety (commit-or-rollback)
- Weak exception safety (no-leak)
- No exception safety

### No-throw guarantee
Recommend to read [this page](https://en.cppreference.com/w/cpp/language/exceptions).

Also known as failure transparency. 
Operations are guaranteed to succeed, even in exceptional circumstances. Exceptions may occur, but are handled internally. No exceptions are visible to the client. This is the same, for all intents and purposes, as noexcept in C++.
Examples:
- Closing a file
- Freeing memory
- Anything done in constructors or moves (usually)
- Creating a trivial object on the stack (made up of only ints)

### Strong exception safety
Also known as "commit or rollback" semantics.

Operations can fail, but failed operations are guaranteed to have no visible effects.

Probably the most common level of exception safety for types in C++. All your copy-constructors should generally follow these semantics.
Similar for copy-assignment:
- Copy-and-swap idiom (usually) follows these semantics (why?)
- Can be difficult when manually writing copy-assignment

To achieve strong exception safety, you need to:
- First perform any operations that may throw, but don't do anything irreversible
- Then perform any operations that are irreversible, but don't throw

### Weak exception safety
Also known as basic exception safety and no-leak guarantee.
Partial execution of failed operations can cause side effects, but:
- All invariants must be preserved
- No resources are leaked(No memory is leakes)

Any stored data will contain valid values, even if it was different now from before the exception
- Does this sound familiar? A "valid, but unspecified state"
- Move constructors that are not noexcept follow these semantics

### No exception safety
No guarantees
Don't write C++ with no exception safety, since
- Very hard to debug when things go wrong
- Very easy to fix - wrap your resources and attach lifetimes
    - This gives you basic exception safety for free

## noexcept specifier
Recommend to read [this page](https://en.cppreference.com/w/cpp/language/noexcept_spec).

Specifies whether a function could potentially throw. It doesn't not actually prevent a function from throwing an exception. STL functions can operate more efficiently on noexcept functions.
```c++
class S {
 public:
  int foo() const; // may throw
}

class S {
 public:
  int foo() const noexcept; // does not throw
}
```

## Testing exceptions
|Expression|Meaning|
|---|---|
|CHECK_NOTHROW(expr);|Checks expr doesn't throw an exception.|
|CHECK_THROWS(expr);|Checks expr throws an exception|
|CHECK_THROWS_AS(expr, type);|Checks expr throws type (or somthing derived from type).|
|namespace Matchers = Catch::Matchers; CHECK_THROWS_WITH(expr,Matchers::Message("message"));|Checks expr throws an exception with a message.|
|CHECK_THROWS_MATCHES(expr, type, Matchers::Message("message"));|CHECK_THROWS_AS and CHECK_THROWS_WITH in a single check.|

REQUIRES_THROWS* also available..

