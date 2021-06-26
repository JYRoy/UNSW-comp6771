# Tutorial 3

## Question 1

How might we use a lambda function in the following example to sort vec by the number of vowels in each string? If the number of vowels is equal, then sort by the number of consonants.

```cpp
std::vector<std::string> vec{"We", "love", "lambda", "functions"};
```

Implement it in `tests/lambda_string.cpp`

### Answer
**题意**：
1. 根据元音字母的数目来对单词进行排序。如果元音字母的数目一样多就根据单词的长度来排序。
2. 使用lambda function
3. 使用sort

**思路**：
1. 统计每一个单词中vowels的个数（std::count_if(arg1, arg2, lambda function)）
2. 如果个数相等，按照长度从大到小排序（if statement）
3. 否则按照vowel个数从大到校排序

```c++
TEST_CASE("Checks sort3(int&, int&, int&)") {
  std::vector<std::string> vec{"We", "love", "lambda", "functions"};

  std::sort(vec.begin(), vec.end(), [] (std::string const& x, std::string const& y) {
    auto const xcount = std::count_if(x.cbegin(), x.cend(), [] (char const& c) { return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u'; });
    auto const ycount = std::count_if(y.cbegin(), y.cend(), [] (char const& c) { return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u'; });
    if (xcount == ycount) {
      return x.length() > y.length();
    }
    return xcount > ycount;
  });

  std::vector<std::string> correct_order{"functions", "lambda", "love", "We"};
  CHECK(correct_order == vec);
}
```

## Question 2

When writing a lambda function, when would you capture by value, and when would you capture by reference?

### Answer
1. Value: i don't want the result of the lambda function changing.
2. Reference: when i want the result of the lambda function changing with the variables, i will use reference.

## Question 3	

This task focuses on using standard algorithms to read a list of newline-seperated words from a file (try /usr/share/dict/words or /usr/dict/words) into a vector (hint: see std::istream_iterator).

### Question 3.1

In `source/dict.cpp`, write a function that takes in the word list as a stream, and outputs a vector of strings that are the words.

#### Answer
1. Input: std::istream
2. std::istream_iterator
3. return: std::vector\<std::string\>
```c++
std::vector<std::string> to_word_list(std::istream& input) {
  return {std::istream_iterator<std::string>{input}, std::istream_iterator<std::string>{}};
}
```

### Question 3.2

In `source/dict.cpp`, write a function that uses standard algorithms to split the string into words, filtered to only words that are in the word dict, and reconstruct this into a string (hint: see std::istringstream, std::istream_iterator, std::copy_if, std::ostringstream, and std::ostream_iterator)

#### Answer
1.  copy_if: 满足在dict中的条件就执行copy
2. copy from istream_iterator to ostream_iterators
3. find在字典中查找能不能找到：std::find(arg1, arg2, arg3) != valid_words.end() 表明可以找到

