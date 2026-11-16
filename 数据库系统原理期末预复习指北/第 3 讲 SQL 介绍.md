# 第 3 讲 SQL 介绍

## `3.1` SQL 语言分类

- `3.1.1` 结构化查询语言

    结构化查询语言 `Structured Query Language (SQL)` ，是用于操作和管理关系型数据库的标准化语言，由美国国家标准协会 `ANSI` 和国际标准化组织 `ISO` 共同制定标准，广泛应用于 MySQL、Oracle、SQL Server 等数据库系统

    ```mermaid
    graph LR
        A["SQL语言<br/>Structured Query Language"] --> B["数据定义语言 DDL"]
        A --> C["数据操纵语言 DML"]
        B --> B1["DDL：负责数据库的逻辑结构操作<br/>create, drop, alter"]
        B --> B2["TCL：负责事务处理<br/>commit, rollback"]
        B --> B3["DCL：负责权限管理<br/>grant, revoke"]
        C --> C1["DML：负责数据的添加、更新、删除<br/>insert, update, delete"]
        C --> C2["DQL：负责数据查询<br/>select"]
    ```

- `3.1.2` 数据定义语言 `Data Definition Language (DDL)` —— SQL 中集中负责**数据结构定义与数据库对象定义**的语言

    - 功能包括创建数据库中的各种对象；创建、删除、修改**表的结构**

    - 可定义的对象包括**表、视图、索引、同义词、聚簇（Cluster）**等

        > DDL操作的是"对象的结构"而非"对象里的数据"
        >
        > 比如 `ALTER TABLE Student ADD COLUMN age INT;` 是给表加一列（改结构），而不是给某个学生加年龄（改数据）

- `3.1.3` 事务控制语言 `Transaction Control Language (TCL)` —— 用于**管理数据库中的事务**，管理由 **DML 语句所做的更改**，并允许把多条语句**分组为逻辑事务**

    - TCL 经常用于快速原型开发、脚本编程、GUI 和测试等方面

        > TCL 是与 DML **配套**使用的：
        >
        > 一个事务 `Transaction` 是“一组不可分割的操作”，例如银行转账=扣款+入账两条 DML 语句，必须同生共死 —— 这就是“逻辑事务”的含义`COMMIT` 让事务永久生效，`ROLLBACK` 撤销整个事务回到起点

- `3.1.4` 数据控制语言 `Data Control Language (DCL)` —— 一种**对数据访问权进行控制**的指令，可控制特定用户账户对**数据表、视图（查看表）、预存程序（存储过程）、用户自定义函数**等数据库对象的控制权

    - DCL 用来授予或回收访问数据库的某种**特权**；控制数据库操纵事务发生的时间及效果；对数据库实行**监视**

        > DCL 服务的是数据库的**安全性与权限管理**，典型使用者是数据库管理员 DBA ，`GRANT` 授权、`REVOKE` 收权，二者一授一收、互为逆操作

-  `3.1.5` 数据操纵语言 `Data Manipulation Language (DML)` —— 实现对数据库的**基本操作**，对数据库中的**对象和数据**运行访问工作的语言

    - DML 的主要功能是**访问数据**，语法以**读写数据库**为主

        > DML 与 DDL —— **DML 改表里的数据行，DDL 改表的定义**：
        >
        > `INSERT` / `UPDATE` / `DELETE` 执行后默认处于"未提交"状态，可用 `ROLLBACK` 撤销；而 `DDL` 执行即隐式提交、不可回滚

