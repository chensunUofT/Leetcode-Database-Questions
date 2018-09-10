### [626. Exchange Seats](https://leetcode.com/problems/exchange-seats/description/ "626. Exchange Seats")
思路：对于id为偶数的学生，id自减1；奇数且不是最后一个，自加1。
```sql
SELECT 
    IF(id % 2 = 0, 
            id-1, 
    IF(id = (SELECT COUNT(*) FROM seat), 
            id, 
            id+1)) 
          AS id, 
    student
    FROM seat
    ORDER BY id ASC
```
此题需要用到的：
- [SQL Aliases（别名）](https://www.w3schools.com/sql/sql_alias.asp "SQL Aliases（别名）")

### [177. Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/description/)
跟176题的区别在于，由于要求第N大，所以需要传一个参数N进去。定义函数、声明变量之类的部分就先跳过吧，直接贴答案：
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  DECLARE M INT;
  SET M = N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT IFNULL((SELECT DISTINCT Salary FROM Employee
                        ORDER BY Salary DESC
                        LIMIT 1 OFFSET M), null) AS getNthHighestSalary
  );
END
```

---

### [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/description/)
与前边比较两天温度的那道题有点类似，区别在于本题中要求判断连续三条记录中的值是否相等，因此需要把三个表（严格来说是把一个表用三次）做INNER JOIN。语法没什么难度，就是把INNER JOIN的结果再INNER JOIN一次。此题的一个小坑在于，为了不出现重复的结果，需要用到一个DISTINCT。
```sql
SELECT DISTINCT A.Num AS ConsecutiveNums
    FROM (Logs A INNER JOIN Logs B ON A.Id = B.Id - 1) INNER JOIN Logs C ON A.Id = C.Id + 1
    WHERE A.Num = B.Num AND B.Num = C.Num
```

### [178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)
此题要实现的是一个排名（rank）功能，思路是，对于每个score，数出有多少个比自己高或相等的distinct score并按score降序输出。
写法一：
对于Scores表中的每条记录，数一下。
```sql
SELECT  Score,
        (SELECT COUNT(DISTINCT Score) From Scores WHERE Score >= S.Score) AS Rank
    FROM Scores S
    ORDER BY Score DESC
```
写法二：
把Scores表跟自己join一下 on S1.score <= S2.score，按Id分组后按序输出。
```sql
SELECT S1.Score, COUNT(DISTINCT S2.Score) AS Rank
    FROM Scores S1 INNER JOIN Scores S2 ON S1.Score <= S2.Score
    GROUP BY S1.Id
    ORDER BY Score DESC
```

### [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/description/)
选出每个部门工资最高的人的名字和工资。JOIN的部分没什么坑，如何把工资最高的人选出来比较费脑筋。
实现的思路：把Employee表按DepartmentID分组（GROUP BY），记下每个DepartmentID对应的最高Salary，然后按这个结果筛选两个表格JOIN之后的记录。
```sql
SELECT D.Name as Department, E.Name as Employee, E.Salary as Salary
    FROM Employee E INNER JOIN Department D ON E.DepartmentID = D.Id
    WHERE (E.DepartmentID, E.Salary) IN 
        (SELECT DepartmentId, MAX(Salary) FROM Employee GROUP BY DepartmentId)
```
