### [601. Human Traffic of Stadium](https://leetcode.com/problems/human-traffic-of-stadium/description/)
Hard难度免费题只有三道，开撸！
这题题意不难理解，找出满足条件【连续三天>=100人】的记录。
我的思路：先找出满足【昨天今天明天都>=100人】的记录，然后在他们的基础上往前后各延伸一天。
```sql
SELECT * FROM stadium 
    WHERE ID in 
    (SELECT s2.ID FROM stadium s1, stadium s2, stadium s3
        WHERE s2.id = s1.id + 1 AND s2.id = s3.id - 1 
                AND s2.people >=100 AND s1.people >= 100 and s3.people >= 100) 
        or ID+1 in 
    (SELECT s2.ID FROM stadium s1, stadium s2, stadium s3
        WHERE s2.id = s1.id + 1 AND s2.id = s3.id - 1 
                AND s2.people >=100 AND s1.people >= 100 and s3.people >= 100)
        or ID-1 in
    (SELECT s2.ID FROM stadium s1, stadium s2, stadium s3
        WHERE s2.id = s1.id + 1 AND s2.id = s3.id - 1 
                AND s2.people >=100 AND s1.people >= 100 and s3.people >= 100)
```
三个小括号里的东西是一样的，不知道能不能简化一下……
discussion区高票思路：
【连续三天】等同于【昨天今天明天】or【今天明天后天】or【今天昨天前天】，于是：
```sql
SELECT s1.* FROM stadium AS s1, stadium AS s2, stadium as s3
    WHERE 
    ((s1.id + 1 = s2.id
    AND s1.id + 2 = s3.id)
    OR 
    (s1.id - 1 = s2.id
    AND s1.id + 1 = s3.id)
    OR
    (s1.id - 2 = s2.id
    AND s1.id - 1 = s3.id)
    )
    AND s1.people>=100 
    AND s2.people>=100
    AND s3.people>=100

    GROUP BY s1.id
```

### [185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/description/)
题目意思跟之前那道184有点像，区别在于要求找出前三高的工资。
我一开始的思路是先找出每个部门第三高的，然后筛选出大于等于它的。但这个思路似乎并不能简化问题。抓耳挠腮之际参考了discuss，发现了一个更好的思路：找出【（每个部门比它高的工资的数量）<3】的。
```sql
SELECT D.Name AS Department, E1.Name AS Employee, E1.Salary AS Salary
    FROM Employee E1 INNER JOIN Department D ON E1.DepartmentId = D.Id
    WHERE 3 > (SELECT COUNT(DISTINCT(e2.Salary)) FROM Employee e2
                WHERE e2.DepartmentId = E1.DepartmentId
                AND e2.Salary > E1.Salary)
```
代码逻辑倒是不复杂，没有想到这个思路可能是因为不熟悉query中的数据可以拿到sub-query中使用的这个思路。另外一个巨坑：
**SQL真的完全 cAsE-INsenSaTiVe !!!** 
我是怎么想到通过大小写区分不同表格的。。。
