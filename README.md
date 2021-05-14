# PingCAP-RangeTest

[TOC]

## 整数类型的Range分区表

### 用例01：建立带有整数类型Range分区的分区表

#### 测试项目

能否成功创建包含整数类型Range分区的分区表

#### 重要级别

高

#### 预置条件

无

#### 测试输入

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

#### 预期结果

能够成功创建并返回成功的语句

#### 实际结果

![image-20210514132941356](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514132941.png)

#### 测试结论

能够成功创建包含整数类型Range分区的分区表

---

### 用例02：插入符合分区条件的数据

#### 测试项目

分区表能否插入符合分区条件的数据

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表

#### 测试输入

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

#### 预期结果

数据能够插入插入分区表中

#### 实际结果

![image-20210514141437785](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141437.png)

#### 测试结论

分区表能够成功插入符合分区条件的数据

---

### 用例03：查看分区表上的分区信息

#### 测试项目

能否成功查看分区表上的分区信息

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区

#### 测试输入

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

#### 预期结果

能够成功显示分区表中的分区信息

#### 实际结果

![image-20210514142127596](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142127.png)

#### 测试结论

能够成功查询分区表的分区信息

---

### 用例04：单个分区查询

#### 测试项目

能否单独查询某个分区的数据

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表并包含数据

#### 测试输入

```sql
SELECT * FROM employees PARTITION(p0);
```

#### 预期结果

能够成功查询出对应Range分区中的数据

#### 实际结果

![image-20210514141527174](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141527.png)

#### 测试结论

能够单独查询单个分区的数据

---

### 用例05：多个分区查询

#### 测试项目

能否单次查询出多个分区的数据并且要查询的每个分区都包含数据

#### 重要级别

高

#### 预置条件

一个包含多个Range分区的分区表

#### 测试输入

```sql
SELECT * FROM employees PARTITION(p0, p1, p2);
```

#### 预期结果

能够单次查询出多个分区的数据

#### 实际结果

![image-20210514141623648](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514141623.png)

#### 测试结论

能够单次查询多个分区的数据

---

### 用例06：包含聚集函数的分区查询

#### 测试项目

能否在进行分区查询的同时使用聚集函数

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表并且查询的分区中包含数据

#### 测试输入

```sql
SELECT COUNT(*) FROM employees WHERE store_id BETWEEN 1 AND 5;
```

#### 预期结果

能够在分区查询时使用聚集函数

#### 实际结果

![image-20210514150520327](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514150520.png)

#### 测试结论

能够在分区查询时使用聚集函数

---

### 用例07：包含GROUP BY的分区查询

#### 测试项目

能否在分区查询的同时使用GROUP BY子句

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表并且查询的分区内包含数据

#### 测试输入

```sql
SELECT job_code, COUNT(store_id) AS c FROM employees PARTITION (p0,p1,p2) GROUP BY job_code HAVING c > 1;
```

#### 预期结果

能够在进行分区查询时使用GROUP BY子句

#### 实际结果

![image-20210514151608511](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514151608.png)

#### 测试结论

能够在进行分区查询时使用GROUP BY子句

---

### 用例08：删除存在的分区

#### 测试项目

能否成功删除存在的分区

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表

#### 测试输入

```sql
ALTER TABLE employees DROP PARTITION p3;
```

#### 预期结果

能够成功删除存在的分区

#### 实际结果

![image-20210514142439699](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142439.png)

![image-20210514142504573](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142504.png)

![image-20210514142532657](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514142532.png)

#### 测试结论

能够成功删除存在的分区

---

### 用例09：删除不存在的分区

#### 测试项目

进行删除不存在的分区时的结果

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表

#### 测试输入

```sql
ALTER TABLE employees DROP PARTITION p4;
```

#### 预期结果

出现错误提示

#### 实际结果

![image-20210514152816437](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152816.png)

#### 测试结论

删除不存在的分区时能够成功提示错误

---

### 用例10：在两个分区之间增加分区

#### 测试项目

能否在已有的两个分区之间增加分区

#### 重要级别

高

#### 预置条件

一个包含两个及以上Range分区的分区表

#### 测试输入

```sql
ALTER TABLE employees ADD PARTITION (PARTITION p3 VALUES LESS THAN (8));
```

#### 预期结果

不能在两个分区之间增加分区

#### 实际结果

![image-20210514145218653](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145218.png)

#### 测试结论

必须按升序增加分区，不能在两个分区之间增加分区

---

### 用例11：在最后一个分区之后增加分区

#### 测试项目

能否按升序增加新的分区

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表

#### 测试输入

```sql
ALTER TABLE employees ADD PARTITION (PARTITION p3 VALUES LESS THAN (21));
```

#### 预期结果

能够成功按升序增加分区

#### 实际结果

![image-20210514145317736](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145317.png)

![image-20210514145412198](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145412.png)

#### 测试结论

能够成功按照升序增加分区

---

### 用例12：插入分区范围外的数据

#### 测试项目

