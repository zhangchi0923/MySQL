
MySQL笔记
============

## 基本概念
- 数据库定义
保存有组织的数据的容器
- 关系型数据库
二维表格模型
- 列（字段）
- 行（记录）
- 主键
  - 本质：一列	
  - 作用：能够唯一标识每一行
  - 要求：任意两行主键值不同；每行都必须具有主键值（且非null）
  - 习惯：不更新、不重用、值不变
## 检索数据
#### 选择数据库
```
USE DATABASE;
SHOW DATABASES;
SHOW TABLES;
SHOW COLUMNS FROM customers / DESCRIBE customers;
```

#### 使用SELECT检索数据
**不同语句用“；”隔开**
```
SELECT prod_name
FROM products;

SELECT prod_name,prod_id,vend_id
FROM products;

SELECT  *
FROM products;

SELECT DISTINCT vend_id
FROM products;

SELECT prod_name
FROM products
LIMIT 5,5;

SELECT products.prod_name
FROM products
```
#### 使用SELECT语句的ORDER BY子句进行排序检索
**ORDER BY 位于 FROM 之后 ，LIMIT 位于ORDER BY 之后**
```
SELECT prod_name
FROM products
ORDER BY prod_name;//默认字母顺序

SELECT prod_id,prod_name,prod_price
FROM products
ORDER BY prod_price,prod_name;

SELECT prod_id,prod_price,prod_name
FROM products
ORDER BY prod_price DESC, prod_name (default:ASC)
```
#### 使用WHERE子句过滤数据
```
//检查单个值
SELECT prod_name
FROM products
WHERE prod_name = 'fuses';//单引号用来限定字符

SELECT prod-name, prod_price
FROM products
WHERE prod_price<10;

//不匹配检查
SELECT prod_name, vend_id
FROM products
WHERE vend_id != 1003;

//范围值检查
SELECT prod_name, prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;

//空值检查
SELECT prod_name
FROM products
WHERE prod_price is NULL;
```
#### 组合WHERE子句过滤数据
```
//（）运算优先级大于AND大于OR
SELECT prod_name, prod_price
FROM products
WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price>10;

//IN操作符用来指定条件范围，合法值之间逗号相隔
SELECT prod_name, prod_price
FROM products
WHERE vend_id IN (1002, 1003)
ORDER BY  prod_name;

//NOT否定其后的所有条件
SELECT prod_name, prod_price
FROM products
WHERE vend_id NOT IN (1002,1003);
```
#### 使用通配符过滤数据
**注意 *尾空格* 可能干扰通配符匹配**
```
// %通配符     
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%'     //检索以jet开头的数据，接受其后任意字符

SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE '%anvil%'      // 搜索模式%anvil%' 匹配任何位置包含anvil的值

SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 's%e'      //匹配以s开头、以e结尾的值

// _下划线通配符(只匹配单个字符）
SELECT  prod_id, prod_name
FROM products
WHERE prod_name LIKE '_ ton anvil';
```

#### *使用通配符注意事项*：

1. 不过度使用，能用其他用其他
2. 尽量不要将通配符放在搜索模式开始处
3. 注意通配符位置

- 正则表达式搜索
```
//基本字符匹配
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP ' .000 ' ;   //  " . "匹配任意单个字符

//OR匹配
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '[123]';     //或者 ' 1|2|3 '  ;集合开始处的“^”表示否定该字符集

//匹配范围、定位、特殊字符
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '^[1-9\\.]';     //“^”在集合外表示从文本开始匹配（“$”从文本  末尾）
```

## 操作数据
#### 创建计算字段
对数据库中的已有数据进行运算（拼接，算术）得到用户所要
```
//拼接vend_name和(vend_address)
SELECT Concat(vend_name,'(', vend_address, ')') AS vend_intro               //AS 后为alias（列别名）
FROM vendors
ORDER BY vend_name;
```

```
//第一句也可用RTRIM()删除数据右侧多余空格
SELECT Concat (RTRIM(vend_name),'(', RTRIM(vend_address), ')' )
```
```
//执行算数计算
SELECT prod_id,
          quantity,
          item_price,
          quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```
SELECT除了检索外还可以进行简单的访问表达式处理
```
SELECT 3*2;

SELECT TRIM('   abc         ')
````
#### 使用数据处理函数
##### 常用文本处理函数

| 函数 |说明  |
| --- | --- |
| Left() | 返回串左边字符 |
| Length() | 返回串长度 |
| Locate()  | 找出串的一个子串 |
| Lower() | 转换小写 |
| LTrim() | 去掉串左边空格 |
| Right() | 返回串右边字符 |
| RTrim() | 去电串右边空格 |
| Soundex() | 返回串SOUNDEX值 |
| Substring() | 返回子串字符 |
| Upper() | 转换大写 |
```
//文本处理函数
SELECT vend_name, Upper(vend_name) AS vend_uppername
FROM vendors
ORDER BY vend_name;

