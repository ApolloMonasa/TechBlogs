---
title: "洽谈C++封装"
date: 2026-01-25T15:07:35+08:00
draft: false 
# draft: true 表示草稿，写完后记得改为 false 发布

author: "ApolloMonasa"
description: "" # 文章摘要，如果不写默认截取正文前70字

# -----------------------------------------------------------------------
# 分类与索引 (核心管理区)
# -----------------------------------------------------------------------
tags: []         # 标签：如 [Java, Docker, 踩坑]
categories: []   # 分类：如 [后端开发]
series: []       # 系列：如 [C++从入门到入土] (填了这个就会自动归档到系列页)
weight: 0        # 排序：数字越小越靠前 (仅在置顶或系列文章中有效)

# -----------------------------------------------------------------------
# 封面图设置 (Cover)
# -----------------------------------------------------------------------
cover:
    image: ""    # 图片路径：直接填图片名，如 "cover.jpg"
    caption: ""  # 图片底部的说明文字
    alt: ""      # 图片无法显示时的替代文本
    relative: true # 【关键】开启相对路径，完美支持 Page Bundles
    hidden: false  # false = 在文章详情页顶部显示这张大图

# -----------------------------------------------------------------------
# 特殊功能开关 (按需取消注释)
# -----------------------------------------------------------------------
# searchHidden: true  # 如果是 true，这篇文章就不会被搜到
# showToc: false      # 如果想单独隐藏这篇文章的目录，解开这行
# comments: false     # 如果想单独关闭这篇文章的评论，解开这行
---

封装主要分为两类，一类是对数据的封装，一类是对功能的封装，并且我们在C语言中就做过这些事情，并不陌生，比如函数封装功能，结构体封装数据。

## 类的基本封装
### 基本认识类和对象
> 说白了，类是一种自定义的类型，我们可以自定义其中的数据结构，还有这个类型可以进行的操作，即`类型 = 类型数据 + 类型操作`，而对象和类的关系类似于变量和数据类型的关系。

#### 成员属性和方法
说白了，成员属性就是上述等式中的**类型数据**，而成员方法就是**类型操作**，例如：
![类型数据和类型操作](image.png)
成员属性的定义和调用和C语言中结构体的部分一模一样，而具体的成员方法实现和C语言的函数编写基本一样，只不过在类内实现，请看接下来的代码演示：
```cpp

#include <iostream>
using namespace std;

class People {
public:
    string name;
    int age;

    void self_intro() {
        cout << "I'm " << name << ", " << age << " years old" << endl;
    }
};

int main() {
    People Jim("Jim", 14);
    Jim.self_intro();
    return 0;
}
```
输出
```
I'm Jim, 14 years old
```
细心的朋友可能发现了类的定义和结构体的不同除了新增属性方法之外，还有一个陌生的关键字`public`，这会在接下来的一节中介绍，这里读者只需要知道，没有这行代码，编译会报错`error: 'void People::self_intro()' is private within this context`。

### 访问控制权限
> C++中有四个用于访问控制的关键字，分别是`public`,`private`,`protected`, `friend`。
> 同时也将在这里挑明C++中结构体和类的唯一不同就是默认权限,前者是public，后者是private，其他特性完全一致，也就是说，其实C++中的结构体和C语言中的结构体是不一样的，也可以拥有自己的成员方法。

下面我们来解释一下四个关键字代表什么含义
- public: 公告访问权限，没有限制，被这个关键字作用的成员能被**外界随意访问**
- private: 私有访问权限，全面限制，被这个关键字作用的成员**外部不能访问**
- protected: 受保护的，只有**自己和子类可以访问**
- friend: 定义友元[函数/类]，让其可以**访问自身所有属性和方法**

但是这里读者可能对**子类**会比较陌生，这里先不管，等到继承一章讲到这个概念我们还会再拿出来谈。
接下来请看代码演示：
```cpp
#include <iostream>
using namespace std;

class People {
public:
    string name;
    int age;
private:
    void self_intro() {
        cout << "I'm " << name << ", " << age << " years old" << endl;
    }
    friend int main();

};

int main() {
    People Jim("Jim", 14);
    Jim.self_intro();
    Jim.age = 15;
    Jim.self_intro();
    return 0;
}
```
可以看到我们把`self_intro`改为私有之后，main函数依旧可以调用这个函数，原因是因为mian函数已经被声明为友元，但是如果main函数再委托其他函数来完成这个函数调用就不行，如下所示：
```cpp
#include <iostream>
using namespace std;

class People {
public:
    string name;
    int age;
private:
    void self_intro() {
        cout << "I'm " << name << ", " << age << " years old" << endl;
    }
    friend int main();

};

void helper(People& p){
    p.self_intro();
}

int main() {
    People Jim("Jim", 14);
    Jim.self_intro();
    Jim.age = 15;
    Jim.self_intro();
    helper(Jim);
    return 0;
}
```
```
error: 'void People::self_intro()' is private within this context
   17 |     p.self_intro();
```

