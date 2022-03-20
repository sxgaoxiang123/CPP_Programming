# C++ 异常处理

C++标准库中定义了一些异常类型，用户也可根据需要自行定义异常类型

利用try-catch语法实现异常获取和异常处理：

- 将要运行的内容放入try代码块中
- 当try中代码出现异常时会向外抛出对应的异常类型
- catch中可根据不同的异常类型做出对应的反馈

```cpp
#include <iostream>
#include <vector>
#include <exception>
#include <stdexcept>
using namespace std;

//自定义零除异常类
class ZeroDivException : public runtime_error {
public:
    ZeroDivException(const string &msg ="zero div") : runtime_error(msg) {}
    const char *what() const noexcept override { //重载what函数
        return "__what__zero__div__";
    }
};

int suml2r(int l, int r, const vector<int> &v)\
        throw(out_of_range, int, exception) { //声明该函数会抛出异常
    int sum = 0;
    for (int i = l; i < r; ++i) {
        sum += v.at(i);
    }
    return sum;
}

int main(int argc, char *argv[]) {
    try { //point 1
        try {
            vector<int> arr(10);
            for (int i = 0; i < 10; ++i) {
                arr[i] = rand();
            }
            suml2r(-1, 100, arr);
            cout << arr[-1] << endl; //point 2
            cout << arr.at(-1) << endl; //point 2
            cout << "check point 1" << endl;

            int n;
            cin >> n;
            if (n == 0) throw(ZeroDivException()); //point 3
            cout << 10 / n << endl;
            cout << "check point 2" << endl;

        } catch (out_of_range &e) {
            cout << "out of range: " << e.what() << endl;
            throw(e); //继续向外抛出异常
        } catch (exception &e) { //point 4
            cout << "exception: " << e.what() << endl;
        }
    } catch (int &e) {
        cout << "int: " << e << endl;
    } catch (string &e) {
        cout << "string: " << e << endl;
    } catch (...) { //point 5
        cout << "catch an exception" << endl;
    }
    cout << "keep going" << endl;
    return 0;
}

```

如上代码：

1. try-catch可以进行嵌套，异常是从内向外逐层抛出的
2. 对于vector来说，at方法是可以抛出异常的，而中括号重载方法不会，根据具体需要进行选择使用
3. 用户可将自定义的异常类型抛出

4. catch中可以捕获标准库中定义的异常类型，如exception、out_of_range等（会有集成关系），也可以捕获一般类型，如int、string等

5. catch中...用于捕获用户自定义的异常类型