- `3.1.6` 数据查询语言 `Data Query Language (DQL)` —— SQL中**负责数据查询、而不会对数据本身进行修改**的语句

    - DQL 只“看”不“改”：`SELECT` 语句对数据库是只读操作，这是它区别于 DML 的本质特征
    - `select` 通常与 `from`、`where`、`group by`、`having`、`order by` 等组成完整查询语句

    | 分类                   | 职责                                  | 关键字                         |
    | ---------------------- | ------------------------------------- | ------------------------------ |
    | **数据定义语言** `DDL` | 定义/修改数据库结构（表、视图、索引） | `CREATE` 、`DROP` 、`ALTER`    |
    | **事务控制语言** `TCL` | 事务提交与回滚                        | `COMMIT` 、`ROLLBACK`          |
    | **数据控制语言** `DCL` | 权限管理（授权/收权）                 | `GRANT` 、`REVOKE`             |
    | **数据操纵语言** `DML` | 表中数据的增、删、改                  | `INSERT` 、`UPDATE` 、`DELETE` |
    | **数据查询语言** `DQL` | 查询表中数据                          | `SELECT`                       |

## `3.2` SQL 数据类型

- `3.2.1` SQL 数据类型概述

    在创建表的列时，必须为其制定数据类型，列的数据类型决定了数据的取值、范围和存储格式

    | 大类       | 小类       | 包含类型                                                | 核心特征                             |
    | ---------- | ---------- | ------------------------------------------------------- | ------------------------------------ |
    | 数值型     | 整数型     | `TINYINT` / `SMALLINT` / `MEDIUMINT` / `INT` / `BIGINT` | 存储整数，字节数 1→8，范围递增       |
    | 数值型     | 精确数值型 | `DECIMAL` / `NUMERIC`                                   | 固定精度，**不丢精度**，适合金额     |
    | 数值型     | 浮点型     | `FLOAT` / `DOUBLE` / `REAL`                             | 近似存储，**可能丢精度**，适合统计量 |
    | 数值型     | 位型       | `BIT(M)`                                                | 按位存储，M 为 1~64 位               |
    | 字符串型   | 字符型     | `CHAR` / `VARCHAR`                                      | 定长补空格 / 变长按实际存储          |
    | 字符串型   | 文本型     | `TINYTEXT` / `TEXT` / `MEDIUMTEXT` / `LONGTEXT`         | 大文本，最大 255 ~ 4G 字符           |
    | 字符串型   | 二进制型   | `BINARY` / `VARBINARY`                                  | 存字节串，无字符集                   |
    | 字符串型   | 大对象型   | `TINYBLOB` / `BLOB` / `MEDIUMBLOB` / `LONGBLOB`         | 存照片、视频等二进制大对象           |
    | 字符串型   | 枚举集合型 | `ENUM` / `SET`                                          | 取值限于预定义列表：单选 / 多选      |
    | 日期时间型 | —          | `DATE` / `TIME` / `DATETIME` / `TIMESTAMP` / `YEAR`     | 5 种日期时间格式                     |

    > 在讨论数据类型时，需要使用精度、小数位数（数值型数据特有）和长度三个概念：
    >
    > | 概念     | 含义                                 | 例子                        |
    > | -------- | ------------------------------------ | --------------------------- |
    > | 精度     | 数值数据中所存储的十进制数据的总位数 | 3560.697 的精度是 **7**     |
    > | 小数位数 | 小数点右边可以有的数字位数的最大值   | 3560.697 的小数位数是 **3** |
    > | 长度     | 存储数据所使用的**字节数**           | 如 INT 长度为 4 字节        |

- `3.2.2` 整数型

    整数型包括 `TINYINT` / `SMALLINT` / `MEDIUMINT` / `INT` / `BIGINT` ，从名称即可看出**表示范围逐渐缩小**

    | 类型              | 有符号范围                                   | 精度 | 小数位数 | 长度（字节） |
    | ----------------- | -------------------------------------------- | ---- | -------- | ------------ |
    | `BIGINT`          | -2^63^ ~ 2^63^-1<br />（约$\pm$ 9.2×10^18^） | 19   | 0        | **8**        |
    | `INT（=INTEGER）` | -2^31^ ~ 2^31^-1<br />（约 $\pm$ 2.1×10^9^） | 10   | 0        | **4**        |
    | `MEDIUMINT`       | -2^23^ ~ 2^23^-1<br />（约$\pm$ 8.4×10^6^）  | 7    | 0        | **3**        |
    | `SMALLINT`        | -2^15^ ~ 2^15^-1<br />（-32768~32767）       | 5    | 0        | **2**        |
    | `TINYINT`         | -2^7^ ~ 2^7^-1<br />（-128~127）             | 3    | 0        | **1**xXQ     |

