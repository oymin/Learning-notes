## 数据库表介绍

### 1.学生表
Student(SId,Sname,Sage,Ssex)  
SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

### 2.课程表
Course(CId,Cname,TId)  
CId 课程编号,Cname 课程名称,TId 教师编号

### 3.教师表
Teacher(TId,Tname)  
TId 教师编号,Tname 教师姓名

### 4.成绩表
SC(SId,CId,score)  
SId 学生编号,CId 课程编号,score 分数

## 学生表

```
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');
```

## 科目表 Course

```
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```

## 教师表 Teacher

```
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```

## 成绩表 SC

```
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

## 练习题目

#### 1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

```
select * from (
    select t1.Sid, class1, class2
    from
      (select Sid,score as class1 from sc where sc.Cid = '01') as t1,
      (select Sid, score as class2 from sc where sc.Cid = '02') as t2
    where t1.Sid = t2.Sid and t1.class1 > t2.class2
)r
left join Student
on Student.Sid = r.Sid
```

```
select * from Student RIGHT JOIN (
    select t1.Sid, class1, class2
    from
      (SELECT Sid, score as class1 from sc where sc.Cid = '01') as t1,
      (SELECT Sid, score as class2 from sc where sc.Cid = '02') as t2
    where t1.Sid = t2.Sid and t1.class1 > t2.class2
)r
on Student.Sid = r.Sid;
```

#### 2. 查询同时存在" 01 "课程和" 02 "课程的情况

```
select * from
  (select * from sc where sc.Cid = '01') as t1,
  (select * from sc where sc.Cid = '02') as t2
where t1.Sid = t2.Sid;
```

#### 3. 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
这一道就是明显需要使用join的情况了，02可能不存在，即为left join的右侧或right join 的左侧即可.

```
select * from
  (select * from sc where sc.CId = '01') as t1
left join
  (select * from sc where sc.CId = '02') as t2
on t1.SId = t2.SId;
```

```
select * from
  (select * from sc where sc.CId = '02') as t2
right join
  (select * from sc where sc.CId = '01') as t1
on t1.SId = t2.SId;
```

### 4. 查询不存在" 01 "课程但存在" 02 "课程的情况

```
SELECT * FROM sc
WHERE Sid NOT IN
  (SELECT Sid FROM sc WHERE sc.Cid = '01')
AND sc.Cid = '02';
```

### 5. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
这里只用根据学生ID把成绩分组，对分组中的score求平均值，最后在选取结果中AVG大于60的即可. 注意，这里必须要给计算得到的AVG结果一个alias.（AS ss）
得到学生信息的时候既可以用join也可以用一般的联合搜索

```
select Student.Sid, sname, ss from Student,(
 select Sid, avg(score) as ss from sc
 group by Sid
 HAVING avg(score) > 60
)r
where Student.Sid = r.Sid;
```

```
select Student.Sid, sname, ss from Student RIGHT JOIN (
    select Sid, AVG(score) as ss from sc
    group by Sid
    HAVING AVG(score) >= 60
  )r
ON r.Sid = Student.Sid;
```

```
select s.Sid, S.sname, ss from
(select Sid, avg(score) as ss from sc
 group by Sid
 HAVING avg(score) > 60
)r
left join
(select Student.Sid, Student.sname from Student)s
on r.Sid = s.Sid;
```

#### 6. 查询在 SC 表存在成绩的学生信息

```
select DISTINCT Student.*
FROM Student , sc
where Student.Sid = sc.Sid;
```

#### 7. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的成绩总和
联合查询不会显示没选课的学生：

```
SELECT Student.Sid , Student.sname, r.coursecount, r. scoresum FROM Student,
(SELECT sc.Sid, SUM(sc.score) AS scoresum, COUNT(sc.Cid) AS coursecount
FROM sc
GROUP BY sc.Sid)r
WHERE r.Sid = Student.Sid;
```
如要显示没选课的学生(显示为NULL)，需要使用join:

```
SELECT s.Sid, s.sname, r.coursecount, r.scoresum
FROM (
  (SELECT Student.Sid , Student.sname FROM Student)s
  LEFT JOIN
  (SELECT sc.Sid , SUM(score) AS scoresum , COUNT(sc.Cid) AS coursecount
  FROM sc
  GROUP BY sc.Sid
  )r
  ON r.Sid = s.Sid
);
```

#### 8. 查有成绩的学生信息
这一题涉及到in和exists的用法，在这种小表中，两种方法的效率都差不多，但是请参考SQL查询中in和exists的区别分析
当表2的记录数量非常大的时候，选用exists比in要高效很多.
EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False.
结论：IN()适合B表比A表数据小的情况
结论：EXISTS()适合B表比A表数据大的情况

```
SELECT * FROM Student
WHERE Student.Sid
IN (SELECT sc.Sid FROM sc );
```

```
SELECT * FROM Student
WHERE EXISTS (SELECT sc.sid FROM sc WHERE Student.Sid = Sc.Sid);
```

#### 9. 查询「李」姓老师的数量

```
SELECT COUNT(*) FROM teacher WHERE tname LIKE '李%';
```

#### 10. 查询学过「张三」老师授课的同学的信息
多表联合查询

```
SELECT Student.* FROM Student, teacher, course, sc
WHERE
  Student.Sid = sc.Sid
  AND sc.Cid = course.Cid
  AND course.TId = teacher.TId
  AND Tname = '张三';
