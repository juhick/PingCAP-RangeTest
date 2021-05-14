# PingCAP-RangeTest

[TOC]

## 整数类型的Range分区表

### 建立带有整数类型Range分区的分区表

#### 输入：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)

PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

#### 输出：

![image-20210514132941356](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514132941.png)

### 插入数据

#### 输入：

```sql
INSERT INTO employees(id, fname, lname, hired, job_code, store_id) VALUES (1,'Tom','John','2020/10/12','12','2'),
(2,'Tom','John','2020/10/13','11','3'),
(3,'Tom','John','2020/10/14','4','11'),
(4,'Tom','John','2020/10/15','5','15'),
(5,'Tom','John','2020/10/16','7','17'),
(6,'Tom','John','2020/10/17','3','20'),
(7,'Tom','John','2020/10/18','1','13'),
(8,'Tom','John','2020/10/19','1','6'),
(9,'Tom','John','2020/10/20','3','5'),
(10,'Tom','John','2020/10/21','7','1');
```

#### 输出：

![image-20210514141437785](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141437.png)

### 查看表上的分区信息

#### 输入：

```sql
select 
  partition_name part,  
  partition_expression expr,  
  partition_description descr,  
  table_rows  
from information_schema.partitions  where 
  table_schema = schema()  
  and table_name='employees';
```

#### 输出：

![image-20210514142127596](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142127.png)

### 分区查询

#### 输入1：

```sql
SELECT * FROM employees PARTITION(p0);
```

#### 输出1：

![image-20210514141527174](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141527.png)

#### 输入2：

```sql
SELECT * FROM employees PARTITION(p0, p1);
```

#### 输出2：

![image-20210514141554587](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141554.png)

#### 输入3：

```sql
SELECT * FROM employees PARTITION(p0, p1, p2);
```

#### 输出3：

![image-20210514141623648](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141623.png)

#### 输入4：

```sql
SELECT COUNT(*) FROM employees WHERE store_id BETWEEN 1 AND 5;
```

#### 输出4：

![image-20210514150520327](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514150520.png)

#### 输入5：

```sql
SELECT COUNT(*) FROM employees WHERE store_id BETWEEN 1 AND 10;
```

#### 输出5：

![image-20210514150543106](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514150543.png)

#### 输入6：

```sql
SELECT COUNT(*) FROM employees WHERE store_id BETWEEN 1 AND 15;
```



#### 输出6：

![image-20210514150606015](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514150606.png)

#### 输入7：

```sql
SELECT job_code, COUNT(*) FROM employees WHERE store_id BETWEEN 6 AND 15 GROUP BY job_code;
```

#### 输出7：

![image-20210514151227242](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514151227.png)

#### 输入8：

```sql
SELECT job_code, COUNT(store_id) AS c FROM employees PARTITION (p0,p1,p2) GROUP BY job_code HAVING c > 1;
```

#### 输出8：

![image-20210514151608511](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514151608.png)

### 删除分区

#### 输入1：

```sql
ALTER TABLE employees DROP PARTITION p3;
```

#### 输出1：

![image-20210514142439699](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142439.png)

![image-20210514142504573](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142504.png)

![image-20210514142532657](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142532.png)

#### 输入2：

```sql
ALTER TABLE employees DROP PARTITION p4;
```

#### 输出2：

![image-20210514152816437](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152816.png)

### 增加分区

#### 输入1：

```sql
ALTER TABLE employees ADD PARTITION (PARTITION p3 VALUES LESS THAN (8));
```

#### 输出1：

![image-20210514145218653](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145218.png)

#### 输入2：

```sql
ALTER TABLE employees ADD PARTITION (PARTITION p3 VALUES LESS THAN (21));
```

#### 输出2：

![image-20210514145317736](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145317.png)

![image-20210514145412198](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145412.png)

### 插入分区范围外的数据

#### 输入：

```sql
INSERT INTO employees(id, fname, lname, hired, job_code, store_id) VALUES (11,'Tom','John','2020/10/15','10','24');
```

#### 输出：

![image-20210514145735037](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145735.png)

### 清空分区

#### 输入1：

