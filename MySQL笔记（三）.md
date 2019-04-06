
## MySQL笔记（三）
#### 之前有很多知识有漏洞，在此完善，相当于进阶篇

##### 将数据表导出csv格式
```
SELECT * FROM world
INTO OUTFILE 'C:/ProgramData/MySQL/MySQL Server 5.7/Uploads/world.csv'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';
```
此处经常报错：
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

需要输入以下命令检查MySQL信任的目录
```
SHOW variables like '%secure%';
```
得到安全的导出导入目录后，使用完整路径即可完成导出。

##### 将csv文件导入MySQL
```
CREATE TABLE newworld
(
name char(100) NOT NULL,
continent char(100) NOT NULL,
area char(50) NOT NULL,
population int NOT NULL,
gdp int NULL
);
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 5.7/Uploads/world.csv'
INTO TABLE newworld
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';
```
#### 项目七：各部门工资最高的员工

```
SELECT Department, Employee.Name, Salary
FROM Department AS d and  Employee AS e
WHERE d.Id = e.DepartmentId AND e.Id IN (SELECT Id, MAX(Salary)
FROM Employee
GROUP BY DepartmentId)
```
先从Employee表中检索出员工名和对应工资
```
SELECT Id, Name, Salary FROM Employee
WHERE Salary IN (SELECT MAX(Salary) FROM Employee GROUP BY DepartmentId)
GROUP BY DepartmentId;
```
然后联结两张表后的版本（值得反复阅读）
```
SELECT Department.Name AS Department, Employee.Name AS Employee, Salary 

FROM Employee INNER JOIN Department
ON Employee.DepartmentId = Department.Id

WHERE Salary IN (SELECT MAX(Salary) FROM Employee GROUP BY DepartmentId)
ORDER BY Salary DESC;
```
###### 结果输出
![5362043e4bc3c2c4353d730e8329b284.png](en-resource://database/542:1)
#### 项目八：调换座位
主要考察高级函数CASE... WHEN ...WHEN...ELSE...END的使用
###### 建表并插入数据
```
CREATE TABLE seat
(
id int NOT NULL,
student char(50) NOT NULL
)ENGINE InnoDB;

INSERT INTO seat
VALUES
(
1,'Abbot'
),
(
2,'Doris'
),
(
3,'Emerson'
),
(
4,'Green'
),
(
5,'Jeames'
);
```
直接使用SELECT返回想要的新表，这里无法UPDATE因为要更新seat但是计数的时候也要检索seat，还不知道该怎么办。
```
SELECT(
CASE
WHEN id%2 = 1 AND id != (SELECT COUNT(*) FROM seat) THEN id+1
WHEN id%2 = 0 THEN id-1
ELSE id
END) AS id, student
FROM seat
ORDER BY id;
```
###### 结果输出
![bc3fc1e5b2eda20b701c0b2c339434a3.png](en-resource://database/546:0)

#### 项目九：分数排名（一）难度：中

###### 建表并插入数据
```
CREATE TABLE score
(
Id int NOT NULL,
Score float(4,2) NOT NULL,
PRIMARY KEY (Id)
)ENGINE InnoDB;

INSERT INTO score
VALUES
(
1,3.50
),
(
2,3.65
),
(
3,4.00
),
(
4,3.85
),
(
5,4.00
),
(
6,3.65
);
```
本题获取RANK的思路是大于等于该分数（DISTINCT）的人数
```
SELECT*FROM score;
SELECT score,
(SELECT COUNT(DISTINCT score)
        FROM score AS s1
        WHERE s1.score >= s2.score) AS RANK
FROM score AS s2
ORDER BY score DESC;
```
###### 结果输出
![b3aa293377c1dba96de211dc11411963.png](en-resource://database/544:0)
#### 项目十：行程与用户     （难度：高）
###### 建表users
注意枚举类型的设置句法
```
CREATE TABLE users
(
user_id int NOT NULL,
banned enum ('Yes','No') NOT NULL,
role enum ('client','driver','partener') NOT NULL,
PRIMARY KEY (user_id)
)ENGINE InnoDB;
INSERT INTO users
VALUES(1,'No','client'),
(2,'Yes','client'),
(3,'No','client'),
(4,'No','client'),
(10,'No','driver'),
(11,'No','driver'),
(12,'No','driver'),
(13,'No','driver');
```
###### 建表trips
注意设置外键的句法
```
CREATE TABLE trips
(
id int NOT NULL AUTO_INCREMENT,
client_id int NOT NULL,
driver_id int NOT NULL,
city_id int NOT NULL,
stat enum('completed','cancelled_by_driver','cancelled_by_client') NOT NULL,
request_at Date NOT NULL,
PRIMARY KEY (id),
FOREIGN KEY (client_id) REFERENCES users(user_id),
FOREIGN KEY (driver_id) REFERENCES users(user_id)
)ENGINE InnoDB;


INSERT INTO trips
(client_id, driver_id, city_id, stat, request_at)
VALUES(1,10,1,'completed','2013-10-01'),
(2,11,1,'cancelled_by_driver','2013-10-01'),
(3,12,6,'completed','2013-10-01'),
(4,13,6,'cancelled_by_client','2013-10-01'),
(1,10,1,'completed','2013-10-02'),
(2,11,6,'completed','2013-10-02'),
(3,12,6,'completed','2013-10-02'),
(2,12,12,'completed','2013-10-03'),
(3,10,12,'completed','2013-10-03'),
(4,13,12,'cancelled_by_driver','2013-10-03');
```
我已经被虐哭了你呢？
目前的体会就是，先把想要的结果SELECT出来，然后再用表间的联结设置筛选条件，可得好好反复体会啊！
```
SELECT t.request_at AS Day, ROUND(SUM(
                                CASE
                                WHEN t.stat = 'completed' THEN 0
                                ELSE 1
                                END),2)/ COUNT(*) AS 'Cancellation Rate'
FROM trips AS t LEFT OUTER JOIN users AS u1
ON t.client_id = u1.user_id
LEFT OUTER JOIN users AS u2
ON t.driver_id = u2.user_id
WHERE u1.banned = 'No' AND u2.banned = 'No' AND t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.request_at;
```
###### 输出结果
![cca4020ec988bd80dcbb18b2474e6e10.png](en-resource://database/550:0)
#### 项目十一：各部门工资前三高的员工（难度：中）
对之前的Employee表进行补充
```
INSERT INTO Employee
(id,name,salary,DepartmentId)
VALUES(5,'Janet',69000,1),
(6,'Randy',85000,1);
```
排序取前N名的算法好好体会
```
SELECT d.Name AS Department, e1.Name AS Employee, Salary
FROM Department AS d JOIN Employee AS e1
ON d.id = e1.DepartmentId
WHERE (SELECT COUNT(DISTINCT e2.Salary)
       FROM Employee AS e2
       WHERE e2.Salary >= e1.Salary AND e2.DepartmentId = e1.DepartmentId)<=3
GROUP BY Department, Salary DESC;
```

#### 项目十二：分数排名（二）难度：中
```
SELECT  score, (SELECT COUNT(*)+1
                FROM score s1
                WHERE s1.score>s2.score) AS RANK
FROM score s2
ORDER BY s2.score DESC;
```
###### 输出结果
![image.png](data:image/png;base64,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)