- `3.2.3` 精确数值型

    精确数值型由整数部分和小数部分构成，其所有的数字都是有效位，能够以完整的精度存储十进制数

    - 声明：`NUMERIC | DECIMAL(p[,s])`
        - p（精度，precision）表示数值的总位数（整数位 + 小数位）
        - s（标度，scale）表示小数点后的位数，**默认值为 0**
    - 精确数值型包括 `decimal` 、`numeric` 两类，两者功能完全等价

- `3.2.4` 浮点型

    不能提供精确表示的精度，存储时可能损失精度，适合**取值范围非常大、对精确度要求不高**的数值

    | 类型   | 语法                       | 取值范围                           | 存储   | 精度           |
    | ------ | -------------------------- | ---------------------------------- | ------ | -------------- |
    | 单精度 | `FLOAT[(M,D)] [ZEROFILL]`  | $\pm$ 3.402 823 466E+38            | 4 字节 | 约 7 位小数位  |
    | 双精度 | `DOUBLE[(M,D)] [ZEROFILL]` | $\pm$ 1.797 693 134 862 315 7E+308 | 8 字节 | 约 15 位小数位 |

    - `M` 是小数总位数，`D` 是小数点后面的位数（如果省略 `M` 和 `D` 将会根据硬件允许的限制来保存值）

    - `FLOAT(p)` 按精度自动选类型

        `FLOAT(p)` 中 `p` 仅表示**精度（以位数表示）**，MySQL 据此决定实际类型：

        - p = 0 ~ 24 → 按没有 M 或 D 的 FLOAT 处理
        - p = 25 ~ 53 → 按没有 M 或 D 的 DOUBLE 处理

- `3.2.5` 位型

    `BIT[(M)]` ，`M` 表示可存储的**位数**，MySQL 官方规定范围为 **1～64**；省略 `M` 时按 `BIT(1)` 理解

    - 示例：位面值写法可用 `b'101'` ，赋给 `BIT(6)` 会左侧补 0 成 `b'000101'`

- `3.2.6` 字符型

    字符型数据用于存储字符串，字符串中可包括字母、数字和其他特殊符号（如# 、@ 、& 等）

    - SQL 字符串常量建议用**单引号** `'abc'`、`'Abc<Cde'` ，双引号在 MySQL 默认可用作字符串引号，但更通用、更符合标准的是单引号（双引号在其他数据库中常表示标识符，混用容易出兼容问题）
    - 定长字符 `CHAR[(N)]`
        - `N` 可选，在 1~255 之间，默认为 1，存短字符串时会在右侧补**空格**到长度 N，因此“逻辑长度”固定
        - 非严格模式下超长可能被截断；**严格 SQL 模式**下超长值不会保存并报错
        - 适用于长度固定的情形，如国家码 `CHAR(2)` 、状态码、定长哈希前缀等
    - `VARCHAR[(N)]`：变长字符
        - `N` 表示（字符串可达到的）**最大字符长度**，在 `0～65535` 之间，但实际有效长度受行大小 65,535 字节与字符集限制
        - 只按实际内容存储，但每个值还要加长度前缀
        - 适用于姓名、邮箱、地址、描述等长度变化大的字段

- `3.2.7` 文本型

    当需要存储大量的字符数据，如较长的备注、日志信息等，字符型数据的最长 65535 个字符的限制可能使它们不能满足应用需求，此时可使用文本型数据

    | 类型       | 最大长度（字节） | 2 的幂表达 | 典型用途                 |
    | ---------- | ---------------- | ---------- | ------------------------ |
    | TINYTEXT   | 255              | 2^8^ − 1   | 短摘要、链接、备注       |
    | TEXT       | 65,535           | 2^16^ − 1  | 评论、文章、商品描述     |
    | MEDIUMTEXT | 16,777,215       | 2^24^ − 1  | 长文章、电子书章节、日志 |
    | LONGTEXT   | 4,294,967,295    | 2^32^ − 1  | 超大文本、归档日志       |

