# 线程池demo
## 线程池简介
多线程技术主要是解决单个处理器单元内多个线程的执行问题，由此诞生了所谓的线程池技术。线程池主要由三个基本部分组成：  
- 线程池管理器（Thread Pool）：负责创建、管理线程池，最基本的操作为：创建线程池、销毁线程池、增加新的线程任务；
- 工作线程（Worker）：线程池中的线程，在没有任务时会处于等待状态，可以循环执行任务；
- 任务队列（Tasks Queue）：未处理任务的缓存队列。

简单来说，一个线程池负责管理了需要执行的多个并发执行的多个线程中可执行数量多的线程、以及他们之间的调度。  

为了更深刻的理解线程池这项技术，我们来看一个实际点的例子。  

在 Web 服务器中，如果一天中服务器需要处理一百万个请求，并且每个请求都需要让一个独立的线程完成。为了保证服务器任务执行的高效性（能执行的赶紧执行），不应该让并发执行的线程数无节制的增长，所以，线程池在这其中就发挥了作用。  

考虑一个处理器完成一项任务会分为创建线程、线程执行任务和销毁线程三个阶段。  

如果在某个访问高峰期同时出现了十万的并发请求，且每个任务的请求都很简单，甚至执行任务的时间还小于这个线程被创建的时间，那么这时处理器必须花费大量的时间来创建这些请求的线程，而很长时间内让各个线程得不到执行。  

有了线程池之后，我们可以在程序启动后创建一定数量的线程，当任务到达后，缓冲队列会将任务加入到线程中进行执行，执行完成后，线程并不销毁，而是等待下一任务的到来。  

## 语言特性
C++11 引入了非常丰富且有用的新特性，尤其是并发编程支持的大量新特性，这才使得我们能够在100行以内编写一个复杂的线程池成为可能。在设计编写线程池之前，我们先回顾一下我们可能会用到的这些特性。  

我们将在接下来的篇幅中复习(学习)下面这些 C++11 的特性、泛型编程以及多线程中的并发模型（互斥锁）。

### 语言特性
#### lambda expression
#### 尾置返回类型
#### 右值引用
### 标准库特性
#### std::thread
#### std::mutex, std::unique_lock
#### std::future, std::packaged_task
#### std::condition_variable
#### std::function, std::bind
#### std::shared_ptr, std::make_shared
#### std::move, std::forward

### Lambda 表达式
Lambda 表达式是 C++11中最重要的新特性之一，而 Lambda 表达式，实际上就是提供了一个类似匿名函数的特性，而匿名函数则是在需要一个函数，但是又不想费力去命名一个函数的情况下去使用的。这样的场景其实有很多很多，所以匿名函数几乎是现代编程语言的标配。  

Lambda 表达式的基本语法如下：  
```CPP
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
    // 函数体
}
```

上面的语法规则除了 \[捕获列表] 内的东西外，其他部分都很好理解，只是一般函数的函数名被略去，返回值使用了一个 -> 的形式进行。  

所谓捕获列表，其实可以理解为参数的一种类型，lambda 表达式内部函数体在默认情况下是不能够使用函数体外部的变量的，这时候捕获列表可以起到传递外部数据的作用。根据传递的行为，捕获列表也分为以下几种：  

#### 值捕获
与参数传值类似，值捕获的前提是变量可以拷贝，不同之处则在于，被捕获的变量在 lambda 表达式被创建时拷贝，而非调用时才拷贝：  
```CPP
void learn_lambda_func_1() {
    int value_1 = 1;
    auto copy_value_1 = [value_1] {
        return value_1;
    };
    value_1 = 100;
    auto stored_value_1 = copy_value_1();
    // 这时, stored_value_1 == 1, 而 value_1 == 100.
    // 因为 copy_value_1 在创建时就保存了一份 value_1 的拷贝
}
```

#### 引用捕获
与引用传参类似，引用捕获保存的是引用，值会发生变化。  

```CPP
void learn_lambda_func_2() {
    int value_2 = 1;
    auto copy_value_2 = [&value_2] {
        return value_2;
    };
    value_2 = 100;
    auto stored_value_2 = copy_value_2();
    // 这时, stored_value_2 == 100, value_1 == 100.
    // 因为 copy_value_2 保存的是引用
}
```

