# auto关键字

C++中auto关键字可以基于变量所对应的值自动赋予变量所属类型

例如：

```cpp
auto a = 3; //int
auto b = a; //int
auto c = 4.5; //float
```

不建议对基本类型使用auto代替

## 应用场景

auto关键字常用于如下场景：

- 自定义类型变量：由于类型名过长，为提高编码效率可以使用auto创建相应的变量
- 自动获取函数返回值类型：定义变量接受函数返回值，此时可以利用auto自动获取函数返回值类型，这样当被调用函数返回值类型变化时，调用语句的代码可以自适应

```cpp
#include <iostream>
#include <map>
#include <typeinfo>
using namespace std;

string getName() {
    return "kkb";
}

int main(int argc, char *argv[]) {

    auto a = 3;
    auto b = a;
    auto c = 3.5;

    cout << sizeof(a) << endl;
    cout << sizeof(b) << endl;
    cout << sizeof(c) << endl;

    map<int, int> m;
    for (int i = 0; i < 10; ++i) {
        m[rand() % 10] = rand();
    }
    auto it = m.begin(); //应用场景1
    for (; it != m.end(); ++it) {
        cout << it -> first << "," << it -> second << endl;
    }

    auto name = getName();
    cout << name << endl;

    cout << typeid(b).name() << endl;
    cout << typeid(it).name() << endl;
    cout << typeid(name).name() << endl;

    if (typeid(c).hash_code() == typeid(float).hash_code()) {
        cout << "c is float" << endl;
    }
    if (typeid(c).hash_code() == typeid(double).hash_code()) {
        cout << "c is double" << endl;
    }

    return 0;
}
```

## auto不适用场景

- 不可作为函数形参的类型

  ```cpp
  string getName(auto x); //不可用于定义形参类型
  ```

- 不能作为模板的参数类型

- auto不能定义数组

  ```cpp
  auto a[3] = {1, 2, 3}; //不可用于定义数组
  ```

- 不能用于声明类中成员

  ```cpp
  class A {
      int x, y;
      auto z; //不可定义类中成员
  };
  ```



# constexpr表达式

## constexpr与const的区别

- const修饰的变量允许在运行时才确定变量代表的值
- constexpr要求在编译时就确定变量代表的值

```cpp
int n;
cin >> n;

const int x = 123 + n; //const允许程序运行时确定x的值
constexpr int y = 123 + n; //constexpr只允许在编译时就确定y的值
```

## constexpr使用场景

1. 用于修饰变量

   ```cpp
   constexpr int x = 123;
   ```

   - 如果被定义为全局变量，则不会占用运行时内存，x = 123会被记录在代码段，实际在读取x的时候直接被替换为123
   - 如果被定义为临时变量，则会分配栈段内存

2. 用于修饰函数

   ```cpp
   constexpr int mul2(int x) {
       return 2 * x;
   }
   
   int main() {
       constexpr int k = mul2(3);
       return 0;
   }
   ```

   - 可以在函数返回类型前加constexpr，将其定义为常函数
   - 在主函数想获取常函数的返回值，也需要定义常变量

3. 用于修饰类型

   ```cpp
   class A {
       constexpr A(){}
   };
   
   int main() {
       constexpr A a;
       return 0;
   }
   ```

   - 可以在类的构造函数前加constexpr修饰，使得创建出来的对象为常对象
   - 主函数中通过默认构造生成对象时也要加constexpr修饰



# nullptr 与 NULL

- NULL既可以表示数字0，也可以表示空指针
- nullptr只能表示空指针

```cpp
#include <iostream>
using namespace std;

void test(int x) {
    cout << __PRETTY_FUNCTION__ << endl;
    return;
}

void test(int *x) {
    cout << __PRETTY_FUNCTION__ << endl;
    return;
}

int main(int argc, char *argv[]) {

    //test(NULL); //歧义
    test(nullptr);

    void *p = NULL;
    void *k = nullptr;
    cout << p << endl;
    cout << k << endl;

    cout << typeid(NULL).name() << endl;
    cout << typeid(nullptr).name() << endl;

    return 0;
}
```

如上代码：

- 实际打印p和k时，都是0
- NULL和nullptr的typeid是不同的，前者是long，后者是地址
- 通过重载实现的两个test函数，如果传入NULL就会产生歧义，此时如果想传入空指针就需要利用nullptr



# 左值与右值

## 如何判断左值右值

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
    int x = 123, y = 456;

    123;    //r
    x;      //l
    x + y;  //r
    x++;    //r
    ++x;    //l

    x + 3;  //r
    x *= 33;//l
    y += 3; //l
    y * 3;  //r

    return 0;
}
```

如上代码：

判断左右值的方式：当前行代码的下一行如果还能通过一个变量访问当前行的结果，那么它就是左值，否则就是右值

## 左右值引用

向函数既可以传递左值引用，也可以传递右值引用

```cpp
#include <iostream>
using namespace std;

#define func(x) __func(x, "func(" #x ")")
#define func2(x) __func2(x, "func2(" #x ")")

void __func2(int &x, string str) { //左值引用
    cout << str << " is left value" << endl;
    return;
}

void __func2(int &&x, string str) { //右值引用
    cout << str << " is right value" << endl;
    return;
}

void __func(int &x, string str) { //左值引用
    cout << str << " is left value" << endl;
    func2(x);
    cout << "================" << endl;;
    return;
}

void __func(int &&x, string str) { //右值引用
    cout << str << " is right value" << endl;
    func2(x);
    cout << "================" << endl;
    return;
}


