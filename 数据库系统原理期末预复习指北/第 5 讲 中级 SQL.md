# 第 5 讲 中级 SQL

## `5.1` 连接表达式

- `5.1.1` 连接查询 `Join`

    - 连接查询，也叫**联接、多表查询**，作用是从多张存在关联的表中联合查询数据，通过 `FROM` 子句实现

        ```mermaid
        graph LR
            A[连接查询] --> B[内连接 Inner Join]
            A --> C[外连接 Outer Join]
            A --> D[交叉连接 Cross Join]
            B --> E[等值连接]
            B --> F[非等值连接]
            B --> G[自然连接 Natural Join]
            C --> H[左外连接 Left Join]
            C --> I[右外连接 Right Join]
            C --> J[全外连接 Full Join]
        ```

    - 连接运算，就是把多张表的行，按照给定条件拼接，生成一张新的结果表

- `5.1.2` 连接表达式

    - 内连接 `Inner Join`

        只返回**两张表中满足连接条件**的记录，不匹配的行会直接丢弃

        - 等值连接 `Equijoin` ：连接条件使用 `=`，结果保留全部列（包含重复同名列）
        - 自然连接 `Natrual Join` ：属于特殊等值连接，自动以**同名公共字段**做等值匹配，自动剔除重复列
        - 非等值连接 `Non-equijoin` ：连接条件使用 `>、>=、<、<=、!=、<>` 等，不用等号

    - 外连接 `Outer Join`

        **保留一张表的全部数据，匹配另一张表**，匹配不到的字段填充`NULL`

        - 左外连接 `Left Join` ：保留左表全部行，右表匹配不上填`NULL`
        - 右外连接 `Right Join` ：保留右表全部行，左表匹配不上填`NULL`
        - 全外连接 `Full Join` ：左右两表所有行都保留，匹配不到的位置填`NULL`

        > MySQL 原生不支持 `FULL JOIN` ，全外连接效果需要用 `LEFT JOIN UNION RIGHT JOIN` 实现

    - 交叉连接 `Cross Join`

        也叫笛卡尔积，不设置连接条件，表 A 每一行和表 B 每一行全部两两组合

    - 自连接 `Self Join`

        一张表自己和自己连接，把一张表当成两张不同表来使用，常用于层级结构（员工 - 上级、分类树）

## `5.2` 内连接

- `5.2.1` 基本语法结构

    1. `JOIN ... ON` 写法（推荐，可读性强）

        ```sql
        SELECT 列名 FROM 表1 [INNER] JOIN 表2 ON 表1.列1 = 表2.列2;
        ```

        > `INNER` 关键字可以省略，只写 `JOIN` 默认就是内连接；`ON` 后面写连接条件

    2. `WHERE` 隐式连接写法

        ```sql
        SELECT 列名 FROM 表1, 表2 WHERE 表1.列1 = 表2.列2;
        ```

- `5.2.2` 自然连接

    自然连接将自动寻找**两张表同名的列**作为等值匹配条件，自动合并、**去除重复列**

    ```sql
    SELECT 列名 FROM 表1 NATURAL JOIN 表2;
    -- 或者
    SELECT 列名 FROM 表1, 表2 WHERE 表1.列1 = 表2.列2;
    ```

    | 对比项   | 等值连接                     | 自然连接                 |
    | -------- | ---------------------------- | ------------------------ |
    | 匹配条件 | 人为指定 `=`，字段可以不同名 | 自动匹配**同名公共字段** |
    | 重复列   | 保留全部列，公共字段重复显示 | 自动去掉重复的公共列     |
    | 从属关系 | 大类，包含自然连接           | 是等值连接的一个子集     |

- `5.2.3` 非等值查询

    连接查询中用来连接两个表的条件成为连接条件或连接谓词，一般格式为：

    ```sql
    [<表名1>.] <列名1> <比较运算符> [<表名2>.] <列名2>
    ```

    当比较运算符不为 `=`  时，这样的连接称为非等值连接，常用于区间的匹配与查询

## `5.3` 外连接

- `5.3.1` 从内连接到外连接

    - 内连接 `Inner Join`：仅把两张表中**满足连接条件**的记录展示出来，不匹配的行会直接丢弃
    - 外连接 `Outer Join`：除满足条件的匹配行之外，**可以保留不满足匹配条件的行**，缺失字段用 `NULL` 填充

- `5.3.2` 左外连接

    以**左侧表**为主表，**保留左表里面的所有行**，右表中和左表无法匹配的记录，对应字段填充 `NULL`

    ```sql
    SELECT 列名列表 FROM 表1 LEFT [OUTER] JOIN 表2 ON 表1.列1 = 表2.列2;
    ```

    示例：查询**所有学生**的选课、成绩情况，没有选课的学生也要出现在结果中

    ```sql
    SELECT s.SNo, s.SName, sc.CID, sc.Score FROM student_info s LEFT OUTER JOIN score_info sc 
    	ON s.SNo = sc.SNo;
    ```

    > 所有学生一定会出现在结果里；没有选课的学生，课程编号 `CID` 、成绩 `Score` 字段为 `NULL`