#### 隐式捕获
手动书写捕获列表有时候是非常复杂的，这种机械性的工作可以交给编译器来处理，这时候可以在捕获列表中写一个 & 或 = 向编译器声明采用引用捕获或者值捕获。  

总结一下，捕获提供了lambda 表达式对外部值进行使用的功能，捕获列表的最常用的四种形式可以是：  

- \[] 空捕获列表
- \[name1, name2, ...] 捕获一系列变量
- \[&] 引用捕获, 让编译器自行推导捕获列表
- \[=] 值捕获, 让编译器执行推导应用列表  

### 尾置返回类型
有时候，当希望编写一个函数来接收某个序列容器中返回的一个元素的应用时候，你可能就不太能够想明白应该如何写出这个函数的返回值类型了：  

```CPP
template <typename T>
return_type &getItem(T begin, T end) {
    return *begin; // 返回序列中一个元素的引用
}
```

这里的 return_type 应该怎么写呢？事实上，我们可能会想到使用 decltype() 来获得这个类型，但是，编译器在读到这个函数定义的时候，begin 甚至还没有出现，这时候我们似乎没有任何办法直接在返回类型的时候写下这个返回类型。  

C++11 提供了一种新的书写返回值的方式，那就是将返回类型尾置。尾置的返回类型允许我们在参数列表之后申明返回的类型，我们的代码可以写成：  

```CPP
template <typename T>
auto &getItem(T begin, T end) -> decltype(*begin) {
    return *begin; // 返回序列中一个元素的引用
}
```

其中，我们使用 decltype 告知了编译器返回类型与参数表中的返回类型相同，而 decltype 会自动推断为元素类型的引用，完成了我们的需求。  

当然，并不是只有这种情况才能够使用尾置返回类型，任函数都可以这么干，这种写法的好处在于能够让我们的返回类型变得清晰，以至于我们不会被各种复杂的返回类型搞得头晕，例如：  

```CPP
int (*)[5]func(int value) {
}
```
可以写成  

```CPP
auto func(int value) -> int (*)[5] {
}
```

## 标准库特性
在 C++11 中引入了一整套完善的并发编程的标准，我们来回顾一下这些特性。  

### std::thread
std::thread 用于创建一个执行的线程实例，所以它是一切并发编程的基础，使用时需要包含头文件，它提供了很多基本的线程操作，例如get_id()来获取所创建线程的线程 ID，例如使用 join() 来加入一个线程等等，例如：  

```CPP
#include <iostream>
#include <thread>
void foo() {
    std::cout << "hello world" << std::endl;
}
int main() {
    std::thread t(foo);
    t.join();
    return 0;
}
```

### std::mutex, std::unique_lock
我们在操作系统的相关知识中已经了解过了有关并发技术的基本知识，mutex 就是其中的核心之一。C++11引入了 mutex 相关的类，其所有相关的函数都放在 头文件中。  

std::mutex 是 C++11 中最基本的 mutex 类，通过实例化 std::mutex 可以创建互斥量，而通过其成员函数 lock() 可以仅此能上锁，unlock() 可以进行解锁。但是在在实际编写代码的过程中，最好不去直接调用成员函数，因为调用成员函数就需要在每个临界区的出口处调用 unlock()，当然，还包括异常。这时候 C++11 还为互斥量提供了一个 RAII 语法的模板类std::lock_gurad。RAII 在不失代码简洁性的同时，很好的保证了代码的异常安全性。  

在 RAII 用法下，对于临界区的互斥量的创建只需要在作用域的开始部分，例如：  

```CPP
void some_operation(const std::string &message) {
    static std::mutex mutex;
    std::lock_guard<std::mutex> lock(mutex);

    // ...操作

    // 当离开这个作用域的时候，互斥锁会被析构，同时unlock互斥锁
    // 因此这个函数内部的可以认为是临界区
}
```

