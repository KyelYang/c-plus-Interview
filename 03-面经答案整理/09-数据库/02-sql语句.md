# sql语句
## show语句
### show columns 或 describe
#### show columns
```sql
SHOW COLUMNS FROM customers;
```
SHOW COLUMNS要求给出一个表名（这个例子中的 FROM customers），它对每个字段返回一行，行中包含字段名、数据类型、是否允许NULL、键信息、默认值以及其他信息（如字段cust_id的auto_increment）。  

#### describe
```sql
DESCRIBE customers; --是SHOW COLUMNS FROM customers;的一种快捷方式。
```
MySQL支持用DESCRIBE作为SHOW COLUMNS FROM的一种快捷方式。  

### show create database/table
分别用来显示创建特定数据库或表的MySQL语句。  

## select语句
### 检索单个列
```sql
SELECT prod_name FROM products;
```
### 检索多个列
```sql
SELECT prod_id,prod_name,prod_price FROM products;
```
### 检索所有列
```sql
SELECT * FROM products;
```
#### 使用通配符  
一般，除非你确实需要表中的每个列，否则最好别使用\*通配符。虽然使用通配符可能会使你自己省事，不用明确列出所需列，但检索不需要的列通常会降低检索和应用程序的性能。  
 
### 检索不同的行(检索的行无重复值，distinct)
 
```sql
SELECT DISTINCT vend_id FROM products;  --SELECT DISTINCT vend_id告诉MySQL只返回不同（唯一）的vend_id行
```
#### distinct  
此关键字指示MySQL只返回不同的值。  

#### 不能部分使用distinct  
DISTINCT关键字应用于所有列而不仅是前置它的列。如果给出SELECT DISTINCT vend_id, prod_price，除非指定的两个列都不同，否则所有行都将被检索出来。  
 
### 限制结果(limit)
```sql
SELECT prod_name FROM products LIMIT 5; --LIMIT 5指示MySQL返回不多于5行

SELECT prod_name FROM products LIMIT 4,3; --LIMIT 4, 3指示MySQL返回从行4开始的3行。第一个数为开始位置，第二个数为要检索的行数

SELECT prod_name FROM products LIMIT 3 OFFSET 4;  --和上一句一样的意思
```

#### 行0  
检索出来的第一行为行0而不是行1。因此，LIMIT 1, 1将检索出第二行而不是第一行。  

### 使用完全限定的表名
```sql
SELECT prod_name FROM products;
 
SELECT products.prod_name FROM sql_learning.products; --此句等价于上一句，有一些情形需要完全限定名
```

## ORDER BY排序语句
### 排序数据
```sql
SELECT prod_name FROM products ORDER BY prod_name;
```
### 通过非选择列进行排序
通常，ORDER BY子句中使用的列将是为显示所选择的列。但是，实际上并不一定要这样，用非检索的列排序数据是完全合法的。  


### 按多个列排序
```sql
SELECT prod_id,prod_price,prod_name FROM products ORDER BY prod_price,prod_name; --检索3个列，并按其中两个列对结果进行排序——首先按价格，然后再按名称排序
```

### 指定排序方向
数据排序不限于升序排序（从A到Z）。这只是默认的排序顺序，还可以使用ORDER BY子句以降序（从Z到A）顺序排序。为了进行降序排序，必须指定DESC关键字。 

```sql
SELECT prod_id,prod_price,prod_name FROM products ORDER BY prod_price DESC; --按价格降序排列
```

#### 在多个列上降序排序DESC（DESCENDING）
DESC关键字只应用到直接位于其前面的列名。  

如果想在多个列上进行降序排序，必须对每个列指定DESC关键字。  

```sql
SELECT prod_id,prod_price,prod_name FROM products ORDER BY prod_price DESC,prod_name DESC; --按价格、产品名降序排列
```

#### 在多个列上升序排序ASC（ASCENDING）
与DESC相反的关键字是ASC（ASCENDING），在升序排序时可以指定它。  

但实际上，ASC没有多大用处，因为升序是默认的（如果既不指定ASC也不指定DESC，则假定为ASC）。  


