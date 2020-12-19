# Task02 基础查询与排序

首先建立本文需要的表

```sql
CREATE TABLE s/*学生表*/
(sno CHAR(9) PRIMARY KEY,/*列级完整性约束条件，Sno是主码*/
 sname CHAR(20) UNIQUE,/*Sname取唯一值*/
 ssex CHAR(2),
 sage SMALLINT,
 sdpet CHAR(20)
);
CREATE TABLE c/*课程表*/
(cno CHAR(4) PRIMARY KEY,
 cname CHAR(40) NOT NULL,/*cname不能为空*/
 cpno char(4),/*先修课*/
 ccredit SMALLINT,
 FOREIGN KEY(cpno) REFERENCES c(cno)/*表级完整性约束条件，cpno是外码，被参照表是c，被参照列是cno*/
);
CREATE TABLE sc
(sno char(9),
 cno char(4),
 grade SMALLINT,
 PRIMARY KEY(sno,cno),/*主码由两个属性构成，必须作为表级完整性进行定义*/
 FOREIGN key(sno) REFERENCES s(sno),
 /*sno是外码，被参照表是s*/
 FOREIGN key(cno) REFERENCES c(cno)
 /*cno是外码，被参照表是c*/
);
```

![image-20201217205717598](C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20201217205717598.png)

![image-20201217205728672](C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20201217205728672.png)

![image-20201217205736381](C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20201217205736381.png)

本例说明参照表与被参照表可以是同一个表



## 2.1.单表查询

### 2.1.1 选择表中的若干列

`SELECT`语法

```sql
SELECT <列名>, 
  FROM <表名>;
```



1. 查询指定列

   - 查询全体学生的学号与姓名

     ```sql
     SELECT sno,sname
     FROM s;
     ```

   - 查询全体学生的姓名、学号、所在系

     ```sql
     SELECT sname,sno,sdept
     FROM s;
     ```

2. 查询全部列

   - 查询全体学生的详细记录

     ```sql
     SELECT *
     FROM s;
     ```

3. 查询经过计算的值

   - 查询全体学生的姓名，出生年份和所在院系，要求用小写字母表达

     ```sql
     SELECT Sname,'Year of Birgh:',2020-Sage BIRTHDAY,
     			 LOWER(Sdept) DEPARTMENT
     FROM s;
     ```

     用户可以通过指定别名来改变查询结果的列标题

     ![image-20201217211458224](C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20201217211458224.png)

### 2.1.2 选择表中的若干元组

1. 消除取值重复的行

   ```sql
   SELECT DISTINCT sno
   FROM sc;
   ```

2. 查询满足条件的元组

   查询满足指定条件的元组可以通过`WHERE`字句实现。

   (1)比较大小

   ​	用于比较的运算符一般包括基本的等于、大于、小于、大于等于，小于等于，！=或<>（不等于），！>（不大于），！<（不小于）。

   - 查询计算机科学系全体学生的信息

     ```sql
     SELECT *
     FROM s
     WHERE sdept='cs'
     ```

   - 查询所有年龄在20岁以下的学生姓名及其年龄

     ```sql
     SELECT sno,sage
     FROM s
     WHERE sage<20
     ```

   - 查询考试成绩不及格的学生的学号

     ```sql
     SELECT DISTINCT sno
     FROM sc
     WHERE grade<60
     ```

   (2)确定范围

   ​	`BETWEEN...AND...`和`NOT BETWEEN ... AND...`可以用来查找属性值在（或不在）指定范围内的元组。

   - 查询年龄不在20~23岁之间的学生姓名、系别和年龄

     ```sql
     SELECT sname,sdept,sage
     FROM s
     WHERE sage NOT BETWEEN 20 AND 23;
     ```

   (3)确定集合

   ​	`IN`可以用来查找属性值属于指定集合的元组，与之相对的是`NOT IN`

   - 查询CS系、MA系和IS系学生的姓名和性别

     ```sql
     SELECT sname,ssex
     FROM s
     where sdept IN('CS','MA','IS');
     ```

   - 查询不在CS系、MA系和IS系学生的姓名和性别

     ```sql
     SELECT sname,ssex
     FROM s
     where sdept NOT IN('CS','MA','IS');
     ```

   (4)字符匹配

   ​	`LIKE`可以用来进行字符串的匹配。一般语法格式如下：

   ```SQL
   [NOT] LIKE '<匹配串>' ['ESCAPE'<换码字符>']
   ```

   其含义是查找指定的属性列值与<匹配串>相匹配的元组。<匹配串>可以是一个完整的字符串，也可以有通配符%和_。其中：

   - %代表任意长度（可以为0）的字符串。例如a%b表示以a开头，以b结尾的任意长度的字符串。如acb、addgb、ab等都满足该字符串。

   - _代表任意单个字符。例如a_b表示以a开头，以b结尾的长度为3的任意字符串。如acb、afb等都满足该字符串。

   - 查询学号为201215121的学生的详细情况

     ```sql
     SELECT *
     FROM s
     WHERE sno LIKE '201215121';
     ```

     等价于

     ```sql
     SELECT *
     FROM s
     WHERE sno = '201215121';
     ```

   - 查询所有姓刘的学生的姓名、学号和性别

     ```sql
     SELECT sname,sno,ssex
     FROM s
     WHERE sname LIKE '刘%';
     ```

   - 查询姓“欧阳”且全名为三个汉字的学生的姓名

     ```sql
     SELECT sname
     FROM s
     WHERE sname LIKE '欧阳_';
     ```

   - 查询名字中第二个字为“阳”的学生的姓名和学号

     ```sql
     SELECT sname,sno
     FROM s
     WHERE sname LIKE '_阳%';
     ```

   - 查询DB_Design课程的课程号和学分

     ```sql
     SELECT cno,ccredit
     FROM c
     WHERE cname LIKE 'DB\_Design' ESCAPE'\'';
     ```

     `ESCAPE'\'`表示“\”为换码字符。这样匹配串中紧跟在“\”后面的字符“_”不再具有通配符的意义，而是转义为普通的字符。

   - 查询以“DB_”开头，且倒数第三个字符为i的课程的详细情况

   ```sql
   SELECT *
   FROM c
   where cname LIKE "DB\_%i__" ESCAPE '\'';
   ```

   (5)涉及空值的查询

   - 某些学生选修课程后没有参加考试，所以有选课记录，但没有考试成绩。查询缺少成绩的学生的学生的学号和相应的课程号

     ```sql
     SELECT sno,cno
     FROM sc
     WHERE grade IS NULL;
     ```

   - 查询所有有成绩的学生学号和课程号

     ```sql
     SELECT sno,cno
     FROM sc
     WHERE grade IS NOT NULL;
     ```

   (6)多重条件查询

   ​	逻辑运算符`AND`和`OR`可以用来连接多个查询条件。`AND`优先级高于`OR`，但用户可以用括号改变优先级

   - 查询计算机科学系年龄在20岁以下的学生姓名

     ```sql
     SELECT sname
     FROM s
     WHERE sdept='CS' AND sage<20;
     ```

   - “(3)确定集合”里的第一个例题中的`IN`实际上是多个`OR`运算符的缩写，因此该例可改写成如下等价形式

     ```sql
     SELECT sname,ssex
     FROM s
     WHERE sdept='CS' OR sdept='MA' OR sdept='IS';
     ```

     ### 

