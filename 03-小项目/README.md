# 需要手撕的内容
### [常用函数strcpy/strcat/strcmp/strlen/memcpy/memset以及对应的实现](https://blog.csdn.net/u012611878/article/details/79222918)

## 将构造函数放在private中——单例模式
```CPP
class Foo {
private:
    Foo() = default;
public:
    Foo& getInstance() { static Foo instance; return instance; }
};
```
