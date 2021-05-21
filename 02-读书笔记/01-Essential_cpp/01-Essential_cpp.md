## 第一章 C++编程基础
### 1.6 指针带来的弹性
- 在使用指针时，必须在*之前确定它一定明确的指向某个对象，否则会导致未知的执行结果。
- 一个未指向任何对象的指针，其地址为0。任何指针多可以被初始化，或是令其值为0。为了防止对null指针进行*操作而导致错误，可以检验该指针所持有的地址是否为0
```
if(pi && ...)  //只有pi持有一个非零值时，其求值结果才会是true
```
- 关于指针的使用
> 取指针所指的对象的地址，用\&  
> 值指针所指的对象的值，用\*  
> 当我们提领pointer的时候，一定要先确定其值并非0。至于reference，则必定会代表某个对象，所以不需要做此检查  
```
int val = 1024;  //对象，类型为int
int *pi = &val;  //指针，指向一个int对象
int &rval = val;  //引用，代表一个int对象
```
> 【注】  
> 1. C++不允许改变reference所代表的对象，它们必须从一而终  
> 2. 对reference的所有操作和对“reference所代表的对象”所进行的操作结果一样，当我们以reference作为函数参数时，也是如此  
> 3. 当我们以reference的方式将对象作为函数参数传入时，对象本身并不会复制出另一份————复制的是对象的地址。函数中对该对象进行的任何操作，都相当于对传入的对象进行间接操作  
> 4. 在传递class object的时候，可以传reference，这样可以降低复制大型对象的额外负担，但在传递内置类型时，不要使用传址方式  
> 5. 将参数声明为reference的理由  
>> 一、希望得以直接对所传入的对象进行修改  
>> 二、降低复制大型对象的额外负担，而且程序效率会变高  

### 1.7 文件的读写
- 写入文件
```
ofstream outfile(filename,ios_base::app);  //以追加的形式将内容写入到文件中
if(!outfile)  //文件打开失败；
else outfile << xxx << endl;  //将结果写入到outfile中
```
- 读取文件
```
ifstream infile(filename);  //读取文件
```
- 读写同一个文件
```
fstream iofile(filename,ios_base::in|ios_base::app);
if(!iofile)  //文件打开失败
else iofile.seekg(0);  //开始读取之前，将文件重定位至起始处
```
> 当我们以追加模式来打开文件，文件位置会位于末尾。如果我们没有先重新定位，就试着读取文件内容，那么立刻就会遇上“文件结束”的状况。seekg()可将iofile重新定位至文件的起始处。由于此文件时以追加模式开启，因此任何写入操作都会将数据添加在文件末尾。

- cerr和cout的区别
> cerr代表标准错误设备。和cout一样，cerr将其输出结果定向到用户的终端。两者唯一的差别是，cerr的输出结果并无缓冲（buffered）情形，它会立即显示于用户终端中  

## 第二章 面向过程的编程风格
### 2.1 如何编写程序
- 想知道某个类型的最小/最大值，可查询标准库中的numeric_limits class
```
#include<limits>
int max_int = numeric_limits<int>::max();
double min_db1 = numeric_limits<double>::min();
```

- 函数必须先被声明，然后才能被调用。因此在编写小型程序时，应该先把函数声明写在using namespace std;的下面，目的是先让编译器知道这个函数的存在...

### 2.2 调用函数
- 函数调用的过程
> 当我们调用一个函数时，会在内存中建立起一块特殊区域，成为“程序堆栈（program stack）”。这块特殊区域提供了每个函数参数的存储空间。它也提供了函数所定义的每个对象的内存空间————我们将这些对象称为local object（局部对象）。一旦函数完成，这些内存就会被释放掉，或者说是从程序堆栈中被pop出来。
- 函数内定义的对象，只存在于函数执行期间。如果将这些所谓局部对象的地址返回，会导致运行时错误
- 内置类型的对象，如果定义在file scope之内，必定被初始化为0。但如果它们被定义于local scope之内，那么除非程序员指定其初值，否则不会被初始化  
- 不论local scope or file scope，对我们而言，都是由系统自动管理。第三种存储形式称为dynamic extent（动态范围）。其内存由程序的空闲空间分配而来，有时也成为heap memory。这种内存必须由程序员自行管理，其分配是通过new表达式来完成，而释放则是通过delete完成  
- C++没有提供任何语法让我们得以从heap分配数组的同时为其元素设定初值  

