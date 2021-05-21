# 0. 导读
- explicit
> 被声明为explicit的构造函数通常比non-explicit更受欢迎，因为它禁止编译器执行非预期的类型转换。但它们仍可以被用来进行显式类型转换  
除非我有一个好的理由允许构造函数被用于隐式类型转换，否则我会把它声明为explicit
```
class A
{
public:
  explicit A();
}
```
- pass-by-value（值传递）意味调用copy构造函数；但以by value传递用户自定义类型通常是个坏主意，pass-by-reference-to-const往往是比较好的选择

# 1. 让自己习惯C++
## 条款01：视C++为一个语言联邦
## 条款02：尽量以const,enum,inline替换#define
- 对于单纯常量，最好以const对象或enums替换#define
- 对于形似函数的宏，最好改用inline函数替换#define
> 有了consts、enums和inlines，我们对预处理器（特别是#define）的需求降低了，但并非完全消除  
#define仍然是必需品，而#ifdef/#ifndef也继续扮演控制编译的重要角色。目前还不到预处理器全面引退的时候，但你应该尽量少用它
## 条款03：尽可能使用const
- 除非有需要改动参数或local对象，否则请将它们声明为const
- const位置不同表示的含义不同
1. 如果关键字const出现在星号左边，表示被指物是常量
2. 如果出现在星号右边，表示指针自身是常量
3. 如果出现在星号两边，表示被指物和指针两者都是常量
```
cgar greeting[] = "Hello";
const char* p = greeting;  //non-const pointer,const data
char* const p = greeting;  //const pointer,non-const data
const char* const p = greeting;  //const pointer,const data
```
- STL迭代器中的const
> STL迭代器是以指针为根据塑造出来的，所以迭代器的作用就像个T\*指针。声明迭代器为const就像申明指针为const一样
（即声明一个T\* const指针），表示这个迭代器不能更改指向，但它所指的对象的值是可以改动的。
如果你希望迭代器所指的对象不可被改动，你需要的是const_iterator
```
std::vector<int> vec;
...
const std::vector<int>iterator iter = vec.begin();  //iter的作用就像T* const指针
*iter = 10;  //没问题
++iter;  //错误，iter是const

std::vector<int>const_iterator cIter = vec.begin();
*cIter = 10;  //错误，*cIter是const
++cIter;  //没问题
```
- const成员函数
```
std::size_t length() const;  //定义一个const成员函数
```
> 当需要在const成员函数中实现赋值等修改操作时，编译器会报错  
解决办法是：在允许修改的变量前面，加上mutable
```
class CTextBlock{
public:
  ...
private:
  mutable std::size_t textLength;  //加了mutable的成员变量总是可以被更改，即使在const成员函数内
  ...
}
```