- `3.2.8` 二进制串和变长二进制串类型

    类似 `CHAR/VARCHAR`，但存的是**二进制字节串**，不是普通字符文本，因此它们使用 binary 字符集/排序规则，比较与排序基于字节数值

    | 类型            | 含义              | 长度 N          | 填充/截断                                               | 比较特点                                              |
    | --------------- | ----------------- | --------------- | ------------------------------------------------------- | ----------------------------------------------------- |
    | BINARY\[(N)]    | 定长 N 字节       | 1–255，默认 1   | 不足右侧补 `0x00`；过长非严格模式截断告警，严格模式报错 | 所有字节都参与比较，`0x00` 与空格不同且 `0x00` 排更前 |
    | VARBINARY\[(N)] | 变长，最多 N 字节 | 1–65535，默认 1 | 插入不补零、读取不去尾                                  | 同样按字节数值比较，但没有定长补零副作用              |

- `3.2.9` 二进制大对象类型

    二进制大对象类型 `Binary Large Obiect (BLOB)` 用来存储可变长度的二进制数据，适合存放图片、照片、视频、扫描文档、音频这类文件二进制流

    - 同样 4 个子类型：`TINYBLOB` / `BLOB` / `MEDIUMBLOB` / `LONGBLOB`
    - BLOB 系列最大字节长度和 TEXT 系列一一对应

- `3.2.10` 日期时间类型

    MySQL 一共 5 种时间类型：`DATE`、`TIME`、`DATETIME`、`TIMESTAMP`、`YEAR`

    1. `DATE` ：只存日期，格式 `YYYY-MM-DD`

        - 合法范围：`1000-01-01` ~ `9999-12-31`
        - 分隔符不限于`-`，`/`、`@`甚至无分隔都支持，例如`19970806`等价`1997-08-06`

        > 超出范围的值可以写入，但日期计算不可靠

    2. `TIME`

        存储一天内的时间，格式 `HH:MM:SS[.微秒]`，微秒最多 6 位，可以省略

        - 必须是合法时间：`10:08:34` 合法；`10:98:10` 非法（分钟不能≥60），非法值自动转为`00:00:00`

    3. `DATETIME` & `TIMESTAMP` 

        两者都同时存「日期 + 时间」，格式 `YYYY-MM-DD HH:MM:SS` ，空格分隔日期和时间

        | 对比项   | DATETIME                   | TIMESTAMP                                      |
        | -------- | -------------------------- | ---------------------------------------------- |
        | 时间范围 | 1000-01-01 ~ 9999-12-31    | 1970-01-01 ~ 2038-01-19（UTC）                 |
        | 时区特性 | 不做时区转换，存什么取什么 | 自动时区转换：存入转 UTC，查询转回当前会话时区 |

    4. `YEAR`

        只存年份，显示格式`YYYY`，范围 `1901 ~ 2155`，仅占用 1 字节

- `3.2.11` 特殊字符串枚举类型

    ENUM 和 SET 类型的合法取值列表在建表时预先定义，字段只能从列表里选值，用来约束数据，减少脏数据

    - 枚举类型 `ENUM` ，只能选列表中的一个值（单选）；最多 65535 个候选项 

        示例：`ENUM('Y','N')` ，这个字段只能存 `Y` 或者 `N`

    - 集合类型 `SET` ，可以同时选 0 个、1 个或多个列表内的值（多选）；最多 64 个候选项 

        示例：`SET('a','b','c')` ，可以存 `'a,b'` 、`'b,c' `、空字符串

## `3.3` SQL 数据定义语言