```c++
void print_valid_words(const std::vector<std::string>& valid_words,
                     std::istream& input,
                     std::ostream& output) {
  std::copy_if(
      std::istream_iterator<std::string>{input},
      std::istream_iterator<std::string>{},
      std::ostream_iterator<std::string>{output, " "},
      [&](const std::string& s) {
        return std::find(valid_words.begin(), valid_words.end(), s) != valid_words.end();
      });
}

```
#### Reference
- [copy_if](https://en.cppreference.com/w/cpp/algorithm/copy)
    - 参数：被复制obj的start, 被复制obj的end, 待复制obj的start， 规则
- [istream_iterator](https://en.cppreference.com/w/cpp/iterator/istream_iterator)
    - start: std::istream_iterator<std::string>{input}
    - end: std::istream_iterator<std::string>{}
- [ostream_iterator](https://en.cppreference.com/w/cpp/iterator/ostream_iterator)
    - std::ostream_iterator<std::string>{output, " "}
    - 参数：待输入的obj，分割符

### Question 3.3

Add your own tests to `test/dict.cpp`. This file has not been created, so you will have to create it yourself and add the appropriate line to `CMakeLists.txt`.

#### Answer
**题意**：
感觉这个题主要考察的是怎么利用std::istringstream, std::istream_iterator, std::copy_if, std::ostringstream, and std::ostream_iterator等来实现获得一个字符串中有效单词的功能，即对上面的function进行测试。

**思路**：
上一问中已经实现了这个功能，这一问其实就是怎么能有效的利用这个function而已。

我们首先要做的就是确定我们的TEST_CASE要比较什么。
- TEST_CASE：判断filter之后是不是我们想要的单词，即str1==str2

接下来要做的就是确定我们的输入和输出：
- Input：std::vector<std::string>valid_words, std::string unfiltered_words
- Output：std::string filtered_words

因为在print_valid_words中我们的输入是std::istream和std::ostream，所以我们要先把我们的输入的std::string转换为std::istream类型。因为std::istream的构造函数是explicit istream (streambuf* sb)，使用streambuf作为输入，所以我们无法通过std::string直接构建一个istream对象。但是istringstream是继承自istream的，目的就是实现string到istream的操作。所以我们在这里使用istingstream。在print_valid_words函数中我们还需要一个ostreaml对象来存储filtered之后的words，我们这里直接使用ostringstream类型，它有一个str() member function返回string，这样就可以实现TEST_CASE中的str1==str2判断。

```c++
#include <istream>
#include <string>
#include <vector>
#include <sstream>

#include "catch2/catch.hpp"
#include "../source/dict.h"

auto get_valid_word(std::vector<std::string>& valid_words, std::string& unfiltered_words) -> std::string {
    std::istringstream in{unfiltered_words};
    std::ostringstream out;
    print_valid_words(valid_words, in, out);
    return out.str();
}

// testing is enough, and discuss the value of each different require that you add.s
TEST_CASE("Empty word list means nothing is valid") {
  REQUIRE(get_valid_words({}, "").empty());
  REQUIRE(get_valid_words({}, "hello").empty());
  REQUIRE(get_valid_words({}, "hello world").empty());
}

TEST_CASE("One word means only that word is valid") {
  REQUIRE(get_valid_words({"hello"}, "").empty());
  // TODO(tutorial): Add more tests here
  CHECK(get_valid_words({"hello"}, "hello world") == "hello ");
}

TEST_CASE("Multiple words means they are all valid") {
  REQUIRE(get_valid_words({"hello", "world"}, "").empty());
  // TODO(tutorial): Add more tests here
  CHECK(get_valid_words({"hello", "world"}, "hello world program") == "hello world ");
}
```

#### Reference
1. [ios](https://www.cplusplus.com/reference/ios/ios/)

### Question 3.4

Discuss why separating your functions you want to test is a good idea.

#### Answer
参考上面的思路部分，function可以进行模块功能的复用，如上TEST_CASE多了之后我们依然可以通过统一的函数实现我们的功能，减少代码的重复。

### Question 3.5

Assume now that the word list and strings are both very large. Discuss how we could make this code run much faster (hint: a different data structure may be required. Tutors, students should know the data type, but not what it is called in C++)

#### Answer
根据刚刚我们的实现来考虑，我们使用的是std::find来实现的vector上的查找，这是一个O(N)时间复杂度的操作。当数据集增大的时候这个操作必然非常耗时。从这个角度来考虑我们需要优化word list的数据结构，减少查找的时间复杂度，那么unordered_set就是一个可行的选择。

unordered_set实现的是hash查找，通常情况下的时间复杂度是O(1)，最差是O(N)的。


### Question 3.6

Discuss the effect the use of automatic type deduction (through the use of auto keyword, and by not having to declare types at all when calling functions) on the quantity of code you had to change, and the depth of the testing required.

#### Answer

## Question 4

Open `source/car.h` and `source/car.cpp`

### Question 4.1

Create a constructor for the car that takes in the manufacturer name (e.g. Toyota) and the number of seats. Ensure that your constructor uses a member initializer list and uniform initialisation. Why is it important to use a member initializer list? Why is uniform initialisation preferred since C++11?

#### Answer

```c++
car(std::string manufacturer, int num): manufacturer_(manufacturer), num_seats_(num) {
    manufacturer_ = manufacturer;
    num_seats = num;
}
```

### Question 4.2

Create a default constructor that delegates to the previous constructor using the values of "unknown" and 4

Create const member functions to get the manufacturer and number of seats. What does it mean for a class or function to be const correct?

#### Answer
In h file.
```c++
car(): car{"unknown", 4}

auto get_manufacturer() const -> const std::string&;
auto get_seats() const -> int;
```

In cpp file.
```c++
auto car::get_manufacturer() const -> const std::string& {
  return manufacturer_;
}

auto car::get_num_seats() const -> int {
  return num_seats_;
}
```

### Question 4.3

Create a static data member to keep count of the number of car objects created. Modify your constructors to ensure that the count increases when a new object is created. Do you need to increase the object count in your delegating constructor?

#### Answer
```c++
private:
    static int num_cars;
public:
  car(std::string manufacturer, int n_seats): manufacturer_{ manufacturer}, num_seats_{n_seats} {
    ++n_objects_;
  }
```

### Question 4.4

Ensure that your static object count is initialised to 0, where should you do this, in the header file or the cpp file?

#### Answer
In cpp file.
```c++
int car::n_objects_ = 0;
```

### Question 4.5

Create a static function to return the object count. What does it mean for an function or data member to be static? Is the static data member part of the object or the class?

#### Answer
In h file.
```c++
static int get_num_cars();
```
In cpp fle.
```c++
auto car::get_num_cars() -> int {
  return n_objects_;
}
```

### Question 4.6

Create a destructor that decreases the object count when an object is destroyed

#### Answer
In h file.
```c++
public:
    ~car() noexcept;
```

In cpp file.
```c++
car::~car() noexcept {
  --n_objects_;
}
```

### Question 4.7

Test your code extensively by adding tests to the test directory, and adjusting the `CMakeLists.txt` file accordingly.

Make sure you keep your code - this example will continue next week.

#### Answer
```c++
TEST_CASE("car Checks") {
    car one;
    car two{"Toyota", 5};
    CHECK(car::get_num_cars() == 2);

    car three{two};
    CHECK(car::get_num_cars() == 3);

    three.set_num_seats(6);
    CHECK(three.get_num_seats() == 6);
}
```

