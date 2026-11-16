# 第 4 讲 SQL 介绍 2

## `4.1` SQL 数据查询语言

- `4.1.1` SQL 数据查询 `SELECT`

    ```sql
    SELECT
      [ALL | DISTINCT] select_expr, ...
      [FROM table1 [, table2] ...]
      [WHERE 条件表达式]
      [GROUP BY 列名 [ASC | DESC], ...]
      [HAVING 分组过滤条件]
      [ORDER BY 列名 [ASC | DESC], ...]
      [LIMIT [offset,] row_count]
    ```

    - `DISTINCT` 关键字

        `DISTINCT` 为去重关键字，用于去除查询结果中的重复记录，保证输出的每一行都是唯一值（对应地， `ALL` 表示保留全部查询结果，包含重复行）

        - **核心规则**：

            1. 作用于 `SELECT` 后的所有列，多列时会对「所有列的组合值」进行去重
            2. 所有的 `NULL` 值会被当作相同的值处理，即多个 NULL 行只会保留一个
            3. 如果不写默认为 `ALL` （不去重）

        - **示例**：查询学生兴趣爱好且不重复

            ```sql
            SELECT DISTINCT Hobby FROM Student_Info;
            ```

    - `FROM` 子句

        用于指定查询语句要处理的数据源表，是 `SELECT` 语句的可选子句（无数据源时可省略，如计算常量）

        - **语法格式**：
            - 非限定表名：`FROM 表名`，例如 `FROM Student_info`
            - 限定表名（指定数据库）：`FROM 库名.表名`，例如 `FROM eas.db.student_info`
            - 表的别名：可在 `FROM` 子句中给表置临时别名，简化后续语句书写，如 `FROM Student_Info S` 中 `S` 就是表别名

    - `WHERE` 子句

        也叫过滤子句，用于根据指定条件筛选记录，只保留满足条件的行，是最常用的行级过滤手段

        - **执行逻辑**：对 FROM 输出的每一行数据进行条件判断，条件为真则保留，为假则丢弃

        - **示例：**从学生信息表和专业信息表中查询所有的学生姓名和所在专业名称

            ```sql
            SELECT SName, MName FROM Student_Info S, Major_Info M WHERE S.Majer = M.MID;
            ```

    - `GROUP BY` 分组子句

        `GROUP BY` 用于把 `WHERE` 过滤之后的数据，按照指定的一列 / 多列进行分组，相同值的行归为一组，一般配合聚合函数一起使用

        > 聚合函数 `Aggregate Function` ：对一组数据计算得到一个汇总值，常见：`COUNT()` 计数、`SUM()` 求和、`AVG()` 平均值、`MAX()` 最大值、`MIN()` 最小值

        - `SELECT` 后面，非聚合的列，必须出现在 `GROUP BY` 里面（MySQL 默认宽松模式可以不遵守，但是标准 SQL、大部分数据库强制要求）

        - 支持多列分组：先按第一个字段分组，同一个组内再按第二个字段细分

        - 也可以写 `ASC / DESC`，指定分组本身的排序（很多 MySQL 版本不推荐在这里写排序，排序优先交给后面的 `ORDER BY`）

        - 示例：课程表 `Course_Info`，字段：`CName课程名, TTerm学期, CScore学分` ，需求是需要统计每个学期的课程数

            ```sql
            SELECT TTerm, COUNT(*) AS course_num FROM Course_Info GROUP BY TTerm;
            -- 所有学期相同的行合并成一组，COUNT(*) 统计每组里面有多少行
            ```

    - `ORDER BY` 子句

        `ORDER BY` 用于排序子句，用于将查询结果按照指定字段的顺序输出

        - **核心规则**：

            1. 支持两种排序方向：`ASC`（升序，默认值）、`DESC`（降序）
            2. 支持多字段排序，每个字段可单独指定排序方向，优先级按书写顺序从左到右

        - **示例**：按课程名称降序、学分升序排序

            ```sql
            SELECT CName, CScore FROM Course_Info ORDER BY CName DESC, CScore ASC;
            ```

    - `HAVING` 分组过滤子句

        `HAVING` 用于对 `GROUP BY` 分组之后、聚合计算完成的结果，再次过滤

        | 子句     | 过滤对象         | 执行时机            | 能不能用聚合函数 |
        | -------- | ---------------- | ------------------- | ---------------- |
        | `WHERE`  | 原始的单行记录   | `GROUP BY` **之前** | ❌ 不可以         |
        | `HAVING` | 分组后的分组结果 | `GROUP BY` **之后** | ✅ 可以           |

        - **示例：**统计每个学期的课程数量，只保留课程数量大于 3 的学期

            ```sql
            SELECT TTerm, COUNT(*) AS course_num FROM Course_Info GROUP BY TTerm HAVING COUNT(*) > 3;
            ```

            > 这里不能写 `WHERE COUNT(*) >3`，`WHERE` 执行的时候还没有分组、还没有算出 `count` 的值，直接报错

        - ` WHERE` + `GROUP BY` + `HAVING` 的组合场景示例：只统计学分 > 2 的课程，按学期分组，只保留课程总数≥2 的学期

            ```sql
            SELECT TTerm, COUNT(*) AS course_num FROM Course_Info
            WHERE CScore > 2      -- 第一步：先筛掉学分<=2的课程行
            GROUP BY TTerm        -- 第二步：剩下的数据按学期分组统计
            HAVING COUNT(*) >= 2; -- 第三步：过滤掉课程数量不足2个的学期
            ```

    - `LIMIT` 子句

        `LIMIT` 为分页限制子句，用于约束查询结果返回的行数，是 MySQL 特有的分页语法

        - **两种语法格式**：
            1. `LIMIT 行数`：返回前 N 条记录，例如 `LIMIT 3` 返回前 3 行
            2. `LIMIT 偏移量, 行数`：跳过指定偏移量的记录后，返回 N 条记录，用于分页，例如 `LIMIT 3, 3` 表示跳过前 3 条，返回第 4-6 条

