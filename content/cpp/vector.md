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
Constructor:         0x7ffc74f4b3f0
Move Constructor:    0x602000000010
Destructor:          0x7ffc74f4b3f0
==========================================
size():              1
capacity():          1
data():              0x602000000010
==========================================
Constructor:         0x7ffc74f4b400
Copy Constructor:    0x602000000034
Move Constructor:    0x602000000030    // 扩容时调用移动构造函数
Destructor:          0x602000000010
==========================================
size():              2
capacity():          2
data():              0x602000000030
Destructor:          0x7ffc74f4b400
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
Constructor:         0x7ffea43b53a0
Move Constructor:    0x602000000010
Destructor:          0x7ffea43b53a0
==========================================
size():              1
capacity():          1
data():              0x602000000010
==========================================
Constructor:         0x7ffea43b53b0
Copy Constructor:    0x602000000034
Copy Constructor:    0x602000000030    // 扩容时调用拷贝构造函数
Destructor:          0x602000000010
==========================================
size():              2
capacity():          2
data():              0x602000000030
Destructor:          0x7ffea43b53b0
Destructor:          0x602000000030
Destructor:          0x602000000034
```

# 容量接口

示例代码：
```
int main() {
    std::vector<MyClass> vec;

    vec.reserve(10); // 预分配容量，避免多次扩容，性能优化

    std::cout << "==========================================" << std::endl;
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

    vec.shrink_to_fit(); // 尝试把 capacity 压缩到 size 大小，可能触发数据搬迁

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
capacity():          10
data():              0x604000000010
==========================================
size():              3
capacity():          10
data():              0x604000000010
==========================================
size():              3
capacity():          3
data():              0x602000000010    // shrink_to_fit() 触发数据搬迁
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

# 遍历接口

示例代码：
```

int main() {
    std::vector<MyClass> vec;
    vec.push_back(MyClass(1));
    vec.push_back(MyClass(2));
    vec.push_back(MyClass(3));

    std::cout << "==========================================" << std::endl;
    std::cout << "size():     " << vec.size() << std::endl;
    std::cout << "capacity(): " << vec.capacity() << std::endl;
    std::cout << "data():     " << vec.data() << std::endl;

    // 方式一：下标遍历
    for (size_t i = 0; i < vec.size(); ++i) {
        std::cout << &vec[i] << ": " << vec[i].a << ", ";
    }
    std::cout << std::endl;

    // 方式二：迭代器遍历
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << &*it << ": " << it->a << ", ";
    }
    std::cout << std::endl;

    // 方式三：Range-based for
    for (const auto& val : vec) {
        std::cout << &val << ": " << val.a << ", ";
    }
    std::cout << std::endl;
    return 0;
}
```

运行结果：
```
size():     3
capacity(): 4
data():     0x602000000050
0x602000000050: 1, 0x602000000054: 2, 0x602000000058: 3, 
0x602000000050: 1, 0x602000000054: 2, 0x602000000058: 3, 
0x602000000050: 1, 0x602000000054: 2, 0x602000000058: 3, 
```

注意：使用 Range-based for 时不同类型对应着不同的传参方式：
```
// 传值遍历，触发拷贝构造函数
for (auto val : vec)

// 传引用遍历，不发生拷贝，可以修改元素
for (auto& val : vec)

// 常量引用遍历，不发生拷贝，不能修改元素，首选
for (const auto& val : vec)
```

# 添加接口

使用 push_back 添加对象是可能触发拷贝构造或者移动构造，示例代码：
```
int main() {
    std::vector<MyClass> vec;
    vec.reserve(10);

    std::cout << "==========================================" << std::endl;
    MyClass obj(10);
    vec.push_back(obj);

    std::cout << "==========================================" << std::endl;
    vec.push_back(MyClass(20));
    return 0;
}
```

运行结果：
```
==========================================
Constructor:         0x7fffc437f530
Copy Constructor:    0x604000000010    // vec.push_back(obj) 触发拷贝构造
==========================================
Constructor:         0x7fffc437f540
Move Constructor:    0x604000000014    // vec.push_back(MyClass(20)) 触发移动构造
Destructor:          0x7fffc437f540
Destructor:          0x7fffc437f530
Destructor:          0x604000000010
Destructor:          0x604000000014
```

对象占用内存很大，可以保存智能指针，示例代码：
```
int main() {
    std::vector<std::unique_ptr<MyClass>> vec1;
    vec1.reserve(10);

    std::unique_ptr<MyClass> ptr = std::make_unique<MyClass>(10);
    vec1.push_back(std::move(ptr));
    std::cout << "ptr: " << ptr.get() << std::endl; // 输出 0
    return 0;
}
```

多态场景，可以保存智能指针，示例代码：
```
struct Base { virtual ~Base() = default; };
struct Derived : Base {};

int main() {
    std::vector<std::shared_ptr<Base>> vec;
    vec.push_back(std::make_shared<Derived>());
    return 0;
}
```

更加推荐使用的是 emplace_back, 传入构造参数，在 vector 的内存里原地构造对象，省去一次临时对象的拷贝 / 移动，性能更好。

示例代码：
```
int main() {
    std::vector<MyClass> vec;
    vec.reserve(10);

    std::cout << "==========================================" << std::endl;
    MyClass obj(10);
    vec.emplace_back(obj);

    std::cout << "==========================================" << std::endl;
    vec.emplace_back(20);
    return 0;
}
```

运行结果：
```
==========================================
Constructor:         0x7ffed44444e0
Copy Constructor:    0x604000000010
==========================================
Constructor:         0x604000000014
Destructor:          0x7ffed44444e0
Destructor:          0x604000000010
Destructor:          0x604000000014
```

# 删除接口

使用 `pop_back()` 删除最后一个元素，容量不变。使用前一定要判断`!empty()`，否则出现未定义行为，可能崩溃。
```
if (!v.empty()) {
    v.pop_back();
}
```

使用 `clear()` 清空所有元素，容量不变。
```
v.clear();
```

使用 `erase(it)` 删除某个元素或者某段元素，入参是迭代器，擦除非末尾元素时会触发数据搬迁。

示例代码：
```
int main() {
    std::vector<MyClass> vec;
    vec.reserve(10);
    for (unsigned int i = 0; i < 5; ++i) {
        vec.emplace_back(i);
    }

    for (unsigned int i = 0; i < vec.size(); ++i) {
        std::cout << &vec[i] << ": " << vec[i].a << ", ";
    }
    std::cout << std::endl;

    vec.erase(vec.begin() + 2);
    for (unsigned int i = 0; i < vec.size(); ++i) {
        std::cout << &vec[i] << ": " << vec[i].a << ", ";
    }
    std::cout << std::endl;

    vec.erase(vec.begin() + 1, vec.begin() + 3); // 前闭后开区间 [first, last)
    for (unsigned int i = 0; i < vec.size(); ++i) {
        std::cout << &vec[i] << ": " << vec[i].a << ", ";
    }
    std::cout << std::endl;
    return 0;
}
```

运行结果：
```
0x604000000010: 0, 0x604000000014: 1, 0x604000000018: 2, 0x60400000001c: 3, 0x604000000020: 4, 
0x604000000010: 0, 0x604000000014: 1, 0x604000000018: 3, 0x60400000001c: 4, 
0x604000000010: 0, 0x604000000014: 4, 
```

遍历 vector 时，删除某个元素，示例代码：
```
int main() {
    std::vector<MyClass> vec;
    vec.reserve(10);
    for (unsigned int i = 0; i < 5; ++i) {
        vec.emplace_back(i);
    }

    for (auto it = vec.begin(); it != vec.end(); ) {
        if (it->a % 2 == 0) {
            std::cout << "Delete: " << it->a << std::endl;
            it = vec.erase(it); 
        } else {
            ++it; 
        }
    }

    for (unsigned int i = 0; i < vec.size(); ++i) {
        std::cout << &vec[i] << ": " << vec[i].a << ", ";
    }
    std::cout << std::endl;
    return 0;
}
```

运行结果：
```
Delete: 0
Delete: 2
Delete: 4
0x604000000010: 1, 0x604000000014: 3, 
```

注意：erase() 自动返回下一个有效迭代器，不要再执行 `++it`。

更多的是配置使用 `erase` 和 `std::remove_if` 将要删除的元素移到末尾，然后再用 vector::erase 统一截断。
```
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    vector<int> vec = {1, 2, 3, 4, 5, 6};

    // 1. remove_if 把不要的元素扔到后面，返回新逻辑结尾的迭代器
    // 2. vec.erase 从这个逻辑结尾真正删除到物理结尾
    vec.erase(std::remove_if(vec.begin(), vec.end(), [](int val) {
        return val % 2 == 0;
    }), vec.end());

    for (int v : vec) cout << v << " "; // 输出: 1 3 5
    return 0;
}
```

如果大部分元素都要删掉，搬迁数据代价很大，可以使用 `swap` 直接构造新的 vector：
```
std::vector<int> tmp;
tmp.reserve(v.size());
for (auto &x : v) {
    if (!need_delete(x)) {
        tmp.push_back(x);
    }
}
v.swap(tmp);
```

# 使用场景
不适合使用的场景：
+ 需要频繁在中间插入或删除元素，改用 std::list 或者 std::deque。
+ 元素很大、频繁扩容拷贝，可以存裸指针或者unique_ptr。
+ 需要固定大小、栈上分配内存，用 std::array。

# 使用技巧
std::vector 使用的 3 个核心经验：
+ 优先使用 emplace_back：在向末尾添加对象时，emplace_back 可以省去拷贝/移动构造的开销。
+ 提前 reserve 内存：防止插入元素时发生动态扩容、拷贝数据，极大提升性能。
+ 安全访问用 at()：如果下标可能不准确，用 .at(i) 越界时会直接抛出异常。

