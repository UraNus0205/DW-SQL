# Task03 复杂一点的查询

## 3.1 数据查询

### 3.1.1 连接查询

前面的查询都是单表查询。若一个查询同时涉及两个以上的表，则称之为连接查询。

1. 等值和非等值连接查询

   连接查询的WHERE子句中用来连接两个表的条件称为**连接条件**或**连接谓词**

   基本格式为：

   ```sql
   [<表名1>.]<列名1><比较运算符>[<表名2>.]<列名2>
   [<表名1>.]<列名1> BETWEEN [<表名2>.]<列名2> AND [<表名3>.]<列名3>
   ```

   连接运算符为等号时称为等值连接，不然就为非等值连接。

   链接谓词中的列名称称为**连接字段**。连接条件中的各**连接字段类型必须是可比的**，但名字不必相同。

   - 查询每个学生及其选修课程的情况

     ```sqL
     SELECT s.*,sc.*
     FROM s,sc
     WHERE s.sno = sc.sno；
     ```

   - 对上一个例子用**自然连接**完成(即**在等值连接中把目标列中重复的属性列去掉**)

     ```sql
     SELECT s.sno,sname,ssex,sage,sdept,cno,grade
     FROM s,sc
     WHERE s.sno = sc.sno;
     ```

   - 查询选修2号课程且成绩在90分以上的所有学生的学号和姓名

     ```sql
     SELECT s.sno,Sname
     FROM s,sc
     WHERE sc.sno=s.sno AND sc.grade>90 AND sc.cno=2
     ```

2. 自身链接

   连接操作不仅可以在两个表之间进行，也可以是<u>一个表和自己进行连接</u>，称为表的**自身连接**。

   - 查询每一门课的间接先修课（即先修课的先修课）

     ```sql
     SELECT c1.cno,c2.cpno
     FROM c c1,c c2
     WHERE c1.cpno = c2.cno
     ```

3. 外连接

   在通常的连接操作中，只有满足连接条件的元组才能作为输出结果。有时想以s表为主体列出每个学生的基本情况及其选课情况。若某个学生没有选课，扔把s的悬浮元组保存在结果关系中，而在sc表的属性上填空值NULL，这时就需要使用外连接。

   - 查询每个学生及其选修课程的情况

     ```sql
     SELECT s.*,sc.*
     FROM s LEFT OUTER JOIN sc ON(s.sno = sc.sno)
     ```

     PS:左外连接列出左边关系中所有的元组，右外连接列出右边关系中的所有元组。

4. 多表链接

   连接操作除了可以是两表连接、一个表与其自身连接外，还可以是两个以上的表进行连接，后者通常称为多表连接。

   - 查询每个学生的学号、姓名、选修的课程名及成绩

     ```sql
     SELECT s.sno,s.sname,c.cname,sc.grade
     FROM s,sc,c
     WHERE s.sno = sc.sno AND sc.cno = c.cno
     ```

### 3.1.2 嵌套查询

在SQL语言中，一个`SELECT-FROM-WHERE`语句称为一个查询块。将一个查询块嵌套在另一个查询块的WHERE子句或HAVING短语的条件中的查询称为**嵌套查询**。例如：	

```sql
SELECT sname
FROM s
WHERE sno IN
		(SELECT sno
		FROM sc
		WHERE cno=2);
```

PS:子程序的`SELECT`的语句不能使用`ORDER BY`子句，`ORDER BY`子句只能对最终查询结果排序。

1. 带有IN谓词的子查询

   - 查询与“刘晨”在同一个系学习的学生

     ```sql
     SELECT *
     FROM s
     WHERE Sdept IN
     		(SELECT sdept
     		FROM s
     		WHERE s.Sname="刘晨")
     ```

     本例中，子查询的查询条件不依赖于父查询，称为**不相关子查询**。

   - 查询选修了课程名为“信息系统”的学生学号和姓名

     ```sql
     SELECT DISTINCT s.sno,s.sname
     FROM s,sc,c
     WHERE s.sno IN
     		(SELECT sc.sno
     		FROM sc
     		WHERE sc.cno IN
     				(SELECT c.cno
     				FROM c
     				WHERE c.cname="信息系统"
     				)
     		);
     ```

     本查询同样可以用连接查询代替（并非所有嵌套查询都可以替换）：

     ```sql
     SELECT DISTINCT s.sno,s.sname
     FROM s,sc,c
     WHERE s.sno = sc.sno AND sc.cno = c.cno AND c.cname = '信息系统'
     ```

2. 带有比较运算符的子查询

   - 找出每个学生超过他自己选修课程平均成绩的课程号

     ```sql
     SELECT sno,cno
     FROM sc sc1
     WHERE grade >=
     		(SELECT AVG(grade)
     		FROM sc sc2
     		WHERE sc2.sno = sc1.sno
     		)
     ```

     本例为**相关子查询**。