> 在此就总结一下`friend`的用法：在类内先写**friend**关键字，后面接上**友元的声明**那部分。

### 对象的生命周期
> 我们在声明并使用一个对象时，一直忽略了一件事，它是怎么来的？总得先内存分配，并进行初始化（构造），然后才能使用吧？并且我们没有关注一件事情，用完对象之后我们是如何抛弃对象（析构）并释放内存的？这就是**构造**和**析构**。
![对象生命周期](image-1.png)

### 构造函数和析构函数
![构造函数和析构函数](image-2.png)
上图很好地概括了C++的主要的构造和析构函数，但是初学者难免感到一头雾水,接下来我们循序渐进的讲解每种构造和析构。

首先是**默认构造**，是由**编译器默认生成的和类同名的无返回值的公有方法**，生成条件是：**没有任何自定义构造**，使用方法如下：
```cpp
#include <iostream>
using namespace std;

class A { };

int main() {
    A a;
}
```
加深理解就是，无参构造函数就是默认构造函数，我们当然也可以人为显示定义这个无参构造函数，记得加上`public`，使用方式不变，但是可以自定义一些初始化规则，比如：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int property;
    A() {
        property = 0;
        cout << "set property to 0" << endl;
    }
};

int main() {
    A a;
}
```
```
// 输出
set property to 0
```

讲到这里我要补充两点，第一是值初始化，当类**没有任何自定义构造**时，用如下的初始化方式可以将成员属性全部赋值为0:
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int property;
};

int main() {
    A a{};
    cout << a.property;
}
```
须知正常情况下应该是随机的垃圾值，这里输出为`0`。

第二是**聚合初始化**（又叫**初始化列表**），是值初始化的带参数形式，可以按顺序给成员属性赋值:
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int property;
    string name;
};

int main() {
    A a{10, "Ten"};
    cout << a.property << " " << a.name;
}
```
```
10 Ten
```
讲到这里，有读者或许会问，为什么不能用()来显式调用默认构造函数呢，就像下面这样：
```
A a();
```
那我问你，你这是不是一个返回类型是A的一个名为a的函数？于是就会发现这有语法歧义，并且编译器通常会像我这样理解，故不可行，通常就采用{}来显式调用默认构造，有自定义默认构造时调用的就是自定义默认构造，没有则是之前说的值初始化。

---

接下来是**有参构造**，就是**和类同名的自定义的有参数列表的构造函数**，如：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int property;
    A(int x) {
        property = x;
    }    
};

int main() {
    A a(101);
    cout << a.property;
}
```
但是这个时候默认构造函数就坏了，需要注意，此时我们需要显式写出默认构造函数了，但好处是，这里因为有参数，使用小括号来调用构造函数就不会产生歧义。

---

接下来请读者看一段代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A();
    A(int x, int y) {
        x = x, y = y; // 这样能够成功赋值吗？
    }
private:
    int x, y;    
};

int main() {
    
}
```
我既然这样问，那大概率不行，这是因为类内要访问自身属性要用到**this指针**，这是一个只想对象自身的指针，一个对象唯有通过this指针才能访问到成员属性，所以正确的写法应该是这样：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A();
    A(int x, int y) {
        this->x = x, this->y = y;
    }
private:
    int x, y;    
};

int main() {
    
}
```
this指针我们之后还会再提，这里先有一个简单的认识。

---

此外还有一个小的知识点叫做**委托构造**，看代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() : A(100, 100) {};//这种在构造函数声明后面加冒号并调用其他构造函数的方式叫做委托构造
    A(int x, int y) {
        x = x, y = y;
    }
private:
    int x, y;    
};

int main() {
    
}
```
这种在构造函数后面的`:`的后面部分叫做`初始化列表`，除了用于此处的委托构造之外，还有其他写法，可以对成员属性进行初始化：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() : x(100), y(100) {};
    A(int x, int y) {
        x = x, y = y;
    }
private:
    int x, y;    
};

int main() {
    
}
```

值得一提，当调用默认构造时，是**先执行初始化列表**中的语句，**后执行其函数体**中的语句。
更深入一点，初始化列表的初始化顺序**并不是**根据初始化列表书写顺序来赋值的，而是按照类中的声明顺序来的。


为什么一定要使用初始化列表呢？其实最重要的考量是提高效率。


---

接下来讲`this指针`，它本质上是一个指针，指向当前对象，所以才能像上面那样访问成员属性，可以通过下面的代码输出结果得到验证：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() {
        cout << this << "| default constructor" << endl;
    };
};