由于 C++ 保证了所有栈对象在声明周期结束时会被销毁，所以这样的代码也是异常安全的。无论 some_operation() 正常返回、还是在中途抛出异常，都会引发堆栈回退，也就自动调用了 unlock()。  

而 std::unique_lock 则相对于 std::lock_guard 出现的，std::unique_lock 更加灵活，std::unique_lock 的对象会以独占所有权（没有其他的 unique_lock 对象同时拥有某个 mutex 对象的所有权）的方式管理 mutex 对象上的上锁和解锁的操作。所以在并发编程中，推荐使用 std::unique_lock。例如：  

```CPP
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void block_area() {
    std::unique_lock<std::mutex> lock(mtx);
    //...临界区
}
int main() {
    std::thread thd1(block_area);

    thd1.join();

    return 0;
}
```

### std::future, std::packaged_task
std::future 则是提供了一个访问异步操作结果的途径，这句话很不好理解。为了理解这个特性，我们需要先理解一下在 C++11之前的多线程行为。  

试想，如果我们的主线程 A 希望新开辟一个线程 B 去执行某个我们预期的任务，并返回我一个结果。而这时候，线程 A 可能正在忙其他的事情，无暇顾及 B 的记过，所以我们会很自然的希望能够在某个特定的时间获得线程 B 的结果。  

在 C++11 的 std::future 被引入之前，通常的做法是：创建一个线程A，在线程A里启动任务 B，当准备完毕后发送一个事件，并将结果保存在全局变量中。而主函数线程 A 里正在做其他的事情，当需要结果的时候，调用一个线程等待函数来获得执行的结果。  

而 C++11 提供的 std::future 简化了这个流程，可以用来获取异步任务的结果。自然地，我们很容易能够想象到把它作为一种简单的线程同步手段。  

此外，std::packaged_task 可以用来封装任何可以调用的目标，从而用于实现异步的调用。例如：  

```CPP
#include <iostream>
#include <future>
#include <thread>

int main()
{
    // 将一个返回值为7的 lambda 表达式封装到 task 中
    // std::packaged_task 的模板参数为要封装函数的类型
    std::packaged_task<int()> task([](){return 7;});
    // 获得 task 的 future
    std::future<int> result = task.get_future();    // 在一个线程中执行 task
    std::thread(std::move(task)).detach();    std::cout << "Waiting...";
    result.wait();
    // 输出执行结果
    std::cout << "Done!" << std:: endl << "Result is " << result.get() << '\n';
}
```

在封装好要调用的目标后，可以使用 get_future() 来获得一个 std::future 对象，以便之后事实线程同步。  

### std::condition_variable
std::condition_variable 是为了解决死锁而生的。当互斥操作不够用而引入的。比如，线程可能需要等待某个条件为真才能继续执行，而一个忙等待循环中可能会导致所有其他线程都无法进入临界区使得条件为真时，就会发生死锁。所以，condition_variable 实例被创建出现主要就是用于唤醒等待线程从而避免死锁。std::condition_variable的 notify_one() 用于唤醒一个线程；notify_all() 则是通知所有线程。  

#### 下面是一个生产者和消费者模型的例子：    


```CPP
#include <condition_variable>
#include <mutex>
#include <thread>
#include <iostream>
#include <queue>
#include <chrono>

int main()
{
    // 生产者数量
    std::queue<int> produced_nums;
    // 互斥锁
    std::mutex m;
    // 条件变量
    std::condition_variable cond_var;
    // 结束标志
    bool done = false;
    // 通知标志
    bool notified = false;

    // 生产者线程
    std::thread producer([&]() {
        for (int i = 0; i < 5; ++i) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            // 创建互斥锁
            std::unique_lock<std::mutex> lock(m);
            std::cout << "producing " << i << '\n';
            produced_nums.push(i);
            notified = true;
            // 通知一个线程
            cond_var.notify_one();
        }   
        done = true;
        cond_var.notify_one();
    }); 

    // 消费者线程
    std::thread consumer([&]() {
        std::unique_lock<std::mutex> lock(m);
        while (!done) {
            while (!notified) {  // 循环避免虚假唤醒
                cond_var.wait(lock);
            }   
            while (!produced_nums.empty()) {
                std::cout << "consuming " << produced_nums.front() << '\n';
                produced_nums.pop();
            }   
            notified = false;
        }   
    }); 

    producer.join();
    consumer.join();
}
```