### 2.3 提供默认参数值
- 关于默认参数值的提供，有两个不成文的规则
> 第一个规则是，默认值的解析操作由最右边开始进行。如果我们为某个参数提供默认值，那么这一参数右侧的所有参数都必须也具有默认参数值才行。下面属于非法操作：
```
void display(ostream &os = cout, const vector<int> &vec);
```
> 第二个规则是，默认值只能够指定一次，可以在函数声明处，也可以在函数定义处，但不能在两个地方都指定。一般情况下，将默认值都放在函数声明处
```
void display(const vector<int> &, ostream &os = cout);
```

### 2.4 使用局部静态对象
- 在函数持续调用时，为了避免重复计算
> 在函数内声明局部vector对象不能解决上述问题，因为局部对象会在每次调用函数时建立，并在函数结束的同时被弃置。如果将vector对象定义于file scope之中，又过于冒险。通常file scope对象会打乱不同函数间的独立性，使他们难以理解  
因此可以使用局部静态对象。局部静态对象所处的内存空间，即使在不同的函数调用过程中，依然持续存在
```
fibon_seq(int size){
  static vector<int> elems;
  //函数的工作逻辑放在此处
  return &elems;
}
```

### 2.5 声明inline函数
- 一般而言，最适合声明为inline的函数，是体积小，常被调用，所从事的计算并不复杂的函数。声明为inline函数可以提高程序的性能
- inline函数的定义，常常被放在头文件中

### 2.6 提供重载函数
### 2.7 定义并使用模板函数
### 2.8 函数指针带来更大的弹性
- 如果有多个类似的函数，每次调用可能不同时，用函数指针更好
> 为了让seq_ptr被视为一个指针，我们必须以小括号改变运算优先级
```
const vector<int>* (*seq_ptr)(int);  //ok
const vector<int>* *seq_ptr(int);  //error
//seq_ptr为指向函数的指针，传入int参数，这其实也是该指针指向的函数的传入参数，返回值同理，为指向int类型vecor的指针
```
- 函数指针的使用，可以作为参数传到函数中
```
bool seq_elem(int pos, int &elems, const vector<int>* *seq_ptr(int)){
  const vector<int> *pseq = seq_ptr(pos);  //函数指针的调用，其调用方式和一般函数相同
  ...
}
```
> 函数指针会间接调用其所指的函数。我们不知道（或不在乎）它所指的函数是哪一个。更明智的做法是，多付出一些检验操作，确定它是否真的指向某个函数  
- 函数指针的初始化
```
const vector<int>* (*seq_ptr)(int) = 0;  //表示并未指向任何函数
const vector<int>* (*seq_ptr)(int) = pell_seq;  //直接将函数名称赋给函数指针，操作十分简单
```

## 第三章 泛型编程风格
### 3.1 指针的算术运算
### 3.2 了解Iterator（泛型指针）
- 对于const vector
```
const vector<string> cs_vec;
//我们使用const_iterator来进行遍历操作
vector<string>::const_iterator iter = cs_vec.begin();  //const_iterator允许我们读取vector的元素，但不允许任何写入操作
```
- 关于find() and find_if()
> find()的实现使用了底部元素所属类型的equality（相等）运算符，只能提供查找是否存在元素等于value的操作，不够通用  
> 将find()演化成为泛型算法，比如find_if()，能够接受函数指针或function object，取代底部元素的equality运算符，可以使用各种操作，大大提高弹性  

### 3.3 所有容器的共通操作
- 下列为所有容器的共通操作
```
equality(==) and inequality(!=)
assignment(=)
empty()
size()
clear()
begin()
end()
//insert()和erase()的行为视容器本身为顺序（sequential）容器或关联（associative）容器而有所不同
insert()  //将单一或某个范围内的元素插入容器内
erase()  //将容器内的单一元素或某个范围内的元素删除
```

### 3.4 使用顺序性容器
- 定义顺序性容器的五种方法
```
1. 定义空容器
list<string> str_list;
vector<int> int_vec;

2.定义特定大小的容器，每个元素有以其默认值作为初值（比如int、double这类语言内置的算术类型，其默认值为0）
list<string> str_list(1024);
vector<int> int_vec(32);

3.定义特定大小的容器，并为每个元素指定初值
list<string> str_list(16, "emmm");
vector<int> int_vec(32, -1);

4.通过一对iterator产生容器
int ia[8] = {1,1,2,3,4,5,6,7}
vector<int> nums(ia, ia + 8);

5.根据某个容器定义新容器
list<string> str_list;
//填充str_list..
list<string> str_list2(str_list);  //定义str_list2，并将str_list复制给str_list2
```

