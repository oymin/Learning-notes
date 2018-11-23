### 用一条SQL 语句 查询出每门课都大于80 分的学生姓名

| name | kecheng | fenshu 	|
| --   |  --     |  --   |  
| 张三 | 语文     | 81		|
| 张三 | 数学     | 75		|
| 李四 | 语文     | 76		|
| 李四 | 数学     | 90		|
| 王五 | 语文     | 81		|
| 王五 | 数学     | 100		|
| 王五 | 英语     | 90		|

```
SELECT name FROM student
GROUP BY name
HAVING COUNT(kecheng)>=3 AND MIN(fenshu)>=80;
```

### 学生表 如下:

| 自动编号 | 学号    |  姓名 |  课程编号| 课程名称| 分数 |
|   ---   |   --    |  --   |    --   |  --    |  --  |
| 1       | 2005001 |  张三 |  0001   |  数学   | 69  |
| 2       | 2005002 |  李四 |  0001   |  数学   | 89  |
| 3       | 2005001 |  张三 |  0001   |  数学   | 69  |
删除除了自动编号不同, 其他都相同的学生冗余信息

```
delete tab_name where 自动编号
not in
    (select min(自动编号) from tab_name group by 学号,姓名,课程编号,课程名称,分数)
```

### 怎么把这样一个表
| year | month |  amount |
| ---- | ----- |  ------ |
| 1991 | 1     |  1.1	|
| 1991 | 2     |  1.2	|
| 1991 | 3     |  1.3	|
| 1991 | 4     |  1.4	|
| 1992 | 1     |  2.1	|
| 1992 | 2     |  2.2	|
| 1992 | 3     |  2.3	|
| 1992 | 4     |  2.4	|

查成这样一个结果

| year | m1  |  m2  |  m3  |  m4  |
| ---- | --  |  --  |  --  |  --  |
| 1991 | 1.1 |  1.2 |  1.3 |  1.4 |
| 1992 | 2.1 |  2.2 |  2.3 |  2.4 |

```
select year,
  (select amount from aaa m where month = 1 and m.year = aaa.year) as m1,
  (select amount from aaa m where month = 2 and m.year = aaa.year) as m2,
  (select amount from aaa m where month = 3 and m.year = aaa.year) as m3,
  (select amount from aaa m where month = 4 and m.year = aaa.year) as m4
from aaa group by year
```

### 拷贝表( 拷贝数据, 源表名：a 目标表名：b)

```
insert into b(a,b,c) select d,e,f from a
```

### 有一张表，里面有3个字段：语文，数学，英语。其中有3条记录分别表示语文70分，数学80分，英语58分，请用一条sql语句查询出这三条记录并按以下条件显示出来（并写出您的思路）：
大于或等于80表示优秀，大于或等于60表示及格，小于60分表示不及格。
显示格式：
语文 数学 英语
及格 优秀 不及格

```
SELECT
  (case when 语文>=80 then '优秀'
  when 语文>=60 then '及格'
  else '不及格') as 语文,
  (case when 数学>=80 then '优秀'
  when 数学>=60 then '及格'
  else '不及格') as 数学,
  (case when 英语>=80 then '优秀'
  when 英语>=60 then '及格'
  else '不及格') as 英语,
from table_name
```
