# 对象地址的内部透传

```cpp
#include <iostream>
using namespace std;

class A {
    int x;
public:
    A() {
        cout << this << " default constructor" << endl;
    }
    A(int x):x(x) {
        cout << this << " param constructor" << endl;
    }
    A(const A &a):x(a.x) {
        cout << this << " copy constructor" << endl;
    }
    ~A() {
        cout << this << " destructor" << endl;
    }
};

A func() {
    A tmp(100);
    cout << "tmp:" << &tmp << endl;
    return tmp;
}

int main() {
    A a = func();
    cout << "  a:" << &a << endl;
    return 0;
}
```

1. A a = func()

   对象a的地址被透传到func()内部

2. func()函数中 A tmp(100)

   - 此时tmp就是对象a的引用，即a和tmp指向同一个地址
   - tmp(100)，将a的地址继续向内透传到有参构造函数中

3. A(int x) : x(x)

   函数默认传入的this指针此时就是a的地址

经过如上步骤，直接将a的地址传递给有参构造函数，完成a的构造，省略了中间拷贝构造的过程，这是编译器优化的结果



# 返回值优化

```cpp
People func() {
    People tmp_a("master");
    return tmp_a;
}

int main() {
    People a = func();
    return 0;
}
```

返回值优化前，程序流程如下：

1. 开辟a对象数据区（申请内存空间）
2. 调用函数func
3. 开辟对象tmp_a数据区（申请内存空间）
4. 调用tmp_a对象的构造函数
5. 使用tmp_a调用临时匿名变量的拷贝构造函数
6. 销毁tmp_a对象
7. 使用临时匿名变量调用a的拷贝构造函数
8. 销毁临时匿名变量
9. 销毁a对象

返回值优化后，程序流程如下：

1. 开辟a对象数据区
2. 调用函数func
3. 将tmp_a设置为a对象的引用
4. 调用tmp_a对象的构造函数（即a对象的构造函数）
5. 销毁a对象

**注意：**由于编译器优化会导致拷贝构造函数调用的次数不同，所以在设计拷贝构造函数的时候不可以改变它的语义，要确保它仅用于拷贝，不要赋予其他意义



# 类属性、类方法

```cpp
#include <iostream>
using namespace std;

class People {
public:
    People() : say_cnt(0) {
        People::total_num++;
    }
    ~People() {
        People::total_num--;
    }
    static void showNum() { //类方法
        cout << People::total_num << endl;
    }
    void say()const { //逻辑上的const
        say_cnt++;
        cout << "hahaha" << endl;
        output();
    }
    void output() {
        cout << "non-const output" << endl;
    }
    void output()const { //output函数重载
        cout << "const output" << endl;
    }
private:
    static int total_num; //类属性声明
    mutable int say_cnt; //加mutable属性，可更改
};

int People::total_num = 0; //类属性的定义（全局变量）
int a = 3, b = 0;

int main() {
    People xp, klss;

    People::showNum(); //类方法的使用要用类名本身引用
    xp.showNum();
    klss.showNum();

    const People xk;
    xk.say();

    return 0;
}
```

