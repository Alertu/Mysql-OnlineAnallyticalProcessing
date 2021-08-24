### MySQL窗口函数（版本MySQL 8.0+）



#### 窗口函数类型：

##### 1.rank() over(partition by Field order by Field desc)

###### 注：

1. **rank() over(order by Field desc)** ：计算排名时会根据order by 中的字段进行排序并输出排名值，**如果遇到数据相同排名并列时**：**排名相同。**下一个非并列排名等于前一行的排序 +N 其中：N等于排名并列多少行；
2. **例如：rank：1 1 3 4 4 6 7 8 8 10**
3. partition by Field : 根据字段进行分组然后进行排名计算,只对每一个分组的数据进行排名

##### 2.row_number() over(partition by Field order by Field desc)

###### 注:

1. **row_number(order by Field desc)** : 计算排名时会根据order by 中的字段进行排序并输出排名值,**如果遇到数据相同排名并列时 : 会根据顺序累加,排序不会间断**
2. **例如:1 2 3 4 5 6 7 8 9 10** 
3. partition by : 根据字段进行分组进行排名,只计算当前在各个分组中的排名

##### 3.dense_rank() over(partition by Field order by Field desc)

###### 注:

1. dence_rank() over(order by Field desc): 计算排名时会根据order by 中的字段进行排序并输出排名值,**如果遇到数据相同排名并列时 : 排名相同. 下一个非并列的排名根据前一个排名进行连续排名**

2. **例如 : 1 2 2 3 4 5 5 5 6 7 8 8 8 8 9 10**

3. partition by : 根据字段进行分组计算排名,只计算当前分组中的分组也就是内部分组

   

#### 注意: partition by Field 可以不填,如果不填则不进行组内排名



#### 实例(力扣数据库185题):

```mysql
Employee 表包含所有员工信息，每个员工有其对应的工号 Id，姓名 Name，工资 Salary 和部门编号 DepartmentId 
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+
Department 表包含公司所有部门的信息。

+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

#### 问题:

1.计算所有部门的工资排名,如果工资相同则排名并列,下一个排名不连续

解:

```mysql
SELECT
	a.*,
	rank () over ( ORDER BY a.Salary DESC ) AS 'rank' 
FROM
	Employee AS a
```

输出:

![image-20210819175327771](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210819175327771.png)

2.计算所有部门工资排名,如果工资相同则排名连续

解:

```mysql
SELECT
	a.*,
	row_number () over ( ORDER BY a.Salary ) AS 'rank' 
FROM
	Employee AS a
```

输出:

![image-20210819181725553](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210819181725553.png)

3.计算所有部门的工资,如果工资相同则并列,下一名连续

```mysql
SELECT
	a.*,
	dense_rank () over ( ORDER BY a.Salary DESC ) as 'rank'
FROM
	Employee AS a
```

输出:

![image-20210819182737758](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210819182737758.png)

4.计算员工在各个部门中的工资,如果遇到工资排名相同下一个排名连续

```mysql
SELECT
	a.*,
	dense_rank () over ( PARTITION BY a.DepartmentId ORDER BY a.Salary ) as 'rank'
FROM
	Employee AS a
```

输出:

![image-20210819183342874](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210819183342874.png)

5.计算员工在各个部门的工资排名。如果遇到工资相同则排名相同不连续排名

```mysql
SELECT
	a.*,
	rank () over ( PARTITION BY a.DepartmentId ORDER BY a.Salary ) AS 'rank' 
FROM
	Employee AS a
```

输出：

![image-20210820091602875](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210820091602875.png)

6.计算员工在各个部门中的排名，如果遇到工资相同则排名不相同排名连续

```mysql
SELECT
	a.*,
	row_number () over ( PARTITION BY a.DepartmentId ORDER BY a.Salary ) AS 'rank' 
FROM
	Employee AS a
```

输出：

![image-20210820091857248](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210820091857248.png)

那么问题来了，找出每个部门获得前三高工资的所有员工。例如，根据上述给定的表，查询结果应返回： 

```mysql
 +------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+

# 解释： 
#
# IT 部门中，Max 获得了最高的工资，Randy 和 Joe 都拿到了第二高的工资，Will 的工资排第三。销售部门（Sales）只有两名员工，Henry
# 的工资最高，Sam 的工资排第二。 
```

解：

```mysql
/*** 注释* 子查询 + 窗口函数 （先根据分组排名（子查询），子查询完成后再根据子查询结果进行筛选）* from型子查询：把内层的查询结果当成临时表，供外层sql再次查询。查询结果集可以当成表看待。临时表要使用一个别名。* 子查询我的理解：如下文查询，from子查询查询完成后 可以当成一个新的表（或者叫临时表）这是内层查询，最外层查询就查询当前临时表或者叫新表的数据进行数据操作* 如下文：先用 子查询构建一个名字为P的表，表中的字段也就是employee.* 和 dense_rank() over(order by )的字段，在根据这个表进行数据操作  如图演示**/SELECT	b.`Name` AS Department,	p.NAME AS Employee,	p.Salary FROM	( SELECT e.*, dense_rank () over ( PARTITION BY e.DepartmentId ORDER BY e.Salary DESC ) AS 'rank' FROM employee AS e ) AS p	LEFT JOIN department AS b ON p.DepartmentId = b.Id WHERE	3 >= p.rank
```



###### 这是子查询运行后的数据一个名为P的新表

```mysql
SELECT	* FROM	( SELECT e.*, dense_rank () over ( PARTITION BY e.DepartmentId ORDER BY e.Salary DESC ) AS 'rank' FROM employee AS e ) AS p
```

![image-20210820095317856](C:\Users\郭康氏\AppData\Roaming\Typora\typora-user-images\image-20210820095317856.png)
