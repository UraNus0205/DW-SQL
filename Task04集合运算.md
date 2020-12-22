# Task04 集合运算

## 4.1 集合查询

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

## 4.2 数据更新

### 4.2.1 插入数据

1. 插入元组

   插入元组的`INSERT`语句的格式为：

   ```sql
   INSERT
   INTO <表名> [<属性列1> [,<属性列2>]...]
   VALUES (<常量1> [,<常量2>]...);
   ```

   其功能即为将新元组插入指定表中。其中新元组的属性列1的值为常量1，属性列2的值为常量2，...。`INTO`子句中没有出现的属性列，新元组在这些列上将取空值。必须注意的是，在表定义时说明了NOT NULL的属性列不能取空值，否则会出错。

   **如果`INTO`子句中没有指明任何属性列名，则新插入的元组必须在每个属性列上均有值。**

   - 将一个新学生元组（学号：201215128，姓名：陈冬，性别：男，所在系：IS，年龄：18岁）插入到s表中

     ```sql
     INSERT
     INTO s(sno,sname,ssex,sdept,sage)
     VALUES('201215128','陈冬','男','IS',18)
     ```

     `INTO`子句中指明了新增加的元组在哪些属性上要赋值，**属性的顺序可以与`CREATE TABLE中`的顺序不一样。**

   - 将学生张成民的信息插入到s表中

     ```sql
     INSERT
     INTO s
     VALUES('201215126','张成民','男',18,'CS')
     ```

     与上一个例子不同的是在`INTO`子句中只指出了表名，没有指出属性名。这表示新元组要在表的所有属性列上都指定值，属性列的次序与`CREATE TABLE`中的次序一致。

   - 插入一条选课记录（‘201215128’，‘1’）

     ```sql
     INSERT
     INTO sc
     VALUES('201215128','1',NULL)
     ```

     因为没有指出sc的属性名，所以在grade列上要明确给出空值。

     或者：

     ```sql
     INSERT
     INTO sc(sno,cno
     VALUES('201215128','1')
     ```

     DBMS会在新插入记录的grade列上自动地赋空值。

2. 插入子查询结果

   - 对每一个系，求学生的平均年龄，并把结果存入数据库

     ```sql
     CREATE TABLE dept_age
     		(sdept CHAR(15),
     		avg_age SMALLINT);
     
     INSERT
     INTO dept_age
     SELECT sdept,AVG(sage)
     FROM s
     GROUP BY sdept;
     ```

### 4.2.2 修改数据

修改操作又称为更新操作，其语句的一般格式为：

```sql
UPDATE <表名>
SET <列名>=<表达式> [,<列名>=<表达式>]...
[WHERE <条件>];
```

其功能是修改指定表中满足WHERE子句条件的元组。其中SET子句给出<表达式>的值用于取代相应的属性列值。**若省略WHERE子句，则表示要修改表中的所有元组。**

1. 修改某一个元组的值

   - 将学生201215121的年龄改为22岁

     ```sql
     UPDATE s
     SET sage=22
     WHERE sno='201215121'
     ```

2. 修改多个元组的值

   - 将所有学生的年龄增加1岁

   ```sql
   UPDATE s
   SET sage=sage+1
   ```

3. 带子查询的修改语句

   - 将CS系全体学生的成绩置零

     ```sql
     UPDATE sc
     SET grade=0
     WHERE sc.sno IN
     		(SELECT sno
     		FROM s
     		WHERE s.sdept='cs')
     ```

     或者

     ```sql
     UPDATE sc
     SET grade=0
     WHERE 'CS' = (
     		SELECT cno
     		FROM s
     		WHERE s.sno = sc.sno)
     ```

### 4.2.3 删除数据

格式：

```sql
DELECT
FROM <表名>
[WHERE <条件>];
```

**`DELECT`语句删除的是表中的数据，而不是关于表的定义。**

1. 删除某一元组的值

   - 删除学号为201215128的学生选课记录

     ```sql
     DELETE
     FROM sc
     WHERE sc.sno='201215128' 
     ```

   PS：**若sc表的该学生记录未删除，则s表中的该学生记录无法直接删除，因为sno是s的主码、sc的外码。**