int main(int argc, char *argv[]) {
    int x = 123, y = 456;

    func(123);    //r
    func(x);      //l
    func(x + y);  //r
    func(x++);    //r
    func(++x);    //l

    func(x + 3);  //r
    func(x *= 33);//l
    func(y += 3); //l
    func(y * 3);  //r

    return 0;
}
```

如上代码：

- &&x就是传递右值引用

- 传递进来的右值引用在函数内部使用时就会变成左值

## 左右值转换

### 左值转换为右值

- 通过move方法

  ```cpp
  move(x); //将左值x转换为右值
  ```

- 通过forward方法

  ```cpp
  forward<int &&>(x); //将左值x按照int类型转换为右值
  ```

### 右值转换为左值

可以通过右值引用的方式实现右值到左值的转换

```cpp
#include <iostream>
using namespace std;

class A {
    int x;
public:
    A(int x = 0) : x(x) {
        cout << this << " :default constructor" << endl;
    }
    A(const A &a) : x(a.x) {
        cout << this << " :copy constructor" << endl;
    }
    A operator+(const A &a) {
        return A(x + a.x);
    }
    ~A() {
        cout << this << " :destructor" << endl;
    }
};

int main(int argc, char *argv[]) {
    A a(1), b(2);
    A &&c = (a + b); //利用右值引用将a+b的匿名对象变为左值
    //A c = a + b; //又调了一次拷贝构造将a+b的匿名对象复制给c，造成资源浪费
    cout << "===========" << endl;

    return 0;
}
```

运行结果：

```shell
0x7ffce5b1db74 :default constructor #创建a
0x7ffce5b1db78 :default constructor #创建b
0x7ffce5b1db34 :default constructor #创建a+b
0x7ffce5b1db7c :copy constructor #拷贝a+b的返回值给匿名对象，进而转换为右值引用c
0x7ffce5b1db34 :destructor #析构a+b
===========
0x7ffce5b1db7c :destructor #析构c
0x7ffce5b1db78 :destructor #析构b
0x7ffce5b1db74 :destructor #析构a
```

如上代码：

- 如果不利用右值引用将右值a+b转换为左值，那么在执行完a+b后匿名对象就会被销毁
- 如果不通过右值引用而直接c=a+b，会再调用一次拷贝构造，将a+b返回后生成的匿名对象再复制一份给c，这样会造成时间空间的浪费

## 右值引用实现常量复用

```cpp
const int &a = 123;
int &&a = 123;
```

如上代码中都可以将右值123转化为左值

- 第一行在a的引用前加const，转换为只读量，才能实现123的左值引用
- 第二行直接利用右值引用，实现右值123转换为左值a，并且a可以读写

## 右值引用的常见应用

C++11标准引入了右值引用的概念，大范围利用与STL中，实现移动拷贝构造，可以大幅提升STL的运行效率

```cpp
#include <iostream>
using namespace std;

#define KKBB namespace kkb{
#define KKBE }

KKBB

class Vector {
    int __size;
    int *data;
public:
    Vector(int n=10) : __size(n), data(new int[n]) {}

    //普通拷贝构造，实现深拷贝
    Vector(const Vector &v) : __size(v.__size), data(new int [v.__size]){
        cout << "deep constructor" << endl;
        for (int i = 0; i < __size; ++i) {
            data[i] = v[i];
        }
    }

    //传递右值引用，实现移动构造
    Vector(Vector &&v) : __size(v.__size), data(v.data) {
        cout << "move constructor" << endl;
        v.__size = 0;
        v.data = nullptr;
    }

    ~Vector() {
        delete[] data;
        data = nullptr;
        __size = 0;
    }

    Vector operator+(const Vector &v) { //实现两个数组相加
        Vector ret(this -> __size + v.__size);
        for (int i = 0; i < this -> __size; ++i) {
            ret[i] = (*this)[i];
        }
        for (int i = __size; i < ret.__size; ++i) {
            ret[i] = v[i - __size];
        }
        return ret;
    }

    int &operator[](int idx) const {
        return this -> data[idx];
    }

    int size() const {
        return __size;
    }
};

KKBE

ostream &operator<<(ostream &out, const kkb::Vector &v) {
    for (int i = 0; i < v.size(); ++i) {
        i && cout << ",";
        cout << v[i];
    }
    return out;
}

int main(int argc, char *argv[]) {

    kkb::Vector v1, v2;
    for (int i = 0; i < v1.size(); ++i)
        v1[i] = rand() % 100;

    for (int i = 0; i < v2.size(); ++i)
        v2[i] = rand() % 100;

    kkb::Vector v3 = v1 + v2;

    cout << v1 << endl;
    cout << v2 << endl;
    cout << v3 << endl;

    //kkb::Vector v4(move(v3));
    kkb::Vector v4(forward<kkb::Vector &&>(v3));
    cout << v4 << endl;

    return 0;
}
```

如上代码：

1. 主函数中在实现v3 = v1 + v2时，如果没有移动构造，那么在调用加法运算符重载时，ret会调用深拷贝在主函数空间内构造出一个匿名对象，然后v3也会调用深拷贝将匿名对象赋值给v3，这样会浪费大量的系统资源
2. 移动构造的实现是在拷贝构造的时候传递右值引用，并将右值引用对象的属性都赋给当前对象，在函数主体中将右值引用对象的属性复位后返回，这样在右值引用对象析构的时候，它的属性就可以被当前对象继承
3. 右值引用实现的移动拷贝构造，可以减少深拷贝所带来的空间时间浪费，使得STL效率提高

## 引用传递的bindorder

引用传递优先级：

- 左值传递优先左值引用
- 右值传递优先右值引用
- const传递优先const引用
- 左值、右值、const都可以通过const引用实现