- `3.3.1` 数据库操作

    - 创建数据库

        ```sql
        CREATE DATABASE [IF NOT EXISTS] db_name;
        ```

        - `IF NOT EXISTS`：可选，**判断数据库不存在才创建**，避免重复创建报错
        - MySQL 中数据库名**不区分大小写**，数据库本质是文件系统里的目录
    - 使用数据库

        ```sql
        USE db_name;
        ```

        作用：切换当前会话的默认数据库，后续操作默认在这个库中执行

    - 删除数据库

        ```sql
        DROP DATABASE [IF EXISTS] db_name;
        ```

        - `IF EXISTS`：目标表不存在时，若不加，在删除时 SQL 会抛出错误，语句终止；若加上，只产生提示信息不终止
        - 删除数据库会**清空库内所有表和数据**，谨慎使用

    - 示例

        ```sql
        -- 创建
        CREATE DATABASE IF NOT EXISTS EAS_DB;
        -- 使用
        USE EAS_DB;
        -- 删除
        DROP DATABASE IF EXISTS EAS_DB;
        ```

- `3.3.2` SQL 模式定义

    > SQL 模式 `Schema` 是表、视图、域、索引等数据库对象的集合，可以理解为数据库下的一个命名空间

    - 创建模式

        ```sql
        CREATE SCHEMA <模式名> AUTHORIZATION <用户名>
        [ <CREATE DOMAIN子句> | <CREATE TABLE子句> | <CREATE VIEW子句> ... ]
        ```

        - `AUTHORIZATION`：指定模式的所有者（用户）
        - 可以在创建模式的同时直接定义表、视图等对象

    - `AUTHORIZATION`：指定模式的所有者（用户）
    - 可以在创建模式的同时直接定义表、视图等对象

    - 删除模式

        ```sql
        DROP SCHEMA <模式名> [CASCADE | RESTRICT];
        ```

        - `CASCADE` 级联：删除模式时，**模式里所有表、视图、约束全部一起删除**
        - `RESTRICT` 限制：如果模式里还有对象，**拒绝删除，只有模式为空才能删**

        > RESTRICT 是安全模式，防止误删数据；CASCADE 是强制清理模式

