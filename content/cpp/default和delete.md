---
title: "default和delete"
date: 2026-10-10T10:00:00+08:00
draft: false
weight: 2
---

# 说明

C++编译器可能会默认生成一些特殊成员函数（构造、拷贝构造、移动构造、赋值运算符、移动赋值、析构函数），default和delete用于告诉编译器是否自动生成这些特殊成员函数：

+ default：告诉编译器“请帮我生成这个函数的默认版本”。
+ delete：告诉编译器“请禁用这个函数，谁调用就报错”。


# default

如果自定义了任何构造函数（包括带参数的构造函数），编译器就不会再自动生成无参的默认构造函数。

示例代码：
```
#include <iostream>

class MyClass {
public:
    MyClass(int value) : a(value) {}

public:
    int a;
};

int main() {
    MyClass obj1;
    return 0;
}
```

编译结果：
```
cpp1.cpp: In function ‘int main()’:
cpp1.cpp:12:13: error: no matching function for call to ‘MyClass::MyClass()’
```

语句 `MyClass() = default;` 强制编译器生成默认构造函数。

示例代码：
```
#include <iostream>

class MyClass {
public:
    MyClass() = default;
    MyClass(int value) : a(value) {}

public:
    int a;
};

int main() {
    MyClass obj1;
    return 0;
}
```

编译通过。

而且，编译器对 `MyClass() = default;` 比 `MyClass() {}` 的优化效果更好。

# delete

明确禁止调用某个函数。

如果在代码中试图调用被 = delete 修饰的函数，编译阶段就会直接报错。

示例代码：
```
#include <iostream>

class MyClass {
public:
    MyClass() : a(0) {}
    MyClass(const MyClass& other) = delete;
public:
    int a;
};

int main() {
    MyClass obj1;
    MyClass obj2(obj1);
    return 0;
}
```

编译结果：
```
cpp1.cpp: In function ‘int main()’:
cpp1.cpp:13:22: error: use of deleted function ‘MyClass::MyClass(const MyClass&)’
```

delete 最常用于禁用拷贝和赋值，以及阻止隐式类型转换。

示例代码：
```
#include <iostream>

class MyClass {
public:
    void printNum(int x) {
        std::cout << x << std::endl;
    }
};

int main() {
    MyClass obj1;
    obj1.printNum(10.1);
    return 0;
}
```

运行结果：
```
10
```

阻止隐式类型转换，禁止传入 double 类型参数：
```
#include <iostream>

class MyClass {
public:
    void printNum(int x) {
        std::cout << x << std::endl;
    }
    void printNum(double x) = delete; 
};

int main() {
    MyClass obj1;
    obj1.printNum(10.1);
    return 0;
}
```

编译失败：
```
cpp1.cpp: In function ‘int main()’:
cpp1.cpp:17:23: error: use of deleted function ‘void MyClass::printNum(double)’
```
