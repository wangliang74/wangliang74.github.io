---
title: "vector"
date: 2026-10-10T10:00:00+08:00
draft: true
weight: 4
---

# 数据结构

std::vector 本质上是一个动态数组，元素在堆上分配内存，可以根据需要自动调整内存大小，支持快速的随机访问。

std::vector 通过三个指针实现：
```
template <class T, class Alloc = allocator<T>>
class vector {
protected:
    T* _M_start;          // 指向已分配内存空间的起始位置
    T* _M_finish;         // 指向当前已存储元素的下一个位置（即数据末尾）
    T* _M_end_of_storage; // 指向整个已分配内存空间的末尾（即容量末尾）
};
```

std::vector 也提供了接口访问这三个指针：
```
_M_start           = vec.data()
_M_finish          = vec.data() + vec.size()
_M_end_of_storage  = vec.data() + vec.capacity()
```

常用接口与指针的关系：
```
size()     = _M_finish - _M_start         // 当前元素个数
capacity() = _M_end_of_storage - _M_start // 最大容量
empty()    = _M_start == _M_finish        // 是否为空
front()    = *_M_start                    // 首个元素
back()     = *(_M_finish - 1)             // 末尾元素
```

执行 `push_back` 添加新元素时，会判断元素个数是否已满 `_M_finish != _M_end_of_storage`。如果没满在 `_M_finish` 位置上构造新对象，然后执行 `++_M_finish`。如果满了，触发扩容处理。

# 扩容处理
扩容时机：当你向 std::vector 添加新元素时，如果发现当前的元素个数 `size()` 等于当前的最大容量 `capacity()`，触发扩容机制。

扩容处理：
+ 计算新的容量（GCC/Clang编译器通常是旧容量的 2 倍）。
+ 按照新的容量，申请一块连续内存。
+ 把旧内存里的元素，通过拷贝/移动构造函数，全部复制/移动到新内存中。
+ 销毁旧内存里的对象，并释放老旧的内存空间。
+ 插入新元素。

# 扩容如何构造

将对象添加到 std::vector 中可能触发拷贝构造和移动构造。

std::vector 扩容时，编译器可能调用拷贝构造，也可能时移动构造。如果移动构造函数绝对安全，则调用移动构造，不然则调用拷贝构造。

std::vector 在扩容搬运元素时，完全取决于一个关键条件：该元素的移动构造函数是否被标记为 noexcept。

示例代码：
```
#include <iostream>
#include <vector>
#include <string>

class MyClass {
public:
    MyClass(int value) : a(value) {
        std::cout << "Constructor:         " << this << std::endl;
    }
    MyClass(const MyClass& other) : a(other.a) {
        std::cout << "Copy Constructor:    " << this << std::endl;
    }
    MyClass(MyClass&& other) noexcept : a(other.a) {
        std::cout << "Move Constructor:    " << this << std::endl;
    }
    ~MyClass() {
        std::cout << "Destructor:          " << this << std::endl;
    }
public:
    int a;
};

int main() {
    std::vector<MyClass> vec;

    std::cout << "==========================================" << std::endl;
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;

    std::cout << "==========================================" << std::endl;
    vec.push_back(MyClass(1));

    std::cout << "==========================================" << std::endl;
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;

    std::cout << "==========================================" << std::endl;
    MyClass obj(2);
    vec.push_back(obj);

    std::cout << "==========================================" << std::endl;
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;
    return 0;
}
```

运行结果：
```
==========================================
size():              0
capacity():          0
data():              0
==========================================
Constructor:         0x7ffed9477620
Move Constructor:    0x602000000010
Destructor:          0x7ffed9477620
==========================================
size():              1
capacity():          1
data():              0x602000000010
==========================================
Constructor:         0x7ffed9477630
Copy Constructor:    0x602000000034
Copy Constructor:    0x602000000030
Destructor:          0x602000000010
==========================================
size():              2
capacity():          2
data():              0x602000000030
Destructor:          0x7ffed9477630
Destructor:          0x602000000030
Destructor:          0x602000000034
```

如果去掉 `MyClass(MyClass&& other) noexcept : a(other.a)` 的 `noexcept`，运行结果：
```
==========================================
size():              0
capacity():          0
data():              0
==========================================
Constructor:         0x7ffe0ae11ba0
Move Constructor:    0x602000000010
Destructor:          0x7ffe0ae11ba0
==========================================
size():              1
capacity():          1
data():              0x602000000010
==========================================
Constructor:         0x7ffe0ae11bb0
Copy Constructor:    0x602000000034
Copy Constructor:    0x602000000030
Destructor:          0x602000000010
==========================================
size():              2
capacity():          2
data():              0x602000000030
Destructor:          0x7ffe0ae11bb0
Destructor:          0x602000000030
Destructor:          0x602000000034
```

# 容量接口

示例代码：
```
v.reserve(100);       // 预分配容量，避免多次扩容，性能优化
v.shrink_to_fit();    // 尝试把capacity压缩到size大小


int main() {
    std::vector<MyClass> vec;

    std::cout << "==========================================" << std::endl;
    vec.reserve(10);      // 预分配容量，避免多次扩容，性能优化
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;

    std::cout << "==========================================" << std::endl;
    vec.push_back(MyClass(1));
    vec.push_back(MyClass(2));
    vec.push_back(MyClass(3));
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;

    std::cout << "==========================================" << std::endl;
    vec.shrink_to_fit(); // 尝试把 capacity 压缩到 size 大小
    std::cout << "size():              " << vec.size() << std::endl;
    std::cout << "capacity():          " << vec.capacity() << std::endl;
    std::cout << "data():              " << vec.data() << std::endl;
    return 0;
}
```

运行结果：
```
1
2
1
3
```

# 访问接口

示例代码：
```
int main() {
    std::vector<MyClass> vec;
    vec.push_back(MyClass(1));
    vec.push_back(MyClass(2));
    vec.push_back(MyClass(3));

    std::cout << vec[0].a << std::endl;      // 不检查越界，速度快
    std::cout << vec.at(1).a << std::endl;   // 会检查越界，越界抛出异常
    std::cout << vec.front().a << std::endl; // 第一个元素
    std::cout << vec.back().a << std::endl;  // 最后一个元素
    return 0;
}
```

运行结果：
```
1
2
1
3
```

# 增删接口
```
v.push_back(6);       // 末尾追加，最常用
v.emplace_back(6);    // 原地构造（推荐，减少拷贝）
v.pop_back();         // 删除最后一个元素，无返回值
v.clear();            // 清空所有元素，容量不变
```

# 使用场景
不适合使用的场景：
（1）需要频繁在中间插入或删除元素，改用 std::list / std::deque。
（2）元素很大、频繁扩容拷贝，可以存指针 /unique_ptr。
（3）需要固定大小、栈上分配内存，用 std::array。

# 使用技巧
std::vector 使用的 3 个核心经验：
（1）优先使用 emplace_back：在向末尾添加对象时，emplace_back 可以省去拷贝/移动构造的开销。
（2）提前 reserve 内存：防止插入元素时发生动态扩容、拷贝数据，极大提升性能。
（3）安全访问用 at()：如果下标可能不准确，用 .at(i) 而不是 [i]，越界时会直接抛出异常。
