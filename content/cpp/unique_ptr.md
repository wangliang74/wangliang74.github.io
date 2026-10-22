---
title: "unique_ptr"
date: 2026-10-12T14:30:00+08:00
draft: false
weight: 2
---

## 说明

unique_ptr 独占内存的所有权，只能移动，不能拷贝。

定义两个 unique_ptr 指针 ptr1 和 ptr2，ptr1指向内存地址A，当将内存地址A移动到ptr2后，ptr1就不能指向A，A只被ptr2独占。

它通过删除拷贝操作来保证“同一时刻只有一个所有者”，移动时只是把内存地址和删除器转移过去，并把源对象置空。

## 接口
```
// 创建
std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);

// 获取裸指针
ptr1.get()

// 获取引用
*ptr1

//  释放当前对象，指针置为空
ptr.reset();

// 交出所有权，指针置为空，不释放内存，返回地址
ptr.release();
```

## 自动释放

与裸指针相比, unique_ptr 离开作用域时，会自动释放其指向的内存。

示例代码：
```
#include <memory>
#include <iostream>

class MyClass {
public:
    explicit MyClass(int value) : a(value) {
        std::cout << "Constructor: " << this << std::endl;
    }
    ~MyClass() {
        std::cout << "Destructor:  " << this << std::endl;
    }
public:
    int a;
};

void Show() {
    std::unique_ptr<MyClass> ptr = std::make_unique<MyClass>(10);
    std::cout << "a:           " << ptr->a << std::endl;
}

int main() {
    std::cout << "Before Show()" << std::endl;
    Show();
    std::cout << "After  Show()" << std::endl;
    return 0;
}
```

运行结果：
```
Before Show()
Constructor: 0x602000000010
a:           10
Destructor:  0x602000000010
After  Show()
```

## 不支持拷贝构造和拷贝赋值

拷贝构造示例代码：
```
int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    std::unique_ptr<MyClass> ptr2 = ptr1;
    return 0;
}
```

编译结果：
```
cpp1.cpp: In function ‘int main()’:
cpp1.cpp:22:37: error: use of deleted function ‘std::unique_ptr<_Tp, _Dp>::unique_ptr(const std::unique_ptr<_Tp, _Dp>&) [with _Tp = MyClass; _Dp = std::default_delete<MyClass>]’
   22 |     std::unique_ptr<MyClass> ptr2 = ptr1;
```

拷贝赋值示例代码：
```
void Show(std::unique_ptr<MyClass> ptr) {
    std::cout << "ptr :        " << ptr.get() << std::endl;
    std::cout << "a:           " << ptr->a << std::endl;
}

int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    std::cout << "Before Show()" << std::endl;
    Show(ptr1);
    std::cout << "After  Show()" << std::endl;
    return 0;
}
```

编译结果：
```
cpp2.cpp: In function ‘int main()’:
cpp2.cpp:24:14: error: use of deleted function ‘std::unique_ptr<_Tp, _Dp>::unique_ptr(const std::unique_ptr<_Tp, _Dp>&) [with _Tp = MyClass; _Dp = std::default_delete<MyClass>]’
   24 |     Show(ptr1);
```

## 支持移动构造和移动赋值

示例代码：
```
#include <iostream>
#include <memory>

class MyClass {
public:
    explicit MyClass(int value) : a(value) {
        std::cout << "Constructor: " << this << std::endl;
    }
    ~MyClass() {
        std::cout << "Destructor:  " << this << std::endl;
    }
public:
    int a;
};

void Show() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    std::unique_ptr<MyClass> ptr2 = std::move(ptr1);
    std::cout << "ptr1:        " << ptr1.get() << std::endl;
    std::cout << "ptr2:        " << ptr2.get() << std::endl;
    std::cout << "a:           " << ptr2->a << std::endl;
}

int main() {
    std::cout << "Before Show()" << std::endl;
    Show();
    std::cout << "After  Show()" << std::endl;
    return 0;
}
```

运行结果：
```
Before Show()
Constructor: 0x602000000010
ptr1:        0
ptr2:        0x602000000010
a:           1
Destructor:  0x602000000010
After  Show()
```

执行 `ptr2 = std::move(ptr1);` 后，内存转移到 ptr2, ptr1 变成空指针。