3. 带有ANY（SOME）或ALL谓词的子查询

   子查询返回单值时可以用比较运算符，但返回多值时要用ANY（有的系统用SMOE）或ALL谓词修饰符。而使用ANY或ALL谓词时必须同时使用比较运算符。其语义如下：

   > `!=ANY`		不等于子查询结果中的某个值
   >
   > `!=ALL`		不等于子查询结果中的任何一个值

   - 查询非计算机科学系中比计算机科学系任意一个学生年龄小的学生姓名和年龄

     ```SQL
     SELECT sname,sage
     FROM s
     WHERE sage<ANY
     		(SELECT sage
     		FROM s
     		WHERE sdept = 'CS') 
     AND sdept != 'CS';
     ```

   - 查询非计算机科学系中比计算机科学系所有学生年龄都小的学生姓名和年龄

     ```sql
     SELECT sname,sage
     FROM s
     WHERE sage<ALL
     		(SELECT sage
     		FROM s
     		WHERE sdept = 'CS') 
     AND sdept != 'CS';
     ```

4. 带有EXISTS谓词的子查询

   `EXISTS`代表存在量词。带有**EXISTS谓词的子查询不返回任何数据，只产生逻辑真值或逻辑假值。**

   - 查询所有选修了1号课程的学生姓名

     ```sql
     SELECT sname
     FROM s
     WHERE EXISTS
     		(SELECT *
     		FROM sc
     		WHERE sc.sno = s.sno AND sc.cno='1');
     ```

     由`EXISTS`引出的子查询，其目标列表达式通常都用*，因为带EXISTS的子查询只返回真值或假值，给出列名无实际意义。

   - 查询没有选修1号课程的学生姓名

     ```sql
     SELECT sname
     FROM s
     WHERE NOT EXISTS
     		(SELECT *
     		FROM sc
     		WHERE  s.sno = sc.sno AND sc.cno='1');
     ```

   - 查询选修了全部课程的学生姓名

     即没有课程是这个学生不选的

     ```sql
     SELECT sname
     FROM s/*学生为主语*/
     WHERE NOT EXISTS/*这个学生没有没选修的课*/
     		(SELECT */*学生没选修的课*/
     		FROM c
     		WHERE NOT EXISTS
     				(SELECT *
     				FROM sc
     				WHERE sc.sno = s.sno AND sc.cno = c.cno/*学生选修了cno课号的课*/
     				)
     		);
     ```

   - 查询被全部学生选修的课程名

     ```sql
     SELECT cname
     FROM c/*课程为主语*/
     WHERE NOT EXISTS/*这个课程没有没选修的学生*/
     		(SELECT */*没选修的学生*/
     		FROM s
     		WHERE NOT EXISTS
     				(SELECT *
     				FROM sc
     				WHERE sc.sno = s.sno AND sc.cno = c.cno/*学生选修了cno课号的课*/
     				)
     		);
     ```

   - 查询至少选修了学生201215122选修的全部课程的学生学号

     即没有课程是201215122选的而另一个学生没选的 

     ```sql
     SELECT DISTINCT sno
     FROM sc scx
     WHERE EXISTS
     		(SELECT *
     		FROM sc scy
     		WHERE scy.sno='201215122' AND EXISTS
     				(SELECT *
     				FROM sc scz
     				WHERE scx.sno = scz.sno AND scy.cno = scz.cno
     				)
     		);
     ```

### 3.1.3 集合查询

SELECT语句的查询结果是元组的集合，所以多个SELECT语句的结果可以进行集合操作。**集合操作主要包括并操作`UNION`、交操作`INTERSECT`和差操作`EXCEPT`。**

<u>参加集合操作的各查询结果的列数必须相同；对应项的数据类型也必须相同。</u>

- 查询计算机科学系的学生及年龄不大于19岁的学生

  ```sql
  SELECT *
  FROM s
  WHERE sdept='CS'
  UNION
  SELECT *
  FROM s
  WHERE sage<=19
  ```

  `UNION`将多个查询结果合并起来时会自动去掉重复元组。如果要保留重复元组则可以使用`UNION ALL`操作符。

- 查询选修了课程1或选修了课程2的学生

  ```sql
  SELECT *
  FROM sc
  WHERE cno='1'
  UNION
  SELECT *
  FROM sc
  WHERE cno='2'
  ```

- 查询计算机科学系的学生与年龄不大于19岁的学生的交集

  ```sql
  SELECT *
  FROM s
  WHERE sdept='CS'
  INTERSECT
  SELECT *
  FROM s
  WHERE sage<=19
  ```

- 查询即选修了课程1又选修了课程2的学生。

  ```sql
  SELECT *
  FROM sc
  WHERE cno='1'
  INTERSECT
  SELECT *
  FROM sc
  WHERE cno='2'
  ```

- 查询计算机科学系的学生与年龄不大于19岁的学生的差集

  ```sql
  SELECT *
  FROM s
  WHERE sdept='CS'
  EXCEPT
  SELECT *
  FROM s
  WHERE sage<=19
  ```

### 3.1.4 基于派生表的查询

子查询不仅可以出现在`WHERE`子句中，也可以出现在`FROM`子句中，这使子查询生成的临时**派生表**成为主查询的查询对象

- 查询每个学生超过他自己选修课程平均成绩的课程号

  ```sql
  SELECT cno
  FROM sc,(SELECT sno,Avg(grade) FROM sc GROUP BY sno ) AS Avg_sc(avg_sno,avg_grade)
  WHERE sc.sno = Avg_sc.avg_sno AND sc.grade > Avg_sc.avg_grade;
  ```

- 查询所有选修了1号课程的学生姓名

  ```sql
  SELECT sname
  FROM s,(SELECT sno FROM sc WHERE cno='1') AS sc1
  WHERE s.sno = sc1.sno
  ```