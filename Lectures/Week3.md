# Lecture 3-Class
[TOC]

## Scope
### What is scope?
The scope of a variable is the part of the program where it is accessible. Scope starts at variable definition. Scope (usually) ends at next "}".  
The principle is that define variables as close to first usage as possible.

### Ways that we create scopes?
- Classes
- Namespaces
- Functions
- Global
- Random braces

### Object Lifetimes
An object is a piece of memory of a specific type that holds some data.
- All variables are objects.
- Unlike many other languages, this does not add overhead.

Object lifetime starts when it comes in scope.
- "Constructs" the object.
- Each type has 1 or more constructor that says how to construct it.

Object lifetime ends when it goes out of scope.
- "Destructs" the object.
- Each type has a different "destructor" which tells the compiler how to destroy it.

### Construction
Generally use () to call functions, and {} to construct objects.
- () can only be used for functions, and {} can be used for either.
- There are some rare occasions these are different. Sometimes it is ambiguous between a constructor and an initialize list
```c++
auto main() -> int {
  // Always use auto on the left for this course, but you may see this elsewhere.
  std::vector<int> v11; // Calls 0-argument constructor. Creates empty vector.
  
  // There's no difference between these:
  // T variable = T{arg1, arg2, ...}
  // T variable{arg1, arg2, ...}
  auto v12 = std::vector<int>{}; // No different to first
  auto v13 = std::vector<int>(); // No different to the first
  
  {
  	auto v2 = std::vector<int>{v11.begin(), v11.end()}; // A copy of v11.
  	auto v3 = std::vector<int>{v2}; // A copy of v2.
  } // v2 and v3 destructors are called here

  auto v41 = std::vector<int>{5, 2}; // Initialiser-list constructor {5, 2}
  auto v42 = std::vector<int>(5, 2); // Count + value constructor (5 * 2 => {2, 2, 2, 2, 2})
} // v11, v12, v13, v41, v42 destructors called here
```
Also works for your basic types. But the default constructor has to be manually called. This potential bug can be hard to detect due to how function stacks work (variable may happen to be 0). Can be especially problematic with pointers.
```c++
#include <iostream>

double f() {
	return 1.1;
}

int main() {
	// One of the reasons we do auto is to avoid ununitialized values.
	// int n; // Not initialized (memory contains previous value)

	int n21{}; // Default constructor (memory contains 0)
	auto n22 = int{}; // Default constructor (memory contains 0)
	auto n3{5};

	// Not obvious you know that f() is not an int, but the compiler lets it through.
	// int n43 = f();

	// Not obvious you know that f() is not an int, and the compiler won't let you (narrowing
	// conversion)
	// auto n41 = int{f()};

	// Good code. Clear you understand what you're doing.
	auto n42 = static_cast<int>(f());

	// std::cout << n << "\n";
	std::cout << n21 << "\n";
	std::cout << n22 << "\n";
	std::cout << n3 << "\n";
	std::cout << n42 << "\n";
}
```

### Namespaces
Used to express that names belong together. Prevent similar names from clashing.
```c++
// lexicon.hpp
namespace lexicon {
    std::vector<std::string> read_lexicon(std::string const& path);

    void write_lexicon(std::vector<std::string> const&, std::string const& path);
} // namespace lexicon

// word_ladder.hpp
namespace word_ladder {
    std::unordered_set<std::string> read_lexicon(std::string const& path);
} // namespace word_ladder
```

#### Nested namespaces
```c++
namespace comp6771::word_ladder {
    std::vector<std::vector<std::string>>
    word_ladder(std::string const& from, std::string const& to);
} // namespace comp6771::word_ladder

namespace comp6771 {
    // ...
    
    namespace word_ladder {
        std::vector<std::vector<std::string>>
        word_ladder(std::string const& from, std::string const& to);
    } // namespace word_ladder
} // namespace comp6771
```

#### Unnamed namespaces
In C you had static functions that made functions local to a file. C++ uses "unnamed" namespaces to achieve the same effect. Functions that you don't want in your public interface should be put into unnamed namespaces. Unlike named namespaces, it's okay to nest unnamed namespaces.
```c++
namespace word_ladder {
    namespace {
        bool valid_word(std::string const& word);
    } // namespace
} // namespace word_ladder
```