- `3.3.3` 数据表操作

    1. 创建表 `CREATE TABLE`

        ```sql
        CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name (
            <列名1> <数据类型> [列选项],
            <列名2> <数据类型> [列选项],
            ...
            [表选项]
        );
        ```

        - `TEMPORARY`：临时表，表示仅当前连接可见，断开数据库连接则自动删除，其他会话看不到这张表
        - `IF NOT EXISTS`：表不存在才创建，防止表已存在时报错

        - **列选项**（字段级约束）

            | 约束              | 含义                                                     |
            | ----------------- | -------------------------------------------------------- |
            | `NULL / NOT NULL` | 是否允许为空；`NOT NULL` 该字段不能存空值                |
            | `DEFAULT value`   | 字段默认值，插入数据不填该列时自动使用这个值             |
            | `AUTO_INCREMENT`  | 自增，只支持整数；一张表**最多一个自增列，且必须是主键** |
            | `UNIQUE KEY`      | 唯一键：该列的值不能重复，可以允许 NULL                  |
            | `PRIMARY KEY`     | 主键：唯一 + 非空；一张表只能一个主键                    |
            | `COMMENT`         | 字段注释，方便阅读表结构                                 |

        - **表选项**（表级约束）

            适合**多列联合主键、联合唯一、外键**，写在所有字段定义的后面， 例：联合主键 `PRIMARY KEY(col1,col2)`

        - 示例

            1. `Student_Info` 学生信息表

                ```sql
                CREATE TABLE Student_Info(
                    SNo varchar(10) NOT NULL PRIMARY KEY,
                    SName varchar(30) NOT NULL,
                    Sex int NULL DEFAULT 1,
                    Birthday datetime NULL,
                    Major int NULL,
                    Scores float NULL DEFAULT 0,
                    Remarks text NULL
                );
                ```

                - `SNo` 学号作为主键，`varchar` 字符串类型，**不要用** `int` **存学号（防止 0 开头丢失）**
                - `Sex` 性别用 `int` 表示，即 `1` 表示男生、`0` 表示女生

            2. `Course_Info` 课程信息表

                ```sql
                CREATE TABLE Course_Info(
                    CID int NOT NULL auto_increment primary key,
                    CName varchar(30) NOT NULL,
                    Term int NULL DEFAULT 1,
                    CScore float NOT NULL
                );
                ```

            3. `Score_Info` 成绩表（**关联表，外键**）

                ```sql
                CREATE TABLE Score_Info(
                    SNo varchar(10) NOT NULL,
                    CID int NOT NULL,
                    Score float NULL,
                    PRIMARY KEY(SNo,CID), -- 联合主键：同一个学生同一门课只能一条成绩
                    FOREIGN KEY (SNo) REFERENCES Student_Info(SNo),
                    FOREIGN KEY (CID) REFERENCES Course_Info(CID)
                );
                ```

                - 成绩表主键是**学号 + 课程号联合主键**，外键分别引用学生表、课程表主键，保证参照完整性

    2. 修改表结构 `ALTER TABLE`

        `ALTER TABLE` 是修改已经存在的表结构的语句，4 种常用操作：

        1. 增加列 `ADD`

            ```sql
            ALTER TABLE 表名 ADD 新列名 数据类型 [约束];
            ```

            示例：学生表增加兴趣爱好字段

            ```sql
            ALTER TABLE Student_Info ADD hobby text NULL;
            ```

        2. 删除列 `DROP`

            ```sql
            ALTER TABLE 表名 DROP 列名 [CASCADE | RESTRICT];
            ```

            - `CASCADE` 级联：删除模式时，**模式里所有表、视图、约束全部一起删除**

            - `RESTRICT` 限制：如果模式里还有对象，**拒绝删除，只有模式为空才能删**

            - 示例：删除 `Student_Info` 表中 `hobby` 列

                ```sql
                ALTER TABLE Student_Info DROP hobby;
                ```

        3. 修改属性列 `MODIFY`

            修改已有字段的数据类型、约束、默认值

            ```sql
            ALTER TABLE 表名 MODIFY 列名 新类型 [约束];
            ```

            - 示例：把 `hobby` 从 `text` 改成 `varchar (50)`

                ```sql
                ALTER TABLE Student_Info MODIFY hobby varchar(50);
                ```

        4. 补充主键、外键（表级约束追加）

            ```sql
            -- 追加主键
            ALTER TABLE 表名 ADD PRIMARY KEY(列列表);
            -- 追加外键
            ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY(本表列) REFERENCES 主表(主表列);
            ```

            - 添加主键的三种方法

                1. 创建表时直接写在字段后

                    ```sql
                    CREATE TABLE tb(
                    id INT IDENTITY(1,1) PRIMARY KEY,
                    name VARCHAR(20)
                    );
                    ```

                2. 创建表时，所有字段写完后使用 PRIMARY KEY

                    ```sql
                    CREATE TABLE table_test(
                    id INT NOT NULL,
                    name VARCHAR(20) NOT NULL,
                    address VARCHAR(20),
                    PRIMARY KEY(id)
                    );
                    ```

                3. 表格创建完成之后添加主键

                    ```sql
                    ALTER TABLE score ADD PRIMARY KEY(id);
                    ```

                > 复合主键写法：
                >
                > ```sql
                > CREATE TABLE table_test(
                > user_id INT NOT NULL,
                > user_name VARCHAR(20) NOT NULL,
                > user_address VARCHAR(20),
                > PRIMARY KEY (user_id, user_name)
                > );
                > ```

            - 添加外键的四种方法

                1. 直接在属性（字段）后面添加

                    ```sql
                    CREATE TABLE score
                    (
                    cscore int(11),
                    st_id int(50) REFERENCES student(id),
                    cs_id int(30) REFERENCES classes(id),
                    PRIMARY KEY(st_id,cs_id)
                    );
                    ```

                2. 在所有属性之后添加

                    ```sql
                    CREATE TABLE score
                    (
                    cscore int(11),
                    st_id int(50),
                    cs_id int(30),
                    PRIMARY KEY (st_id,cs_id),
                    FOREIGN KEY (st_id) REFERENCES student(id),
                    FOREIGN KEY (cs_id) REFERENCES classes(id)
                    );
                    ```

                3. 使用 CONSTRAINT 添加命名约束（建表时）

                    ```sql
                    CREATE TABLE score
                    (
                    cscore int(11),
                    st_id int(50),
                    cs_id int(30),
                    PRIMARY KEY(st_id,cs_id),
                    CONSTRAINT fk_id_cd FOREIGN KEY (st_id) REFERENCES student(id),
                    CONSTRAINT fk_id_cs FOREIGN KEY (cs_id) REFERENCES classes(id)
                    );
                    ```

                4. 在表的定义之外进行添加（ALTER TABLE）

                    ```sql
                    ALTER TABLE score ADD CONSTRAINT FK_ID FOREIGN KEY (st_id) REFERENCES student(id);
                    ```

    3. 删除表 `DROP TABLE`

        ```sql
        DROP [TEMPORARY] TABLE [IF EXISTS] tbl_name [,tbl_name...];
        ```

        - `TEMPORARY`：`DROP TEMPORARY TABLE`，只会删掉临时表，不会碰普通数据表
        - `IF EXISTS`：目标表不存在时，若不加，在删除时 SQL 会抛出错误，语句终止；若加上，只产生提示信息不终止

        - ` DROP TABLE` 删除整张表结构 + 里面所有数据；`DELETE` 是 DML，只删数据、保留表结构（后面 DML 内容）

