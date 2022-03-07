# 动态类型转换

对于存在多态的情况，父类指针可以指向不同的子类对象，可以通过动态类型转换 dynamic_cast<>() 方法将该父类指针强转为对应对象的地址

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>
using namespace std;

class Animal {
public:
    virtual void run() {
        cout << "i don't know how to run" << endl;
    }
};

class Horse:public Animal {
public:
    void run() override {
        cout << "horse run fast" << endl;
    }

    void say() {
        cout << "say horse" << endl;
    }
};

class Donkey:public Animal {
public:
    void run() override {
        cout << "Donkey run slow" << endl;
    }

    void swim() {
        cout << "swim happily" << endl;
    }
};

int main(int argc, char *argv[]) {
    srand(time(0));
    Animal *p;

    switch (rand() % 2) {
        case 0:
            p = new Horse;
            break;
        case 1:
            p = new Donkey;
            break;
    }
    p -> run();

    cout << "===================" << endl;
    cout << dynamic_cast<Horse *>(p) << endl; //point 1
    cout << dynamic_cast<Donkey *>(p) << endl; //point 1

    Horse *h;
    Donkey *d;
    if (h = dynamic_cast<Horse *>(p)) {
        h -> say(); //point 2
    }
    if (d = dynamic_cast<Donkey *>(p)) {
        d -> swim(); //point 2
    }

    cout << "==================" << endl;
    Horse x;
    Donkey y;
    Animal *k = &x;
    cout << dynamic_cast<Donkey *>(k) << endl;
    *(void **) &x = *(void **) &y; //point 3
    cout << dynamic_cast<Donkey *>(k) << endl;

    return 0;
}
```

如上代码：

1. 如果指针p的指向和转换类型的指针相同，转换生效，返回对应的指针地址，否则转换失败，返回0；
2. 可以利用dynamic_cast<>()的返回值判断是否转换成功，进而判断p到底指向的是哪一个类型的对象，进而访问相应类型的成员；
3. dynamic_cast<>()是利用虚表中的运行时类型信息 rtti (run time typeinfo) 来进行判断的，如果当前的指针指向和要转换的目标类型不匹配，也可以通过改变虚表指针指向的方法，让dynamic_cast强制生效（尽量避免这么操作）