插入不符合分区条件的数据是否会进行错误提示

#### 重要级别

高

#### 预置条件

一个不包含`less than MAXVALUE`Range分区的分区表

#### 测试输入

```sql
INSERT INTO employees(id, fname, lname, hired, job_code, store_id) VALUES (11,'Tom','John','2020/10/15','10','24');
```

#### 预期结果

能够给出错误插入提示

#### 实际结果

![image-20210514145735037](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514145735.png)

#### 测试结论

对不符合分区的数据插入能够正确进行错误提示

---

### 用例13：清空已有分区

#### 测试项目

能否清空已有分区中的数据

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表并且要清空的分区内有数据

#### 测试输入

```sql
ALTER TABLE employees TRUNCATE PARTITION p0;
```

#### 预期结果

能够成功清空分区内的数据

#### 实际结果

![image-20210514152453410](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152453.png)

![image-20210514152546813](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152546.png)

#### 测试结论

能够成功清空分区内的数据

---

### 用例14：清空不存在的分区

#### 测试项目

清空不存在的分区时能否正确给出错误提示

#### 重要级别

高

#### 预置条件

一个包含Range分区的分区表

#### 测试输入

```sql
ALTER TABLE employees TRUNCATE PARTITION p4;
```

#### 预期结果

能够成功给出错误提示

#### 实际结果

![image-20210514152636651](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514152636.png)

#### 测试结论

清空不存在的分区时能够给出正确的错误提示

---

## 时间戳类型的Range分区表

### 用例01：建立带有时间戳类型Range分区的分区表

#### 测试项目

能否建立包含时间戳类型Range分区的分区表

#### 重要级别

高

#### 预置条件

无

#### 测试输入

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

#### 预期结果

能够成功创建并给出成功提示

#### 实际结果

![image-20210514155600297](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514155600.png)

#### 测试结论

能够建立包含时间戳类型Range分区的分区表

---

### 用例02：向时间戳类型的分区表插入数据

#### 测试项目

能否向时间戳类型的分区表插入数据

#### 重要级别

高

#### 预置条件

一个包含时间戳类型Range分区的分区表

#### 测试输入

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

#### 预期结果

能够成功插入数据

#### 实际结果

![image-20210514161017094](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161017.png)

#### 测试结论

能够成功向时间戳类型的分区表插入数据

---

### 用例03：查看分区表的分区信息

#### 测试项目

能否成功显示时间戳类型分区表的分区信息

#### 重要级别

高

#### 预置条件

一个包含时间戳Range分区的分区表

#### 测试输入

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

#### 预期结果

能够成功显示分区信息

#### 实际结果

![image-20210514161152719](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161152.png)

#### 测试结论

能够成功显示时间戳类型分区表的分区信息

---

### 用例04：时间戳分区表的分区查询

#### 测试项目

能否成功在时间戳分区表中进行分区查询

#### 重要级别

高

#### 预置条件

一个包含时间戳类型分区的分区表并且查询的分区内包含数据

#### 测试输入

```sql
SELECT COUNT(*) FROM quarterly_report_status WHERE report_updated BETWEEN '2008-01-01 00:00:00' AND '2009-01-01 00:00:00';
```

#### 预期结果

能够成功进行分区查询

#### 实际结果

![image-20210514161823755](https://raw.githubusercontent.com/juhick/picJuhick/master/20210514161823.png)

#### 测试结论

能够成功在时间戳分区表中进行分区查询

---

## 分区表与普通表的查询和删除效率对比

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

---

### 用例01：对大量数据进行范围查询

#### 测试项目

对包含大量数据的分区表及普通表进行范围查询

#### 重要级别

高

#### 预置条件

一个包含大量数据的分区表和一个包含相同数据的普通表

#### 测试输入

```sql
select * from employees where store_id >= 6 and store_id <= 15;//分区表
select * from employees_noRange where store_id >= 6 and store_id <= 15;//无分区表
```

#### 预期结果

分区表进行范围查询比普通表快

#### 实际结果

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

#### 测试结论

分区表进行范围查询时效率低于普通表

---

### 用例02：按范围进行数据删除

#### 测试项目

对包含的大量数据的分区表和普通表进行范围删除

#### 重要级别

高

#### 预置条件

一个包含大量数据的分区表和一个包含相同数据的普通表

#### 测试输入

```sql
ALTER TABLE employees DROP PARTITION pN;//N = 0, 1, 2
DELETE FROM employees_noRange WHERE store_id <= M;//M = 5, 10, 15
```

#### 预期结果

分区表进行范围删除比普通表效率高得多

#### 实际结果

| N    | M    | 分区表消耗时间 | 无分区表消耗时间 |
| ---- | ---- | -------------- | ---------------- |
| 0    | 5    | 0.37           | 3.97             |
| 1    | 10   | 0.25           | 3.88             |
| 2    | 15   | 0.24           | 4.45             |

#### 测试结论

分区表进行范围删除比普通表效率高得多