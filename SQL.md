## 一、查询

1. 基本

```
SELECT column1, column2, ...
FROM table_name;
-- column1, column2, ...：要选择的字段名
-- table_name：要查询的表名称

SELECT * FROM table_name;
-- *：通配符，表示选择表中的所有列
```



2. 限制返回字段数量

```
-- DISTINCT：用于返回唯一不同的值
-- 当指定多个字段时，会基于这些字段的组合来进行去重
SELECT DISTINCT column1, column2, column3
FROM table_name;

-- LIMIT 与 OFFSET（分页）
-- 查询第2页，每页3条记录
SELECT * FROM employees 
LIMIT 3 OFFSET 3;
```

​	

3. 函数

```
WHERE 关键字不能与聚合函数一起使用
COUNT() 函数返回匹配指定条件的行数
MAX(), MIN(), SUM() 
LENGTH() 函数返回文本字段中值的长度

NOW() 函数返回当前系统的日期和时间 CURDATE();日期 CURTIME();时间
YEAR('2023-10-23 15:30:45');HOUR('15:30:45'); 从日期时间中提取各部分

DAYNAME('2023-10-23');DAYOFWEEK，WEEKDAY，获取星期
DAYOFYEAR('2023-10-23');一年中的第几天

DATEDIFF('2023-10-30', '2023-10-23');天数差
TIMEDIFF('15:30:45', '14:20:30');时间差
SELECT TIMESTAMPDIFF(YEAR, '2000-05-10', '2023-10-23');

DATE_FORMAT(NOW(), '%Y-%m-%d');日期格式化-- 2023-10-23
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日'); -- 2023年10月23日
-- 常用格式符
-- %Y 四位年份, %y 两位年份
-- %m 月份(01-12), %c 月份(1-12)
-- %d 日期(01-31), %e 日期(1-31)
-- %H 24小时制, %h 12小时制
-- %i 分钟, %s 秒
-- %W 星期名, %a 缩写星期名
-- 字符串转日期
SELECT STR_TO_DATE('2023-10-23', '%Y-%m-%d');
```





4. AS 别名：

```
-- 为函数结果指定别名
SELECT 
    first_name,
    last_name,
    CONCAT(first_name, ' ', last_name) AS 全名,
    UPPER(first_name) AS 大写名字,
    YEAR(hire_date) AS 入职年份,
    DATEDIFF(CURDATE(), hire_date) AS 工作天数
FROM employees;
GROUP BY YEAR(hire_date)  -- 使用原始表达式，而不是别名
ORDER BY 入职年份;         -- 但可以在 ORDER BY 中使用别名

-- 多表连接使用别名
SELECT 
    e.name AS 员工姓名,
    p.project_name AS 项目名称,
    m.name AS 经理姓名,
    d.department_name AS 部门名称
FROM employees e
JOIN projects p ON e.project_id = p.id
JOIN employees m ON e.manager_id = m.id
JOIN departments d ON e.department_id = d.id;

SELECT 
    department,
    CASE 
        WHEN salary < 6000 THEN '低工资'
        WHEN salary BETWEEN 6000 AND 8000 THEN '中等工资'
        ELSE '高工资'
    END as 工资级别,
    COUNT(*) as 人数,
    AVG(salary) as 平均工资
...
```



5. WHERE

```
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

​	`WHERE`：用于提取那些满足指定条件的记录

​	WHERE子句中的运算符：

- =：等于
- \>=：大于等于
- <>：不等于；在 SQL 的一些版本中，该操作符可被写成 !=
- `BETWEEN`：选取介于两个值之间的数据范围内的值：`... WHERE column_name BETWEEN value1 AND value2;`
- `LIKE`：搜索列中的指定模式，通常与通配符一起使用
  - `%`：表示零个、一个或多个字符
  - `_`：表示一个单个字符
  - `... WHERE name LIKE '%三_';`：选取名字倒数第二个字是’三‘的人
- `REGEXP`：正则表达式在 SQL 中用于进行复杂的模式匹配，比 `LIKE` 操作符更强大和灵活
  - `... WHERE name REGEXP '^张';`：名字以"张"开头的员工；`name REGEXP '三$'`：以"三"结尾
  - `... WHERE name REGEXP '[a-zA-Z]';`：包含英文字母
  - `... WHERE name REGEXP '^.{2,4}$';`：名字长度2-4个字符
  - `... WHERE email REGEXP '.+@.+\..+';`：简单的邮箱验证
- `IN`：检查某列的值是否在给定的值列表中：
  - `... WHERE column_name IN (value1, value2, ...);`
  - `... NOT IN ...`：排除特定值
  - 值列表是子查询：`... WHERE department IN (SELECT department FROM departments WHERE location = '北京');`
  - 转义符：`WHERE ename LIKE '%\_%' ESCAPE '\';`模式中的'_'表示字面量的下划线