int main() {
    A a, b;
    cout << "a: " << &a << endl;
    cout << "b: " << &b << endl;
}
```
```
0x5ec01ffe7f| default constructor
0x5ec01ffe7e| default constructor
a: 0x5ec01ffe7f
b: 0x5ec01ffe7e
```

---

下面讲，有参构造的一种特殊情况——**转换构造**，即**只有一个参数**的情况，这样的构造函数会在**将参数类型的变量赋值给一个对象**时**隐式**地转换为该类对象，请看代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A(int z) : x(z), y(z) {
        cout << this << " convert constuctor" << endl;
    };

    // 重载运算符的知识，看不懂可以先不管
    void operator=(const A &a) {
        this->x = a.x;
        this->y = a.y;
        cout << this << " operator=" << endl;
        return ;
    }
private:
    int x, y;
};

int main() {
    A a(3);
    a = 4;
}
```
```
0xa7087ff960 convert constuctor
0xa7087ff968 convert constuctor
0xa7087ff960 operator=
```
从输出结果看得出这里`4`被**转换构造**转换为了一个A类型的对象，然后再赋值给了a。


> 这个时候就不得不提一个关键字`explicit`，只要在构造函数前加上它，就可以ban掉这种隐式转换，这是C++11引入的新特性，目的是增强代码可读性和可维护性。当然，除了ban掉这种隐式转换，初始化列表隐式转换也可以ban，只需要在多参数初始化函数前加上这个关键字即可。



---

接下来介绍一组很重要的概念，**左值和右值**。

我们必须先理解另一组概念：**值和引用**，说白了，引用就是变量的别名，其必须在声明时完成绑定，其语法是这样的：
```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 123;
    cout << a << endl;
    int &b = a; // 左值引用
    b = 456;
    cout << a << endl;
    int &&c = 11; // 右值引用
    c = 12;
    cout << c << endl;
}
```
- 左值: 持续存在的，可以取地址的，可以出现在赋值语句左侧
- 右值: 字面量或匿名对象，不能出现在赋值语句左侧
- **非const左值引用不能绑定在右值上**
- **左值引用优先绑定到左值上**
- **右值引用优先绑定到右值上**
- 无论是左值引用还是右值引用本质都是引用，除了绑定时的优先级不同，其他都一样，所以根据引用的特性我们容易知道，引用，**无论是左值引用还是右值引用都是左值**。

接下来看一段代码和输出：
```cpp
#include <iostream>
using namespace std;

void tell(int &x) {
    cout << "is left value" << endl;
}
void tell(int &&x) {
    cout << "is right value" << endl;
}
int main() {
    tell(12 + 23);
    tell(123);      // 字面量
    tell(int());    // 匿名变量
    int a;
    tell(a);        // 左值
}
```
```
is right value
is right value
is right value
is left value
```
这里的两个tell函数不会冲突，这涉及到了C++的多态特性，和C语言很不一样，先不用理会，只需要知道，左值会触发第一个tell函数，而右值会触发第二个tell函数。经过实验，我们发现了常见的左值和右值。

当然也有几个不那么常规的判断，我们不在这里演示，交给读者自行思考和实验。
- `++a`
- `a++`

> 说到这里，最后补充一下`std::move(x)`和`std::forward<int&&>(x)`，这俩函数可以把左值转右值。

---

之前我们已经讲完了默认构造和有参构造，接下来我们讲一讲**拷贝构造**。

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() {
        cout << this << " default constructor" << endl;
    }
    A(const A& a) {
        this->x = a.x;
        this->y = a.y;
        cout << this << " copy constructor" << endl;
    }
    void operator=(const A& a) {
        this->x = a.x;
        this->y = a.y;
        cout << this << " copy operator=" << endl;
    }
private:
    int x, y;
};

int main() {
    A a, b = a;
    b = a; // 一旦进入使用阶段，就和构造函数无关了
}

```
上述代码中`b = a`所用到的就是拷贝构造。

> 那为什么拷贝构造一定要传入**const左值引用**呢？
>
> 首先必须是引用，这是**编译器强制要求**的，因为如果不是引用就会无限调用拷贝构造，但是这是个无限递归。
> 其次是const，这不是必须的，但是如果**被拷贝的是const类型的对象**，那么传入参数不加const就会报错，因为**const变量只能绑定到const类型的引用上面**。

---

接下来我们来聊聊**深拷贝和浅拷贝**这个问题。

### 属性和方法
### 总结

## 运算符重载
### 函数重载
### 类外运算符重载
### 类内运算符重载
### 函数对象、数组对象与指针对象
> 通过运算符重载诞生的三种对象，说白了xx对象就是外在表现像xx的对象。
### 总结运算符重载

## 附加内容：返回值优化
### RVO返回值优化
### NRVO命名返回值优化
### 优化前后效果对比