#### 区分大小写和排序顺序
在字典（dictionary）排序顺序中，A被视为与a相同，这是MySQL（和大多数数据库管理系统）的默认行为。但是，许多数据库管理员能够在需要时改变这种行为（如果你的数据库包含大量外语字符，可能必须这样做）。  

这里，关键的问题是，如果确实需要改变这种排序顺序，用简单的ORDER BY子句做不到。你必须请求数据库管理员的帮助。  


### 找出一个列中最高或最低的值：ORDER BY和LIMIT组合
```sql
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1; --找出价格最高的一个值
```

#### ORDER BY子句的位置  
在给出ORDER BY子句时，应该保证它位于FROM子句之后。如果使用LIMIT，它必须位于ORDER BY之后。使用子句的次序不对将产生错误消息。  

## 过滤数据where
### 简单的相等测试
```sql
SELECT prod_name,prod_price FROM products WHERE prod_price = 2.50; --找出价格等于2.50的条目
```

#### WHERE子句的位置  
在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误。  

### WHERE子句操作符
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/139.jpg" width = 60% height = 60% /></div>

### 检查单个值
```sql
SELECT prod_name,prod_price FROM products WHERE prod_name = 'fuses'; --MySQL在执行匹配时默认不区分大小写，所以fuses与Fuses匹配

SELECT prod_name,prod_price FROM products WHERE prod_price <= 10;
```

### 不匹配检查
```sql
SELECT vend_id,prod_name,prod_price FROM products WHERE prod_id != 1003; 

SELECT vend_id,prod_name,prod_price FROM products WHERE prod_id <> 1003; --这两句等价
```

### 范围值检查between
```sql
SELECT prod_name,prod_price FROM products WHERE prod_price BETWEEN 5 AND 10; --两个指定值必须由低到高，不然筛选结果为空集
```

### 空值检查 IS NULL语句
在创建表时，表设计人员可以指定其中的列是否可以不包含值。在一个列不包含值时，称其为包含空值NULL。  

NULL无值（no value），它与字段包含0、空字符串或仅仅包含空格不同。  

```sql
SELECT prod_name FROM products WHERE prod_price IS NULL; --返回没有价格（空prod_price字段，不是价格为0）的所有产品
```

### AND操作符
为了通过不止一个列进行过滤，可使用AND操作符给WHERE子句附加条件。  
```sql
SELECT prod_id,prod_price,prod_name FROM products WHERE vend_id = 1003 AND prod_price <= 10; 
```
上述例子中使用了只包含一个关键字AND的语句，把两个过滤条件组合在一起。还可以添加多个过滤条件，每添加一条就要使用一个AND。  

### OR操作符
OR操作符与AND操作符不同，它指示MySQL检索匹配任一条件的行。  

```sql
SELECT prod_name,prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003; 
```

### 计算次序
```sql
SELECT prod_name,prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10; 
```
SQL（像多数语言一样）在处理OR操作符前，优先处理AND操作符。  

当SQL看到上述WHERE子句时，它理解为由供应商1003制造的任何价格为10美元（含）以上的产品，或者由供应商1002制造的任何产品，而不管其价格如何。  

换句话说，由于AND在计算次序中优先级更高，操作符被错误地组合了。  

此问题的解决方法是使用圆括号明确地分组相应的操作符。  

```sql
SELECT prod_name,prod_price FROM products WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10; 
```

### IN操作符
圆括号在WHERE子句中还有另外一种用法。  

IN操作符用来指定条件范围，范围中的每个条件都可以进行匹配。IN取合法值的由逗号分隔的清单，全都括在圆括号中。  
```sql
SELECT prod_name,prod_price FROM products WHERE vend_id IN (1002,1003) ORDER BY prod_name; 

SELECT prod_name,prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003 ORDER BY prod_name; --这两句等价
```

#### 为什么要使用IN操作符？其优点具体如下
- 在使用长的合法选项清单时，IN操作符的语法更清楚且更直观
- 在使用IN时，计算的次序更容易管理（因为使用的操作符更少）
- IN操作符一般比OR操作符清单执行更快
- IN的最大优点是可以包含其他SELECT语句，使得能够更动态地建立WHERE子句  


