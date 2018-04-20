---
layout	: page
title		: "perfect forwarding"
date       : 2018-03-23
author      : "pshung"
---

This article explains the perfect forwarding techniquie introduced in c++11.
Look at the example below.

```c++
class Foo {
public:
  Foo() { std::cout << "Foo ctor\n"; }
  Foo(const Foo &f) { std::cout << "Foo copy ctor\n"; }
  ~Foo() { std::cout << "Foo dtor\n"; }
};

void func(Foo f) {
  std::cout << "Foo\n";
}

Foo getFoo() {return Foo();}
int main(int argc, char *argv[]) {

  func(getFoo());
  return 0;
}
```
Compile with option -fno-elide-constructors to disable [copy-elision](https://en.wikipedia.org/wiki/Copy_elision) optimization, or you are not able to see the calling of copy constructor.
```sh
Foo ctor
Foo copy ctor
Foo dtor
Foo copy ctor
foo
Foo dtor
Foo dtor
```

Copy may introduce a significant overhead if Foo contains huge memory
resource. 
Thus, move semantic is introduced to solve this in c++11.

```c++
class Foo {
public:
  Foo() { std::cout << "Foo ctor\n"; }
  Foo(const Foo &f) { std::cout << "Foo copy ctor\n"; }
  Foo(Foo &&f) { std::cout << "Foo move ctor\n"; }
  ~Foo() { std::cout << "Foo dtor\n"; }
};

void func(Foo f) {
  std::cout << "Foo\n";
}

Foo getFoo() {return Foo();}
int main(int argc, char *argv[]) {

  func(getFoo());
  return 0;
}
```
```sh
Foo ctor
Foo move ctor
Foo dtor
Foo move ctor
foo
Foo dtor
Foo dtor
```
If a class has a move constructor, the temporary variable of rvalue type will be called instead of the copy constructor.

Now, let's consider an usual case in template function, it forwards its
template parameter to another function and the type of the parameter donesn't
have move constructor.

```c++
class Foo {
public:
  Foo() { std::cout << "Foo ctor\n"; }
  Foo(const Foo &f) { std::cout << "Foo copy ctor\n"; }
  ~Foo() { std::cout << "Foo dtor\n"; }
};

void func(Foo f) {
  std::cout << "Foo\n";
}

template <typename T> void funcTemp(T arg) { func(arg); }
Foo getFoo() {return Foo();}
int main(int argc, char *argv[]) {

  func(getFoo());
  return 0;
}
```
```sh
Foo ctor
Foo copy ctor
Foo dtor
Foo copy ctor
foo
Foo dtor
Foo dtor
```sh

The problem is "Can we pass the parameter to the receiver function without overhead?".

Solution : use reference.
```
template <typename T> void funcTemp(T& arg) { func(arg); }
```
It's unable to bind a type of T& to rvalue.

```
template <typename T> void funcTemp(const T& arg) { func(arg); }
```
It limits the receive function to take a const parameter.
```
template <typename T> void funcTemp(T&& arg) { func(arg); }
```
It works well, but can we pass the parameter to the best receiver function?

```c++
class Foo {
public:
  Foo() { std::cout << "Foo ctor\n"; }
  Foo(const Foo &f) { std::cout << "Foo copy ctor\n"; }
  ~Foo() { std::cout << "Foo dtor\n"; }
};

void func(Foo& f) {
  std::cout << "Foo&&\n";
}
void func(Foo&& f) {
  std::cout << "Foo&&\n";
}

template <typename T> void funcTemp(T&& arg) { func(arg); }
Foo getFoo() {return Foo();}
int main(int argc, char *argv[]) {

  func(getFoo());
  return 0;
}
```

It always calls func(Foo&& f).

What we wanna do is if the input is a lvalue/rvalue, pass it to a function that takes
a parameter of lvalue/rvalue reference type.

```
template <typename T> void funcTemp(T&& arg) { func(std::move(arg)); }
```

This will always call the version of rvalue version.
```
template <typename T> void funcTemp(T&& arg) { func(std::forward<T>(arg)); }
```
std::forward keeps the original type.
If input is a rvalue, convert it from a lvalue to a rvalue, same as std::move.
If inptu is a lvalue, do nothing.










