---
title: "explicit"
date: 2026-10-10T10:00:00+08:00
draft: false
weight: 3
---

## 说明
explicit 作用于编译器的类型检查和重载决议阶段，禁止构造函数或转换函数发生隐式类型转换。

默认不加 explicit 情况下，发现代码中存在类型不匹配时，编译器默认会找到被赋值对象的构造函数，如果存在构造函数的参入类型和传入对象类型匹配，自动创建对象。

手动添加 explicit 后，等同于告诉编译器：“禁止自动调用构造函数创建对象，赋值时类型不同直接报错！”

在现代 C++ 开发中，“默认将构造函数和类型转换操作符声明为 explicit” 是一项公认的行业最佳实践。

## 不加 explicit
示例代码：
```
#include <iostream>

class MyClass {
public:
    MyClass(int value) : a(value) {
        std::cout << "Constructor: " << this << std::endl;
    }

public:
    int a;
};

void Show(MyClass obj) {
    std::cout << "a:           " << obj.a << std::endl;
}

int main() {
    int value = 10;
    Show(value);
    return 0;
}
```

运行结果：
```
Constructor: 0x7ffec6f11d20
a:           10
```

向函数 `Show` 传入 `int value`，编译器自动调用构造函数，创建 MyClass 对象。

## 添加 explicit
示例代码：
```
#include <iostream>

class MyClass {
public:
    explicit MyClass(int value) : a(value) {
        std::cout << "Constructor: " << this << std::endl;
    }

public:
    int a;
};

void Show(MyClass obj) {
    std::cout << "a:           " << obj.a << std::endl;
}

int main() {
    int value = 10;
    Show(value);
    return 0;
}
```

运行结果：
```
cpp.cpp: In function ‘int main()’:
cpp.cpp:19:10: error: could not convert ‘value’ from ‘int’ to ‘MyClass’
```
