lambda表达式，又叫匿名函数对象，主要用在不需要知晓函数名称，不需要定义函数对象的情况，而只想运行该函数

# 定义

```cpp
[capture](parameters) mutable return-type {
    statement
}
```

## capture

- [var]：表示值传递方式捕捉变量var；
- [=]：表示值传递方式捕捉所有父作用域的变量（包括this）；
- [&var]：表示引用传递捕捉变量var；
- [&]：表示引用传递捕捉所有父作用域的变量（包括this）；
- [this]：表示值传递方式捕捉当前的this指针；
- []：不可以捕获全局变量，全局变量是必须可以修改的；