- 关于插入删除操作
```
push_back();  //在末端插入一个元素
pop_back();  //在末端删除一个元素

除此之外，list和deque（但不包括vector）还提供以下函数
push_front();  //在前端插入一个元素
pop_front();  //在前端删除一个元素

如果要读取最前端或者最后端的元素
xx.front();
xx.back();
```

- push_front()和push_back()都属于特殊化的插入操作。每个容器除了拥有通用的插入函数insert()，还支持四种变形
```
iterator insert(iterator position,elemType value);  //可将value插入postion之前，它会返回一个iterator，指向被插入的元素
//例如
str_list.insert(it,string);

void insert(iterator position,int count, elemType value);  //可在position之前插入count个元素，这些元素的值都和value相同
//例如
str_list.insert(it, 8, string("emmm"));

void insert(iterator1 position, iterator2 first, iterator2 last);  //可在position之前插入[first,last)所标示的各个元素
//例如
str_list.insert(it, ia2, ia2 + 4);

iterator insert(iterator position);  //可在position之前插入元素所属类型的默认值
//例如
str_list.insert(it);
```

- pop_front()和pop_back()都属于特殊化的删除操作。每个容器除了拥有通用的删除函数erase()，还支持两种变形
```
iterator erase(iterator position);  //可在position所指的元素，返回删除元素下一个位置的it
//例如
str_list.erase(it);

iterator erase(iterator first, iterator last);  //可删除[first,last)范围内的元素
//例如
str_list.erase(it1, it2);

//注：list并不支持iterator的偏移运算，因为不是连续存储的
str_list.erase(it1, it1 + num_tries); //错误
```

### 3.5 使用泛型算法
- 欲使用泛型算法，首先得包含对应的algorithm头文件
```
#include <algorithm>
```
- 简单罗列几个常用的泛型算法
```
find()  //用于搜索无序（unordered）集合中是否存在某值。若找到，返回it；若不存在，返回it的last
binary_search()  //用于搜索有序（sorted）集合中是否存在某值，要求其作用对象必须经过排序（sorted）。若找到，返回true；若不存在，返回flase
count()  //返回数值相符的元素数目
search()  //比对某个容器中是否存在某个子序列。若找到，返回it指向子序列起始处；若不存在，返回it的last
```

### 3.6 如何设计一个泛型算法
- 将普通函数转化成为泛型算法
> 1.让函数与元素类型无关  
> 2.让函数与容器类型无关  
> 3.让函数与比较操作无关  

- 可以用泛型算法find_if()来取代for循环
- Function Object
> Function Object是某种class的实例对象，这类class对function call运算符做了重载操作，如此一来可使function object被当成一般函数来使用  
> Function Object与函数指针相对应，其存在是为了让我们可以令call运算符成为inline，从而消除“通过函数指针来调用函数”时需付出的额外代价  
> 标准库事先定义了一组Function Object，分为算术运算、关系运算和逻辑运算三大类  
> 欲使用事先定义的Function Object，首先得包含相关头文件
```
#include <functional>
```
> 默认情况下，sort()会使用底部元素的类型所提供的less-than运算符，将元素升序排序。如果我们传入greater_than function Object，元素就会以降序排序
```
sort(vec.begin(),vec.end(),greater<int>());
```
- Function Object Adapter
> function object adapter会对function object进行修改操作。所谓binder adapter(绑定适配器)会将function object的参数绑定至特定值，使二元function object转化为一元function object  
> 标准库提供了两个binder adapter  
>> bind1st会将指定值绑定至第一操作数  
>> bind2nd会将指定值绑定至第二操作数  
```
A < B  //若使用bind1st，则将操作数绑定到操作符左边，即A位置上
```

> 另一种adapter是所谓negator，它会对function object真伪值取反
>> not1可对一元function object的真伪值取反  
>> not2可对二元function object的真伪值取反  

### 3.7 使用Map
- 欲查询map内是否存在某个key，有三种方法
> 1.把key当成索引来使用
```
int count = 0;
if(!(count == words["emm"]))
//emm并不存在与words map内
```
>> 这种做法存在一个缺点：会把key自动加入到map中，而其相应的value会被设置为所属类型的默认值

> 2.利用map的find函数进行查询（不要和泛型算法find()搞混了）
```
words.find("emm");
```
>> 如果key已放在其中，find()会返回一个iterator，指向key/value形成的一个pair；反之则返回end()

