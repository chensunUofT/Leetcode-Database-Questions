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
