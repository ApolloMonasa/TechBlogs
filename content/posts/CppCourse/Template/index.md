---
title: '洽谈C++模板'
date: 2026-01-25T14:38:38+08:00
draft: false
# weight: 1
# aliases: ["/first"]
tags: []
author: "ApolloMonasa"
description: ""

# -----------------------------------------------------------------------
# 页面功能设置 (PaperMod)
# -----------------------------------------------------------------------
showToc: true
TocOpen: true      # 默认展开目录，方便查看
hidemeta: false    # 是否隐藏元数据(作者、时间等)
comments: true    # 是否开启评论
disableShare: false
disableHLJS: false # false=开启代码高亮
searchHidden: false # true=不显示在搜索结果中

# -----------------------------------------------------------------------
# 封面图设置 (Cover)
# -----------------------------------------------------------------------
cover:
    image: "" # 图片路径：建议使用 assets/image.png 或 /images/xxx.png
    alt: "<alt text>"
    caption: ""
    relative: true # 如果使用 Page Bundles (文件夹形式的文章)，改为 true
    hidden: false   # false=在文章内容页顶部显示封面

# -----------------------------------------------------------------------
# 界面显示微调 (通常在 hugo.yaml 全局设置，这里可单独覆盖)
# -----------------------------------------------------------------------
# ShowReadingTime: true
# ShowBreadCrumbs: true
# ShowPostNavLinks: true
# ShowWordCount: true
# ShowRssButtonInSectionTermList: true
# UseHugoToc: true

# -----------------------------------------------------------------------
# SEO 与 编辑链接
# -----------------------------------------------------------------------
# canonicalURL: "https://canonical.url/to/page"
# editPost:
#     URL: "https://github.com/ApolloMonasa/Tech/content"
#     Text: "Suggest Changes"
#     appendFilePath: true
---

# 模板

>  模板是另一种**运行时多态**，模板在两种编程范式中应用，一是泛型编程，二是模板元编程，本章主要介绍前者。
>
>  如果说函数是把参数抽象出来，那模板就是把参数类型抽象出来，模板真正的作用是抽象出任意类型的概念，在程序设计中引入任意类型。

## 模板函数

### 基础语法

```cpp
#include <iostream>
using namespace std;

template<typename T> // 当然typename也可以定义多个，逗号隔开即可	
T add(T a, T b) {
    return a + b;
}

int main(){
    // 隐式确定参数类型
    cout << add(1, 2) << endl;
    // 显式调用模板实例化
    cout << add<double>(1.1, 2.2) << endl;
    return 0;
}
```

### 实现完美的add模板函数

上面的例子中实现得太过简单，导致如果两个参数的类型不一致就会出现类型推导失败的bug，我们当然也可以强制显式实例化为某一种类型，不过也可以增加模板的类型参数：

```cpp
#include <iostream>
using namespace std;

#define P(msg) {\
    cout << #msg << " : " << msg << endl;\
}

template<typename T>
T add1(T a, T b) {
    return a + b;
}

///@brief 带类型推断的add
///@param a 
///@param b 
///@return decltype(T() + U()) 两个类型相加应该是什么类型就返回什么类型
template<typename T, typename U>
decltype(T() + U()) add2(T a, U b) {
    return a + b;
}

int main(){
    P(add1(1, 2));
    P(add1<double>(1.1, 2.2));
    // P(add(1, 2.2));
    // 1.显式指定
    P(add1<double>(1, 2.2));
    // 2.模板修复
    P(add2(1, 2.2));
    P(add2(1.1, 2));

    return 0;
}
```

这样貌似没有bug了，但是要是T和U没有默认构造函数呢？

 ```cpp
template<typename T, typename U>
auto add3(T a, U b) -> decltype(a + b) {
    return a + b;
}
 ```

这种是C++比较新的一个语法，可把返回值后置，这样就可以直接使用a + b作为类型推导的表达式了，不用调用默认构造。

## 类型推导机制

只有在使用模板函数(**模板的实例化**)时，才会去推导确定相关的参数类型。

> [!NOTE]
>
> 要验证这一点，读者可以使用如下命令查看生成的对象文件中有哪些对象：
>
> ```cmd
> g++ -c filename.cpp
> nm -C filename.o
> ```

