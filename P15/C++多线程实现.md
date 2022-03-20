# C++ 线程基础

C++ 中对线程做了面向对象的封装，一个线程就是一个对象，可以利用标准库中提供的线程API进行对象操作

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

#define BEGIN(x) namespace x{
#define END(x) }

//辅助函数：判断x是否为素数
bool is_prime(int x) {
    for (int i = 2; i * i <= x; ++i) {
        if (x % i == 0) {
            return false;
        }
    }
    return true;
}

int cnt = 0; //用于统计素数的个数
mutex mtx; //线程锁

//功能函数
void count_prime(int l, int r) {
    for (int i = l; i <= r; ++i) {
        int ret = is_prime(i);
        unique_lock<mutex> lock(mtx); //生成锁对象，线程锁定
        cnt += ret; //尽可能减少锁定的内容
        lock.unlock(); //释放锁
    }
}

//回调函数
void worker(int l, int r) {
    cout << this_thread::get_id() << ":begin" << endl;
    count_prime(l, r);
    cout << this_thread::get_id() << ":end" << endl;
}

int main(int argc, char *argv[]) {

    const int batch = 5000000;
    thread *trr[10]; //声明线程池
    int ret[10];
    for (int i = 0, j = 1; i < 10; ++i, j += batch) {
        //生成线程对象，传入回调函数入口和参数及返回值
        trr[i] = new thread(worker, j, j + batch - 1); //构造线程对象
    }

    for (int i = 0; i < 10; ++i)
        trr[i] -> join();

    for (auto x : trr)
        delete x;

    cout << cnt << endl;

    return 0;
}
```

如上代码：实现五千万以内素数个数的统计

- 将整个任务分为10个批次，对应10个线程解决，每个批次对应五百万的数字
- 线程的构造可以利用标准库中的线程对象构造方法
  - 线程构造需要传入回调函数，参数和返回值（一般为引用）
  - 线程的调用，一般不直接调用功能函数，而是再封装一层回调函数，这样可以将线程和功能解耦，使程序更灵活
- C++中的线程锁也是一个对象，利用类模板unique_lock<mutex>创建一个线程锁对象
- 当多个线程同时需要操作相同的资源时，会出现资源争抢的现象，此时就需要加锁以实现线程同步
- 加锁要尽可能对原子操作来进行，这样可以尽可能减少系统等待时间，提高程序运行效率



# 线程池实现

```cpp
#include <iostream>
#include <functional>
#include <thread>
#include <vector>
#include <unordered_map>
#include <queue>
#include <mutex>
#include <condition_variable>
using namespace std;

class Task {
private:
    function<void()> func; //定义高级函数指针，实现任务入口
public:
    template <typename FUNC_T, typename ...ARGS>
    Task(FUNC_T f, ARGS ...args) {
        func = bind(f, forward<ARGS>(args)...); //利用函数绑定，实现回调
    }

    void run() {
        func();
    }
};

class ThreadPool {
public:
    ThreadPool(int n = 1) : trr(n), starting(false) {
        this -> start();
        return;
    }

    void worker() {
        thread::id id = this_thread::get_id();
        running[id] = true;
        while (running[id]) {
            //获取任务
            Task *t = get_task();
            //执行任务
            t -> run();
            delete t;
        }
        return;
    }

    void start() {
        if (starting == true) return;

        for (int i = 0; i < trr.size(); ++i) {
            trr[i] = new thread(&ThreadPool::worker,this);
        }
        starting = true;
    }

    void stop() { //封装接口：终止所有任务
        if (starting == false) return;
        for (int i = 0; i < trr.size(); ++i) { //调用类私有方法终止任务
            add_task(&ThreadPool::stop_running, this);
        }
        for (int i = 0; i < trr.size(); ++i) { //等待线程结束
            trr[i] -> join();
        }
        for (int i = 0; i < trr.size(); ++i) { //线程池复位
            delete trr[i];
            trr[i] = nullptr;
        }
        starting = false;
    }

    //封装模板函数，向任务队列中添加任务
    template <typename FUNC_T, typename ...ARGS>
    void add_task(FUNC_T f, ARGS ...args) {
        unique_lock<mutex> lock(mtx); //上锁
        task_q.push(new Task(f, forward<ARGS>(args)...));
        //解锁:同时通知队列中有任务的条件已经达成
        condi.notify_one();
        return;
    }

    virtual ~ThreadPool() {
        this -> stop();
        while (!task_q.empty()) {
            delete task_q.front();
            task_q.pop();
        }
    }

private:
    void stop_running() { //终止单个任务方法
        auto id = this_thread::get_id();
        running[id] = false;
        return;
    }

    Task *get_task() { //线程池获取任务
        unique_lock<mutex> lock(mtx);
        while (task_q.empty()) {
            condi.wait(lock); //队列不为空，条件达成，并释放锁
        }
        Task *t = task_q.front();
        task_q.pop();
        return t;
    }

    vector<thread *> trr; //线程池
    unordered_map<thread::id, bool> running; //记录线程运行状况
    queue<Task *> task_q; //任务队列
    mutex mtx; //线程锁
    condition_variable condi; //线程条件变量
    bool starting; //表征整个线程池的状态
};

//辅助函数：判断x是否为素数
bool is_prime(int x) {
    for (int i = 2; i * i <= x; ++i) {
        if (x % i == 0) {
            return false;
        }
    }
    return true;
}

//功能函数
int count_prime(int l, int r) {
    int cnt = 0;
    for (int i = l; i <= r; ++i) {
        int ret = is_prime(i);
        cnt += ret; //尽可能减少锁定的内容
    }
    return cnt;
}

//回调函数
void worker(int l, int r, int &ret) {
    cout << this_thread::get_id() << ":begin" << endl;
    ret = count_prime(l, r);
    cout << this_thread::get_id() << ":end" << endl;
    return;
}

int main(int argc, char *argv[]) {

    const int batch = 5000000;
    int ret[10];
    ThreadPool tp(5);
    for (int i = 0, j = 1; i < 10; ++i, j += batch) {
        //生成线程对象，传入回调函数入口和参数及返回值
        tp.add_task(worker, j, j + batch - 1, ref(ret[i]));
    }

    tp.stop();

    int ans = 0;
    for (auto x : ret) ans += x;

    cout << ans << endl;

    return 0;
}
```