```

#### 11. 查询没有学全所有课程的同学的信息
因为有学生什么课都没有选，反向思考，先查询选了所有课的学生，再选择这些人之外的学生.

```
select * from Student where Student.Sid not in
(
 select sc.Sid from sc
 group by sc.Sid
 HAVING count(sc.Cid) = (select count(Cid) from course)
)
```

#### 12. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
这个用联合查询也可以，但是逻辑不清楚，我觉得较为清楚的逻辑是这样的：从sc表查询01同学的所有选课cid--从sc表查询所有同学的sid如果其cid在前面的结果中--从student表查询所有学生信息如果sid在前面的结果中

```
SELECT * FROM Student WHERE
Student.Sid IN
	(
	    SELECT sc.SId FROM sc
	    WHERE sc.CId
	    IN (SELECT cid FROM sc WHERE sid = '01')
	);
```

#### 13. 查询和" 01 "号的同学学习的课程完全相同的其他同学的信息

```
select * from Student where sid in (
SELECT sid FROM SC WHERE
cid in (SELECT cid FROM SC WHERE sid=01) AND sid!=01 GROUP BY sid HAVING COUNT(*)=(SELECT COUNT(*) FROM SC WHERE sid=01)
)
```

#### 14. 把“SC”表中“张三”老师教的课的成绩都更改为此课程的平均成绩；

```
UPDATE sc INNER JOIN
(
SELECT course.`CId` AS courseid, AVG(score) AS sumscore FROM course,teacher,sc
WHERE course.`TId` = teacher.`TId`
AND sc.`CId` = course.`CId`
AND teacher.`Tname` = '张三'
) r
SET sc.`score` = r.sumscore
WHERE sc.`CId` = r.courseid;
```

#### 15. 查询没学过"张三"老师讲授的任一门课程的学生姓名

```
select * from Student WHERE Sid NOT in
(
  select sid from sc , course , teacher
  where
    teacher.Tname = '张三'
    AND course.TId = teacher.TId
  AND sc.Cid = course.CId
)
```

#### 16. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```
SELECT s.sid, s.sname, AVG(score) FROM sc c
JOIN Student s ON s.sid = c.sid
WHERE c.score < 60
GROUP BY s.sid,s.sname
HAVING COUNT(c.score) >= 2;
```

#### 17. 检索"01"课程分数小于60，按分数降序排列的学生信息

双表联合查询，在查询最后可以设置排序方式，语法为ORDER BY ******* DESC\ASC;

```
SELECT student.* , sc.score FROM student , sc
WHERE student.SId = sc.SId
AND sc.score < 60
AND sc.CId = '01'
ORDER BY sc.score DESC;
```

#### 18. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```
SELECT sc.*, scoreavg FROM sc
LEFT JOIN (
    SELECT sid , AVG(score) AS scoreavg FROM sc
    GROUP BY sid
  )r
