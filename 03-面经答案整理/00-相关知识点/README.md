## gdb的使用
- [GBD调试大法](https://zhuanlan.zhihu.com/p/68465617)

## 回调函数
- [完全理解回调函数](https://zhuanlan.zhihu.com/p/30280776)
- [协程和异步回调方案实现高io并发](https://zhuanlan.zhihu.com/p/65154410)
- [并发之痛 Thread，Goroutine，Actor](http://jolestar.com/parallel-programming-model-thread-goroutine-actor/)
- [C++屌屌的观察者模式-同步回调和异步回调](https://zhuanlan.zhihu.com/p/73434055)
- [深入 C++ 回调](https://zhuanlan.zhihu.com/p/88434924)

## 负载均衡怎么实现
- [什么是负载均衡原理？](https://www.zhihu.com/question/61783920)
- [什么是负载均衡？什么是高可用？说说常见的负载均衡案例](https://zhuanlan.zhihu.com/p/76918261)
- [面试刷题35:负载均衡有哪几种实现方式？](https://zhuanlan.zhihu.com/p/128565873)
- [负载均衡的几种实现方式](https://zhuanlan.zhihu.com/p/36054372)

## 设计模式
## 观察者模式
- [漫画：什么是“观察者模式”？](https://zhuanlan.zhihu.com/p/158537313)
- [设计模式之观察者模式](https://zhuanlan.zhihu.com/p/25045567)
- [观察者模式 vs 发布订阅模式](https://zhuanlan.zhihu.com/p/51357583)
## 设计模式介绍
- [C++ 常用设计模式](https://www.cnblogs.com/schips/p/12306851.html)
- [图说设计模式阅读笔记](https://wangpengcheng.github.io/2019/11/07/design_patterns_01/)
- [关于c++设计模式的总结](https://www.cnblogs.com/Chary/p/12860004.html)
- [C++的设计模式](https://zhuanlan.zhihu.com/p/111492986)

### 单例模式（饿汉模式与懒汉模式）
#### 懒汉模式（C++11线程安全写法）
```CPP
class Singleton
{
private:
	Singleton() { };
	~Singleton() { };
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() 
        {
		static Singleton instance;
		return instance;
	}
};
```
#### 懒汉模式（C++11之前的线程安全写法）
```CPP
#include<pthread.h>
class singleton{
    private:
        static singleton *instance;
        static pthread_mutex_t mutex;
        singleton(){};
    public:
        static singleton* get_instance(){
            if(instance == nullptr){
                pthread_mutex_lock(&mutex);
                if(instance == nullptr){
                    instance = new singleton();
                }
                pthread_mutex_unlock(&mutex);
            }
            return instance;
        }
    private:
        class deletor{
            ~deletor(){
                if(singleton::instance != nullptr){
                    delete singleton::instance;
                }
            }
        };
        static deletor del;
};
pthread_mutex_t singleton::mutex = PTHREAD_MUTEX_INITIALIZER;
singleton *singleton::instance = nullptr;
```
#### 饿汉模式
```CPP
class Singleton
{
private:
	static Singleton instance;
private:
	Singleton();
	~Singleton();
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() {
		return instance;
	}
}

// initialize defaultly
Singleton Singleton::instance;
```

- [C++设计模式之单例模式](https://light-city.club/sc/design_pattern/singleton/singleton/)
- [C++ 单例模式](https://zhuanlan.zhihu.com/p/37469260)
- [C++单例模式（懒汉和饿汉）与线程安全](https://www.cnblogs.com/lfri/p/12743685.html)
### 工厂方法（Factory Method）模式
- 对象创建型模式的一种——来源：[浅谈C++设计模式之工厂方法（Factory Method）](https://www.cnblogs.com/whc-uestc/p/4750948.html)
> 工厂方法模式的意义是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类当中。核心工厂类不再负责产品的创建，这样核心类成为一个抽象工厂角色，仅负责具体工厂子类必须实现的接口，这样进一步抽象化的好处是使得工厂方法模式可以使系统在不修改具体工厂角色的情况下引进新的产品。  
