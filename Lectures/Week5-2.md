# Week 5-Smart Pointer
[TOC]

## Basis

Why we need smart pointer?
- Managing unnamed / heap memory can be dangerous, as there is always the chance that the resource is not released / free'd properly. We need solutions to help with this.

## RAII - Making unnamed objects safe
This demonstration shows a way to release heap memory when the stack memory war released. But it's just a demo, don't use it in our own code.
```c++
// myintpointer.h

class MyIntPointer {
 public:
  // This is the constructor
  MyIntPointer(int* value);

  // This is the destructor
  ~MyIntPointer();

 int* GetValue();

 private:
  int* value_;
};
```
```c++
// myintpointer.cpp
#include "myintpointer.h"

MyIntPointer::MyIntPointer(int* value): value_{value} {}

int* MyIntPointer::GetValue() {
  return value_
}

MyIntPointer::~MyIntPointer() {
  // Similar to C's free function.
  delete value_;
}
```
```c++
void fn() {
  // Similar to C's malloc
  MyIntPointer p{new int{5}};
  // Copy the pointer;
  MyIntPointer q{p.GetValue()};
  // p and q are both now destructed.
  // What happens?
}
```
## Raw pointer
裸指针的使用通常会出现非常多的问题：
1. 裸指针在声明中没有指出涉及的是一个对象还是一个数组。然而在使用delete运算符的时候我们是需要明确知道要使用单个对象形式的delete还是数组形式的delete[]。如果用错会出发undefinited behavior。
2. 裸指针的声明中无法看出指针是否还拥有它涉及的对象。
3. 有时候难以判断我们需要的析构方式，是用delete还是把指针传入一个专门的析构函数
4. 难以保证在所有代码路径上只析构一次。如果少析构会导致资源泄漏，如果多执行一次会导致undefinited behavior。
5. 无法检测指针是否空悬（dangle）。即对象被析构了但是指针仍然涉及到它。

## Smart pointer
Ways of wrapping unnamed (i.e. raw pointer) heap objects in named stack objects so that object lifetimes can be managed much easier.

### Types
|Type|Shared ownership|Take ownership|
|---|---|---|
|std::unique_ptr<T>|No|Yes|
|raw pointers|No|No|
|std::shared_ptr<T>|Yes|Yes|
|std::weak_ptr<T>|No|No|

Usually two ways of approaching problems:
1. unique_ptr + raw pointers ("observers")
2. shared_ptr + weak_ptr/raw pointers

The ownership of resource is that an owner of a resource is the one who's responsible for correct cleanup of that resource.
- Take ownership: 

## Unique pointer
std::unique_pointer<T>
- The unique pointer owns the object
- When the unique pointer is destructed, the underlying object is too

raw pointer (observer)
- Unique Ptr may have many observers
- This is an appropriate use of raw pointers (or references) in C++
- Once the original pointer is destructed, you must ensure you don't access the raw pointers (no checks exist)
- These observers do not have ownership of the pointer
```c++
#include <memory>
#include <iostream>

int main() {
  auto up1 = std::unique_ptr<int>{new int};
  auto up2 = up1; // no copy constructor
  std::unique_ptr<int> up3;
  up3 = up2; // no copy assignment

  up3.reset(up1.release()); // OK
  auto up4 = std::move(up3); // OK
  std::cout << up4.get() << "\n";
  std::cout << *up4 << "\n";
  std::cout << *up1 << "\n";
}
```