#### Namespace aliases
Gives a namespace a new name. Often good for shortening nested namespaces.
```c++
namespace chrono = std::chrono;
namespace views = ranges::views;
```
## OOP
OOP is object-oriented programming.
A class uses data abstraction and encapsulation to define an abstract data type:
- **Abstraction**: separation of interface from implementation
    - Useful as class implementation can change over time
- **Encapsulation**: enforcement of this via information hiding

This abstraction leads to two key parts of the abstract data type:
- **Interface**: the operations used by the user (an API)
- **Implementation**: the data members the bodies of the functions in the interface and any other functions not intended for general use

## C++ Classes
### Basis
A class:
- Defines a new type
- Is created using the keywords class or struct
- May define some members (functions, data)
- Contains zero or more public and private sections
- Is instantiated through a constructor

A member function:
- must be declared inside the class
- may be defined inside the class (it is then inline by default)
- may be declared const, when it doesn’t modify the data members
- The data members should be private, representing the state of an object.

### Member access control
```c++
class foo {
 public:
  // Members accessible by everyone
  foo(); // The default constructor.

 protected:
  // Members accessible by members, friends, and subclasses
  // Will discuss this when we do advanced OOP in future weeks.

 private:
  // Accessible only by members and friends
  void private_member_function();
  int private_data_member_;

 public:
  // May define multiple sections of the same name
};
```

### Constructor