## `3.4` SQL 数据操作语言

- `3.4.1` 插入表数据 `INSERT`

    ```sql
    INSERT [INTO] tbl_name [(col_name,...)] VALUES ({expr|DEFAULT},...),(...),...
    ```

    - `[(col_name,...)]` ：可选，指定要插入值的字段列表；省略时，值顺序必须和表定义字段顺序完全匹配

    - `{expr|DEFAULT} `：每一个位置可以写表达式，或者写 `DEFAULT`，使用字段的默认值

    - 如果列清单和 `VALUES` 清单都为空，则 `INSERT` 会创建一行，每列都设置成默认值

    - 示例

        ```sql
        -- 显式指定字段
        INSERT INTO Student_Info (SNo,姓名,性别,出生时间,专业,总学分,备注) 
        VALUES('20200101','王林',1,'1990-02-10',1,50,NULL);
        
        -- 省略字段列表，单行插入
        INSERT INTO Student_Info VALUES('20200101','王林',1,'1990-02-10',1,50,NULL);
        
        -- 批量多行插入
        INSERT INTO Student_Info (SNo,姓名) VALUES('20200101','王林'),('20200102','李华');
        ```

- `3.4.2` 修改表数据 `UPDATE`

    ```sql
    UPDATE tbl_name SET col_name1=expr1 [, col_name2=expr2 ...] [WHERE where_definition]
    ```

    - `SET` 子句：根据 `WHERE` 子句中指定的条件对符合条件的数据行进行修改，若语句中不设定 WHERE 子句，则更新所有行

    - `WHERE` 子句：通过设定条件确定要修改哪些行，where_definition 用于指定条件

    - 示例

        ```sql
        -- 将学生信息表中学号为 20220102 的学生的备注值改为 “三好生”
        UPDATE Student_Info SET Remark='三好生' WHERE SNo='20220102';
        
        -- 将学生信息表中的所有学生的总学分增加 10
        UPDATE Student_Info SET Scores = Scores + 10;
        
        -- 将姓名为 “陈东” 的同学的专业改为 “软件工程”（编号为 3），备注改为 “提前修完学分”
        UPDATE Student_Info SET Major = 3, 备注 = '提前修完学分' WHERE 姓名 = '陈东';
        ```

- `3.4.3` 删除表数据 `DELETE`

    ```sql
    DELETE FROM tbl_name [WHERE where_definition]
    ```

    - 示例

        ```sql
        delete FROM course_Info WHERE Term != 5;
        ```

        

    