- `AND`：使用AND连接两个条件时，两个条件都必须为真，整个条件才为真
- `OR`：使用OR连接两个条件时，只要有一个条件为真，整个条件就为真；优先级低于`AND`



6. JOIN连接

<img src="./sql-join.png" style="zoom:67%;float:left; margin:10px" />

由于mysql中没有 full outer join

所以图中6为1 UNION 4，7为2 UINION 5

| **INNER JOIN**      | 返回两个表中满足连接条件的记录（交集）。                     |
| ------------------- | ------------------------------------------------------------ |
| **LEFT JOIN**       | 返回左表中的所有记录，即使右表中没有匹配的记录（保留左表）。 |
| **RIGHT JOIN**      | 返回右表中的所有记录，即使左表中没有匹配的记录（保留右表）。 |
| **FULL OUTER JOIN** | 返回两个表的并集，包含匹配和不匹配的记录。                   |
| **CROSS JOIN**      | 返回两个表的笛卡尔积，每条左表记录与每条右表记录进行组合。   |
| **SELF JOIN**       | 将一个表与自身连接。                                         |
| **NATURAL JOIN**    | 基于同名字段自动匹配连接的表。                               |



7. ORDER BY 和 GROUP BY

```
SELECT * 
SELECT * FROM employees 
WHERE department = '技术部'
ORDER BY salary DESC;
- 指定升序（ASC）或降序（DESC）；如果不指定，默认是升序
- `ORDER BY`子句通常放在查询语句的最后

-- 按字段位置排序
SELECT id, name, department 
FROM employees 
ORDER BY 3 DESC, 1 ASC;
- 按第3列(department)降序，第1列(id)升序

-- 使用CASE WHEN自定义排序规则
SELECT * FROM employees 
ORDER BY 
  CASE department
    WHEN '技术部' THEN 1
    WHEN '销售部' THEN 2
    WHEN '人事部' THEN 3
    ELSE 4
  END;
 
-- ORDER BY + LIMIT
SELECT * FROM employees 
ORDER BY salary DESC 
LIMIT 5 OFFSET 5;
- 查询工资高到低第6-10名（分页查询）
```

```
-- 在GROUP BY查询中，SELECT列表只能包含：
-- GROUP BY子句中的列，聚合函数（COUNT, SUM, AVG, MAX, MIN等）
-- 换句话说：SELECT 中出现的非聚合列必须在 GROUP BY 中出现
-- GROUP BY +  ORDER BY
SELECT department, AVG(salary) as avg_salary
FROM employees 
GROUP BY department 
ORDER BY avg_salary DESC;
- 按部门分组统计平均工资，按平均工资降序

-- GROUP BY + HAVING
-- HAVING 用于对分组后的结果进行过滤：
-- 找出平均薪资大于2000的部门
SELECT deptno, AVG(sal) as 平均薪资
FROM emp
GROUP BY deptno
HAVING AVG(sal) > 2000;

-- WHERE vs HAVING
-- WHERE 在分组前过滤，HAVING 在分组后过滤
SELECT deptno, AVG(sal) as 平均薪资
FROM emp
WHERE job != 'PRESIDENT'  -- 先排除总裁
GROUP BY deptno
HAVING AVG(sal) > 1500;   -- 再筛选平均薪资>1500的部门
```



8. UNION, MINUS, INTERSECT

```
-- 交集，合并两个或多个 SELECT 语句的结果
-- 可以从多个表中选择数据，并将结果集组合成一个结果集
-- 使用 UNION 时，每个 SELECT 语句必须具有相同数量的列，且对应列的数据类型必须相似
-- 默认会去除重复的记录，如果需要保留所有重复记录，可以使用 UNION ALL 操作符
SELECT column1, column2, ...
FROM table1
UNION
SELECT column1, column2, ...
FROM table2;

MINUS差集，INTERSECT交集
```



9. EXISTS