### NOT操作符：是否定它之后所跟的任何条件
```sql
SELECT prod_name,prod_price FROM products WHERE NOT IN (1002,1003) ORDER BY prod_name; 
```

#### MySQL中的NOT 
MySQL支持使用NOT对IN、BETWEEN和EXISTS子句取反，这与多数其他DBMS允许使用NOT对各种条件取反有很大的差别。  

## 用通配符(用来匹配值的一部分的特殊字符)进行过滤
### LIKE操作符
为在搜索子句中使用通配符，必须使用LIKE操作符。LIKE指示MySQL，后跟的搜索模式利用通配符匹配而不是直接相等匹配进行比较。  

### 百分号（%）通配符
在搜索串中，%表示任何字符出现任意次数。  
```sql
SELECT prod_id,prod_name FROM products WHERE prod_name LIKE 'jet%'; --检索任意以jet起头的词。%告诉MySQL接受jet之后的任意字符，不管它有多少字符
```
#### 区分大小写
根据MySQL的配置方式，搜索可以是区分大小写的。如果区分大小写，'jet%'与JetPack 1000将不匹配。  

#### 匹配0个字符
除了一个或多个字符外，%还能匹配0个字符。%代表搜索模式中给定位置的0个、1个或多个字符。  

#### 注意NULL
虽然似乎%通配符可以匹配任何东西，但有一个例外，即NULL。即使是WHERE prod_name LIKE '%'也不能匹配用值NULL作为产品名的行。  


### 下划线（_）通配符
下划线的用途与%一样，但下划线只匹配单个字符而不是多个字符。  

与%能匹配0个字符不一样，_总是匹配一个字符，不能多也不能少。  

```sql
SELECT prod_id,prod_name FROM products WHERE prod_name LIKE '_ ton anvil'; 
```

## 用正则表达式进行搜索
### 基本字符匹配REGEXP
```sql
SELECT prod_id,prod_name FROM products WHERE prod_name REGEXP '1000'; --它告诉MySQL：REGEXP后所跟的东西作为正则表达式（与文字正文1000匹配的一个正则表达式）处理
```
#### LIKE和REGEXP的重要的差别
- LIKE匹配整个列。如果被匹配的文本在列值中出现，LIKE将不会找到它，相应的行也不被返回（除非使用通配符） 
- 而REGEXP在列值内进行匹配，如果被匹配的文本在列值中出现，REGEXP将会找到它，相应的行将被返回
- REGEXP可以使用^和$定位符来匹配整个列值

#### 匹配不区分大小写
MySQL中的正则表达式匹配（自版本3.23.4后）不区分大小写（即，大写和小写都匹配）。  

为区分大小写，可使用BINARY关键字，如：  
```sql
WHERE prod_name REGEXP BINARY 'JetPack .000';
```

### .匹配任意一个字符
.是正则表达式语言中一个特殊的字符。它表示匹配任意一个字符，因此，1000和2000都匹配且返回。  
```sql
SELECT prod_id,prod_name FROM products WHERE prod_name REGEXP '.000';
```

### 进行OR匹配 |
为搜索两个串之一（或者为这个串，或者为另一个串），使用|    

|为正则表达式的OR操作符。它表示匹配其中之一，因此1000和2000都匹配并返回。  

```sql
SELECT prod_id,prod_name FROM products WHERE prod_name REGEXP '1000 | 2000';
```

使用|从功能上类似于在SELECT语句中使用OR语句，多个OR条件可并入单个正则表达式。  

### 匹配几个字符之一
通过指定一组用\[和]括起来的字符来匹配任何单一字符。  

\[123]定义一组字符，它的意思是匹配1或2或3，因此，1 ton和2 ton都匹配且返回（没有3 ton）。  

```sql
SELECT prod_id,prod_name FROM products WHERE prod_name REGEXP '[123] Ton';
```

正如所见，\[]是另一种形式的OR语句。事实上，正则表达式\[123]Ton为\[1|2|3]Ton的缩写，也可以使用后者，但匹配特定字符时，千万不可将括号\[]省略，不然匹配不当。  