### std::function, std::bind
std::function 是一种通用、多态的函数封装，它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作，它也是对 C++ 中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调用不是类型安全的），换句话说，就是函数的容器。  

而 std::bind 则是用来绑定函数调用的参数的，当我们使用 std::packaged_task 封装一个执行函数时，自然有必要对其调用参数进行传递，这时候就可以使用 std::bind 将实参绑定到调用函数上。例如：  
```CPP
#include <iostream>
#include <functional>

int Foo(int a, int b, int c) {
    ;
}
int main() {
    // 将参数1,2绑定到函数 Foo 上，但是使用 std::placeholders::_1 来对第一个参数进行占位
    auto bindFoo = std::bind(Foo, std::placeholders::_1, 1,2);
    // 这时调用 bindFoo 时，只需要提供第一个参数即可
    bindFoo(1);
}
```

### std::shared_ptr, std::make_shared
C++11 在内存管理上同样做了很多改进，std::make_shared 就是其中之一。它是和 std::shared_ptr 共同出现的，std::shared_ptr 是一种智能指针，它能够记录多少个 shared_ptr 共同指向一个对象（熟悉 Objective-C 的可能知道，这种特性叫做引用计数），能够消除显示的调用 delete，当引用计数变为0的时候就会将对象自动删除。  

但还不够，因为使用 std::shared_ptr 仍然需要使用 new来调用，这使得代码出现了某种程度上的不对称。因此就需要另一种手段(工厂模式)来解决这个问题。  

std::make_shared 就能够用来消除显示的使用 new，所以std::make_shared 会分配创建传入参数中的对象，并返回这个对象类型的std::shared_ptr指针。例如：  

```CPP
#include <iostream>
#include <memory>

void foo(std::shared_ptr<int> i)
{
    (*i)++;
}
int main()
{
    // 构造了一个 std::shared_ptr
    auto pointer = std::make_shared<int>(10);
    foo(pointer);
    std::cout << *pointer << std::endl;
}
```

### std::move, std::forward, std::result_of
std::move 和 std::forward 从名称上看似乎是用来转移某些东西的但实际上他们不转移任何东西，在运行时，它们不会产生一行代码。它们的功能就是就行类型转换。而 std::move 会无条件将自己的参数转换为右值。举个例子：  

```CPP
#include <iostream> // std::cout
#include <utility>  // std::move
#include <vector>   // std::vector
#include <string>   // std::string

int main() {

    std::string str = "Hello world.";
    std::vector<std::string> v;

    // 将使用 push_back(const T&), 即产生拷贝行为
    v.push_back(str);
    // 将输出 "str: Hello world."
    std::cout << "str: " << str << std::endl;

    // 将使用 push_back(const T&&), 即产生右值引用，这时，不会出现拷贝行为
    // 而整个字符串会被移动到 vector 中，所以有时候 std::move 会用来减少拷贝出现的开销
    // 这步操作后, str 中的值会变为空
    v.push_back(std::move(str));
    // 将输出 "str: "
    std::cout << "str: " << str << std::endl;

    return 0;
}
```

而 std::forward 顾名思义就是进行转发，它的行为和 std::move 十分类似，区别在于它会把参数被绑定到一个右值的时候将其转化为右值。  

对于 std::result_of，它的作用是可以在编译的时候推导出一个函数调用表达式的返回值类型，例如：  

```CPP
struct S {
    double operator()(char, int&); // 这个函数的返回类型是 double
};

int main()
{

    std::result_of<S(char, int&)>::type foo = 3.14; // 使用这样的写法会推导出模板参数中函数的返回值类型
    typedef std::result_of<S(char, int&)>::type MyType; // 是 double 类型吗?
    std::cout << "foo's type is double: " << std::is_same<double, MyType>::value << std::endl;
    return 0;
}
```

上面的代码最终会输出：  

```CPP
foo's type is double: true
```