```
EXISTS 运算符用于判断查询子句是否有记录，如果有一条或多条记录存在返回 True，否则返回 False
SELECT e.ename, e.job
FROM emp e
WHERE EXISTS (
    SELECT 1
    FROM dept d
    WHERE d.deptno = e.deptno  -- 这里关联了外部查询的 e.deptno
      AND d.loc = 'NEW YORK'
);
-- 这个子查询是关联子查询，它对于 emp 表中的每一行都会执行一次。
执行步骤：
1. 从 emp 表取出第一行员工记录（比如员工A，deptno=10）
2. 执行子查询：SELECT 1 FROM dept WHERE deptno=10 AND loc='NEW YORK'
3. 如果子查询返回结果（deptno=10的部门确实在NEW YORK），则 EXISTS 返回 TRUE，该员工被选中
	如果子查询无结果，则 EXISTS 返回 FALSE，该员工被排除
4. 对 emp 表中的每一行重复步骤1-4
```



10. 表复制

```
INSERT INTO table2
(column_name(s))
SELECT column_name(s)
FROM table1;
```



11. SELECT执行顺序

```
select      ... 5
  from      ... 1
 where      ... 2
 group by   ... 3
having      ... 4
 order by   ... 6
```

---



## 二、插入

```
指定列名插入：
-- 插入一条完整记录
INSERT INTO employees (name, department, salary, hire_date, email)
VALUES ('张三', '技术部', 5000.00, '2023-01-15', 'zhangsan@company.com');

-- 使用函数生成数据和计算字段
INSERT INTO employees (name, department, salary, hire_date, email)
VALUES (
    '吴九', 
    DEFAULT,  -- 使用DEFAULT关键字插入默认值
    5000.00 - 4000.00,  -- 计算字段
    CURDATE(),  -- 当前日期
    LOWER('WUJIU@COMPANY.COM')  -- 转换为小写
);

-- 插入部分字段（允许为NULL或有默认值的字段可省略）
INSERT INTO employees (name, department, salary)
VALUES ('李四', '销售部', 4500.00);
```



```
不指定列名插入：
-- 必须为所有列提供值，包括自增ID
INSERT INTO employees
VALUES (NULL, '王五', '人事部', 4800.00, '2023-02-20', 'wangwu@company.com');
```



```
批量插入：
-- 单条语句多值插入，一次插入多条记录（高效）
INSERT INTO employees (name, department, salary, hire_date)
VALUES 
    ('赵六', '技术部', 5200.00, '2023-03-10'),
    ('孙七', '销售部', 4700.00, '2023-03-15'),
    ('周八', '技术部', 5500.00, '2023-03-20');

从查询结果插入：
-- 从临时表或另一张表插入数据
INSERT INTO employees (name, department, salary)
SELECT candidate_name, applied_department, expected_salary
FROM job_applications
WHERE application_status = 'approved';
```



```
插入时处理重复键（MySQL）
-- 如果存在则更新，不存在则插入
INSERT INTO employees (id, name, department, salary)
VALUES (1, '张三', '技术部', 6000.00)
ON DUPLICATE KEY UPDATE 
    name = VALUES(name),
    department = VALUES(department),
    salary = VALUES(salary);
```



---



## 三、更新

```
-- 更新李四的部门和工资
UPDATE employees 
SET department = '市场部', salary = salary + 500
WHERE name = '李四';

-- 从其他表获取数据更新
UPDATE employees e
SET e.salary = (
    SELECT AVG(salary) 
    FROM department_budgets db 
    WHERE db.department = e.department
)
WHERE e.department IN ('技术部', '销售部');

-- 根据条件不同进行不同的更新
UPDATE employees 
SET salary = salary * 1.10,
    department = 
    CASE 
        WHEN salary > 8000 THEN '高级' + department
        ELSE department
    END;
    
--限制更新行数（防止误操作） （MySQL）
UPDATE employees 
SET salary = salary * 1.05 
WHERE department = '技术部'
LIMIT 10;
```



---



## 四、删除

```
-- 删除离职状态且入职超过3年的员工
DELETE FROM employees 
WHERE status = 'inactive' 
  AND hire_date < DATE_SUB(CURDATE(), INTERVAL 3 YEAR);
  
-- 删除没有订单的客户
DELETE FROM customers 
WHERE id NOT IN (SELECT DISTINCT customer_id FROM orders);

-- MySQL 多表删除
DELETE e
FROM employees e
JOIN departments d ON e.department = d.name
WHERE d.status = 'closed';

-- MySQL：分批删除，避免锁表太久
DELETE FROM large_table 
WHERE condition 
LIMIT 1000;
```