### 否定字符集^
为否定一个字符集，在集合的开始处放置一个^即可。  

因此，尽管\[123]匹配字符1、2或3，但\[^123]却匹配除这些字符外的任何东西。  

### 匹配范围-
范围不限于完整的集合，\[1-3]和\[6-9]也是合法的范围。  

此外，范围不一定只是数值的，\[a-z]匹配任意字母字符。  

```sql
SELECT prod_id,prod_name FROM products WHERE prod_name REGEXP '[1-3] Ton';
```

### 匹配特殊字符-转义
为了匹配特殊字符，必须用\\为前导。\\-表示查找-，\\.表示查找.。  

\\也用来引用元字符（具有特殊含义的字符），如表所列:  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/140.jpg" width = 60% height = 60% /></div>

#### 匹配\
为了匹配反斜杠（\）字符本身，需要使用\\\。  

#### \或\\?
多数正则表达式实现使用单个反斜杠转义特殊字符，以便能使用这些字符本身。  

但MySQL要求两个反斜杠（MySQL自己解释一个，正则表达式库解释另一个）。  

### 匹配字符类
存在找出你自己经常使用的数字、所有字母字符或所有数字字母字符等的匹配。为更方便工作，可以使用预定义的字符集，称为字符类
（character class）:
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/141.jpg" width = 60% height = 60% /></div>

### 匹配多个实例-重复元字符
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/142.jpg" width = 60% height = 60% /></div>

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '\\([0-9] sticks?\\)';
```
\\(匹配)，\[0-9]匹配任意数字（这个例子中为1和5），sticks?匹配stick和sticks（s后的?使s可选，因为?匹配它前面的任何字符的0次或1次出现），\\)匹配)。没有?，匹配stick和sticks会非常困难。  

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[[:digit:]]{4}';
```
\[:digit:]匹配任意数字，因而它为数字的一个集合。{4}确切地要求它前面的字符（任意数字）出现4次，所以\[\[:digit:]]{4}匹配连在一起的任意4位数字。  


需要注意的是，在使用正则表达式时，编写某个特殊的表达式几乎总是有不止一种方法。上面的例子也可以如下编写：  
```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[0-9][0-9][0-9][0-9]';
```

### 定位符
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/143.jpg" width = 60% height = 60% /></div>  


```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '^[0-9\\.]';
```
^匹配串的开始。因此，^\[0-9\\.]只在.或任意数字为串中第一个字符时才匹配它们。没有^，则还要多检索出4个别的行（那些中间有数字的行）。  

#### ^的双重用途
- 在集合中（用\[和]定义），用它来否定该集合
- 用来指串的开始处

### 简单的正则表达式测试
可以在不使用数据库表的情况下用SELECT来测试正则表达式。  

REGEXP检查总是返回0（没有匹配）或1（匹配）。可以用带文字串的REGEXP来测试表达式，并试验它们。相应的语法如下：  
```sql
SELECT 'hello' REGEXP '[0-9]';
```
这个例子显然将返回0（因为文本hello中没有数字）。

## 使用别名
一个未命名的列不能用于客户机应用中，因为客户机没有办法引用它，因此此时必须使用别名。  

### 别名关键字as
```sql
SELECT Concat(vend_name,'(',vend_country,')')  AS vend_title FROM vendors ORDER BY vend_name;
```

### 别名的其他用途
别名还有其他用途。常见的用途包括在实际的表列名包含不符合规定的字符（如空格）时重新命名它，在
原来的名字含混或容易误解时扩充它，等等。  

## 执行算术计算
MySQL支持下表中列出的基本算术操作符。此外，圆括号可用来区分优先顺序。  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/147.jpg" width = 60% height = 60% /></div>  

比如可以使用算术计算汇总物品的价格（单价乘以订购数量）：  
```sql
SELECT prod_id,quantity,item_price,quantity * item_price AS expanded_price FROM orderitems;
```