> 3.map的count函数（推荐）
```
int count = 0;
if(count == words.count("emm"))
```
>> 如果key已放在其中，count()会返回其value；如果不在其中，则返回int的默认值0,正好最为true/false来判断

- 如果需要存储多份相同的key值，就必须使用multimap

### 3.8 使用Set
- 在图遍历算法中，我们可以使用set存储每个遍历过的节点。在移至下一个节点前，我们可以先查询set，判断该节点是否已经遍历过
- 判断set中是否包含某一元素推荐使用set.count()方法，若存在则返回1，即true；若不存在则返回0，即false
- 默认情况下，set中的元素会被默认排序。即set元素皆依据其所属类型默认的less-than运算符进行排列
- 如果需要存储多份相同的key值，就必须使用multiset

### 3.9 如何使用Iterator Inserter
### 3.10 如何使用isotream Iterator
- 【注】以上两章节不太好总结，但都写的非常好，以后复习的时候，建议再看看

## 第四章 基于对象的编程风格
### 4.1 如何实现一个class
- private member 只能在member function或是class friend内被访问
- class主体之外定义member function
> 要在clss主体之外定义member function，必须使用特殊的语法，目的在于分辨该函数究竟属于哪一个class。  
如果希望该函数为inline，应该在最前面指定关键字inline
```
inline bool Stack::empty()
{
...
}
```

- 代码存放规则
> class定义及其inline member function通常都会被放在与class同名的头文件中  
non-inline member function应该在程序代码文件中定义，该文件通常和class同名，其后接着拓展名，c、.cc、.cpp等

### 4.2 什么是构造函数和析构函数
### 4.3何为mutable(可变)和const(不变)
- const
> const修饰符紧接宇函数参数列表之后。凡是在class主体以外定义的，如果它是一个const member function，那就必须同时在声明与定义中指定const
```
int Triangular::elem(int pos) const
{
return _elems[pos - 1];
}
```
> 设计class时，鉴定其const member function是一件十分重要的事情  
没有一个const reference class参数可以调用公开接口中的non-const成分（但目前许多编译器对此情况都只给出警告）

- Mutable Data Member(可变的数据成员)
对于一个含有指针的数据结构，如果数据结构内部的元素没有发生任何改变，
但只改变了指针的指向，从这种意义上来说，不能视为改变class object的状态，或者说不算是破坏了对象的常量性（constness）  
关键字mutable可以让我们做出这样的声明
```
class Triangular{
public:
	bool next(int &val) const;
	void next_reset() const {_next = _beg_pos - 1;}
	//...
	
private:
	mutable int _next;
	int _beg_pos;
	int _length;
};
```
### 4.4 什么是this指针
- this指针是在,member function内用来指向其调用者（一个对象）
> this指向tr1，这是怎么办到的呢？内部工作过程是，编译器自动将this指针加到每一个member function的参数列表中
```
tr1.copy(tr2);
```

### 4.5静态类成员
- static data member用来表示唯一的、可共享的member。它可以在同一类的所有对象中被访问
< 比如利用一个容器来存储Fibonacci数列元素。可以使用static关键词，使得class只需要唯一的容器来存储数列，而不用每一个对象都复制一份  
< 对class而言，static data member只有唯一的一份实体，因此我们必须在程序代码文件中提供其清楚的定义。
这种定义会看起来很像全局对象的定义。唯一的差别是，其名称必须附上class scope运算符
```
vector<int> Triangular::elems;
//也可以指定初值
int Triangular::_initial_size = 8;
```

- Static Member Function(静态成员函数)
> 当一个函数并未访问任何non-static data member时，它的工作就和任何对象都没有关联，因为可以很方便的以一般non-member function的方式来调用
```
if (is_elem(8)) {}  //错误写法
if (Triangular::is_elem(8)) {}  //正确写法
```
> 于是static member function便可以在这种”与任何对象都无瓜葛“的情况下被调用。注意，member function只有在“不访问任何non-static member”的条件
下能够被声明为static，声明方式是在声明之前加上关键字static
```
class Triangular{
public:
	static bool is_elem(int);
	...
}
```
> 当我们在class主体外部进行member function的定义时，无须重复加上关键字static(这个规则也适用于static data member)  
> 以下说明我们应该如何独立使用is_elem()
```
bool is_elem Triangular::is_elem(ival);
...
```

### 4.6 打造一个Iterator Class
### 4.7 合作关系必须建立在友谊的基础上