由此我们也知道了，**模板实例化就发生在编译阶段**。

### 隐式推导

```
1. 匹配模板
2. 类型推导
```

```cpp
#include <iostream>
using namespace std;

template<typename T>
void Type(T a) {
    cout << "Type(T a) = " << a << endl;
}

template<typename T>
void Type(T *a) {
    cout << "Type(T *a) = " << a << "," << *a << endl;
}

int main(){
    int a = 123;
    double b = 45.6;
    Type(a);
    Type(b);

    int *p1 = &a;
    double *p2 = &b;
    Type(p1);
    Type(p2);

    return 0;
}
```



所谓匹配模板，就是上述代码中选择模板函数的过程，是用`Type(T a)`还是`Type(T *a)`?

而匹配类型就是根据参数列表确定T的过程，比如如果调用的是传入int*的Type(T *)，T就被推导为int。

### 显式推导

即用尖括号直接指定模板参数类型，略去类型推导，只进行模板匹配，其实在add模板里面早就见过了。

### 间接推导

> 无论是隐式还是显式，其实都是通过给模板传参来推导，这种比较好理解，那么什么是**间接推导**？即**当模板作为参数被使用时，被迫推导**。

```cpp
#include <iostream>
using namespace std;

template<typename T, typename U>
T template_param(U a) {
    cout << "in template : a = " << a << endl;
    return 2 * a;
}

void func1(int (*func)(double)) {
    int val = func(12.3);
    cout << "in func1 : func(12.3) = " << val << endl;
}

int main(){
    func1(template_param);
    return 0;
}
```

```
in template : a = 12.3
in func1 : func(12.3) = 24
```

### 引用类型的推导

先用一段代码看看问题：

```cpp
#include <iostream>
using namespace std;

///@brief 希望是一个能接受左值和右值的模板
///@param a 
template<typename T>
void Print(T &a) {
    cout << a << endl;
}

int main(){
    // Print(123);
    // 当前的模板只能传入左值
    int val = 123;
    Print(val);
    return 0;
}
```

但是如果在模板中写的是&&，我们就发现，此时能够传入左值，也能够传入右值，这是为什么？

> 遵循两步，**推导T**和**引用折叠**，规则是，传入**左值T被推导为左值，传入右值T被推导为右值**，引用折叠的规则是，**左值无论和模板形参列表中怎么叠加都是左值，右值叠右值才是右值。**因此传入右值时，T被推导为&&，和形参的&&叠加还是&&，传入左值时，T被推导为&，和&&叠加还是&，故(T &&)在模板函数中也被称为通用引用。

## 模板类

### 语法基础

```cpp
#include <iostream>
using namespace std;

template<typename T>
class Point {
public:
    void out() {
        cout << x << ", " << y << endl;
    }
    T x, y;
};

int main(){
    Point<int> p1{3, 4};
    p1.out();
    return 0;
}
```

## 全特化和偏特化

**全特化就是没有模板参数(模板函数/模板类)**，是原模板(就是原版存在一个同名同参数的模板函数)的一个特殊版本，不能独立存在，是一个补丁，比如针对某一种类型推导情况做一些额外的动作，这就可以用到全特化，此时如果把`template<>`删除也能跑，不过就变成了一个普通函数，而且这样做会导致显式指定时会忽略掉这个普通函数，转而实例化原模板。

**偏特化也是原模板的一个特殊版本，但保有模板参数。**

```cpp
#include <iostream>
using namespace std;

template<typename T>
class Point {
public:
    void out() {
        cout << x << ", " << y << endl;
    }
    T x, y;
};

///@brief 偏特化
template<typename T>
class Point<T *> {
public:
    void out() {
        cout << *x << ", " << *y << endl;
    }
    T *x, *y;
};

int main(){
    int a = 3, b = 4;
    Point<int *> p1{&a, &b};
    p1.out();
    return 0;
}
```

> [!WARNING]
>
> **模板函数没有偏特化**，本质上是函数重载，所谓的偏特化是假象。

---

## 变参模板



---

## 体验模板元编程