### 如何测试计算
SELECT提供了测试和试验函数与计算的一个很好的办法。虽然SELECT通常用来从表中检索数据，但可以省略FROM子句以便简单地访问和处理表达式。  
例如:
```sql
SELECT 3*2; --将返回6
SELECT Trim(' abc '); --将返回abc
SELECT Now(); --利用Now()函数返回当前日期和时间
```

# 函数
## 文本处理函数
### 拼接字段
拼接（concatenate） 将值联结到一起构成单个值。  
#### MySQL的不同之处
多数DBMS使用+或||来实现拼接，MySQL则使用Concat()函数来实现。当把SQL语句转换成MySQL语句时一定要把这个区别铭记在心。  

#### 使用Concat()函数来拼接多个列
```sql
SELECT Concat(vend_name,'(',vend_country,')') FROM vendors ORDER BY vend_name;
```
Concat()拼接串，即把多个串连接起来形成一个较长的串。  

Concat()需要一个或多个指定的串，各个串之间用逗号分隔。  

### 去掉空格函数
#### Trim() 去掉串左右两边的空格
#### RTrim() 去掉串右边的空格
#### LTrim() 去掉串左边的空格

### 常见的文本处理函数
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/148.jpg" width = 60% height = 60% /></div>  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/149.jpg" width = 60% height = 60% /></div>  

SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。  
```sql
SELECT cust_name,cust_contact FROM customers WHERE Soundex(cust_contact) = Soundex('Y Lie');
```
在这个例子中，WHERE子句使用Soundex()函数来转换cust_ contact列值和搜索串为它们的SOUNDEX值。因为Y.Lee和Y.Lie发音相似，所以它们的SOUNDEX值匹配，因此WHERE子句正确地过滤出了所需的数据。  

## 日期和时间处理函数
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/150.jpg" width = 60% height = 60% /></div>  

需要注意的是MySQL使用的日期格式。无论你什么时候指定一个日期，不管是插入或更新表值还是用WHERE子句进行过滤，日期必须为格式yyyy-mm-dd。因此，2005年9月1日，给出为2005-09-01。虽然其他的日期格式可能也行，但这是首选的日期格式，因为它排除了多义性（如，04/05/06是2006年5月4日或2006年4月5日或2004年5月6日或……）。  

### 如果要的是日期，请使用Date()
```sql
SELECT cust_id,order_num FROM orders WHERE Date(order_date) = '2005-09-01';

--如果你想检索出2005年9月下的所有订单
SELECT cust_id,order_num FROM orders WHERE Year(order_date) = 2005 AND  Month(order_date) = 9;
```

## 数值处理函数
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/151.jpg" width = 60% height = 60% /></div>  


## 聚集函数
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/152.jpg" width = 60% height = 60% /></div>  

### AVG()函数
AVG()通过对表中行数计数并计算特定列值之和，求得该列的平均值。AVG()可用来返回所有列的平均值，也可以用来返回特定列或行的平均值。  
```sql
SELECT AVG(prod_price) AS avg_price FROM products WHERE vend_id = 1003;
```
#### 只用于单个列
 AVG()只能用来确定特定数值列的平均值，而且列名必须作为函数参数给出。为了获得多个列的平均值，必须使用多个AVG()函数。

#### NULL值
AVG()函数忽略列值为NULL的行。

### COUNT()函数
COUNT()函数进行计数。可利用COUNT()确定表中行的数目或符合特定条件的行的数目。  
```sql
-- 使用COUNT(*)对表中行的数目进行计数，不管表列中包含的是空值（NULL）还是非空值。

SELECT COUNT(*) AS num_cust FROM customers;

-- 使用COUNT(column)对特定列中具有值的行进行计数，忽略NULL值。

SELECT COUNT(cust_email) AS num_cust FROM customers;
```

### MAX()、MIN()函数
MAX()返回指定列中的最大值。MAX()要求指定列名。  

MIN()的功能正好与MAX()功能相反，它返回指定列的最小值。  

```sql
SELECT MAX(prod_price) AS max_price FROM products;

SELECT MIN(prod_price) AS min_price FROM products;
```