2. 删除多个元组的值

   - 删除所有的系平均年龄记录

     ```sql
     DELETE
     FROM dept_age
     ```

3. 带子查询的删除语句

   - 删除CS系所有学生的选课记录

     ```sql
     DELETE
     FROM sc
     WHERE sno IN(
     		SELECT sno
     		FROM s
     		WHERE sdept='cs')
     ```

<u>对某个数据的增、删、改操作可能会破坏参照完整性。</u>

## 4.3 视图

视图是从一个或几个基本表（或视图）导出的表。它与基本表不同，是一个**<u>虚表</u>**。**数据库只存放视图的定义，而不存放视图对应的数据，这些数据仍存放在原来的基本表中。**所以一旦基本表中的数据发生变化，从视图中查询出的数据也就随之改变了。（视图就像一个窗口，对应模式结构的**外模式**）

### 4.3.1 定义视图

1. 建立视图

   格式：

   ```sql
   CREATE VIEW <视图名> [(<列名> [,<列名>]...)]
   AS <子查询>
   [WITH CHECK OPTION];
   ```

   `WITH CHECK OPTION`表示对视图进行更删改操作时要保证更新、插入或删除的行满足试图定义中的谓词条件（即子查询中的条件表达式）。

   组成视图的属性列名或者全部省略或者全部指定，没有第三种选择。如果省略了视图的各个属性列名，则隐含该视图由子查询中`SELECT`子句目标列中的各个字段组成。但在下面三种情况下必须明确指定组成视图的所有列名：

   ①某个目标列不是单纯的属性名，而是聚集函数或列表达式；

   ②多表连接时选出了几个同名列作为视图的字段；

   ③需要在视图中为某个列启用新的更合适的名字。

   - 建立信息系学生的视图

     ```sql
     CREATE VIEW IS_s
     AS
     SELECT sno,sname,sage
     FROM s
     WHERE sdept='IS'
     ```

     DBMS执行`CREATE VIEW`语句的结果只是把视图的定义存入数据字典，并不执行其中的`SELECT`语句。只是在对视图查询时，才按视图的定义从基本表中将数据查出。

   - 建立信息系学生的视图，并要求进行修改和插入操作时仍需保证该视图只有信息系的学生。

     ```sql
     CREATE VIEW IS_s
     AS
     SELECT sno,sname,sage
     FROM s
     WHERE sdept='IS'
     WITH CHECK OPTION
     ```

     由于在定义IS_s视图时加上了`WITH CHECK OPTION`子句，以后对该视图进行插入、修改和删除操作时，BBMS会自动加上`sdept='IS'`的条件。

     若一个视图是从单个基本表导出的，并且只是去掉了基本表的某些行和某些列，但保留了主码，则称这类视图为**行列子集视图。**本例的视图即为一个行列子集视图。

     视图不仅可以建立在单个基本表上，也可以建立在多个基本表上。

   - 建立信息系选修了1号课程的学生的视图（包括学号、姓名、成绩）。

     ```sql
     CREATE VIEW IS_s1(sno,sname,grade)
     AS
     SELECT s.sno,sname,sage
     FROM s,sc
     WHERE sdept='IS'AND s.sno=sc.sno AND sc.cno='1'
     ```

     视图也可以建立在**一个或多个已定义好的视图上，或建立在基本表与视图上。**

   - 建立信息系选修了1号课程且成绩在90分以上的学生的视图

     ```sql
     CREATE VIEW IS_s2
     AS
     SELECT sno,sname,grade
     FROM IS_s1
     WHERE grade>=90
     ```

   - 定义一个反映学生出生年份的视图

     由于视图中的数据并不实际存储，所以定义视图时可以根据应用的需要设置一些派生属性列。这些派生属性由于在基本表中并不实际存在，也称它们为虚拟列，这就减少了数据库的冗余数据。**带虚拟列的视图也称为带表达式的视图。**

     ```sql
     CREATE VIEW BT_s(sno,sname,sbirth)
     AS
     SELECT sno,sname,2020-Sage
     FROM s
     ```

   - 将学生的学号及平均成绩定义为一个视图

     还可以用带有聚集函数和`GROUP BY`子句的查询来定义视图，这种视图称为**分组视图**。

     ```sql
     CREATE VIEW S_G(sno,gavg)
     AS
     SELECT sno,AVG(grade)
     FROM sc
     GROUP BY sno
     ```

   - 将s表中所有女生记录定义为一个视图

     ```sql
     CREATE VIEW F_S(F_sno,name,sex,age,dept)
     AS
     SELECT *
     FROM s
     WHERE ssex='女'
     ```

     这里的视图F_S是由子句`SELECT *`建立的。F_S视图的属性列和S的属性列一一对应。若以后修改了基本表S的结构，二者的映像关系会被破坏，该视图就不能正常工作了。为避免该问题，最好在修改基本表后删除该视图，然后重建该视图。