- 当const和non-const成员函数要实现的功能完全一样，为了避免撰写重复代码，可以令non-const版本调用const版本；但是令non-const调用const成员函数会报错
```
class TextBlock{
public:
  ...
  const char& operator[](std::size_t position) const
  {
    ...
    return text[position];
  }
  char& operator[](std::size_t position)
  {
    return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
    //先通过static_cast将non-const转换成const,然后将返回结果通过const_cast转换成non-const;
  }
}
```
- [C++显式转换运算static_cast/const_cast/reinterpret_cast/dynamic_cast](http://yaoyao.codes/c++/2015/03/18/cpp-explicit-cast-operator-static_cast-const_cast-reinterpret_cast-dynamic_cast)

## 条款04：确定对象被使用前已被初始化
> 对于无任何成员的内置类型，必须手工完成初始化  
至于内置类型以外的任何其他东西，初始化责任落在构造函数身上  
规则很简单：确保每一个构造函数都将对象的每一个成员初始化
```
class ABEntry{
public:
  ABEntry(const string& name,const string& address,const list<PhoneNumber>& phones);
private:
  string theName;
  string theAddress;
  list<PhoneNumber>& thePhones;
  int numTimesConsulted;
};

ABEntry:ABEntry(const string& name,const string& address,const list<PhoneNumber>& phones){
  //这些都是赋值，而非初始化
  theName = name;
  theAddress = address;
  thePhones = phones;
  numTimesConsulted = 0;
}
```
- ABEntry构造函数一个较好的写法是：使用所谓的成员初始化列表替换赋值动作
```
ABEntry:ABEntry(const string& name,const string& address,const list<PhoneNumber>& phones)
:theName(name),theAddress(address),thePhones(phones),numTimesConsulted(0)
{}
```
> 这个构造函数和上一个的最终结果相同，但通常效率较高。基于赋值的那个版本首先调用default构造函数为theName,theAddress和thePhones设初值，
然后立刻再对它们赋予新值。也因此default构造函数的一切作为被浪费掉了  
对大对数类型而言，比起先调用default构造函数然后再调用copy assignment操作符，单只调用一次copy构造函数是比较高效的，有时甚至高效得多。
对于内置对象如numTimesConsulted，其初始化和赋值的成本相同，但为了一致性最好也通过成员初始化列表来进行初始化
- 为避免跨编译单元之初始化次序问题，可以以local static对象替换non-local static对象
```
//具体做法
Class FileSystem {...};
FileSystem& tfs()
{
  static FileSystem fs; //定义并初始化一个local static对象
  return fs;  //返回它
}
```

# 2. 构造/析构/赋值运算
## 条款05：了解C++默默编写并调用哪些函数
- 如果定义一个空类，编译器可以暗自为该class创建default构造函数、copy构造函数、copy assignment构造函数、析构函数

## 条款06：若不想使用编辑器自动生成的函数，就该明确拒绝
- 为驳回编译器自动提供的功能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法

## 条款07：为多态基类声明virtual析构函数
- 带多态性质的基类应该声明一个virtual析构函数。如果该类带有任何的virtual函数，它就应该拥有一个virtual析构函数
> 当派生类对象经由基类指针删除时，而该基类带着一个non-virtual析构函数，实际执行时通常发生的是对象的派生类成分没被销毁，即资源泄露
- Classes的设计目的如果不是作为基类使用，或不是为了具备多态性，就不该声明virtual析构函数，不然会导致如下问题
  1.额外开销
  2.程序不再具备移植性
  
 - 如果一个类带有pure virtual函数，那么该类为抽象类——即不能被实体化的类
 > 如果想构造一个抽象类，但手上没有任何纯虚函数时，可以声明一个pure virtual析构函数
 
 ## 条款08：别让异常逃离析构函数
 - 析构函数绝对不要吐出异常。如果一个呗析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们（不传播）or结束程序
 - 如果客户需要对某个操作运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作
 - abort函数和exit函数
 > abort函数
 >> 功能: 异常终止一个进程  
 说明：abort函数是一个比较严重的函数，当调用它时，会导致程序异常终止，而不会进行一些常规的清除工作，比如释放内存等。
 
  > exit函数
 >> 功能：正常终结目前进程的执行，并把参数status返回给父进程，而进程所有的缓冲区数据会自动写回并关闭未关闭的文件。  
说明：它并不像abort那样不做任何清理工作就退出，而是在完成所有的清理工作后才退出程序。

## 条款09：绝不在构造和析构过程中调用virtual函数
- 在执行子类的构造函数期间，会先执行基类的构造函数，但在基类构造期间，虚函数不会下降到子类阶层，即在基类构造期间，虚函数不是虚函数
> 由于基类构造函数的执行更早于子类构造函数，当基类构造函数执行时，子类的成员变量尚未初始化，如果此期间调用的虚函数下降至子类阶层，
要知道子类的函数几乎必然取用local成员变量，而那些成员变量尚未初始化，因此会产生问题
- 相同的问题也适用于析构函数
- 解决的办法是在构造和析构的过程中不要调用虚函数。可以借由令子类将必要的构造信息向上传递至基类构造函数以解决该问题

## 条款10：令operator= 返回一个reference to \*this

## 条款11：在operator= 中处理“自我赋值”
- 如果一个函数中操作一个以上的对象，而其中多个对象是同一个对象时，确保该函数的行为不会出现问题
- 解决以上的办法
> 1.比较“来源对象”与“目标对象”是否一致  
> 2.将delete尽量放到最后
> 3.copy-and-swap

## 条款12：复制对象时勿忘其每一个成员变量
- Copying函数应该确保复制“对象内的所有成员变量”及“所有base class成分”
> 任何时候只要你承担起“为派生类撰写copying函数”的重大责任，必须很小心地也复制其父类成分。那些成分往往是private，
所以你无法直接访问它们，你应该让派生类的copying函数调用相应的父类函数
```
PirorityCustomer::PirorityCustomer(const PirorityCustomer& rhs):Customer(rhs),priority(rhs.priority)
{
  ...
}

PirorityCustomer& PirorityCustomer::operator=(const PirorityCustomer& rhs)
{
  ...
  Customer::operator=(rhs);
  priority = rhs.priority;
  return *this;
}
```
- 不要尝试以某个copying函数实现另一个copying函数
> 如果你发现你的copy构造函数和copy assignment函数有相近的代码，消除重复代码的正确做法是，建立一个新的成员函数给两者调用。
这样的函数往往是private而且常被命名为init。这个策略可以安全消除copy构造函数和copy assignment函数之间的代码重复

# 3. 资源管理

## 条款13：以对象管理资源
- RAII(Resource Acqusition Is Initialization):资源取得时机便是初始化时机
- 为了确保资源总是被释放，我们需要将资源放进对象内，这样当控制流离开时，该对象的析构函数会自动释放那些资源（C++析构函数自动调用机制可以确保资源被释放）
```
void f()
{
  investment* pInv = createInvestment();
  ... //如果中途出现return或异常等发生，会导致delete被略过去，泄露的不只是内涵投资对象的那块内存，还包括哪些投资对象所保存的任何资源
  delete pInv;
}
```
- 标准库提供的auto_ptr智能指针正式针对以上问题而设计的；auto_ptr是个“类指针（pointer-like）的对象”，其析构函数自动对其所指对象调用delete
```
void f()
{
  std::auto_ptr<investment> pInv(createInvestment());
  ...
}
```
> 由于auto_ptr被销毁时会自动删除它所指之物，所以一定要注意别让多个auto_ptr同时指向同一对象
- auto_ptr有一个不寻常的性质
> 若ptr_1通过copy构造函数or copy assignment构造函数复制它，那么ptr_1会变成null，而赋值所得的指针ptr_2将取得资源的唯一控制权
```
std::auto_ptr<investment> pInv_1(createInvestment());
std::auto_ptr<investment> pInv_2(pInv_1); //pInv_2指向对象，pInv_1被设位null
pInv_1 = pInv_2;  //pInv_1指向对象，pInv_2设为null
```
> 以上行为导致auto_ptr并非管理动态分配的利器，因为STL容器要求其元素发挥正常的复制行为，而auto_ptr并不支持 
- auto_ptr的替代方案是“引用计数型智慧指针RCSP”
> RCSP也是个智能指针，持续追踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源  
RSCPs提供的行为类似垃圾回收，不同的是RCSPs无法打破环装引用，例如两个其实已经没被使用的对象彼此互指，因而好像还处在“被使用”状态  
由于tr1::shared_ptrs的复制行为一如预期，它们可被用于STL容器以及其他auto_ptr不适用的场合
```
void f()
{
  ...
  std::tr1::shared_ptr<investment> pInv_1(createInvestment());
  std::tr1::shared_ptr<investment> pInv_2(pInv_1); //pInv_1和pInv_2指向同一个对象
  pInv_1 = pInv_2;  //同上
  ... //pInv_1和pInv_2被销毁，它们所指的对象也被自动销毁
}
```
- auto_ptr和tr1_shared_ptr两者都在其析构函数内做delete而不是delete\[]动作，因此想要解决此情况可以用boost::scoped_array和boost::shared_array classes
- 最后，不要忘记了对于createInvestment返回的未加工指针的delete，调用者极易忽略这个指针。后面有条款针对此进行调整
- 以对象管理资源的两个重要思想
> 1.获得资源后立刻放进管理对象内  
> 2.管理对象运用析构函数确保资源被释放

## 条款14：在资源管理类中小心coping行为
- 当一个RAII对象被复制时,可能出现不符合预期的行为，为了避免这种情况，通常有以下方式解决
> 1. 禁止复制：如果复制动作对RAIIclass并不合理，你应该禁止它。禁止的方法是将copying操作声明为private并继承  
> 2. 对底层资源使用“引用计数法”。这种情况下复制RAII对象时，应该将资源的“被引用数”递增，tr1::shared_ptr便是如此  
>> 其中为了防止在使用tr1::shared_ptr的时候出现“当引用次数为0时删除其所指物（比如我们需要的只是释放锁，而不是删除）”，
tr1::shared_ptr允许指定所谓的“删除器”，将删除器作为第二参数即可  
>> 同时以上可能不需要再声明析构函数。因为当引用次数为0时会自动调用指定的删除器  

> 3.复制底部资源（深拷贝）  
> 4. 转移底部资源的拥有权，比如auto_ptr的功能
- copying函数有可能被编译器自动创建出来，因此除非它创建的正是你想要的，否则你得自己编写copying  

## 条款15：在资源管理类中提供对原始资源的访问
- APIs往往要求访问原始资源，所以每一个RAII class应该提供一个“取得其所管理的资源”的方法  
- 对原始资源的访问可能经由显示转换or隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便  
- 该条款中实际提供了很多个显式/隐式转换的方法，该选择哪一种视情况而定  
- 一般而言，设计一个显式转换函数如get()来获取原始资源是一个比较推荐的方法，因为它将“非故意类型转换”的可能性降到最低  

## 条款16：成对使用new和delete时要采取相同形式
- 如果你调用new时使用\[]，那么必须在对应调用delete\[]；如果你调用new时没有使用\[]，那么就不该在对应调用delete时使用\[]  
- 为了避免delete错误，最好尽量不要对数组形式命别名
```
using arr_ = arr[5];  //如何delete？
```
## 条款17：以独立语句将newed对象置入智能指针
- 以独立语句将newed对象置入智能指针内，如果不这么做，一旦异常被抛出，有可能导致难以察觉的资源泄露  
```
processWidget(std::tr1::shared_ptr<Widget>(new Widget),priority());
```
> Java和C#这两种语言总是以特定次序完成函数参数的核算。但C++不同，C++编译器对于语句执行顺序有很大的弹性  
new一定执行在tr1::shared_ptr构造函数之前，但对于priority()的调用可以在任何时候，如果编译器以下面顺序执行  
>> 1. 执行new Widget  
>> 2. 调用priority()  
>> 3. 调用shared_ptr构造函数  

> 如果在priority()调用过程中发生异常，则会引发资源泄露，所以一定要将new语句独立出来

# 4. 设计与声明

## 条款18：让接口容易被正确使用，不易被误用
- 对于处理具有一定特性的数据 
> 1. 要处理特定的数据，使用成熟且经充分锻炼的classes并封装其内数据，比简单的使用structs或enums要好  
> 2. 对于该数据的操作和限制，封装在该classes里面，能预防客户的错误  
> 3. 阻止客户错误使用的方法包括：新建新类型、限制类型上的操作、束缚对象值、以及消除客户的资源管理责任
- 令types与内置types一致
> 1. 除非有好理由，否则应该尽量令types的行为与内置types一致，如果不确定，可以参考内置types或标准库里面的types  
> 2. 避免无端与内置类型不兼容，真正的理由是为了提供行为一致的接口。很少有其他性质比得上“一致性”更能导致“接口容易被正确使用”，
任何接口如果要求客户必须记得做某些事情，就是有着“不正确”的倾向，因为客户可能会忘记做那件事  
- 关于tr1::shared_ptr支持定制删除器，这可防范cross-DLL problem，可被用来自动解除互斥锁等等
> cross-DLL problem：对象在动态链接程序库（DLL）中被new创建，却在另一个DLL内被delete销毁；详见P82  

## 条款19：设计class犹如设计type
- 应该带着和“语言设计者当初设计语言内置类型”一样的谨慎来研讨class的设计
- 设计高效的classes必须面对的问题
> 1. 新type的对象应该如何被创建和销毁？  
> 2. 对象的初始化和对象的赋值该有什么样的差别？  
>> 别混淆了初始化和赋值，因为它们对应于不同的函数调用  
> 3. 新type的对象如果被passed by value，意味着什么？  
> 4. 什么是新type的“合法值”？  
> 5. 你的新type需要配合某个集成图系吗？  
>> 如果你允许其他classes继承你的class，那会影响你所声明的函数———尤其是析构函数———是否为virtual  
> 6. 你的新type需要什么样的转换？  
> 7. 什么样的操作符和函数对此新type而言是合理的？  
> 8. 什么样的标准函数应该驳回？  
> 9. 谁该取用新type的成员？  
> 10. 什么是新type的“未声明接口”？  
> 11. 你的新type有多么一般化？  
> 12. 你真的需要一个新type吗？  

## 条款20：宁以pass-by-reference-to-const替换pass-by-value
- pass-by-value
> 值传递会导致函数构造以实参为初值的副本对象，会调用构造函数，同时在函数结束时，会调用析构函数，这肯能使得值传递成为昂贵费时的操作  
- pass-by-reference-to-const
> 这种传递方式的效率高得多：没有任何构造函数or析构函数被调用，因为没有任何新对象被创建。同时将原始对象传到函数中时，为了防止该对象在函数中被改变，
将它声明为const是非常重要的  
> 以by-reference方式传递参数也可以避免对象切割问题  
>> 当一个子类对象以by value方式传递并被视为一个父类对象时，父类对象的构造函数会被调用，而“造成此对象的行为像个子类对象”的那些特性全被切割掉了，
仅仅留下一个父类对象，但这几乎绝不会是你想要的  
- 一般而言，你可以合理假设，“pass-by-value并不昂贵”的唯一对象就是内置类型、STL的迭代器和函数对象；至于其他，尽量以pass-by-reference-to-const替换pass-by-value  

## 条款21：必须返回对象时，别妄想返回其reference
- 任何时候看到一个reference声明式，你都应该立刻问自己，它的另一个名称是什么？因为它一定是某物的另一个名称  
- 如果函数要返回一个reference指向一个对象，它必须要自己创建那个对象
> 函数创建新对象的途径如下  
>> 1. 在stack空间上创建（通过构造函数），这个函数返回一个reference指向某一个对象，但该对象是local对象，而local对象在函数退出前会被销毁掉  
>> 2. 在heap上构造一个对象（通过new），但是没有合理的办法让使用者进行delete调用，这绝对会导致内存泄露  
>> 3. 让返回的reference指向一个呗定义于函数内部的static对象，这既会造成我们对多线程安全性的疑虑，也会产生更深层的问题（具体见P93）
> 一个“必须返回新对象”的函数的正确写法是：就让那个函数返回一个新对象呗，尽管那得承受构造和析构成本，然而长远来看那只是为了获得正确行为而付出的一点点小小的代价  
- 总结就是：当函数最终要返回的结果必须保存在一个新的对象中时（不需要返回原本传入的对象），那就返回一个对象而不是reference  

## 条款22：将成员变量声明为private  
- 理由如下
> 1. 客户使用的一致性  
>> 1.1 访问成员变量的唯一方法就是通过成员函数  
>> 1.2 使用函数可以让你对成员变量的处理有更精确的控制。如果你令成员变量为public，每个人都可以读写它；但如果你以函数取得或设定其值，
你就可以实现“不准访问”、“只读访问”、“读写访问”等  
> 2. 封装性  
>> 如果你通过函数访问成员变量，日后对成员变量进行调整，而class客户一点也不会知道内部实现已经起了变化  
>> 将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性  
- protected并不比public更具封装性  
- 其实只有两种访问权限：private（提供封装）和其他（不提供封装）  

## 条款23：宁以non-member non-friend替换member函数  
- 面向对象守则要求数据应该尽可能被封装，然而与直观相反的是，member函数带来的封装性比non-member non-friend函数低  
> 衡量封装性的粗略标准：有多少代码访问到class内的private成分  
越多代码可访问它，数据的封装性就越低；因此导致较大封装性的是non-member non-friend函数，因为它并不增加能够访问到class内private成分的代码  
- 在上一点上有两件事情值得注意  
> 1. 这个论述只适用于non-member non-friend函数。friends函数对class private成员的访问权利和member函数相同，因此两者对封装的冲击力道也相同  
> 2. 这里的选择关键并不在member和non-member函数之间，而是在member和non-member non-friend函数之间  
> 3. 只因在意封装性而让函数“成为该class的non-member”并不意味它不可以是另一个class的member；只要它不是该class的一部分，就不会影响到该class的private成员的封装性  
- 在C++中，比较自然的做法是，让该函数成为一个non-member non-friend函数并且位于与该class所在的同一个namespcace内
```
namespace WebBrowserStuff{
  class WebBrowser {...}; //class内的member函数尽可能只是满足访问private的基本功能函数，剩下的复杂功能交给便利函数
  void clearBrowser(WebBrowser& wb);  //这是一个提供便利的函数
}
```
- C++标准程序库的组织方式
> 将所有满足某个机能的便利函数放在多个头文件内但隶属于同一个命名空间，意味客户可以轻松扩展这一组便利函数。他们需要做的就是添加更多的non-member non-friend函数到此命名空间  
> class定义式对客户而言是不能拓展的。客户可以派生出新的classes，但子类无法访问父类中被封装的private成员，于是如此的“拓展机能”没有上面更实用  

## 条款24：若所有参数皆需要类型转换，请为此采用non-member函数
- 令classes支持隐式类型转换通常是个糟糕的主意  
- 但如果需要为某个函数的所有参数（包括被this指针所指的那个隐藏参数）进行类型转换，那么这个函数必须是个non-member，否则会报错  
- member函数的反面是non-member函数，不是friend函数
> 无论何时，如果可以避免friend函数就该避免，因为就想真实世界一样，朋友带来的麻烦往往多过其价值  
> 当然有时候friend有其正当性，但这个事实依然存在：不能够只因为函数不该成为member，就自动让它成为friend  

## 条款25：考虑写一个不抛异常的swap函数（不是很懂，以后再看看P106）
- 关于swap的程序设计思路
> 1. 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛异常  
> 2. 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes（而非templates），也请特化std::swap  
> 3. 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”  
> 4. 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西  

# 5. 实现


