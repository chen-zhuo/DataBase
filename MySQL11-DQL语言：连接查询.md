# DQL语言：连接查询

### 语法特点

```sql
/*
含义：又称多表查询，当查询的字段来自多个表时，就会用到连接查询。

笛卡尔乘积现象：表1有m行，表2有n行，结果=m*n行
发生原因：没有有效的连接条件
如何避免：添加有效的连接条件

分类：
   按年代分类：
           SQL92标准：仅仅支持内连接
           SQL99标准【推荐】：支持内连接+外连接（左外和右外）+交叉连接
   按功能分类：
           内连接：
                等值连接
                非等值连接
                自连接
           外连接：
                左外连接
                右外连接
                全外连接
           交叉连接
*/
```

### SQL92内连接

##### 等值连接

```sql
/*
等值连接：
1、多表等值连接的结果为多表的交集部分
2、n个表连接，至少需要n-1个连接条件
3、多表的顺序没有要求
4、一般需要为表起别名
5、可以搭配任意子句使用，比如：排序、分组、筛选
*/
```

等值连接联系案例：

```sql
-- 查询girls库中beauty表中女生名对应的boys表中男生名
SELECT
	`name`,
	`boyName`
FROM
	beauty,
	boys
WHERE
	beauty.boyfriend_id = boys.id;
```

![QQ截图20201206183846](image/QQ截图20201206183846.png)

```sql
-- 查询employees表中员工名和对应的部门名
SELECT
	last_name,
	department_name
FROM
	employees,
	departments
WHERE
	employees.department_id = departments.department_id;
```

![QQ截图20201206184529](image/QQ截图20201206184529.png)

```sql
/*
为表起别名：
1、提高语句的简洁度
2、区分多个重名的字段
3、两个表的顺序可以调换
注意：如果为表起了别名，则查询的字段就不能使用原来的表名去限定
*/
-- 查询员工名、工种号、工种名
SELECT
	last_name,
	-- 因为两个表都有job_id字段，所以需要指定表名
	e.job_id,
	job_title
FROM
	employees e,
	jobs j
WHERE
	e.job_id = j.job_id;
```

![QQ截图20201206205128](image/QQ截图20201206205128.png)

```sql
-- 添加筛选条件
-- 查询有奖金的员工名、部门名、奖金率
SELECT
	last_name,
	department_name,
	commission_pct
FROM
	employees e,
	departments d
WHERE
	e.department_id = d.department_id
AND 
	commission_pct IS NOT NULL;
```

![QQ截图20201206205812](image/QQ截图20201206205812.png)

```sql
-- 三表连接
-- 查询员工名、部门名和所在的城市
SELECT
	last_name,
	department_name,
	city
FROM
	employees e,
	departments d,
	locations l
WHERE
	e.department_id = d.department_id
AND 
	d.location_id = l.location_id;
```

![QQ截图20201206210838](image/QQ截图20201206210838.png)

##### 非等值连接

```sql
-- 查询员工的工资和工资级别
SELECT
	salary,
	grade_level
FROM
	employees e,
	job_grades g
WHERE
	salary BETWEEN g.lowest_sal
AND 
  g.highest_sal;
```

![QQ截图20201206213504](image/QQ截图20201206213504.png)

##### 自连接

自连接：两次查询使用的是同一张表。

```sql
-- 查询员工名和上级的名称
SELECT
	e.employee_id,
	e.last_name,
	m.employee_id,
	m.last_name
FROM
	employees e,
	employees m
WHERE
	e.manager_id = m.employee_id;
```

![QQ截图20201206214158](image/QQ截图20201206214158.png)

### SQL99内连接

##### 语法特点

```sql
/*
语法：
	SELECT 查询列表
	FROM 表1 别名 【连接类型】
	JOIN 表2 别名
	ON 连接条件
	【WHERE 筛选条件】
	【GROUP BY 分组】
	【HAVING 筛选条件】
	【ORDER BY 排序列表】
分类：
	内连接：INNER
	外连接：
		左外：LEFT 【OUTER】
		右外：RIGHT 【OUTER】
		全外：FULL 【OUTER】
	交叉连接：CROSS
*/
```

##### 等值连接