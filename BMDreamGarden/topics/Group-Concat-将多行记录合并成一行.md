# Group Concat - 将多行记录合并成一行

有以下两张表：

Candidate：

| 列名         | 数据类型        | 描述               |
|------------|-------------|------------------|
| id         | int         | pk, candidate id |
| first_name | varchar(64) | first name       |
| last_name  | varchar(64) | last name        |

Results:

| 列名           | 数据类型        | 描述           |
|--------------|-------------|--------------|
| candidate_id | int         | Candidate id |
| state        | varchar(19) | US State     |

要求：
1. 用一条SQL语句查询出每个候选人的得票情况，结果如下：  

| 列名        | 数据类型          | 描述                           |
|-----------|---------------|------------------------------|
| state     | varchar(19)   | US State                     |
| votes     | varchar(5000) | votes, eg:john x 1,jerry x 2 | 

Solution:
```sql
SELECT sub.state, 
       GROUP_CONCAT(
                    CONCAT(c.first_name, ' ', c.last_name, ' x ', sub.vote_count)
                    ORDER BY sub.vote_count DESC,
                             c.first_name ASC
                             c.last_name ASC
                    SEPARATOR ','
       ) AS votes
FROM Candidate c
JOIN 
    (select candidate_id, state, count(*) as vote_count
     from Results
     group by candidate_id, state 
    ) sub 
        ON c.id = sub.candidate_id
GROUP BY sub.state;
```
