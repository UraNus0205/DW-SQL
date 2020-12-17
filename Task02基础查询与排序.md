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

     