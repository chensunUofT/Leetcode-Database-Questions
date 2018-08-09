### [595. Big Countries](https://leetcode.com/problems/big-countries/description/ "595. Big Countries")
SELECT最简单的用法，WHERE clause中可以用AND/OR连接多个条件
```sql
SELECT name,population,area FROM World WHERE area>3000000 OR population>25000000
```

---

### [627. Swap Salary](https://leetcode.com/problems/swap-salary/description/ "627. Swap Salary")
说是换工资，其实是变性，要求所有人对换性别，男变女，女变男。
肯定是要使用UPDATE query对表格进行操作了，问题在于条件语句怎么写。
- 解法1：[CASE](https://www.w3schools.com/sql/func_mysql_case.asp "CASE")
```sql
UPDATE salary
    SET sex = CASE sex
                	WHEN 'm' then 'f'
                	WHEN 'f' then 'm'
                   END
```

- 解法2：[IF](https://www.w3schools.com/sql/func_mysql_if.asp "IF")
```sql
UPDATE salary
    SET sex = IF(sex='m', 'f', 'm')
```
这种做法的局限性在于，幸亏这道题里只有两种性别，如果情况比较多的话就只能用CASE了。

---

### [620. Not Boring Movies](https://leetcode.com/problems/not-boring-movies/description/ "620. Not Boring Movies")
除了SELECT和WHERE，其它要用到的：
- WHERE中的运算符： `<>`
- [ORDER BY](https://www.w3schools.com/sql/sql_orderby.asp "ORDER BY")

```sql
SELECT * FROM cinema
    WHERE id % 2 = 1 AND description <> 'boring'
    ORDER BY rating DESC
```

---

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/ "175. Combine Two Tables")
名字叫Combine，其实就是最基础的[Left Join](https://www.w3schools.com/sql/sql_join_left.asp "Left Join")
```sql
SELECT FirstName, LastName, City, State
    FROM Person LEFT JOIN Address ON Person.PersonId = Address.PersonID
```

### [182. Duplicate Emails](http://https://leetcode.com/problems/duplicate-emails/description/ "182. Duplicate Emails")
思路：要SELECT出一些Email，要求他们满足：出现次数>1
开始想的是用WHERE clause，发现怎么都行不通。偷瞄了一眼discussion之后发现要用到HAVING，搜了一下：[SQL HAVING Clause](http://https://www.w3schools.com/sql/sql_having.asp "SQL HAVING Clause")
文档中给的例子和这道题就很像了，于是直接敲出答案：
```sql
SELECT Email FROM Person GROUP BY Email HAVING COUNT(*)>1
```
解释一下：先通过GROUP BY按照Email进行分组，然后筛选出满足条件(行数>1)的行

---

### [181. Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/description/ "181. Employees Earning More Than Their Managers")
这个题是典型的需要用到INNER JOIN的情况
```sql
SELECT A.Name AS Employee 
    FROM Employee A INNER JOIN Employee B ON A.ManagerID=B.ID
    WHERE A.Salary>B.Salary
```
还有另一种不需要用到JOIN关键字（但含义相同）的写法：
```sql
SELECT Emp.Name AS Employee
    FROM Employee Emp, Employee Mng
    WHERE Emp.ManagerID=Mng.Id AND Emp.Salary>Mng.Salary
```


---

### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/description/ "183. Customers Who Never Order")
需要在Customers表中找到一些Name，满足：他们的ID在Orders表中没有对应的CustomerID。这里要用到LEFT JOIN：对于左表（Customers）中的每一条记录，去JOIN Orders表中的对应的ID行，有就SELECT出来。

```sql
SELECT Name AS Customers
    FROM Customers LEFT JOIN Orders ON Customers.ID=Orders.CustomerID
    WHERE Orders.ID IS NULL
```
看了一眼discussion，发现不用JOIN也是可以做的：
```sql
SELECT Name AS Customers
    FROM Customers
    WHERE Id NOT IN (SELECT CustomerID FROM Orders)
```


---

### [596. Classes More Than 5 Students](https://leetcode.com/problems/classes-more-than-5-students/description/ "596. Classes More Than 5 Students")
要选出一些student数目大于等于5的class，很明显要用到GROUP BY和HAVING。一个小坑是，由于可能出现重复行，所以需要用到DISTINCT关键字。
```sql
SELECT class
    FROM courses
    GROUP BY class
    HAVING COUNT(DISTINCT student) > 4
```

---

### [197. Rising Temperature](https://leetcode.com/problems/rising-temperature/description/ "197. Rising Temperature")
需要INNER JOIN自身，同样也是两种写法：
```sql
SELECT A.Id as Id
    FROM Weather A INNER JOIN Weather B ON DATEDIFF(A.RecordDate, B.RecordDate)=1 
    WHERE A.Temperature > B.Temperature
```
```sql
SELECT A.Id
    FROM Weather A, Weather B
    WHERE DATEDIFF(A.RecordDate,B.RecordDate)=1 AND A.Temperature>B.Temperature
```

---

### [196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/description/ "196. Delete Duplicate Emails")
这道题费的时间有点多，慢慢说。
题目要求很简单，删除重复的Email，仅保留Id最小的记录。实现的思路：使用DELETE语句删掉一些不是最小的Id。此处在HAVING和WHERE之间纠结了好久，看了一眼discussion发现正确做法大概是WHERE+GROUP BY（分组找出最小Id），写下了下面这么个玩意：
```sql
DELETE FROM Person
    WHERE Id NOT IN (SELECT MIN(Id) FROM Person GROUP BY Email)
```
此时踩进一个坑：[You can't specify target table for update in FROM clause](https://blog.csdn.net/fdipzone/article/details/52695371 "You can't specify target table for update in FROM clause")
解决方案就是把最小Id那个表再SELECT一次并且起个名字：
```sql
DELETE FROM Person
    WHERE Id NOT IN (SELECT MIN_ID FROM
                        (SELECT MIN(Id) AS MIN_ID FROM Person GROUP BY Email)
                        as MIN_ID_TABLE)
```
OK，再看看另外的解法：
考虑把所有Email相同且满足【其中一条的Id比另一条大】的记录INNER JOIN起来：
```sql
SELECT p1.*
	FROM Person p1, Person p2
	WHERE p1.Email = p2.Email AND p1.ID > p2.ID
```
通过上边的语句可以SELECT出所有满足上述条件的行，然后我们需要删除掉他们，以此来保留重复Email记录的最小Id行：
```sql
DELETE p1 FROM Person p1, Person p2
	WHERE p1.Email = p2.Email AND p1.ID > p2.ID
```
Discussion中对这个解法的解释：[https://leetcode.com/problems/delete-duplicate-emails/discuss/55553/Simple-Solution](https://leetcode.com/problems/delete-duplicate-emails/discuss/55553/Simple-Solution)

---

### [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/description/ "176. Second Highest Salary")
喜迎Easy难度最后一题，通过率仅为百分之二十多，瑟瑟发抖。题意倒是简单，找出第二高，没有则返回null。
Trivial idea：第二高就是除了第一高之外的第一高，那就选出【不带最高的玩】的表格里的最高的。
```sql
SELECT MAX(Salary) AS SecondHighestSalary
    FROM Employee
    WHERE Salary NOT IN (SELECT MAX(Salary) FROM Employee)
```
没想到居然一遍过了……
扫了一眼discussion，用到了个叫做[LIMIT（链接）](http://www.sqltutorial.org/sql-limit/ "LIMIT")的玩意。用这个就简单多了：
```sql
SELECT Salary AS SecondHighestSalary
    FROM Employee
    ORDER BY Salary DESC
    LIMIT 1 OFFSET 1
```
燃鹅，出现了null的问题，挖坑明天填……
solution: [IFNULL](https://www.w3schools.com/sql/sql_isnull.asp "IFNULL")
```sql
SELECT IFNULL(MAX(Salary), null) AS SecondHighestSalary
    FROM Employee
    WHERE Salary NOT IN (SELECT MAX(Salary) FROM Employee)
```