- `5.3.3` 右外连接

    以**右侧表**为主表，**保留右表里面的所有行**，左表中和右表无法匹配的记录，对应字段填充 `NULL`

    ```sql
    SELECT 列名列表 FROM 表1 RIGHT [OUTER] JOIN 表2 ON 表1.列1 = 表2.列2;
    ```

    示例：查询**所有开设课程**的选修情况，没有学生选修的课程也要展示

    ```sql
    SELECT DISTINCT sc.CID, c.* FROM score_info sc RIGHT OUTER JOIN course_info c
    	ON c.CID = sc.CID;
    ```

    > 全部课程一定出现在结果里；没人选的课程，学生成绩相关字段为 `NULL`

    可以发现，**右外连接完全可以改写为左外连接**，交换两张表位置即可

- `5.3.4` 全外连接

    **同时保留左表、右表的全部行**，两边无法互相匹配的行，缺失字段填充 `NULL`

    ```sql
    SELECT 列名列表 FROM 表1 FULL [OUTER] JOIN 表2 ON 表1.列1 = 表2.列2;
    ```

    示例：对于学生表和宿舍表，有的学生不住宿、有的床位空闲，需要列出**全部学生和全部床位**的分配情况

    ```sql
    SELECT s.s_id, s.s_name, b.bed_id, b.building, b.room_no FROM student s FULL OUTER JOIN bed b
    	ON s.bed_id = b.bed_id;
    
    -- MySQL 没有 FULL JOIN，需要 UNION
    
    -- 第一部分：左外连接 → 拿到【所有学生】，包含无床位的学生
    SELECT s.s_id, s.s_name, b.bed_id, b.building, b.room_no FROM student s LEFT JOIN bed b
        ON s.bed_id = b.bed_id
    
    UNION
    
    -- 第二部分：右外连接 → 拿到【所有床位】，只保留匹配不到学生的空闲床位
    SELECT s.s_id, s.s_name, b.bed_id, b.building, b.room_no FROM student s RIGHT JOIN bed b
        ON s.bed_id = b.bed_id;
    ```

    | 连接类型              | 保留范围                | 不匹配行填充        | 等价改写思路                      |
    | --------------------- | ----------------------- | ------------------- | --------------------------------- |
    | 左外连接 `Left Join`  | 左表全部行 + 右表匹配行 | 右表字段 `NULL`     | -                                 |
    | 右外连接 `Right Join` | 右表全部行 + 左表匹配行 | 左表字段 `NULL`     | 调换两表，改为 Left Join          |
    | 全外连接 `Full Join`  | 左表全部行 + 右表全部行 | 缺失一侧字段 `NULL` | MySQL：Left Join UNION Right Join |

## `5.4` 交叉连接

- `5.4.1` 交叉连接 `Cross Join`

    交叉连接，又称**笛卡尔连接** `Cartesian Join` 、叉乘 `Product`，是指对两张表做笛卡尔积（**表 A 每一行，和表 B 的每一行两两组合**，生成全部组合结果）

- `5.4.2` 基本语法

    ```sql
    -- 语法1：标准 cross join 写法
    SELECT 列名 FROM 表1 CROSS JOIN 表2;
    
    -- 语法2：逗号分隔（旧写法，等价cross join）
    SELECT 列名 FROM 表1,表2;
    
    -- 注意和以下内连接的写法区分
    SELECT 列名 FROM 表1, 表2 WHERE 表1.列1 = 表2.列2;
    ```

    > 交叉连接是内连接、外连接的底层基础，其他连接本质 = 笛卡尔积 + 过滤条件

## `5.5` 自连接

自连接 `SELF JOIN` ，是指将同一个表的不同行连接起来，可以看作一张表的两个副本之间进行的连接

> 本质依然是内连接 / 外连接，只是参与连接的两张表来源于同一个物理表

示例：`Student_Info(SNo, MID, SName, Sex, Birthday)` 和 `Score_Info(SNo, CID, Score)`

```sql
-- 1. 查询和“王林”一个专业的其他学生
select b.* from Student_Info a, Student_Info b where a.SName='王林' 
	and a.MID = b.MID and a.SNo != b.SNo;

-- 2. 列出全部学生，并且标出哪些选了 2 号课，没选 2 号课的学生也要保留（选课字段为 NULL）
SELECT s.SNo, s.SName, sc.CID FROM Student_Info s LEFT JOIN Score_Info sc 
    ON s.SNo = sc.SNo AND sc.CID = 2;
```