ON sc.sid = r.sid
ORDER BY scoreavg DESC ;
```

#### 19. 查询各科成绩最高分、最低分和平均分：
以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率

及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90

要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```
SELECT cid, MAX(score) AS 最高分, MIN(score) AS 最低分 ,
AVG(score) AS 平均分 , COUNT(*) AS 选修人数,
SUM(CASE WHEN sc.`score` >= 60 THEN 1 ELSE 0 END )/COUNT(*) AS 及格率,
SUM(CASE WHEN sc.`score` >= 70 AND sc.score <= 80 THEN 1 ELSE 0 END)/COUNT(*) AS 中等率,
SUM(CASE WHEN sc.`score` >= 80 AND sc.`score` <= 90 THEN 1 ELSE 0 END)/COUNT(*) AS 优良率,
SUM(CASE WHEN sc.`score` >= 90 THEN 1 ELSE 0 END)/COUNT(*) AS 优秀率
FROM sc
GROUP BY cid
ORDER BY COUNT(*) DESC , cid ASC;
```

#### 20. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
这一道题有点tricky，可以用变量，但也有更为简单的方法，即自交（左交）
用sc中的score和自己进行对比，来计算“比当前分数高的分数有几个”。

```
SELECT a.cid, a.sid, a.score, COUNT(b.score)+1 AS rank
, b.score AS bscore
FROM sc AS a
LEFT JOIN sc AS b
ON a.score<b.score AND a.cid = b.cid
GROUP BY a.cid, a.sid,a.score
ORDER BY a.cid, rank ASC;
```

#### 21. 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
这里主要学习一下使用变量。在SQL里面变量用@来标识。

```
SET @crank = 0;
SELECT r.sid , total , @crank := @crank + 1 AS 排名 FROM
(
  SELECT sc.sid, SUM(score) AS total FROM sc
  GROUP BY sc.sid
  ORDER BY SUM(score) DESC
)r ;
```

#### 22. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比

有时候觉得自己真是死脑筋。group by以后的查询结果无法使用别名，所以不要想着先单表group by计算出结果再从第二张表里添上课程信息，而应该先将两张表join在一起得到所有想要的属性再对这张总表进行统计计算。这里就不算百分比了，道理相同。
注意一下，用case when 返回1 以后的统计不是用count而是sum

```
select course.cname, course.cid,
sum(case when sc.score<=100 and sc.score>85 then 1 else 0 end) as "[100-85]",
sum(case when sc.score<=85 and sc.score>70 then 1 else 0 end) as "[85-70]",
sum(case when sc.score<=70 and sc.score>60 then 1 else 0 end) as "[70-60]",
sum(case when sc.score<=60 and sc.score>0 then 1 else 0 end) as "[60-0]"
from sc left join course
on sc.cid = course.cid
group by sc.cid;
```

#### 23. 查询各科成绩前三名的记录

大坑比。mysql不能group by 了以后取limit，所以不要想着讨巧了，我快被这一题气死了。思路有两种，第一种比较暴力，计算比自己分数大的记录有几条，如果小于3 就select，因为对前三名来说不会有3个及以上的分数比自己大了，最后再对所有select到的结果按照分数和课程编号排名即可。

```
select * from sc
where (
select count(*) from sc as a
where sc.cid = a.cid and sc.score<a.score
)< 3
order by cid asc, sc.score desc;
```

```
select * from sc a
left join sc b on a.cid = b.cid and a.score<b.score
order by a.cid,a.score;
```

#### 24. 查询每门课程被选修的学生数

```
select cid , count(sid) from sc
group by cid;
```

#### 25. 查询出只选修两门课程的学生学号和姓名

嵌套查询

```
SELECT * FROM student WHERE student.`SId` IN
(
  SELECT sc.SId FROM sc
  GROUP BY sc.SId
  HAVING COUNT(sc.CId) = 2
);
```

联合查询

```
select student.* from student , sc
where student.sid = sc.sid
group by sc.sid
having count(*) = 2;
```

#### 26. 查询男生、女生人数

```
select ssex , count(*) from student
group by ssex;
```

#### 27. 查询名字中含有「风」字的学生信息

```
select * from student where sname like '%风%';
```

#### 28. 查询同名学生名单，并统计同名人数

找到同名的名字并统计个数

```
select sname , count(*) from student
group by sname
having count(*) > 1;
```

嵌套查询列出同名的全部学生的信息

```
select * from student
where sname in 
	( select sname from student
	  group by sname
	  having count(*)>1
	);
```

#### 29. 查询 1990 年出生的学生名单

```
select * from student sage like substr(sage)
```