SELECT cust_name, cust_contact
FROM customers
WHERE Soundex(cust_contact) = Soundex('Y Lie');
```
##### 时间和日期处理函数
***默认日期格式为 " yyyy-xx-mm "***
| 函数 | 说明 |
| --- | --- |
| AddDate()  | 增加一个日期（天、周等） |
| AddTime()  | 增加一个时间（时、分等） |
| CurDate() | 返回当前日期 |
| CurTime() | 返回当前时间 |
| Date() | 返回当前日期的日期部分 |
| DateDiff() | 计算两个日期之差 |
| Date_Add() | 灵活的日期计算函数 |
| Date_Format() | 返回一个格式化的日期或字符串 |
| Day() | 返回一个日期的天数部分 |
| DayOfWeek() | 对于一个日期，返回对应的星期几 |
| Hour() | 返回时间的小时部分 |
| Minute() | 返回日期的分钟部分 |
| Month() | 返回日期的月份 |
| Now() | 返回当前日期和时间 |
| Second() | 返回一个时间的秒部分 |
| Time() | 返回一个日期的时间部分 |
| Year() | 返回一个日期的年份 |
```
//日期和时间处理函数
SELECT cust_id, order_num
FROM orders
WHERE order_date = '2005-09-01';

SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) = '2005-09-01';

SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';

SELECT cust_id, order_num
FROM orders
WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
```
##### 数值处理函数

| 函数 | 说明 |
| --- | --- |
|Abs()  |返回绝对值  |
| Cos() | 返回余弦值 |
|Exp()  | 返回指数值 |
|Mod()  | 返回除操作的余数 |
| Pi() | 返回圆周率 |
| Rand() | 返回一个随机数 |
| Sin() | 返回正弦值 |
| Sqrt() | 返回平方根 |
| Tan() | 返回正切值 |

#### 数据汇总
##### 聚集函数
*运行在行组上，返回单个值。*

**常用聚集函数**

| 函数 | 说明 |
| --- | --- |
| AVG( ) | 返回列平均值 |
| SUM( ) | 返回某列的和 |
| COUNT( ) | 返回某列行数 |
| MIN( ) | 返回某列最小值 |
| MAX( ) | 返回某列最大值 |
```
//聚集彼此不同值
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;

//组合聚集函数
SELECT Count(*) AS num_items,
       MIN(prod_price) AS min_price,
       MAX(prod_price) AS max_price,
       AVG(prod_price) AS avg_price
FROM products;
```
#### 分组函数
##### GROUP BY 创建分组
```
SELECT vend_id, COUNT(*) AS num_prod
FROM products
GROUP BY vend_id;
```

**GROUP BY 的一些规定*

1. 可包含任意数目的列，为分组提供精细控制
2. 若嵌套了分组则按分组返回，即不能从个别列取回数据
3. GROUP BY的子句每一列都必须是检索列或表达式；若SELECT中有表达式，则GROUP BY必须用同样的表达式，不能用别名！
4. 除聚集表达式，SELECT出现的每一列都必须出现在GROUP BY子句中

##### HAVING 过滤分组
HAVING 和 WHERE 很相似。HAVING 过滤分组，WHERE 过滤行。
```
SELECT vend_id, COUNT(*) AS orders
FROM products
WHERE prod_price>=10
GROUP BY vend_id
HAVING orders>=2;
```
##### 排序与分组
```
SELECT order_num, SUM(quantity*item_price) AS ordertotalprice   
FROM orderitems
GROUP BY order_num                                  //有几个个分组就有几个聚集函数返回值
HAVING ordertotalprice >= 50
ORDER BY ordertotalprice;
```
### 项目一：查找重复邮箱
##### 建表
```
CREATE TABLE email (
ID INT NOT NULL PRIMARY KEY,
Email VARCHAR(255) NOT NULL
);
```
##### 插入数据
```
INSERT INTO email VALUES('1','a@b.com');
INSERT INTO email VALUES('2','c@d.com');
INSERT INTO email VALUES('3','a@b.com');
```
##### 查询语句

```
SELECT Email, COUNT(*) AS num
FROM email
GROUP BY Email
HAVING num>1;
```
##### 显示结果
![查找结果](https://img-blog.csdnimg.cn/20190402203413104.png)

### 项目二：查找大国
##### 建表
```
CREATE TABLE World (
name VARCHAR(50) NOT NULL,
continent VARCHAR(50) NOT NULL,
area INT NOT NULL,
population INT NOT NULL,
gdp INT NOT NULL
);
```
##### 插入数据
```
INSERT INTO World
  VALUES('Afghanistan','Asia',652230,25500100,20343000);
INSERT INTO World 
  VALUES('Albania','Europe',28748,2831741,12960000);
INSERT INTO World 
  VALUES('Algeria','Africa',2381741,37100000,188681000);
INSERT INTO World
  VALUES('Andorra','Europe',468,78115,3712000);
INSERT INTO World
  VALUES('Angola','Africa',1246700,20609294,100990000);
```
##### 查询代码
```
SELECT name, population, area
FROM World
WHERE area>3000000 OR population>25000000 AND gdp>20000000;
```
##### 显示结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190402210632320.png)
