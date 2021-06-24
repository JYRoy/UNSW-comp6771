# Lecture 1-C++ Basic
[TOC]
## Basic Types
Types have defined storage requirements and behaviours.
```c++
// `int` for integers.
int meaning_of_life = 42;
    
// `double` for rational numbers.
double six_feet_in_metres = 1.8288;
    
// report if this expression is false
CHECK(six_feet_in_metres < meaning_of_life);

// `string` for text.
std::string course_code = std::string("COMP6771");
    
// `char` for single characters.
char letter = 'C';
    
CHECK(course_code.front() == letter);

// `bool` for truth
bool is_cxx = true;
bool is_danish = false;

CHECK(is_cxx != is_danish);
```
C++ runs directly on hardware, which means the value of some types may differ depending on the system.
```c++
#include <iostream>
#include <limits>

int main() {
  std::cout << std::numeric_limits::max() << "\n";
  std::cout << std::numeric_limits::min() << "\n";
}
```

## Auto
Auto keyword that allows the compiler to statically infer the type of a variable based on what is being assigned to it on the RHS(right hand side).
```c++
#include <iostream>
#include <limits>
#include <vector>

int main() {
  auto i = 0; // i is an int
  auto j = 8.5; // j is a double
  // auto k; // this would not work as there is no RHS to infer from
  std::vector<int> f;
  auto k = f; // k is std::vector<int>
  k.push_back(5);
  std::cout << k.size() << "\n";
}
```
## Const
The const keyword specifies that a value cannot be modified.
**Principle**: [Everything should be const unless you know it will be modified.](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#con1-by-default-make-objects-immutable) 
**Reasons**:
1. Clearer code (you can know a function won't try and modify something just by reading the signature).
2. Immutable objects are easier to reason about.
3. The compiler may be able to make certain optimisations.
4. Immutable objects are much easier to use in multithreading situations.
5. Prevents accidental or hard-to-notice change of value.

```c++
// `int` for integers.
auto const meaning_of_life = 42;
    
// `double` for rational numbers.
auto const six_feet_in_metres = 1.8288;

meaning_of_life++; // NOT ALLOWED - compile error!!!
    
// report if this expression is false
CHECK(six_feet_in_metres < meaning_of_life);
```

## Expressions
In computer science, an expression is a combination of values and functions that are interpreted by the compiler to produce a new value.
### Integral expressions
```c++
auto const x = 10;
auto const y = 173;

auto const sum = 183;
CHECK(x + y == sum);

auto const difference = 163;
CHECK(y - x == difference);
CHECK(x - y == -difference);

auto const product = 1730;
CHECK(x * y == product);

auto const quotient = 17;
CHECK(y / x == quotient);

auto const remainder = 3;
CHECK(y % x == remainder);
```
### Floating-point expressions
```c++
auto const x = 15.63;
auto const y = 1.23;

auto const sum = 16.86;
CHECK(x + y == sum);

auto const difference = 14.4;
CHECK(x - y == difference);
CHECK(y - x == -difference);

auto const product = 19.2249;
CHECK(x * y == product);

auto const expected = 12.7073170732;
auto const actual = x / y;
auto const acceptable_delta = 0.0000001;
CHECK(std::abs(expected - actual) < acceptable_delta);
```
### String expressions
```c++
auto const expr = std::string("Hello, expressions!");
auto const cxx = std::string("Hello, C++!");

CHECK(expr != cxx);
CHECK(expr.front() == cxx[0]);

auto const concat = absl::StrCat(expr, " ", cxx);
CHECK(concat == "Hello, expressions! Hello, C++!");

auto expr2 = expr;

// Abort TEST_CASE if expression is false
REQUIRE(expr == expr2);
```

### Boolean expressions
```c++
auto const is_comp6771 = true;
auto const is_about_cxx = true;
auto const is_about_german = false;

CHECK((is_comp6771 and is_about_cxx));
CHECK((is_about_german or is_about_cxx));
CHECK(not is_about_german);
```

## Type Conversion
In C++ we are able to convert types **implicitly** or **explicitly**. Recommend to use explicit conversion.
### Implicit promoting conversions
```c++
{
  auto d = 0.0;
  REQUIRE(d == 0.0);

  d = i; // Silent conversion from int to double
  CHECK(d == 42.0);
  CHECK(d != 41);
}
```
### Explicit promoting conversions
```c++
auto const i = 0;
{
  // Preferred over implicit, since your intention is clear
  auto const d = static_cast<double>(i);
  CHECK(d == 42.0);
  CHECK(d != 41);
}
```

### 转换运算符
|关键字|说明|
|---|---|
|static_cast|用于良性转换，一般不会导致意外发生，风险很低。|
|const_cast|用于 const 与非 const、volatile 与非 volatile 之间的转换。|
|reinterpret_cast|高度危险的转换，这种转换仅仅是对二进制位的重新解释，不会借助已有的转换规则对数据进行调整，但是可以实现最灵活的 C++ 类型转换.|
|dynamic_cast|借助 RTTI，用于类型安全的向下转型（Downcasting）|
语法格式：```xxx_cast<newType>(data)```
#### static_cast
- static_cast 是“静态转换”的意思，也就是在编译期间转换，转换失败的话会抛出一个编译错误。

- static_cast 只能用于良性转换，这样的转换风险较低，一般不会发生什么意外，例如：
    1. 原有的自动类型转换，例如 short 转 int、int 转 double、const 转非 const、向上转型等；
    2. void 指针和具体类型指针之间的转换，例如void *转int *、char *转void *等；  

- static_cast 不能用于无关类型之间的转换，因为这些转换都是有风险的，例如：
    1. 两个具体类型指针之间的转换，例如int *转double *、Student *转int *等。不同类型的数据存储格式不一样，长度也不一样，用 A 类型的指针指向 B 类型的数据后，会按照 A 类型的方式来处理数据：如果是读取操作，可能会得到一堆没有意义的值；如果是写入操作，可能会使 B 类型的数据遭到破坏，当再次以 B 类型的方式读取数据时会得到一堆没有意义的值。
    2. int 和指针之间的转换。将一个具体的地址赋值给指针变量是非常危险的，因为该地址上的内存可能没有分配，也可能没有读写权限，恰好是可用内存反而是小概率事件。

- 不能将 const/volatile 类型转换为非 const/volatile 类型。

#### 宽转换和窄转换
宽转换 Widening Conversion：表数范围小的数据类型转到表数范围大的数据类型，不会丢失信息。例如：int -> long
窄转换 Narrowing Conversion：表数范围大的数据类型转表数范围小的数据类型，可能会丢失信息。例如：int -> char; long -> int

## Values and references
A reference is an alias for another object: You can use it as you would the original object. A reference to const means you can't modify the object using the reference.
Similar to a pointer, but:
1. Don't need to use -> to access elements
2. Can't be null
3. You can't change what they refer to once set
  
Further Reading: [This page](https://isocpp.org/wiki/faq/references)
### 引用和取地址符
- 引用的格式：类型名  &  别名 = var;
    - 定义的时候必须初始化，即& 前面有类名或类型名，&别名后面一定带 “=” （在= 左边）
    - &后面的别名是新的名字，之前不存在。
 
- &取地址时：
    - 如果&是取址运算符，也就意味着取一个变量的地址并付给指针变量。&后面紧跟的是变量（已存在）

## Functions
### Function Types
Basing on the parameter list, the function types could be divided into three categories: nullary functions (no parameters), unary functions (one parameter) and binary functions (two parameters).
```c++
bool is_about_cxx() { // nullary functions (no parameters)
  return true;
}

int square(int const x) { // unary functions (one parameter)
  return x * x;
}

int area(int const width, int const length) { // binary functions (two parameters)
  return width * length;
}
```

### Function Syntax
You can use either, just make sure you're consistent.
First type:
```c++
#include <iostream>

auto main() -> int {
  // put "Hello world\n" to the character output
  std::cout << "Hello, world!\n";
}
```
Second type:
```c++
#include <iostream>

int main() {
  // put "Hello world\n" to the character output
  std::cout << "Hello, world!\n";
}
```

### Default Arguments
Recommend to read [this page](https://www.codingtag.com/actual-argument-and-formal-argument-in-c-plus-plus/) about formal and actual paramters in cpp.

Functions can use **default arguments**, which is used if an actual argument is not specified when a function is called. Default values are used for the trailing parameters of a function call - this means that ordering is important. 
**Formal parameters形参**: Those that appear in function definition. 形参则是你在写一个被调函数时，为了说明用到的自变量的类型、要进行什么操作而定义的，在调用函数前它不会被分配内存空间，更不会被赋予具体的值。
**Actual parameters实参(arguments)**: Those that appear when calling the function. 实参是程序中已经分配了内存空间的参数，它可以被赋予一个具体的值，比如常数、数组、地址（指针），也可以是一个变量名、数组名或表达式，当然也包括指针变量。
```c++
std::string rgb(short r = 0, short g = 0, short b = 0);
rgb();// rgb(0, 0, 0);
rgb(100);// Rgb(100, 0, 0);
rgb(100, 200);     // Rgb(100, 200, 0)
rgb(100, , 200);   // error
```

### Function Overloading
重载
Function overloading refers to a family of functions in the same scope that have the same name but different formal parameters.
```c++
auto square(int const x) -> int {
  return x * x;
}

auto square(double const x) -> double {
  return x * x;
}
```
#### Overload Resolution
This is the process of "function matching".
Step 1: Find candidate functions: Same name.
Step 2: Select viable ones: Same number arguments + each argument convertible.
Step 3: Find a best-match: Type much better in at least one argument.
Errors in function matching are found during compile time.
```c++
auto g() -> void;
auto f(int) -> void;
auto f(int, int) -> void;
auto f(double, double = 3.14) -> void;
f(5.6); // calls f(double, double)
```
### Passing Parameters in Functions
Recommend to read [this page](https://www.geeksforgeeks.org/parameter-passing-techniques-in-c-cpp/) about passing parameters in cpp.
#### Pass by value
The actual argument is copied into the memory being used to hold the formal parameters value during the function call/execution.
```c++
#include <iostream>

auto swap(int x, int y) -> void {
  auto const tmp = x;
  x = y;
  y = tmp;
}

auto main() -> int {
  auto i = 1;
  auto j = 2;
  std::cout << i << ' ' << j << '\n'; // prints 1 2
  swap(i, j);
  std::cout << i << ' ' << j << '\n'; // prints 1 2... not swapped?
}
```
Other examples:
```c++
auto by_value(std::string const sentence) -> char;
// takes ~153.67 ns
by_value(two_kb_string);

auto by_value(std::vector<std::string> const long_strings) -> char;
// takes ~2'920 ns
by_value(sixteen_two_kb_strings);
```
#### Pass by reference
The formal parameter merely acts as an alias for the actual parameter. Anytime the method/function uses the formal parameter (for reading or writing), it is actually using the actual parameter.
Pass by reference is useful when:
1. The argument has no copy operation
2. The argument is large.
```c++
#include <iostream>

auto swap(int& x, int& y) -> void {
  auto const tmp = x;
  x = y;
  y = tmp;
}

auto main() -> int {
  auto i = 1;
  auto j = 2;
  std::cout << i << ' ' << j << '\n'; // 1 2
  swap(i, j);
  std::cout << i << ' ' << j << '\n'; // 2 1
}
```
Other examples:
```c++
auto by_reference(std::string const& sentence) -> char;
// takes ~8.33 ns
by_reference(two_kb_string);
auto by_reference(std::vector<std::string> const& long_strings) -> char;
// takes ~13 ns
by_reference(sixteen_two_kb_strings);
```
## Condifitional Expressions
### If statement
Conditionally executes another statement.
Standard if statement:
```c++
auto collatz_point_if_statement(int const x) -> int {
  if (is_even(x)) {
    return x / 2;
  }

  return 3 * x + 1;
}
```
Short-hand condifitional expressions:
```c++
auto is_even(int const x) -> bool {
  return x % 2 == 0;
}

auto collatz_point_conditional(int const x) -> int {
  return is_even(x) ? x / 2
                    : 3 * x + 1;
}
```
### Switch statement
Transfers control to one of the several statements, depending on the value of a condition.
```c++
auto is_digit(char const c) -> bool {
	switch (c) {
	case '0': [[fallthrough]];
	case '1': [[fallthrough]];
	case '2': [[fallthrough]];
	case '3': [[fallthrough]];
	case '4': [[fallthrough]];
	case '5': [[fallthrough]];
	case '6': [[fallthrough]];
	case '7': [[fallthrough]];
	case '8': [[fallthrough]];
	case '9': return true;
	default: return false;
	}
}
```

## Declarations vs Definitions
Rcommend to read [this page](https://stackoverflow.com/questions/1410563/what-is-the-difference-between-a-definition-and-a-declaration).
术语“声明”和“定义”的区别，可以简单的归纳为：涉及内存分配的就是“定义”，否则就是“声明”。“声明”可以有多次，但“定义”只能有一次.
- **Declaration**: A declaration makes known the type and the name of a variable.
- **Definition**: A definition is a declaration, but also does extra things. Everything must have precisely one definition
    - A variable definition allocates storage for, and constructs a variable.
    - A class definition allows you to create variables of the class' type.
    - You can call functions with only a declaration, but must provide a definition later.
```c++
int i1; // Definition。涉及变量i1的内存分配
extern int i2; // Declaration。不会分配内存
 
void Func1(); // Declaration。不会分配内存
extern void Func2(); // Declaration。不会分配内存


void declared_fn(int arg);  // Declaration
class declared_type;  // Declaration

// This class is defined, but not all the methods are.
class defined_type {  // Definition
  int declared_member_fn(double);  // Declaration
  int defined_member_fn(int arg) { return arg; }  // Definition
};

// These are all defined.
int defined_fn() { return 1; }

int i;
int const j = 1;
auto vd = std::vector<double>{};
```

## Range-for-statements
```c++
auto all_computer_scientists(std::vector<std::string> const& names) -> bool {
	auto const famous_mathematician = std::string("Gauss");
	auto const famous_physicist = std::string("Newton");

	for (auto const& name : names) {
		if (name == famous_mathematician or name == famous_physicist) {
			return false;
		}
	}

	return true;
}

auto square_vs_cube() -> bool {
	// 0 and 1 are special cases, since they're actually equal.
	if (square(0) != cube(0) or square(1) != cube(1)) {
		return false;
	}

	for (auto i = 2; i < 100; ++i) {
		if (square(i) == cube(i)) {
			return false;
		}
	}

	return true;
}
```

## Enumerations
Enumerations are user-defined types. An enumeration is a distinct type whose value is restricted to a range of values
```c++
enum class computing_courses {
	intro,
	data_structures,
	engineering_design,
	compilers,
	cplusplus,
};

auto const computing101 = computing_courses::intro;
auto const computing102 = computing_courses::data_structures;
CHECK(computing101 != computing102);
```
```c++
enum Color { red, green, blue };
Color r = red;
switch(r)
{
    case red  : std::cout << "red\n";   break;
    case green: std::cout << "green\n"; break;
    case blue : std::cout << "blue\n";  break;
}
```

## Program Errors
There are 4 types of program errors that we will discuss
1. Compile-time
2. Link-time
3. Run-time
4. Logic
### Compile-time Errors
```c++
auto main() -> int {
  a = 5; // Compile-time error: type not specified
}
```
### Link-time Errors
```c++
#include <iostream>

auto is_cs6771() -> bool;

int main() {
	std::cout << is_cs6771() << "\n";
}
```

### Run-time Error
```c++
// attempting to open a file...
if (auto file = std::ifstream("hello.txt"); not file) {
    throw std::runtime_error("Error: file not found.\n");
}
```

### Logic(Programmer) Errors
```c++
auto const empty = std::string("");
CHECK(empty[0] == 'C'); // Logic error: bad character access
```

## File Input and Output
```c++
#include <iostream>
#include <fstream>

int main () {
  // Below line only works C++17
  std::ofstream fout{"data.out"};
  if (auto in = std::ifstream{"data.in"}; in) { // attempts to open file, checks it was opened
    for (auto i = 0; in >> i;) { // reads in
      std::cout << i << '\n';
      fout << i;
    }
    if (in.bad()) {
      std::cerr << "unrecoverable error (e.g. disk disconnected?)\n";
    } else if (not in.eof()) {
      std::cerr << "bad input: didn't read an int\n";
    }
  } // closes file automatically <-- no need to close manually!
  else {
    std::cerr << "unable to read data.in\n";
  }
  fout.close();
}
```

## Reference
1. [转换运算符](http://c.biancheng.net/cpp/biancheng/view/3297.html)
2. [Actual Argument and Formal Argument in C++](https://www.codingtag.com/actual-argument-and-formal-argument-in-c-plus-plus)
3. [What is the difference between a definition and a declaration?](https://stackoverflow.com/questions/1410563/what-is-the-difference-between-a-definition-and-a-declaration)
4. [Enumeration declaration](https://en.cppreference.com/w/cpp/language/enum)
5. [Reference](https://isocpp.org/wiki/faq/references)