#### 对非数值数据使用MAX()、MIN() 
MIN()函数与MAX()函数类似，MySQL允许将它用来返回任意列中的最小值，包括返回文本
列中的最小值。在用于文本数据时，如果数据按相应的列排序，则MIN()返回最前面的行。  

#### NULL值
 MAX()、MIN()函数忽略列值为NULL的行。  
 
### SUM()函数
```sql
SELECT SUM(quantity) AS items_ordered FROM orderitems;

SELECT SUM(item_price * quantity) AS items_ordered FROM orderitems;
```

#### 在多个列上进行计算
如本例所示，利用标准的算术操作符，所有聚集函数都可用来执行多个列上的计算。

#### NULL值
SUM()函数忽略列值为NULL的行。  

### 聚集不同值DISTINCT
```sql
--下面的例子使用AVG()函数返回特定供应商提供的产品的平均价格，由于使用了DISTINCT参数，因此平均值只考虑各个不同的价格
SELECT AVG(DISTINCT prod_price) AS avg_price FROM products;
```

如果指定列名，则DISTINCT只能用于COUNT()。DISTINCT不能用于COUNT(\*)，因此不允许使用COUNT（DISTINCT），否则会产生错误。类似地，DISTINCT必须使用列名，不能用于计算或表达式。  

将DISTINCT用于MIN()和MAX() 虽然DISTINCT从技术上可用于MIN()和MAX()，但这样做实际上没有价值。一个列中的最小值和最大值不管是否包含不同值都是相同的。  

### 组合聚集函数
```sql
SELECT COUNT(*) AS num_items,MIN(prod_price) AS price_min,MAX(prod_price) AS price_max,AVG(prod_price) AS price_avg FROM products;
```

### 取别名
在指定别名以包含某个聚集函数的结果时，不应该使用表中实际的列名。虽然这样做并非不合法，但使用唯一的名字会使你的SQL更易于理解和使用（以及将来容易排除故障）。  

#分组数据
## 创建分组GROUP BY
分组是在SELECT语句的GROUP BY子句中建立的。  
```sql
-- GROUP BY子句指示MySQL按vend_id排序并分组数据。这导致对每个vend_id而不是整个表计算num_prods一次。
SELECT vend_id,COUNT(*) AS num_prods FROM products GROUP BY vend_id;
```
GROUP BY子句指示MySQL分组数据，然后对每个组而不是整个结果集进行聚集。  

### GROUP BY的重要规定
- GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
- 如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算（所以不能从个别的列取回数据）。
- GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
- 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
- GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

### 使用ROLLUP
使用WITH ROLLUP关键字，可以得到每个分组以及每个分组汇总级别（针对每个分组）的值。  
```sql
SELECT vend_id,COUNT(*) AS num_prods FROM products GROUP BY vend_id WITH ROLLUP;
```

## 过滤分组HAVING
```sql
-- 使用HAVING过滤COUNT(*) >=2（两个以上的订单）的那些分组。
SELECT cust_id,COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >= 2;
```
### HAVING支持所有WHERE操作符
所学过的有关WHERE的所有这些技术和选项都适用于HAVING。它们的句法是相同的，只是关键字有差别。  

### HAVING和WHERE的差别
WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤。  

这是一个重要的区别，WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中基于这些值过滤掉的分组。

### 同时使用WHERE和HAVING
```sql
-- WHERE子句过滤所有prod_price至少为10的行。然后按vend_id分组数据，HAVING子句过滤计数为2或2以上的分组。
SELECT cust_id,COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;
```

## 分组和排序
### ORDER BY与GROUP BY的区别
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/153.jpg" width = 60% height = 60% /></div>  

### 不要忘记ORDER BY 
一般在使用GROUP BY子句时，应该也给出ORDER BY子句。这是保证数据正确排序的唯一方法。千万不要仅依赖GROUP BY排序数据。  
```sql
-- GROUP BY子句用来按订单号（order_num列）分组数据，以便SUM(*)函数能够返回总计订单价格。HAVING子句过滤数据，使得只返回总计订单价格大于等于50的订单。最后，用ORDER BY子句排序输出。

SELECT order_num,SUM(quantity * item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity * tiem_price) >= 50 ORDER BY ordertotal;
```