### 2.1.3 ORDER BY 子句

用户可以用`ORDER BY`子句对查询结果按一个或多个属性列的升序或降序排列，默认为升序。

- 查询选修了三号课程学生的学号及其成绩，查询结果按分数的降序排列

  ```sql
  SELECT sno,grade
  FROM sc
  WHERE cno='3'
  ORDER BY grade DESC
  ```

- 查询全体学生学生情况，查询结果按所在系的系号升序排列，同一系中的学生按年龄降序排列

  ```sql
  SELECT *
  FROM S
  ORDER BY sdept,sage DESC
  ```

  PS:**空值NULL可以当最大值处理**

### 2.1.4 聚集函数

SQL中用于汇总的函数叫做聚合函数。以下是最常用的聚合函数：

- COUNT(*)：统计元组个数

- COUNT([DISTINCT|ALL]<列名>)：计算一列中值的个数
- SUM([DISTINCT|ALL]<列名>)：计算表中**数值**列中数据的合计值
- AVG([DISTINCT|ALL]<列名>)：计算表中**数值**列中数据的平均值
- MAX([DISTINCT|ALL]<列名>)：求出表中任意列中数据的最大值
- MIN([DISTINCT|ALL]<列名>)：求出表中任意列中数据的最小值

若指定DISTINCT短语，则表示在计算时要取消指定列中的重复值。

- 查询学生总人数

  ```sql
  SELECT COUNT(*)
  FROM s
  ```

- 查询选修了课程的人数

  ```sql
  SELECT COUNT(DISTINCT sno)
  FROM sc
  ```

- 计算选修1号课程的学生平均成绩

  ```sql
  SELECT AVG(grade)
  FROM sc
  WHERE cno='1'
  ```

- 查询选修1号课程的学生最高分数

  ```sql
  SELECT MAX(grade)
  FROM sc
  WHERE cno='1'
  ```

- 查询学生201215121选修课程的总学分数

  ```sql
  SELECT SUM(ccredit)
  FROM sc,c
  WHERE sc.sno='201215121' AND sc.cno=c.cno
  ```

  PS：1.聚集函数遇到空值时，除了COUNT(*)外，都跳过空值只处理非空值。

  ​		2.WHERE子句不能用聚集函数作为表达式，**聚集函数只能用于SELECT子句和GROUP BY中的HAVING子句。**

### 2.1.5 GROUP BY子句

GROUP BY子句将查询结果按某一列或多列的值分组，值相等的为一组。

对查询结果分组的目的是为了**细化聚集函数的作用对象。**

- 求各个课程号及相应的选课人数

  ```sql
  SELECT cno,COUNT(sno)
  FROM sc
  GROUP BY cno
  ```

- 查询选修了三门以上课程的学生学号

  ```sql
  SELECT sno
  FROM sc 
  GROUP BY sno
  HAVING COUNT(cno)>3
  ```

- 查询平均成绩大于等于90分的学生学号和平均成绩

  ```sql
  SELECT sno,AVG(grade)
  FROM sc
  GROUP BY sno
  HAVING AVG(grade)>90
  ```

  

