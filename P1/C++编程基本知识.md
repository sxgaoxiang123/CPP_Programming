# C++编程模式

1. 面向过程编程
2. 面向对象编程
3. 泛型编程
4. 函数式编程





# 学习C++方法

## 视C++为一个语言联邦



## 尽量以const/enum/inline替换#define

#define在预处理阶段会完成替换过程，所有宏对编译器不可见

1. 对于单纯常量，最好以const对象或者enums替换#define
2. 对于形似函数的宏，最好改用inline替换#define



## 尽可能使用const

将某些东西声明为const可以帮助编译器检测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体

编译器强制实施bitwise constness（数据意义的const），但编写程序时应该使用概念上的常量性conceptual constness（逻辑上的const）

当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可以避免代码重复

- const int *p 常量指针，指向常量的指针
- int const *q 常量指针
- int *const k = NULL 指针常量，指针的指向是固定的，声明时必须初始化



## 确定对象被使用前已经先被初始化

为内置类型对象进行手工初始化，因为C++并不保证会初始化他们

构造函数最好使用成员初始列，而不要在构造函数本体内使用赋值操作

初始列列出的成员变量，其排列次序应该和他们在class中的生命次序相同

为免除“跨编译单元之初始化次序”问题，请已local static对象提花对象non-local static对象