## SELECT子句顺序
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/154.jpg" width = 60% height = 60% /></div>  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/155.jpg" width = 60% height = 60% /></div>

# 使用子查询
## 利用子查询进行过滤
```sql
-- 为了执行上述SELECT语句，MySQL实际上必须执行3条SELECT语句。
--最里边的子查询返回订单号列表，此列表用于其外面的子查询的WHERE子句。外面的子查询返回客户ID列表，此客户ID列表用于最外层查询的WHERE子句。最外层查询确实返回所需的数据。

SELECT cust_name,cust_contact
FROM customers
WHERE cust_id IN (
    SELECT cust_id 
    FROM orders
    WHERE order_num IN (
        SELECT order_num 
        FROM orderitems
        WHERE prod_id = 'TNT2')
    );
```

## 作为计算字段使用子查询
```sql
-- orders是一个计算字段，它是由圆括号中的子查询建立的。该子查询对检索出的每个客户执行一次。在此例子中，该子查询执行了5次，因为检索出了5个客户。

SELECT cust_name,cust_state,(
    SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```
## 逐渐增加子查询来建立查询 
用子查询测试和调试查询很有技巧性，特别是在这些语句的复杂性不断增加的情况下更是如此。  

用子查询建立（和测试）查询的最可靠的方法是逐渐进行，这与MySQL处理它们的方法非常相同。  

首先，建立和测试最内层的查询。然后，用硬编码数据建立和测试外层查询，并且仅在确认它正常后才嵌入子查询。这时，再次测试它。对于要增加的每个查询，重复这些步骤。这样做仅给构造查询增加了一点点时间，但节省了以后（找出查询为什么不正常）的大量时间，并且极大地提高了查询一开始就正常工作的可能性。  

## 联结表
相同数据出现多次决不是一件好事，此因素是关系数据库设计的基础。  

关系表的设计就是要保证把信息分解成多个表，一类数据一个表。各表通过某些常用的值（即关系设计中的关系（relational））互相关联。  

### 外键（foreign key）
外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。  

### 如果数据存储在多个表中，怎样用单条SELECT语句检索出数据？
答案是使用联结。简单地说，联结是一种机制，用来在一条SELECT语句中关联表，因此称之为联结。使用特殊的语法，可以联结多个表返回一组输出，联结在运行时关联表中正确的行。  

### 等值联结（内部联结）
```sql
SELECT vend_name,prod_name,prod_price FROM vendors,products WHERE vendors.vend_id = products.vend_id;
```

### 内部联结
以下方式和上面的等值联结效果一样，不过推荐使用INNER JOIN。
```sql
SELECT vend_name,prod_name,prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
```

### 使用表别名
这样做有两个主要理由：
- 缩短SQL语句
- 允许在单条SELECT语句中多次使用相同的表

```sql
SELECT cust_name,cust_contact FROM customers AS c,orders AS o,orderitems AS oi WHERE c.cust_id = o.cust_id AND oi.order_num = o.order_num AND prod_if = 'TNT2';
```

### 自联结
使用子查询的情况：
```sql
SELECT prod_id,prod_name FROM products WHERE vend_id = (SELECT vend_id FROM products WHERE prod_id = 'DTNTR');
```

使用自联结的相同查询：
```sql
SELECT p1.prod_id,p1.prod_name FROM products AS p1, products AS p2 WHERE p1.vend_id = p2.vend_id AND p2.prod_id = 'DTNTR';
```

#### 为什么用自联结而不用子查询
自联结通常作为外部语句用来替代从相同表中检索数据时使用的子查询语句。虽然最终的结果是相同的，但有时候处理联结远比处理子查询快得多。应该试一下两种方法，以确定哪一种的性能更好。  

### 自然联结（排除多次出现，使每个列只返回一次）
无论何时对表进行联结，应该至少有一个列出现在不止一个表中（被联结的列）。标准的联结（前一章中介绍的内部联结）返回所有数据，甚至相同的列多次出现。自然联结排除多次出现，使每个列只返回一次。  