2. 删除视图

   格式：

   ```
   DROP VIEW <视图名> [CASCADE];
   ```

   **视图删除后视图的定义将从数据字典中删除。**如果该视图上还导出了其他视图，则使用CASCADE级联删除语句将该视图和由它导出的所有视图一起删除。

   **基本表删除后，由该基本表导出的所有视图均无法使用了，但是视图的定义没有从字典中清除。删除这些试图定义需要显示地使用`DROP VIEW`语句。**

   - 删除视图BT_S和视图IS_S1

     ```sql
     DROP VIEW BT_S;
     DROP VIEW IS_S1 CASCADE;/*IS_S1视图上还导出了IS_S2，必须使用级联删除语句*/
     ```

### 4.3.2 查询视图

- 在IS系学生的视图中找出年龄小于20岁的学生

  ```sql
  SELECT * 
  FROM IS_S
  WHERE sage<20;
  ```

- 查询选修了1号课程的信息系学生

  ```sql
  SELECT * 
  FROM IS_S,sc
  WHERE IS_S.sno = sc.sno AND sc.cno='1'
  ```

- 在S_G视图中查询平均成绩在90分以上的学生学号和平均成绩

  ```sql
  SELECT *
  FROM S_G
  WHERE gavg>=90
  ```

  或者：

  ```sql
  SELECT sno,AVG(grade)
  FROM sc
  GROUP BY sno
  HAVING AVG(grade)>=90
  ```

  定义视图并查询视图与基于派生表的查询是有区别的。**视图一旦定义，其定义将永久保存在数据字典中，之后的所有查询都可以直接引用该视图。****而派生表只是在语句执行时临时定义，语句执行后该定义即被删除。**

### 4.3.3 更新视图

由于视图是不实际存储数据的虚表，因此对视图的更新最终要转换为对基本表的更新。

- 将IS系学生视图IS_S中学号为‘201215122'的学生姓名改为“刘辰”

  ```sql
  UPDATE IS_S
  SET sname='刘辰'
  WHERE sno='201215122'
  ```

- 向IS系学生视图IS_S中插入一个新的学生记录，其中学号为“201215129”，姓名为赵新，年龄为20岁

  ```sql
  INSERT
  INTO IS_S
  VALUES('201215129','赵新',20)
  ```

- 删除IS系学生视图IS_S中学号为“201215129”的记录

  ```sql
  DELETE
  FROM IS_S
  WHERE sno='201215129'
  ```

### 4.3.4 视图的作用

1. 视图可以简化用户的操作

2. 视图使用户能以多种角度看待同一数据

3. 数据对重构数据库提供了一定程度的逻辑独立性

4. 视图能够对机密数据提供安全保护

5. 适当利用视图可以更清晰地表达查询

   - 对每个同学找出他获得最高成绩的课程号

     ```sql
     SELECT sno,cno
     FROM sc
     GROUP BY sno
     HAVING MAX(grade)
     ```

     或者：

     ```sql
     CREATE VIEW VMGRADE
     AS
     SELECT sno,MAX(grade) Mgrade
     FROM sc
     GROUP BY sno;
     
     SELECT sc.sno,cno
     FROM sc,VMGRADE
     WHERE sc.sno = VMGRADE.sno AND sc.grade = VMGRADE.Mgrade;
     ```

     

