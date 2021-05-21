# 算法题笔记
- 字符串的题目，可以考虑创建一个vector<int>vec(128)的数组来代替hash_map，因为字符的ASCI在0~128之间
  
- 利用滑动窗口，可以在O(n)时间复杂度内解决子串、子数组子序列匹配问题。例leetcode 76题
- 买卖股票系列要有一个总结

- 当题目中有乘法或加法的时候，要考虑溢出！！这种情况尽量别用乘法或加法，用其他表达式替换一下。

- 对于需要用递归进行遍历的题目，能用循环用循环，实在不行再用递归，递归用的越少，时间复杂度越低——尤其是N皇后问题

- 遍历二维数组，如果需要重新创建一个二维数组来记录是否已经访问过该节点————有一个更好的方法，完全不需要新建一个二维数组
```CPP
//用一个变量来保存当前dp[x][y]的值，然后将dp[x][y]的值设置为'#'，递归的时候，如果遇到dp[x][y] = '#'则直接跳过，递归完成后再将dp[x][y]的值还原
char c = dp[x][y];
dp[x][y] = '#';
helper(...);
dp[x][y] = c;
```
- 定义一个递增的数组
```CPP
vector<int> vec(5);
iota(vec.begin(),vec.end(),1);  //vec：1,2,3,4,5
```

- 删除数组重复元素：
```CPP
ForwardIterator unique ( ForwardIterator first, ForwardIterator last );
```
>返回的iter为无重复数组的最后一个元素的位置+1。

- 返回iterators之间的距离： 
```CPP
distance (InputIterator first, InputIterator last);
```

- 判断是否为空，直接用
```CPP
if(xxx.empty()) return 0;
```
- 当遇到要处理两个字符串，两个字符串长度不同，且影响后续操作：
> 一是可以将短的字符串补成跟长的字符串一样，补的部分可以是0或者其他，按题目要求  
二是两个字符串一起遍历，当短的遍历完时，继续返回补的结果，直到长的也遍历完毕
```CPP
//例如：
while(m >= 0 || n >= 0){
  int p = (m >= 0) ? (a[m--] - '0') : 0;
  int q = (n >= 0) ? (b[n--] - '0') : 0;
}
```

- to_string函数：将数值转化为字符串，返回对应的字符串形式
```CPP
string to_string (int val);
string to_string (long val);
string to_string (long long val);
string to_string (unsigned val);
string to_string (unsigned long val);
string to_string (unsigned long long val);
string to_string (float val);
string to_string (double val);
string to_string (long double val);
```

- sub_str函数：返回一个新建的初始化为string对象的子串的拷贝string对象
```CPP
string sub1 = s.substr(5);  //只有一个数字5表示从下标为5开始一直到结尾：sub1 = "56789"
string sub2 = s.substr(5, 3);  //从下标为5开始截取长度为3位：sub2 = "567"
```

- 多维数组初始化的方式
```CPP
int dp[n][n] = {0};  //初始化二维数组，且各位置元素全为0
```
> 【注】 int型的二维数组访问执行速度完爆std的vector，所以在刷算法题的时候尽可能的还是用最原始的数据类型  