如果 ptr2 本身指向内存，移动赋值操作会先释放其内存，保证内存安全。

示例代码：
```
void Show() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    std::unique_ptr<MyClass> ptr2 = std::make_unique<MyClass>(12);

    ptr2 = std::move(ptr1);
    std::cout << "ptr1:        " << ptr1.get() << std::endl;
    std::cout << "ptr2:        " << ptr2.get() << std::endl;
    std::cout << "a:           " << ptr2->a << std::endl;
}

int main() {
    std::cout << "Before Show()" << std::endl;
    Show();
    std::cout << "After  Show()" << std::endl;
    return 0;
}
```

运行结果：
```
Before Show()
Constructor: 0x602000000010
Constructor: 0x602000000030
Destructor:  0x602000000030
ptr1:        0
ptr2:        0x602000000010
a:           1
Destructor:  0x602000000010
After  Show()
```

## 作为入参

(1) 如果当前当前不再使用 ptr, 由子函数接管所有权、负责内存释放，使用unique_ptr作为参数类型。
示例代码：
```
void Show(std::unique_ptr<MyClass> ptr) {
    std::cout << "ptr :        " << ptr.get() << std::endl;
    std::cout << "a:           " << ptr->a << std::endl;
}

int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    Show(std::move(ptr1));
    std::cout << "ptr1:        " << ptr1.get() << std::endl;
    return 0;
}
```

运行结果：
```
Constructor: 0x602000000010
ptr :        0x602000000010
a:           1
Destructor:  0x602000000010
ptr1:        0
```

(2) 如果只想查看对象属性、调用方法，不涉及生命周期管理，使用const裸指针或const引用作为参数类型。
示例代码：
```
// 传递裸指针
void Show2(const MyClass *ptr) {
    std::cout << "ptr :        " << ptr << std::endl;
    std::cout << "a:           " << ptr->a << std::endl;
}

// 传递引用
void Show3(const MyClass& obj) {
    std::cout << "ptr :        " << &obj << std::endl;
    std::cout << "a:           " << obj.a << std::endl;
}

int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    Show2(ptr1.get());
    Show3(*ptr1);
    return 0;
}
```

(3) 不推荐使用 `void func(const std::unique_ptr<MyClass>& ptr)` 方式传递 unique_ptr, 直接使用传递裸指针，表示不涉及生命周期管理。

(4) 需要更改 unique_ptr 指向的内存地址时，可以用非const的unique_ptr引用 `void func(std::unique_ptr<MyClass>& ptr)`。
示例代码：
```
void Show(std::unique_ptr<MyClass>& ptr) {
    ptr.reset();
    std::cout << "ptr :         " << ptr.get() << std::endl;
}

int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(1);
    std::cout << "before, ptr1: " << ptr1.get() << std::endl;
    Show(ptr1);
    std::cout << "after , ptr1: " << ptr1.get() << std::endl;
    return 0;
}
```

运行结果：
```
Constructor:  0x602000000010
before, ptr1: 0x602000000010
Destructor:   0x602000000010
ptr :         0
after , ptr1: 0
```

## 作为返回值

(1) 转交所有权，在函数内部创建 unique_ptr, 按值返回。
```
std::unique_ptr<MyClass> Create(int value) {
    auto ptr = std::make_unique<MyClass>(value);

    std::cout << "ptr : " << ptr.get() << std::endl;

    return ptr;
}

int main() {
    std::unique_ptr<MyClass> ptr1 = Create(1);
    std::cout << "ptr1: " << ptr1.get() << std::endl;
    return 0;
}
```

(2) 不转交所有权，unique_ptr 的内存不是在函数内创建, 可以返回指针。
```
#include <iostream>
#include <memory>

class MyClass2 {};

class MyClass {
public:
    explicit MyClass() {
        a = std::make_unique<MyClass2>();
    }
    MyClass2* GetA() const {
        return a.get();
    }
public:
    std::unique_ptr<MyClass2> a;
};

int main() {
    MyClass obj;
    std::cout << "a: " << obj.GetA() << std::endl;
    return 0;
}
```

注意：

不要返回 unique_ptr& 或 unique_ptr*，可能导致 use-after-free 问题；
返回局部变量，不需要 std::move, 编译器会自动进程返回值优化；
返回返回全局、成员变量，需要 std::move 进行移动赋值。
