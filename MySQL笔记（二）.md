## MySQL笔记（二）
#### 使用子查询
子查询本质是“定语从句”,总是从内向外处理
```
SELECT cust_id, order_num
FROM orders
WHERE order_num IN ( SELECT order_num
                     FROM orderitems
                     WHERE prod_id = 'TNT2');
```
```
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN(SELECT cust_id
                 FROM orders
                 WHERE order_num IN (SELECT order_num
                                     FROM orderitems
                                     WHERE prod_id = 'TNT2'));
```

#### 项目三：创建课表并查询
##### 建表
```
DROP TABLE course;
CREATE TABLE course
(
student char(10) NOT NULL,
class char(50) NOT NULL
)ENGINE = InnoDB;
INSERT INTO course(
student, class
)
VALUES(
'A','Math'),
(
  'B','English'),
(
  'C','Math'),
(
  'D','Biology'),
(
  'E','Math'),
(
  'F','Computer'),
(
  'G','Math'),
(
  'H','Math'),
(
  'I','Math'),
(
  'A','Math');
```
##### 查询
```
SELECT class
FROM (SELECT class, COUNT(DISTINCT student) AS num
      FROM course
      GROUP BY class) AS newclass
WHERE newclass.num>=5;
```
##### 输出结果
![# 图](https://img-blog.csdnimg.cn/20190404182226196.png)
#### 项目四：交换工资
##### 建表
```
CREATE TABLE salary
(
  id int NOT NULL AUTO_INCREMENT,
  name char(50) NOT NULL,
  sex char(10) NOT NULL,
  salary int NOT NULL,
  PRIMARY KEY (id)
)ENGINE = InnoDB;
```
##### 插入数据
```
INSERT INTO salary
(
  name,sex,salary
)
VALUES
(
  'A','m',2500
),
(
  'B','f',1500
),
(
  'C','m',5500
),
(
  'D','f',500
);
```
##### 更新数据
```
UPDATE salary
SET sex = CASE sex
WHEN 'm' THEN 'f'
WHEN 'f' THEN 'm'
END
WHERE sex IN('m','f');

SELECT id, name, sex, salary
FROM salary;
```
##### 输出结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190404181725123.png)
#### 项目五：组合两张表
##### 建两张表并插入自定义数据
```
CREATE TABLE person
(
  personid int NOT NULL AUTO_INCREMENT,
  firstname char(30) NOT NULL,
  lastname char(30) NOT NULL,
  PRIMARY KEY (personid)
) ENGINE = InnoDB;
INSERT INTO person
(
  firstname, lastname)
VALUES
(
  'Yang','Michael'
),
(
  'Zhang','Tim'
),
(
  'Yu','Phoeb'
);

CREATE TABLE address
(
  addressid int NOT NULL,
  personid int NOT NULL,
  city char(30) NULL,
  state char(30) NULL,
  PRIMARY KEY (addressid)
)ENGINE = InnoDB;
INSERT INTO address
(
  addressid, personid, city, state
)
VALUES
(
  10002,3,'Seattle','Washington'
),
(
  10003,1,'Los Angels','California'
),
(
  10001,2,'Austin','Texas'
);
```
##### 内部关联查询
```
SELECT firstname, lastname, city, state
FROM person INNER JOIN address
      ON person.personid = address.personid
```
##### 结果输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190404181930764.png)
#### 项目六：删除重复邮箱
*使用项目一的Email表*

##### 删除操作
```
DELETE FROM email
WHERE id IN (SELECT *
             FROM(SELECT MAX(id)
                  FROM email
                  GROUP BY Email
                  HAVING COUNT(Email)>1)
                  AS a);
SELECT*FROM email;
```
##### 输出结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190404181250768.png)