Recommend to read [this page](https://en.cppreference.com/w/cpp/language/constructor) about class constructor.

#### Basis

Constructor is a special non-static member function of a class that is used to initialize objects of its class type. A constructor has the same name as the class and no return type. Default initalisation is handled through the default constructor. Unless we define our own constructors the compile will declare a default constructor. This is known as the synthesized default constructor.

```bash
for each data member in declaration order
  if it has an used defined initialiser
    Initialise it using the used defined initialiser
  else if it is of a built-in type (numeric, pointer, bool, char, etc.)
    do nothing (leave it as whatever was in memory before)
  else
    Initialise it using its default constructor
```
All of these things happen prior to your actual constructor body being called.

Constructors are declared using member function declarators of the following form:
```c++
class-name ( parameter-list(optional) ) except-spec(optional) attr(optional)	(1)	
```
Example:
```c++
#include <iostream>

class myclass {
public:
    myclass(int i) {
        i_ = i;
    }
    getval() {
        return i_;
    }

private:
    int i_;
};

int main() {
    auto mc = myclass{1};
    std::cout << myclass.getval() << "\n";
}
```

#### Constructor initialiser list
The initialisation phase occurs before the body of the constructor is executed, regardless of whether the initialiser list is supplied.

A constructor will:
1. Construct all data members in order of member declaration (using the same rules as those used to initialise variables)
2. Construct any undefined member variables that weren't defined in step (1)
3. Execute the body of constructor: the code may assign values to the data members to override the initial values.
```c++
#include <string>
#include <iostream>

class myclass {
public:
    myclass(int i, int j) : i_{i} { 
        j_=j;
    }
    int getval() {
        return i_ + j_;
    }

private:
	int i_;
    int j_
};

int main() {
    auto mc = myclass{5, 4};
    std::cout << mc.getval() << "\n";
}
```
So, if we go through the above code, we could find that i_{i} goes through all the initializer list, it populated the i data member to be the value of i that passed in. The second thing it does is it goes in default constructs if applicable all of the remaining data members that won't initialize a list. Then step three is executing the body, j_{j}.


#### The synthesized default constructor
In cpp, if we don't  create a constructor,  it will figure out what to do for you. It is actually like kind of hidden constructor.
- Is generated for a class only if it declares no constructors.
- For each member, calls the in-class initialiser if present. Otherwise calls the default constructor (except for trivial types like int).
- Cannot be generated when any data members are missing both in-class initialisers and default constructors.

#### Delegating constructors
Delegating constructors: A constructor may call another constructor inside the initialiser list.
```c++
#include <string>

class dummy {
public:
    explicit dummy(int const& i) : s_{"Hello world"}, val_{i} {
    }
    explicit dummy() : dummy(5) {
    }
    std::string const& get_s() {
        return s_;
    }
    int get_val() {
        return val_;
    }

private:
    std::string s_;
    const int val_;
};

auto main() -> int {
    dummy d1(5);
    dummy d2{};
}
```

### Destructors
Functions that are called when the object goes out of scope which also means that objects at the end of its lifetime.

Why might destructors be handy?
1. Freeing pointers
2. Closing files
3. Unlocking mutexes (from multithreading)
4. Aborting database transactions

Noexcept states no exception will be thrown(It will be included in the next lecture).
```c++
class MyClass {
    ~MyClass()
}
```

### Explicit keyword
If a constructor for a class has 1 parameter, the compiler will create an implicit type conversion from the parameter to the class. You have to opt-out of this implicit type conversion with the explicit keyword. 
```c++
#include <vector>

class intvec {
public:
	// This one allows the implicit conversion
	// intvec(std::vector<int>::size_type length)
	// : vec_(length, 0);

	// This one disallows it.
	explicit intvec(std::vector<int>::size_type length)
	: vec_(length, 0) {}

private:
	std::vector<int> vec_;
};

auto main() -> int {
	int const size = 20;
	// Explictly calling the constructor.
	intvec container1{size}; // Construction
	intvec container2 = intvec{size}; // Assignment

	// Implicit conversion.
	// Probably not what we want.
	// intvec container3 = size;
}
```

### This pointer

Recommend to read [this page](https://www.runoob.com/cplusplus/cpp-this-pointer.html) for this pointer.

A member function has an extra implicit parameter, named this.
- ​This is a pointer to the object on behalf of which the function is called.
- A member function does not explicitly define it, but may explicitly use it.
- The compiler treats an unqualified reference to a class member as being made through the this pointer.
- Generally we use a "_" suffix for class variables rather than a this-> to identify them
```c++
#include <iostream>

class myclass {
public:
    myclass(int i) {
        i_ = i;
    }
    int getval() {
        return i_;
    }

private:
    int i_;
};

int main() {
    auto mc = myclass{1};
    std::cout << mc.getval() << "\n";
}
```
```c++
#include <iostream>

class myclass {
public:
    myclass(int i) {
        this->i_ = i;
    }
    int getval() {
        return this->i_;
    }

private:
    int i_;
};

int main() {
    auto mc = myclass{1};
    std::cout << mc.getval() << "\n";
}
```

### Class Scope
Anything declared inside the class needs to be accessed through the scope of the class. Scopes are accessed using "::" in C++.
```c++
// foo.h

class Foo {
 public:
  // Equiv to typedef int Age
  using Age = int;

  Foo();
  Foo(std::istream& is);
  ~Foo();

  void member_function();
};
```
```c++
// foo.cpp
#include "foo.h"

Foo::Foo() {
}

Foo::Foo(std::istream& is) {
}

Foo::~Foo() {
}

void Foo::member_function() {
  Foo::Age age;
  // Also valid, since we are inside the Foo scope.
  Age age;
}
```

### Const objects

Recommend to read [this page](https://www.learncpp.com/cpp-tutorial/const-class-objects-and-member-functions/) for const member functions.

Member functions are by default only callable by non-const objects. But, we can declare a const member function which is valid on const objects and non-const objects.

A const member function is a member function that guarantees it will not modify the object or call any non-const member functions (as they may modify the object).
```c++
#include <iostream>
#include <string>

class person {
 public:
    person(std::string const& name) : name_{name} {}
    auto set_name(std::string const& name) -> void {  // non-const member function
    	name_ = name;
    }
    auto get_name() -> std::string const& {  // non-const member function
    	return name_;
    }

    //auto get_name() const -> std::string const& {  // const member     //function
    //	return name_;
    //}

 private:
    std::string name_;
};

auto main() -> int {
    person p1{"Hayden"};
    p1.set_name("Chris");
    std::cout << p1.get_name() << "\n";  // const member function can be called by non-const objects.
    
    person const p1{"Hayden"};
    p1.set_name("Chris"); // WILL NOT WORK... WHY NOT?
    std::cout << p1.get_name() << "\n"; // WILL NOT WORK... WHY NOT?
}
```
Because const objects can't call non-const member functions. Non-const objects can call non-const member functions and const member functions.


### Static members
Static members belong to the class (i.e. every object), as opposed to a particular object.

These are essentially globals defined inside the scope of the class
- Use static members when something is associated with a class, but not a particular instance(object)
- Static data has global lifetime (program start to program end)
```c++
// For use with a database
class user {
  public:
	user(std::string const& name) : name_{name} {}
	static auto valid_name(std::string const& name) -> bool {
		return name.length() < 20;
	}
  private:
	std::string name_;
}

auto main() -> int {
	auto n = std::string{"Santa Clause"};
	if (user::valid_name(n)) {
		user user1{n};
	}
}
```
Static member fields are usually defined outside of the class scope. The week3 tutorial includes it.

## Structs
A class and a struct in C++ are almost exactly the same. The only difference is that:
1. All members of a struct are public by default
2. All members of a class are private by default
3. People have all sorts of funny ideas about this. This is the only difference

We use structs only when we want a simple type with little or no methods and direct access to the data members (as a matter of style): 
1. This is a semantic difference, not a technical one
2. A std::pair or std::tuple may be what you want, though
```c++
class foo {
 int member_; // default private
};
```
```c++
struct foo {
 int member_; // default public
};
```

## Incomplete Types
Incomplete types are where you essientially use a type that the compiler does not yet know what it is. The example is
```c++
struct node {
	int data;
	// Node is incomplete - this is invalid
	// This would also make no sense. What is sizeof(Node)
	node next;
};
```
But the following is legal, since a class is considered declared once its class name has been seen:
```c++
struct node {
	int data;
	node* next;
};
```

The following types are incomplete types:
1. the type void (possibly cv-qualified);
2. incompletely-defined object types:
3. class type that has been declared (e.g. by forward declaration) but not defined;
4. array of unknown bound;
5. array of elements of incomplete type;
6. enumeration type from the point of declaration until its underlying type is determined.

All other types are complete.

Another examples:
An incompletely-defined object type can be completed: A class type (such as class X) might be incomplete at one point in a translation unit and complete later on; the type class X is the same type at both points:
```c++
struct X;             // X is an incomplete type
extern X* xp;         // xp is a pointer to an incomplete type
 
void foo() {
  xp++;               // ill-formed: X is incomplete
}
 
struct X { int i; };  // now X is a complete type
 
X x;
void bar() {
  xp = &x;            // OK: type is “pointer to X”
  xp++;               // OK: X is complete
}
```

## Deleting unused default member functions

```c++
#include <vector>

class intvec {
public:
	explicit intvec(std::vector<int>::size_type length)
	: vec_(length, 0) {}
private:
	std::vector<int> vec_;
};

auto main() -> int {
    auto a = std::vector<int>{4};
    auto b = intvec{a.size()};
}
```

```c++
#include <vector>

class intvec {
public:
	explicit intvec(std::vector<int>::size_type length)
	: vec_(length, 0) {}
private:
	std::vector<int> vec_;
};

auto main() -> int {
    auto a = std::vector<int>{4};
    auto a = intvec{}; // compiler error: redefinition of 'a'
}
```

```c++
#include <vector>

class intvec {
public:
	explicit intvec(std::vector<int>::size_type length)
	: vec_(length, 0) {}

	intvec() = default;  // force the compiler uses synthesized constructor.


private:
	std::vector<int> vec_;
};

auto main() -> int {
    auto c = intvec{};  // Pass
}
```

```c++
#include <vector>

class intvec {
public:
	// This one allows the implicit conversion
	explicit intvec(std::vector<int>::size_type length)
	: vec_(length, 0) {}

	intvec(intvec const& v) = delete;

private:
	std::vector<int> vec_;
};

auto main() -> int {
	auto a = std::vector<int>{4};
    auto c = intvec{a.size()};
    auto b = intvec{c};  // error: call to deleted constructor of 'intvec'
}
```

## References
1. [Classes](https://en.cppreference.com/w/cpp/language/classes)
2. [const member function](https://www.learncpp.com/cpp-tutorial/const-class-objects-and-member-functions/)
3. 