怎样完成这项工作呢？答案是，系统不完成这项工作，由你自己完成它。自然联结是这样一种联结，其中你只能选择那些唯一的列。这一般是通过对表使用通配符（SELECT \*），对所有其他表的列使用明确的子集来完成的。下面举一个例子：  
```sql
SELECT c.*,o.order_num,o.order_date,oi.prod_id,oi.quantitymoi.item_price FROM customers AS c,orders AS o,orderitems AS oi WHERE c.cust_id = o.cust_id AND oi.order_num = o.order_num AND prod_id = 'FB';
```
在这个例子中，通配符只对第一个表使用。所有其他列明确列出，所以没有重复的列被检索出来。  

事实上，迄今为止我们建立的每个内部联结都是自然联结，很可能我们永远都不会用到不是自然联结的内部联结。  

### 外部联结
许多联结将一个表中的行与另一个表中的行相关联。但有时候会需要包含没有关联行的那些行。联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。  
```sql
SELECT customers.cust_id,orders.order_num FROM customers LEFT OUTER JOIN orders ON orders.cust_id = customers.cust_id;
```
在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字指定包括其所有行的表（RIGHT指出的是OUTER JOIN右边的表，而LEFT指出的是OUTER JOIN左边的表）。  

上面的例子使用LEFT OUTER JOIN从FROM子句的左边表（customers表）中选择所有行。  

为了从右边的表中选择所有行，应该使用RIGHT OUTER JOIN。  

### 使用带聚集函数的联结
聚集函数用来汇总数据。虽然至今为止聚集函数的所有例子只是从单个表汇总数据，但这些函数也可以与联结一起使用。  

如果要检索所有客户及每个客户所下的订单数，下面使用了COUNT()函数的代码可完成此工作：  
```sql
SELECT customers.cust_name,customers.cust_id,COUNT(orders.order_num) AS num_ord FROM customers.cust_id = orders.cust_id GROUP BY customers.cust_id;
```
此SELECT语句使用INNER JOIN将customers和orders表互相关联。GROUP BY 子句按客户分组数据，因此，函数调用 COUNT(orders.order_num)对每个客户的订单计数，将它作为num_ord返回。  

## 组合查询UNION
### 组合查询的应用场景
有两种基本情况，其中需要使用组合查询：  
- 在单个查询中从不同的表返回类似结构的数据
- 对单个表执行多个查询，按单个查询返回数据

### 组合查询和多个WHERE条件
多数情况下，组合相同表的两个查询完成的工作与具有多个WHERE子句条件的单条查询完成的工作相同。  
换句话说，任何具有多个WHERE子句的SELECT语句都可以作为一个组合查询给出，在以下段落中可以看到这一点。这两种技术在不同的查询中性能也不同。因此，应该试一下这两种技术，以确定对特定的查询哪一种性能更好。  

### 使用组合查询
```sql
-- 没有使用组合查询的写法
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <= 5 AND vend_id IN (1001,1002);

-- 使用组合查询的写法
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <= 5 
UNION
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id IN (1001,1002); 
```

### UNION使用规则
- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个UNION关键字）。  
- UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。  

### 包含或取消重复的行
UNION从查询结果集中自动去除了重复的行（换句话说，它的行为与单条SELECT语句中使用多个WHERE子句条件一样）。  

这是UNION的默认行为，但是如果需要，可以改变它。事实上，如果想返回所有匹配行，可使用UNION ALL而不是UNION。  
```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <= 5 
UNION ALL
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id IN (1001,1002); 
```
UNION ALL为UNION的一种形式，它完成WHERE子句完成不了的工作。如果确实需要每个条件的匹配行全
部出现（包括重复行），则必须使用UNION ALL而不是WHERE。  

### 对组合查询结果排序
在用UNION组合查询时，只能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后。  

对于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况，因此不允许使用多条ORDER BY子句。  
```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <= 5 
UNION
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id IN (1001,1002) ORDER BY vend_id,prod_price; 
```
虽然ORDER BY子句似乎只是最后一条SELECT语句的组成部分，但实际上MySQL将用它来排序所有SELECT语句返回的所有结果。  

## 全文本搜索

 