- 字符串匹配算法：[马拉车算法（ Manacher‘s Algorithm ）](https://www.cxyxiaowu.com/2665.html)、[KMP算法](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247485939&idx=1&sn=b25f39b5644da92c4047bbbd9936f73c&chksm=fa0e6672cd79ef64dda0a21e23c2817edf4a64cbb75b9bed328d6519c6cd4fef36d03a4cb309&scene=21#wechat_redirect)  

- 访问数组或vector的元素
```CPP
vec[0];  //访问第一个元素
vec.back();  //访问最后一个元素
```

- 查看字符串str中第一个与某个s(char)不相同的字符的iter
```CPP
find_if(str.end(),str.end(),bind1st(not_equal_to<char>(),*s));
```

- 查看子串的长度
```CPP
distance(str.begin(),str.end());
```

- 字符串的拼接，使用stringstream更高效

- 查看字符串从右向左第一个字母
```CPP
find_if(s.rbegin(),s.rend(),::isalpha);
//同理，数字::isdigit;  字母or数字::isalnum;
```

- 查看字符串从右向左第一个非字母
```CPP
find_if_not(s.rbegin(),s.rend(),::isalpha);
```

- 对于多维数组的题目
> 如果想要将空间复杂度降到最低（常数复杂度），可以考虑将相应的信息存储在第一行or第一列。同时第一行or第一列的信息存储成标记变量flag

- 将数组i-j位置上的值设置为0
```CPP
fill(&arr[i], &arr[j], 0);
fill(&arr[i],j - i,0);
```

- 容器复制
> 在给vector复制之前，尽量给vector指定长度才行，不然leetcode后台会报错
```CPP
copy_n(arr.begin(),k,res.begin());
```

- 数组题目，可以考虑从左到右、从右到左两次遍历数组，说不定能实现题目要求的某些性质

- 一维动态规划消耗的空间一般都可以简化到常数级，因为每次循环的结果都只与前面几个结果相关

- 求两个数的最大公约数
```CPP
gcd(x,y);
```

- 字符串比较
```CPP
strcmp(str1,str2);
```
> strcmp函数是用来比较2个字符串的函数，从第一个字符开始比较，如果到最后两个字符串完全相同，则strcmp（）函数输出的值为0；若开始出现不同的字符，根据这个字符ASCII码进行比较，若字符串1的ASSCII值大于2，则输出值大于0；反之输出值小于 0；

```CPP
strncmp(str1,str2,n);
```
> 原理同上，只不过比较的是前n个字符

- extern
```CPP
extern int a;
```
> 表明int类型的变量a（也可以是函数）定义在其他的文件中，引入的方式是用include  

- 显示错误信息：perror(3)
> perror(string)这个函数，它会自动查找错误代码，在标准错误输入中显示出相应的错误信息，参数string时要同时显示出描述性信息
```CPP
perror("Cannot open file");
报错信息：Cannot open file: Not such file or directory
```
> 显示的第一部分是用户传递进去的描述性信息，第二部分是根据错误代码查找到的错误提示

- 链表逆转的标准递归代码
```CPP
ListNode* reverseList(ListNode* head) {
        if(!head || !head->next) return head;
        ListNode *p = reverseList(head->next);
        head->next->next = head;
        head->next = NULL;
        return p;  //节点p为逆转后的头结点，如果不需要该节点，可以删掉返回值
    }
```
- 链表逆转的递推思路
> 用头插法的思路
>> pre指向逆转的前一个节点（不用变化该指针）  
cur指向逆转的首节点（不用变化该指针）  
late指向需要头插法的指针（每次递推，初始化为 late = cur->next，只用变化该指针即可完成逆转操作）

- 链表的题目中，如果需要修改链表，则一定要定义一个新的节点pre_head，且pre_head->next = head；并保证不会改动pre_head，最终返回pre_head->next；其他操作的节点另外定义

- 对于链表划分的题目，可以将满足A要求的节点保留在原始链表中，满足B要求的节点单独取出来组成一新的B链表，最后将A、B链表合并
- 对于链表递归删除不符合要求的节点
```CPP
//典型例子见leetcode82题
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (!head || !head->next) return head;  //退出递归的边界条件
        head->next = deleteDuplicates(head->next);  //连接头结点和当前递归节点
        return (删除条件) ? head->next : head;  //符合删除条件，就删除头结点，保存头结点下一个节点，否则两个节点都保存
    }
};
```

- 链表的复制，可以将新节点复制在原始节点后面，复制完以后，再将复制后的新节点一一摘出来，这样就形成新的链表，而且时间复杂度最低

- 如果要做乘2或乘4的操作，最好用位操作符，效率更高
```CPP
tmp * 4 相当于 tmp << 2;
```
- 若还有位运算，最好加上括号
```CPP
int tmp = 1;
tmp << 2 + 2  //结果为16
(tmp << 2) + 2  //结果为6
```

- 对于一些空间题or找规律的题目，如果整体上没有思路，可以从一个最小单元上下手

- 链表的逆转，最快、最简单的方法就是用栈。但是会牺牲空间
```CPP
//stack的使用
empty() 堆栈为空则返回真
pop() 移除栈顶元素
push() 在栈顶增加元素
size() 返回栈中元素数目
top() 返回栈顶元素
```
- 如果题目明确要求返回int类型，就不要用size_t类型，这样混着用会报错
- 如果在遍历过程中，数组可能会越界，可以用下面这种形式
```CPP
dp[i] = i - 2 >= 0 ? dp[i - 2] + 2: 2;
```
- 动态规划的题目，把各个可能的分支结果想清楚，把动态方程写出来就可以
  1.起始条件
  2.动态方程
  3.结束条件
  
- 矩阵题目，要查看中心点周围的8个点时，考虑用以下方式
```CPP
vector<int> dx = {-1,-1,-1,0,0,1,1,1};
vector<int> dy = {-1,0,1,-1,1,-1,0,1};
for(int k = 0;k < 8;++k){
  int x = i + dx[k];
  int y = j + dy[k];
  ...
}
```
- 数组题，要找前后关系时，可以分别考虑3种初始情况： 以i为左侧，以i为中心，以i为结束！
- 解决木桶接水问题，使用递增栈or递减栈，详见leetcode 84、42题

- string转其他类型
```CPP
std::string str;
int i = std::stoi(str); //转int
long j = std::stol(str);  //转long
double k = std::stod(str);  //转double
```
- 树的深度优先遍历（中后序遍历）
> 1.递归——正常递归，最简单——时复O(n)，空复O(n)  
> 2.迭代——使用栈stack——时复O(n)，空复O(n)  
> 3.迭代——线索二叉树，非常巧妙——时复O(n)，空复O(1)——推荐这种写法来遍历树！
>> 1. [Morris Traversal方法遍历二叉树（非递归，不用栈，O(1)空间）](https://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html)  
>> 2. [[LeetCode] 94. Binary Tree Inorder Traversal 二叉树的中序遍历](https://www.cnblogs.com/grandyang/p/4297300.html)  
- 树的广度优先遍历（层序遍历）
> 1.递归——正常递归，递归的过程中，要传递depth参数——时复O(n)，空复O(n)  
> 2.递推——使用队列queue，简单；或稍微麻烦一点，使用两个栈stack——时复O(n)，空复O(n)  

- 矩阵顺时针旋转90度
> trick方法，时复O(n^2)，空复O(1)  
>> 1. 先按行上下翻转，再沿着左上-右下对角线交换元素即可  
>> 2. 先按照从左下-右上交换元素，再按行上下翻转即可  
>> 3. 先按左上-右下交换元素，再按列左右翻转即可 

> 常规方法，时复O(n^2)，空复O(1)  
>> 按顺时针的顺序去覆盖前面的数字，从四个顶角开始，然后往中间去遍历，每次覆盖的坐标都是同理  
```CPP
(i, j)  <-  (n-1-j, i)  <-  (n-1-i, n-1-j)  <-  (j, n-1-i)
```
- 广度优先遍历（层次遍历）时，如果要按层存储信息，根本不需要做额外操作，每层只用这样循环就行
```CPP
while(!que.empty()){
  for(size_t i = que.size();i > 0;--i){
    TreeNode* p = que.front();que.pop();
    ...
  }
}
```
- 字符串去掉首尾指定字符
```CPP
//去掉首尾空格
s.erase(0,s.find_first_not_of(" "));
s.erase(s.find_last_not_of(" ") + 1);
```
- 字符串去掉首尾、中间多余空格（word之间只保留一个）
```CPP
istringstream is(s);  //将字符串存储在string流中
string tmp; 
is >> s;  //将第一个单词保存在s中  
while(is >> tmp) s += tmp + " ";  //将后续单词依次保存在s中
if(!s.empty() && s[0] == ' ') s = ""; //如果s全为空格，则将s置位空
}

//使用getline也可
istringstream is(s);
s = "";
string t = "";
while (getline(is, t, ' ')) {
    if (t.empty()) continue;
    s = (s.empty() ? t : (s + " " + t));
}
```
- 需要重建树分支之间的关系——参考leetcode114题
> 1. 为了在遍历的过程中不破坏原本节点间的关系，方便后面遍历过程，最好采用后序遍历 + 重建节点关系  
> 2. 如果要建立节点及前一个节点的关系，可以在递归的过程中传递两个节点
```CPP
pre = helper(root->left,pre);
pre = helper(root->right,pre);
```
> 3. 非递归思路：找到前一个节点，一般为root的左孩子的最右节点pre，然后处理root与pre的关系即可  

- pair的使用
```CPP
vector<pair<int, int>> vec;
vec.push_back(make_pair(1,2));
```

- 求两数之和
> 这种题最典型的地方在于整数类型存储会越界，因此只能边计算边存储  
> 如果要从大往小计算时，要记录val = 9的节点位置

- 链表逆转——最简单的方法还是使用stack，但是会牺牲空间

- 树的递归中，如果希望记录某个节点，但不以返回值的形式，可以传节点的引用作为参数
```CPP
bool helper(TreeNode* cur,TreeNode*& pre){...}  //希望用到pre，但不以返回值的形式
```

- 广度优先遍历一定要用queue，别用stack

- 如果需要根据矩阵中点(x,y)周围的坐标来更新该点，一个比较巧妙的做法是——两次遍历
> 第一次遍历从左上角往右下角，相当于从左上这个方向来整体更新矩阵  
> 第二次遍历从右下角王左上角，相当于从右下这个方向来整体更新矩阵  
> 两次更新就能将所有方向的所有坐标都更新完毕

- 使用unique或unique_copy函数的时候，需要先排序，才能去重

- 写代码时，如果遇到指针or引用，一定要看底层是否为const，如果为const则要加上const，否则会报错

- split函数（自写）
复杂版本：  
```CPP
char c = ' ';
vector<string> split(const string& str, const char c){
  vector<string> vec;
  for(auto it = str.begin();it != str.end();++it){
      if(*it == c) continue;
      auto it_end = find(it,str.end(),c);
      vec.push_back(string(it,it_end));
      it = it_end;
  }
  return vec;
}


```
简单版本:  
```CPP
void split(const string& target,char t,vector<string>& ans){
    string str = target;
    while(!str.empty()){
        string sub_s = str.substr(0,str.find(t)); //使用string.find()函数找到对应字符的在string的pos
        str = str.substr(str.find(t) + 1);  //更新str，切掉str前一个分割的substr部分
        ans.push_back(sub_s); //将得到的子串sub_s加到结果数组中
    }
}
```

### string.find()函数与find(string)函数
#### string.find()系列函数，返回结果为元素下标

```CPP
string s = "hello,world!";

//string.find(c)
size_t pos = s.find(','); //返回','在s中的下标位置

//string.find(c，5)
size_t pos = s.find(','); //从下标5开始查找，d返回','在s中的下标位置

//string.rfind(c)
size_t pos = s.rfind(','); //从右向左查找，返回','在s中的下标位置

//string.find_first_of(c)
size_t pos = s.find_first_of(','); //返回','在s中的第一次出现的下标位置

//string.find_first_not_of(c)
size_t pos = s.find_first_not_of(','); //返回','在s中查找第一个与‘,’不匹配的字符的下标位置

//string.find_last_of(c)
size_t pos = s.find_last_of(','); //返回','在s中的最后一次出现的下标位置

//string.find_last_not_of(c)
size_t pos = s.find_last_not_of(','); //返回','在s中查找最后一个与‘,’不匹配的字符的下标位置
```
#### find(string)函数，返回结果为iterator
```CPP
#include<algorithm>

string s = "hello,world!";

//find()
string_iterator iter = find(s.begin(),s.end(),','); //返回一个iterator，指向该元素。如果没有任何相符的元素，就返回容器的end()

//find_end()
sub_s = "world";
string_iterator iter = find(s.begin(),s.end(),sub_s.begin(),sub_s.end()); //返回某个子序列的最后一次出现地点

//find_first_of()
sub_s = "world";
string_iterator iter = find_first_of(s.begin(),s.end(),sub_s.begin(),sub_s.end()); //返回某个子序列的第一次出现地点
```


- 单链表实现快排
```CPP
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        quick_sort(head,NULL);
        return head;
    }

private:
    void quick_sort(ListNode* head,ListNode* tail){
        if(head == tail) return;
        ListNode* pos = helper(head,tail);
        quick_sort(head,pos);
        quick_sort(pos->next,tail);
    }

    ListNode* helper(ListNode* head,ListNode* tail){
        ListNode *p = head;
        ListNode *q = p->next;
        int tmp = p->val;
        while(q != tail){
            if(q->val < tmp){
                p = p->next;
                swap(p->val,q->val);
            }
            q = q->next;
        }
        swap(head->val,p->val);
        return p;
    }
};
```
### 面试题之strcpy/strlen/strcat/strcmp的实现
- [面试题之strcpy/strlen/strcat/strcmp的实现](https://songlee24.github.io/2015/03/15/string-operating-function/)

- 单链表实现归并排序
```CPP
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head || !head->next) return head;
        ListNode *pre,*slow,*fast;
        pre = slow = fast = head;
        while(fast && fast->next){
            pre = slow;
            slow = slow->next;
            fast = fast->next->next;
        }
        pre->next = NULL;
        return merge(sortList(head),sortList(slow));
    }

private:
    ListNode* merge(ListNode* l1,ListNode* l2){
        ListNode* pre_head = new ListNode(-1);
        ListNode* cur = pre_head;
        while(l1 && l2){
            if(l1->val < l2->val){
                cur->next = l1;
                l1 = l1->next;
            }else{
                cur->next = l2;
                l2 = l2->next;
            }
            cur = cur->next;
        }
        if(l1) cur->next = l1;
        if(l2) cur->next = l2;
        return pre_head->next;
    }
};
```

- 在函数内部定义数组，数组是不会默认初始化的，因此必须得手动初始化数组
```CPP
int arr[n]; //定义长度为n的数组
memset(arr,0,sizeof(arr));  //将arr数组全部初始化为0
```
> 注意：只能用memset将数组初始化为0或-1，若初始化为其他数字则会产生错误  
> 如果要对数组赋其他数字（例如1），那么请使用fill函数（但memset的执行速度快）
```CPP
fill(arr,arr + n,1);
fill(vec.begin(),vec.end(),1);
fill_n(arr,n,i);
```
- 如果要用下标hash，可以先考虑用数组下标进行hash，如果数组下标hash不能做，再考虑unordered_map，因为map效率比数组低很多  
- 如果一个题目要求满足条件的k个元素有多少种组合，即找含有k个元素的子数组时，可以考虑用一个数组从头到尾记录每个位置含有多少个符合要求的元素，在通过hash求即可  
> 先求取nums中满足条件元素的前缀和arr，即arr\[i]表示nums数组中前i个数有多少个符合条件的元素  
> 对于每一个子数组nums\[i...j]，其中的符合条件的元素个数可以采用arr\[j]-arr\[i-1]求得  
> 则问题转换成有多少个数对(i,j)，其中arr\[j]-arr\[i] =k 这就是我们喜闻乐见的two sum问题，用hash表即可O(n)解决  


- priority_queue：优先队列具有队列的所有特性，包括基本操作，只是在这基础上添加了内部的一个排序，它本质是一个堆
```CPP
//默认为降序队列，即大顶堆
priority_queue <int,vector<int>,greater<int> > q; //升序队列

priority_queue <int,vector<int>,less<int> >q; //降序队列

//greater和less是std实现的两个仿函数（就是使一个类的使用看上去像一个函数。其实现就是类中实现一个operator()，这个类就有了类似函数的行为，就是一个仿函数类了）

//如果只用默认的大顶堆，可以直接定义
 priority_queue<ListNode*> que;
 //如果需要小顶堆，则得传入一个函数指针 or lamda函数
 auto cmp = [](ListNode*& a, ListNode*& b) { return a->val > b->val;};
 priority_queue<ListNode*, vector<ListNode*>, decltype(cmp) > que(cmp); //必须得将cmp当做参数传入，不然会报错
```

- 二分查找——非常容易出错！ 详细讲解[二分查找有几种写法？它们的区别是什么？](https://www.zhihu.com/question/36132386)  
- [LeetCode Binary Search Summary 二分搜索法小结](https://www.cnblogs.com/grandyang/p/6854825.html)
> 1. 第一种写法——左闭合、右闭合区间\[left,right]
```CPP
int searchInsert(vector<int>& nums, int target) {
        int left = 0,right = nums.size() - 1;
        while(left <= right){ //正确的终结条件：也就是搜索空间为空，即对于左闭合、右闭合区间，当left > right时，搜索空间为空
            int mid = left + (right - left) / 2;  //等价于mid = (left + right) / 2，但是加法会溢出
            if(nums[mid] == target) return mid;
            else if(nums[mid] < target) left = mid + 1; //更新左【闭合】区间
            else right = mid - 1; //更新右【闭合】区间
        }
        return left;
    }
```

> 2. 第二种写法——左闭、右开区间\[left,right)
```CPP
int searchInsert(vector<int>& nums, int target) {
        int left = 0,right = nums.size(); //区间为[0,n)
        while(left < right){  //正确的终结条件：也就是搜索空间为空，即对于左闭合、右开区间，当left == right时，搜索空间为空
            int mid = left + (right - left) / 2;
            if(nums[mid] < target) left = mid + 1;  //更新左【闭合】区间
            else right = mid; //更新右【开】区间
        }
        return left;  //循环结束的条件是left == right，因此返回left或right都可
    }
```
- 用二分法找有序数组中target在最左边的位置
```CPP
//上述两种写法都是，或者以下写法
int left = distance(nums,lower_bound(nums,nums + n,target));
```

- 用二分法找有序数组中target在最右边的位置（trick：查找第一个大于taget的值，并返回pos - 1）
```CPP
//上述两种写法，将条件判断改为nums[mid] <= target即可，或下述写法
int right = distance(nums,prev(upper_bound(nums,nums + n,target)));
```
- 根据上述，数组中\[first, last)中与value等价的元素的范围就是：
```CPP
[lower_bound(value), upper_bound(value)); //它们分别是这个区间的(左闭)下界和(右开)上界，因此得名。equal_range(value)的作用是同时返回这两个位置
```

### 动态规划
#### 1. 什么是动态规划
动态规划是一种用来解决一类最优化问题的算法思想。  
简单来说，动态规划将一个复杂的问题分解成若干个子问题，通过综合子问题的最优解来得到原问题的最优解。  
#### 2. 什么情况能使用动态规划
一个问题必须拥有重叠子问题和最优子结构，才能用动态规划去解决。
- 重叠子问题——分治与动态规划的区别：都将问题分解为子问题，然后合并子问题的解得到原问题的解。但是分治过程不出现重叠子问题
> 如果一个问题可以被分解为若干个子问题，且这些子问题会重复出现，那么就称这个问题有用重叠子问题。  
> 动态规划通过记录重叠子问题的解，来使下次碰到相同的子问题时直接使用之前记录的结果，以此避免大量重复计算  
>> 一般可以将复杂度从O(2^n)降低到O(n^2)  
>> 对于路径选择类问题，复杂度的计算方法是一共有n个节点，每个节点可以选择的子路径有m条，那么时间复杂度为n^m  
- 最优子结构——贪心与动态规划的区别：贪心为局部最优，动态规划为全局最优
> 如果一个问题的最优解可以由其子问题的最优解有效的构造出来，那么成这个问题拥有最优子结构。也即状态转移，第i层的状态只与第i+1层的状态有关，而与其他层的状态无关  
- 动态规划的递归和递推  
> 递推（自底向上）：从边界出发，通过状态转移方程扩散到整个数组  
> 递归（自顶向下）：从目标问题开始，将它分解成子问题的组合，直到分解至边界为止

- 最长上升子序列O(nlogn)解法
> 构建一个目标数组，时刻更新数组中的值，使得每个值都保持最小。即在目标数组中找到第一个大于nums\[i]的位置，将该位置上的数更新为nums\[i]，如果不存在大于nums\[i]的数，那么将nums\[i]加到数组最后面  
```CPP
int lengthOfLIS(vector<int>& nums) {
  vector<int> v;
  for (auto a : nums) {
    auto it = lower_bound(v.begin(), v.end(), a);
    if (it == v.end()) v.push_back(a);
    else *it = a;
  }
  return v.size();
}
```
#### 3. 动态规划的思路总结
##### 1.关于序列或字符串的问题（一般来说，“子序列”可以不连续，“子串”必须连续）——推荐枚举长度来进行循环
- 令dp\[i]表示以dp\[i]结尾的xx（单连续序列）
一般这种情况，dp\[i]的状态只与dp\[i-1]有关，表面上空间复杂度为O(n)，但是可以降维到O(1)
- 令dp\[i]\[j]表示A\[i]至A\[j]区间的xx(单非连续序列，双序列)
一般这种情况，dp\[i]\[j]的状态与dp\[i-1]\[0]-dp\[i-1]\[j]的状态都有关，表面上空间复杂度是O(n^2)，但是可以降维到O(n)

##### 2.关于背包问题（多路径选择问题）
- 令dp数组表示为恰好为i、恰好为j（或前j）的xx,接下来可以通过端点的特点去考虑状态转移方程
般这种情况，dp\[i]\[j]的状态与dp\[i-1]\[0]-dp\[i-1]\[j]的状态都有关，表面上空间复杂度是O(n^2)，但是可以降维到O(n)

##### 3. 关于动态规划数组降维的思路
- 降维前，数组更新顺序无所谓，从前往后、从后往前都可以。但是降维后，为了不覆盖后面更新所需要的值，数组应该以特定顺序进行更新，具体的更新顺序最好是画一个示意图！！示意图可以确保不会出错！！  
比如：  
  01背包问题的数组降维需要从右往左更新数组  
  <div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/01-%E7%AE%97%E6%B3%95%E9%A2%98%E7%AC%94%E8%AE%B0/03-image/01.png" width = 70% height = 70% /></div>  
  
  完全背包问题的数组降维需要从左往右更新数组

- 空间复杂度最低的方法：只在原始数组上进行修改和计算

##### 4. 关于多字符串匹配最最简单的办法  

一般来说字符串匹配问题都是更新一个二维dp数组，核心就在于找出状态转移方程  

**技巧：通过将下面的二维数组列出来，找规律，就能找到状态转移方程！！**  

参考leetcode 第115题  

```CPP
  Ø r a b b b i t
Ø 1 1 1 1 1 1 1 1
r 0 1 1 1 1 1 1 1
a 0 0 1 1 1 1 1 1
b 0 0 0 1 2 3 3 3
b 0 0 0 0 1 3 3 3
i 0 0 0 0 0 0 3 3
t 0 0 0 0 0 0 0 3
```
这一类题的核心思路：从左上递推到右下，dp\[n]\[m]的值就是最终动态规划的结果  

- 先求边界初始值：dp[i][0]与dp[0][j]
- 2个for循环（外层i，内层j），进行递推
- 返回dp\[n]\[m]  

```CPP
vector<vector<int>> dp(n + 1, vector<int> (m + 1)); 
dp[0][0] = 1;

for (int i = 1; i <= n; ++i) dp[i][0] = dp[i - 1][0] && (判断条件);
for (int i = 1; i <= m; ++i) dp[0][i] = dp[0][i - 1] && (判断条件);

for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= m; ++j)
        动态转移方程;
        
return dp[n][m];
```
##### 5. 关于找动态转移方程的技巧  
- 只用考虑当前状态和前一个状态的关系，怎么利用动态转移方程从前一个状态推导成当前状态即可  
- 如果2维不够，那么可以考虑3维动态规划  
- dp数组维度>=2维时，要考虑是否有重复计算  
> 一般在递归中使用（因为递归会多次访问一个节点，递推只会访问一次）：可以将dp数组初始化为-1，不符合条件设置为0，符合条件设置为1。这样可以直接用-1来判断该节点是否已经被计算过了  


##### 6. 字符串的题目基本上都能用动态规划来做！！



### 水槽接水、柱状图最大面积等（木桶原理）的题目
只用关注满足nums\[i] > nums\[i + 1]的点i即可  

且这种题一般有O(n)解法，就是使用一个递增stack，如果存在nums\[i] > nums\[i + 1]，就弹出栈顶，否则入栈。  

这样既能保持一个递增栈，且能在nums\[i] > nums\[i + 1]处理水槽面积

