##  有一个字符串列表，从中找出按字典序最大和最小的串。
题目描述：  
```CPP
char * strlist[ N ] = {
    "abc",
    "123",
    ....
    "def"
};
void find( char * strlist[], char ** strmin, char ** strmax );
```
解答：  
```CPP
#include<string.h>
#include<stdio.h>

void find( char * strlist[], int list_len,char ** strmin,char ** strmax);

int main(){

    char * strlist[] = {
                        "abc",
                        "123",
                        "asd",
                        "958",
                        "def"
                        };

    int n = sizeof(strlist)/sizeof(strlist[0]);
    
    char * strmin,* strmax;
  
    //由于没法在函数中计算数组长度，所以得把数组长度传到函数中
    find(strlist,n,&strmin,&strmax);  //如果要修改strmin、strmax中的值，那么就将地址传到函数中

    if(strmin) printf("result: strmin: %s\n",strmin);
    if(strmax) printf("result: strmax: %s\n",strmax);
    return 0;
}

void find( char * strlist[], int list_len, char ** strmin,char ** strmax){

     *strmin = strlist[0];
     *strmax = strlist[0];

    for(int i = 1;i < list_len;i++){
        if(strcmp(strlist[i],*strmin) < 0){
            *strmin = strlist[i];
        }

        if(strcmp(strlist[i],*strmax) > 0){
            *strmax = strlist[i];  
        } 
    }
}
```

- [c中自定义函数通过sizeof来输出数组的长度为何不正确？](https://blog.csdn.net/jiandanokok/article/details/50517837)
- [深入 理解char * ,char ** ,char a[ ] ,char \*a[] 的区别](https://zhuanlan.zhihu.com/p/114161428)
- [指针数组 char * 和 char []到底有什么不一样?](https://www.zhihu.com/question/307261590)


## 写一个函数StrToInt，实现把字符串转换成整数这个功能（或实现atoi）
你可能已经注意到了，实现atoi需要考虑下面这些场景：
- 输入正负号
- 开头有空格
- 转换后的数值超出int的表示范围
- 出错时返回0与正确转换0的区别
- 输入非数字
- 空字符串 

```CPP
#include<string>
#include<iostream>
#include<cstdlib>
using namespace std;

int strToInt(const string& str);

int main(){
    string str = "   ";
    int res = strToInt(str);
    cout << "res: " << res << endl;
    return 0;
}

int strToInt(const string& str){
    int i = 0, res = 0, tmp = 1,len = str.size();

    //跳过开头空格
    while(i < len && str[i] == ' ') i++;

    //确定正负号
    if(i < len && str[i] == '-'){
        tmp = -1;
        i++;
    }
    else if(i < len && str[i] == '+') i++;

    //如果此时字符为空，说明是空字符
    if(i >= len){
        cout << "emtpy string." << endl;
    }

    //开始遍历
    for(;i < str.size();i++){
        //判断字符是否为数字
        if(str[i] < '0' || str[i] > '9'){   //也可以用isdigit()函数
            cout << "invalid char: " << str[i] << endl;
            exit(1);
        }
        //判断中间结果是否越界
        if(res > INT_MAX / 10 || res < INT_MIN / 10){
            cout << "reslut out of the size of Int." << endl;
            exit(1);
        }
        //将结果累加到res上
        res = res * 10 + str[i] - '0';
    }
    
    return tmp * res;
}
```

- [实现atoi函数时需要注意什么？](https://zhuanlan.zhihu.com/p/60113285)