- `4.1.2` 整条 `SELECT` 真实执行顺序

    > 书写顺序： `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT`

    1. `FROM`：找到表，加载数据源
    2. `WHERE`：筛选原始行，留下符合条件的记录
    3. `GROUP BY`：对筛选后的行分组
    4. `HAVING`：过滤分组，丢掉不符合条件的组
    5. `SELECT`：计算表达式、别名、聚合结果，确定要输出哪些列
    6. `ORDER BY`：对最终输出结果排序
    7. `LIMIT`：截取指定行数，返回给客户端

    > 这个顺序直接解释两个经典问题：
    >
    >  ① `WHERE` 不能用聚合函数：分组和聚合在 `WHERE` 之后才执行 
    >
    > ② `WHERE` 不能用 `SELECT` 里定义的列别名：别名是 `SELECT` 阶段才生成的

- `4.1.3` 在 `SELECT` 中使用附加运算

    - 更名运算 `Alias`（列别名）

        为查询结果的列重新设置显示名称，只改变**返回结果里的表头名称**，**不会修改数据库里原始表的列名**

        ```sql
        SELECT 原列名 [AS] 新名称 FROM 表名;
        ```

        - 说明：`AS` 关键字可以省略，别名如果包含中文、空格，需要用单引号或者反引号包裹
        - **示例：**查询 `Hobby` 这一列，输出的时候表头文字显示为「兴趣」

        ```sql
        SELECT Hobby as '兴趣' FROM Student_Info;
        ```

    - 字符串匹配运算 `LIKE`

        `LIKE` 是 SQL 标准的模糊匹配关键字，用来判断字符串是否符合给定的匹配模板，放在 `WHERE` 中使用

        ```sql
        [NOT] LIKE <匹配串> [ESCAPE <换码字符>]
        ```

        - 两个核心通配符：

            1. `%`（百分号）：匹配**任意长度（包括 0 个）**的任意字符
                - `'软件%'`：以 “软件” 开头
                - `'%软件'`：以 “软件” 结尾
                - `'%软件%'`：字符串里面包含 “软件”
            2. `_`（下划线）：匹配**恰好 1 个**任意字符
                - `a_b`：总长度固定 3 位，第一位 a，第三位 b，中间任意一个字符，例如 `acb` 、`afb`
            3. `_` 和 `%` 的区别
                - `_` 数量严格对应字符个数，一个下划线只能匹配 1 个字符
                - `%` 不限制长度，可以匹配 0、1、多个字符
                - 示例：`LIKE '_'` 只能匹配长度为 1 的字符串，`LIKE '%'` 匹配所有非 NULL 字符串

        - `NOT LIKE`：取反，匹配**不满足模板**的记录

        - `ESCAPE` 转义子句：当你本身要查找 `%` 或者 `_` 这个符号本身的时候，需要定义转义符 

            **示例：**查找字符串中包含 `_` 的内容，指定 `\` 作为转义符

            ```sql
            WHERE name LIKE 'user\_%' ESCAPE '\';
            ```

            > 这里 `\_` 代表普通下划线，不再是通配符，后面的 `%` 依旧是通配符

    - 各类运算符详解

        - 集合成员运算符 `IN / NOT IN`

            - `IN`：判断字段的值是否**出现在给定集合里面**

                ```sql
                -- 查询专业是计算机、软件工程的学生
                SELECT * FROM Student_Info WHERE Major IN ('计算机','软件工程');
                -- 等价于 `Major = '计算机' OR Major = '软件工程'`
                ```

            - `NOT IN`：判断字段**不在集合中**

        - 空值比较运算符 `IS NULL / IS NOT NULL`

            - `IS NULL`：该字段的值是空（NULL）

            - `IS NOT NULL`：该字段不为空

            ```sql
            -- 查询兴趣爱好为空的学生
            SELECT * FROM Student_Info WHERE Hobby IS NULL;
            ```

            > 不能用 `= NULL` 或者 `!= NULL` 判断空值！`NULL` 代表「未知、不知道」，不是一个可以拿来比较的具体值

        - 比较运算符

            `>` 大于、`>=` 大于等于、`<` 小于、`<=` 小于等于、`=` 等于、`!=` 不等于

            > 注意：SQL 里等于只用一个 `=`，不是编程语言里的 `==`

        - 逻辑运算符 `AND / OR / NOT`

            用来拼接多个条件：

            - `AND`（与）：两边条件**同时成立**，结果才为真
            - `OR`（或）：任意一个条件成立，结果就为真
            - `NOT`（非）：对条件取反

            > 优先级：`NOT > AND > OR`，复杂条件建议用括号 `()` 明确顺序

            - **示例：**学时大于 40 **并且** 学分大于 2

                ```sql
                SELECT CName FROM Course_Info WHERE CTime > 40 AND CScore > 2;
                ```

        - 范围运算符 `BETWEEN ... AND ...`

            判断值是否落在**闭区间**（包含左右边界） 

            ```sql
            字段 BETWEEN 下界 AND 上界
            ```

            - **示例：**生日在 1990-01-01 到 2022-12-31 之间

                ```sql
                SELECT * FROM Student_Info
                WHERE Birthday between '1990-01-01' AND '2022-12-31';
                ```

                > 等价于：`Birthday >= '1990-01-01' AND Birthday <= '2022-12-31'`

## `4.2` 集合运算

集合运算作用于两个 `SELECT` **查询的结果集**，对应离散数学集合：交集 `∩` 、并集 `∪` 、差集 `−` 

> 区分：`AND / OR / NOT` 是**行内条件逻辑**；`INTERSECT / UNION / EXCEPT` 是**两个查询结果集之间的集合运算**

- `4.2.1` 交集 `INTERSECT`

    `INTERSECT` 返回**同时出现在第一个查询结果、第二个查询结果里**的记录，也就是两个集合的公共部分，默认自动去重

    ```sql
    (SELECT ...)
    INTERSECT
    (SELECT ...)
    ```

    - **示例：**查询第 5 学期 **并且** 学分大于 4 的课程名称

        ```sql
        (SELECT CName FROM Course_Info WHERE Term=5)
        INTERSECT
        (SELECT CName FROM Course_Info WHERE CScore>4)
        
        -- 等价于
        SELECT CName FROM Course_Info WHERE Term=5 AND CScore>4;
        ```

    - MySQL 8 没有 `INTERSECT` 关键字，PostgreSQL、SQL Server、Oracle 支持

- `4.2.2` 并集 `UNION` / `UNION ALL`

    把两个查询的结果合并在一起

    | 关键字      | 行为                                   | 性能             |
    | ----------- | -------------------------------------- | ---------------- |
    | `UNION`     | 合并后**自动删除重复行**               | 慢，需要排序去重 |
    | `UNION ALL` | 直接拼接，**保留所有重复记录，不去重** | 快，不需要排序   |

    ```sql
    (SELECT ...)
    UNION
    (SELECT ...)
    
    (SELECT ...)
    UNION ALL
    (SELECT ...)
    ```

    - **示例：**查询第 5 学期或者第 6 学期的课程名称

    ```sql
    (SELECT CName FROM Course_Info WHERE Term=5)
    UNION
    (SELECT CName FROM Course_Info WHERE Term=6)
    
    -- 等价于
    SELECT CName FROM Course_Info WHERE Term=5 OR Term=6;
    ```

    > 什么时候不能简单用 OR？两个查询来自**不同的表**的时候，只能用 UNION

- `4.2.3` 差集 `EXCEPT`（部分数据库也叫 `MINUS`，Oracle）

    返回**在第一个查询结果里面存在，但是第二个查询结果里面不存在**的记录，顺序很关键：`A EXCEPT B` $\neq$ `B EXCEPT A`

    ```sql
    (SELECT ...)
    EXCEPT
    (SELECT ...)
    ```

    - 课件示例：查询第 5 学期，并且学分**不大于 4**的课程名称

    ```sql
    (SELECT CName FROM Course_Info WHERE Term=5)
    EXCEPT
    (SELECT CName FROM Course_Info WHERE CScore>4)
    ```

    > 含义：先拿出第 5 学期全部课程，再去掉其中学分 > 4 的，剩下就是学分≤4 的

    - MySQL 8 不支持 `EXCEPT`，课件给出替代方案：`NOT EXISTS`，也可以用 `LEFT JOIN + IS NULL`

        `NOT EXISTS` 替代写法

        ```sql
        SELECT C1.CName FROM Course_Info C1 WHERE C1.Term = 5
          AND NOT EXISTS (
            SELECT *
            FROM Course_Info C2
            WHERE C2.CScore > 4
              AND C2.CName = C1.CName
        );
        ```

        1. 外层遍历第 5 学期的每一门课程 C1
        2. 内层检查：有没有同一个课程名，学分 > 4
        3. `NOT EXISTS`：内层查不到 → 保留这条记录，实现差集效果

    > 补充 `LEFT JOIN` 版本

    ```sql
    SELECT DISTINCT C1.CName FROM Course_Info C1
    LEFT JOIN Course_Info C2
      ON C1.CName = C2.CName AND C2.CScore>4
    WHERE C1.Term=5 AND C2.CName IS NULL;
    ```

- `4.2.4` 集合运算中的 NULL

    > 普通编程语言是二值逻辑：只有真、假；而 SQL 是**三值逻辑**：`TRUE`、`FALSE`、`UNKNOWN` 
    >
    > 所有和 `NULL` 做 `> < = != AND OR` 这类运算，结果几乎都是 `UNKNOWN`

    1. 普通比较运算（`=`、`>`）中 `NULL` 和任何值运算结果都是 `UNKNOWN`；
    2. 但是在 `INTERSECT` / `UNION` / `EXCEPT` 集合判等的时候，SQL 标准规定：`NULL` 和 `NULL` 视为相等

    > 两个结果集都有一行 `(NULL, '张三')`，`INTERSECT` 会把它当成相同记录保留
    >
    > 而 `WHERE NULL = NULL` 是 `UNKNOWN`，不会匹配

## `4.3` 聚集函数

- `4.3.1` 五大聚集函数

    聚集函数是指对**一组多行数据**做汇总计算，返回**单个值**

    ```sql
    函数名 ( [DISTINCT | ALL] <列名> )
    ```

    > `ALL`：默认，全部记录；`DISTINCT`：先去掉重复值再计算

    1. `COUNT()`

        - `COUNT(*)`：统计表里**所有行**，哪怕整行都是 `NULL`，也计数
        - `COUNT(列名)`：统计这一列**不为 NULL** 的记录，`NULL` 直接跳过

        ```sql
        -- 统计学生表总行数
        SELECT COUNT(*) FROM Student_Info;
        -- 统计Hobby不为NULL的学生数量
        SELECT COUNT(Hobby) FROM Student_Info;
        ```

    2. **SUM()**：数值列求和，**只能用于数值型**，`NULL` 跳过

    3. **AVG()**：数值列求平均值，`NULL` 跳过（`NULL` 不参与分母）

    4. **MAX()**：求最大值，支持数值、字符串、日期

    5. **MIN()**：求最小值，支持数值、字符串、日期

- `4.3.2` `GROUP BY` 分组子句

    `GROUP BY`：把**相同分组字段值**的行归为一组，聚集函数会**每组单独计算一次**

    ```sql
    -- 每个学生的平均分：按学号SNo分组
    SELECT SNo, AVG(Score) FROM Score_Info GROUP BY SNo;
    ```

    - `SELECT` 后面，**没有被聚集函数包裹的列，必须全部写在 GROUP BY 中**

    - 错误示范：`SELECT SNo, CNo, AVG(Score) FROM Score GROUP BY SNo;` 

        > CNo 不在 GROUP BY，也没有聚集，语法报错

    - `GROUP_CONCAT ()`（MySQL 特有）

        把同一组内多条记录的某个字段拼接成一条字符串

        ```sql
        SELECT SNo, GROUP_CONCAT(Score) AS Scores FROM Score_Info GROUP BY SNo;
        ```

        结果示例：`20200101 | 90,85,88`

    - `WITH ROLLUP`（分组汇总）

        `WITH ROLLUP` 是 `GROUP BY` 的修饰符，表示**沿着分组字段从右往左逐层向上汇总**，生成小计、总计行，**汇总行的分组字段会填充为** `NULL`

        ```sql
        SELECT SNo, SUM(Score) as TotalScore FROM Score_Info GROUP BY SNo WITH ROLLUP;
        ```

        最后一行 SNo 为 NULL，SUM 是全部学生总分总和

- `4.3.3` HAVING 子句

    `HAVING` 用于分组完成之后，对聚集结果过滤

    ```sql
    -- 只保留总分大于520的学生分组
    SELECT SNo, SUM(Score) as TotalScore
    FROM Score_Info
    GROUP BY SNo
    HAVING SUM(Score) > 520;
    ```

    一句话口诀：`WHERE` 筛原始行，`HAVING` 筛分组；`WHERE` 不能用 `SUM` / `AVG`，`HAVING` 可以

## ` 4.4` 嵌套子查询

嵌套子查询是把**一条 SELECT 查询的结果，嵌入到另一条查询的 WHERE 条件里**，内层叫子查询，外层叫父查询 

```sql
SELECT * FROM Table1
WHERE ColumnName IN (
    SELECT ColumnName FROM Table2 WHERE ...
);
```

`IN` 的含义：父查询这一行的字段值，只要**出现在子查询返回集合里**，就满足条件

- **示例：**查询选了 C01 课程的学生姓名

    ```sql
    SELECT SName FROM Student WHERE Sno IN (SELECT Sno FROM SC WHERE Cno='C01');
    ```