```sql
ALTER TABLE employees TRUNCATE PARTITION p0;
```

#### 输出1：

![image-20210514152453410](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152453.png)

![image-20210514152546813](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152546.png)

#### 输入2：

```sql
ALTER TABLE employees TRUNCATE PARTITION p4;
```

#### 输出2：

![image-20210514152636651](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152636.png)

## 时间戳类型的Range分区表

### 建立带有时间戳类型Range分区的分区表

#### 输入：

```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)

PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
    PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
    PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
    PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
    PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
    PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
    PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
    PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
    PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
    PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
);
```

#### 输出：

![image-20210514155600297](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514155600.png)

### 插入数据

#### 输入：

```sql
INSERT INTO quarterly_report_status VALUES
(1,'OK','2007-01-01 09:09:00'),
(2,'OK','2008-03-06 00:00:00'),
(3,'OK','2008-02-24 00:00:00'),
(4,'OK','2008-04-24 00:00:00'),
(5,'OK','2008-05-25 00:00:00'),
(6,'OK','2008-07-02 00:00:00'),
(7,'OK','2008-08-01 00:00:00'),
(8,'OK','2008-10-01 00:00:00'),
(9,'OK','2008-12-20 00:00:00'),
(10,'OK','2009-03-24 00:00:00'),
(11,'OK','2009-02-12 00:00:00'),
(12,'OK','2009-04-25 00:00:00'),
(13,'OK','2009-05-22 00:00:00'),
(14,'OK','2009-07-05 00:00:00'),
(15,'OK','2009-08-08 00:00:00'),
(16,'OK','2009-12-24 00:00:00'),
(17,'OK','2009-10-10 00:00:00'),
(18,'OK','2009-12-31 00:00:00'),
(19,'OK','2010-12-20 00:00:00'),
(20,'OK','2010-10-25 00:00:00');
```

#### 输出：

![image-20210514161017094](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161017.png)

### 查看分区信息

#### 输入：

```sql
select 
  partition_name part,  
  partition_expression expr,  
  partition_description descr,  
  table_rows  
from information_schema.partitions  where 
  table_schema = schema()  
  and table_name='quarterly_report_status';
```

#### 输出：

![image-20210514161152719](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161152.png)

### 分区查询

#### 输入：

```sql
SELECT COUNT(*) FROM quarterly_report_status WHERE report_updated BETWEEN '2008-01-01 00:00:00' AND '2009-01-01 00:00:00';
```

#### 输出：

![image-20210514161823755](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161823.png)

### 大量数据插入

[插入语句](./data.sql)

### 带有Range分区的表

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)

PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

![image-20210514165213854](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514165213.png)

### 未带有Range分区的表

```sql
CREATE TABLE employees_noRange (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
);
```

![image-20210514190944973](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514190945.png)

### 大量数据范围查询

#### 输入：

```sql
select * from employees where store_id >= 6 and store_id <= 15;//分区表
select * from employees_noRange where store_id >= 6 and store_id <= 15;//无分区表
```

#### 输出：

| 序号   | 分区表消耗时间 | 无分区表消耗时间 |
| ------ | -------------- | ---------------- |
| 1      | 1.39           | 1.60             |
| 2      | 0.82           | 0.43             |
| 3      | 0.80           | 0.44             |
| 4      | 0.72           | 0.42             |
| 5      | 1.03           | 0.55             |
| 6      | 0.75           | 0.40             |
| 7      | 0.81           | 0.61             |
| 8      | 0.78           | 0.36             |
| 9      | 0.98           | 0.57             |
| 10     | 0.77           | 0.40             |
| 平均值 | 0.885          | 0.578            |

### 按范围删除数据

#### 输入：

```sql
ALTER TABLE employees DROP PARTITION pN;
DELETE FROM employees_noRange WHERE store_id <= M;
```

#### 输出：

| N    | M    | 分区表消耗时间 | 无分区表消耗时间 |
| ---- | ---- | -------------- | ---------------- |
| 0    | 5    | 0.37           | 3.97             |
| 1    | 10   | 0.25           | 3.88             |
| 2    | 15   | 0.24           | 4.45             |