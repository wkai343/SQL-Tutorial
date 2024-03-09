# SQL Tutorial

---

## 前置

- 关系模型

数据结构：关系
数据操作：查询和更新
完整性约束：实体、参照、用户定义

- 架构

  - PostgreSQL使用一种客户端/服务器的模型
  - 一个PostgreSQL数据库集簇中包含一个或更多命名的数据库。角色和一些其他对象类型被整个集簇共享，连接到服务器的客户端只能访问单个数据库中的数据，在连接请求中指定的那一个

- 数据库基本操作

  - 创建数据库集簇与服务启动
  - 连接数据库
  - pgsql内置createdb和dropdb, createdb可以有默认值
  - `psql -U <用户名> -d <数据库> -h <ip> -p <port>`; `psql <数据库>`, 若不提供数据库名, 默认值为用户账号名
  - 其他

---

## sql语法

### 词法结构

#### 注释

- 单行注释

```sql
-- 注释
```

- 多行注释

```sql
/* */
```

#### 标识符与关键字

- 字母、数字和下划线，不能以数字开头
- 关键词和不被双引号修饰的标识符是大小写不敏感的
- 被双引号修饰的标识符不会被解析为关键字
- pgsql将不带引号的名称折叠为小写，SQL标准规定不带引号的名称应折叠为大写

#### 常量

- 三种隐式类型常量：字符串、位串和数字

##### 字符串常量

- 一个字符串常量是一个由单引号（'）包围的任意字符序列
- 两个只由空白及至少一个新行分隔的字符串常量会被连接在一起，并且将作为一个写在一起的字符串常量来对待
- 一个转义字符串常量可以通过在开单引号前面写一个字母E（大写或小写形式）来指定
- PostgreSQL提供了另一种被称为“美元引用”的方式来书写字符串常量

##### 位串常量

- 二进制使用一个前导B（大写或小写形式）
- 十六进制使用一个前导X（大写或小写形式）

##### 数字常量

```sql
digits
digits.[digits][e[+-]digits]
[digits].digits[e[+-]digits]
digitse[+-]digits
```

- 如果一个不包含小数点和指数的数字常量的值适合类型integer（32 位），它首先被假定为类型integer。否则如果它的值适合类型bigint（64 位），它被假定为类型bigint。再否则它会被取做类型numeric。包含小数点和/或指数的常量总是首先被假定为类型numeric。

##### 其他类型的常量

```sql
type 'string'
'string'::type
CAST ('string' AS type)
typename ('string')
```

- type 'string'语法上的另一个限制是它无法对数组类型工作，指定一个数组常量的类型可使用::或CAST()
- 带有::的语法是PostgreSQL的历史用法，就像函数调用语法一样

#### 基础操作符

- 操作符可以放在OPERATOR()中

```sql
SELECT 3 OPERATOR(pg_catalog.+) 4;
```

#### 算术操作符

```sql
+ - * / -- 四则
% ^ |/ ||/ @ -- 取余、指数、平方根、立方根、绝对值
~ & | # << >> -- 按位求反(NOT)、按位与(AND)、按位或(OR)、按位异或(exclusive OR)、按位左移、按位右移
```

#### 逻辑操作符

```sql
<boolean> AND <boolean> → <boolean>
<boolean> OR <boolean> → <boolean>
NOT <boolean> → <boolean>
```

- SQL使用三值的逻辑系统，包括真、假和null，null表示“未知”
- 操作符AND和OR是可交换的

| a | b | a AND b | a OR b |
|---|---|---|---|
| TRUE | TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE | TRUE |
| TRUE | NULL | NULL | TRUE|
| FALSE | FALSE | FALSE | FALSE |
| FALSE | NULL | FALSE | NULL |
| NULL | NULL | NULL | NULL |

| a | NOT a |
|---|---|
| TRUE | FALSE |
| FALSE | TRUE |
| NULL | NULL |

##### 比较操作符

| 操作符 | 描述 |
|---|---|
| datatype < datatype → boolean |小于 |
| datatype > datatype → boolean |大于 |
| datatype <= datatype → boolean| 小于等于 |
| datatype >= datatype → boolean| 大于等于 |
| datatype = datatype → boolean |等于 |
| datatype <> datatype → boolean| 不等于 |
| datatype != datatype → boolean| 不等于 |

> != 是一个别名
> 通常也可以比较相关数据类型的值；例如integer > bigint 将起作用。 这种排序的某些情况直接由“cross-type” 比较操作符实现，但是，如果没有这种操作符，解析器将把不太通用的类型强制为更通用的类型，并应用后者的比较操作符。

###### 比较谓词

| 描述 | 谓词 | 示例(s) |
|---|---|---|

- 之间, 包括范围端点

```sql
<datatype> BETWEEN <datatype> AND <datatype> → <boolean>
2 BETWEEN 1 AND 3 → t
2 BETWEEN 3 AND 1 → f
```

- 不在之间, BETWEEN的否定

```sql
<datatype> NOT BETWEEN <datatype> AND <datatype> → <boolean>
2 NOT BETWEEN 1 AND 3 → f
```

- 之间, 在对两个端点值排序之后

```sql
<datatype> BETWEEN SYMMETRIC <datatype> AND <datatype> → <boolean>
2 BETWEEN SYMMETRIC 3 AND 1 → t
```

- 不在之间, 在对两个端点值排序之后

```sql
<datatype> NOT BETWEEN SYMMETRIC <datatype> AND <datatype> → <boolean>
2 NOT BETWEEN SYMMETRIC 3 AND 1 → f
```

- 不相等, 将空(null)视为可比值

```sql
<datatype> IS DISTINCT FROM <datatype> → <boolean>
1 IS DISTINCT FROM NULL → t (而不是 NULL)
NULL IS DISTINCT FROM NULL → f (而不是 NULL)
```

- 相等, 将空(null)视为可比值

```sql
<datatype> IS NOT DISTINCT FROM <datatype> → <boolean>
1 IS NOT DISTINCT FROM NULL → f (而不是 NULL)
NULL IS NOT DISTINCT FROM NULL → t (而不是 NULL)
```

- 测试值是否为空

```sql
<datatype> IS NULL → <boolean>
1.5 IS NULL → f
```

测试值是否不为空

```sql
<datatype> IS NOT NULL → <boolean>
'null' IS NOT NULL → t
```

- 测试布尔表达式是否为真

```sql
<boolean> IS TRUE → <boolean>
true IS TRUE → t
NULL::<boolean> IS TRUE → f (而不是 NULL)
```

- 测试布尔表达式是否为假或未知

```sql
<boolean> IS NOT TRUE → <boolean>
true IS NOT TRUE → f
NULL::<boolean> IS NOT TRUE → t (而不是 NULL)
```

- 测试布尔表达式是否为假

```sql
<boolean> IS FALSE → <boolean>
true IS FALSE → f
NULL::<boolean> IS FALSE → f (而不是 NULL)
```

- 测试布尔表达式是否为真或未知

```sql
<boolean> IS NOT FALSE → <boolean>
true IS NOT FALSE → t
NULL::<boolean> IS NOT FALSE → t (而不是 NULL)
```

- 测试布尔表达式是否为未知

```sql
<boolean> IS UNKNOWN → <boolean>
true IS UNKNOWN → f
NULL::<boolean> IS UNKNOWN → t (而不是 NULL)
```

- 测试布尔表达式是否为真或假

```sql
<boolean> IS NOT UNKNOWN → <boolean>
true IS NOT UNKNOWN → t
NULL::<boolean> IS NOT UNKNOWN → f (而不是 NULL)
```

##### 优先级(由高到低)与结合性

| 操作符/元素 | 结合性 | 描述 |
|---|---|---|
| . | 左 | 表/列名分隔符 |
| :: | 左 | PostgreSQL-风格的类型转换 |
| [ ] | 左 | 数组元素选择 |
| + - | 右 | 一元加、一元减 |
| ^ | 左 | 指数 |
| * / % | 左 | 乘、除、模 |
| + - | 左 | 加、减 |
| （任意其他操作符）| 左 | 所有其他本地以及用户定义的操作符 |
| BETWEEN IN LIKE ILIKE SIMILAR|| 范围包含、集合成员关系、字符串匹配 |
| \< \> = \<= \>= \<\> || 比较操作符 |
| IS ISNULL NOTNULL || IS TRUE、IS FALSE、IS NULL、IS DISTINCT FROM等 |
| NOT | 右 | 逻辑否定 |
| AND | 左 | 逻辑合取 |
| OR | 左 | 逻辑析取 |

#### 特殊字符

- 跟随在一个美元符号（$）后面的数字被用来表示在一个函数定义或一个预备语句中的位置参数。在其他上下文中该美元符号可以作为一个标识符或者一个美元引用字符串常量的一部分
- 圆括号（()）具有它们通常的含义，用来分组表达式并且强制优先。在某些情况中，圆括号被要求作为一个特定 SQL 命令的固定语法的一部分
- 方括号（[]）被用来选择一个数组中的元素
- 逗号（,）被用在某些语法结构中来分割一个列表的元素
- 分号（;）结束一个 SQL 命令。它不能出现在一个命令中间的任何位置，除了在一个字符串常量中或者一个被引用的标识符中
- 冒号（:）被用来从数组中选择“切片”。在某些 SQL 的“方言”（例如嵌入式 SQL）中，冒号被用来作为变量名的前缀
- 星号（*）被用在某些上下文中标记一个表的所有域或者组合值。当它被用作一个聚集函数的参数时，它还有一种特殊的含义，即该聚集不要求任何显式参数
- 句点（.）被用在数字常量中，并且被用来分割模式、表和列名

### 值表达式

- 一个常量或文字值
- 一个列引用
- 在一个函数定义体或预备语句中的一个位置参数引用
- 一个下标表达式
- 一个域选择表达式
- 一个操作符调用
- 一个函数调用
- 一个聚集表达式
- 一个窗口函数调用
- 一个类型转换
- 一个排序规则表达式
- 一个标量子查询
- 一个数组构造器
- 一个行构造器
- 另一个在圆括号（用来分组子表达式以及重载优先级）中的值表达式

> 子表达式的计算顺序没有被定义

### 调用函数

- 位置参数、关键字参数、混合

--

## 基本数据类型

- 用户可以使用CREATE TYPE命令为 PostgreSQL增加新的数据类型

### 布尔类型

布尔常量可以表示为SQL关键字TRUE, FALSE,和 NULL.

> 注意语法分析程序会把TRUE 和 FALSE 自动理解为boolean类型，但是不包括NULL，因为它可以是任何类型的。因此在某些语境中你也许要将 NULL 转化为显示boolean类型，例如NULL::boolean.

### 数字类型

|类型|字节|
|---|---|
|smallint|2|
|integer, int|4|
|bigint|8|
|numeric|可变|
|real|4|
|double precision|8|

#### 任意精度

```sql
NUMERIC(precision, scale) -- 精度必须为正数，小数位数可以为零或者正数

NUMERIC(precision) -- 选择小数位数为 0 

NUMERIC -- 创建一个“无约束的数值”列，其中可以存储任意长度的数值，直到被实现所限制
```

支持：

```sql
Infinity(inf)
-Infinity(-inf)
NaN
```

> 在SQL命令中将这些值作为常量写入时，必须在其周围加引号
> 无穷大只能存储在无约束的numeric中，因为它名义上超过了任何有限精度限制
> 类型decimal和numeric是等效的。两种类型都是SQL标准的一部分

#### 浮点数

> PostgreSQL还支持 SQL 标准表示法float和float(p)用于声明非精确的数字类型。在这里，p指定以二进制位表示的最低可接受精度。在选取real类型的时候，PostgreSQL接受float(1)到float(24)，在选取double precision的时候，接受float(25)到float(53)。在允许范围之外的p值将导致一个错误。没有指定精度的float将被当作是double precision
> 在对值进行圆整时，numeric类型会圆到远离零的整数，而（在大部分机器上）real和double precision类型会圆到最近的偶数上

支持：

```sql
Infinity(inf)
-Infinity(-inf)
NaN
```

### 字符类型

|类型|描述|
|---|---|
|character varying(n), varchar(n)|有限制的变长|
|character(n), char(n)|定长，空格填充|
|text|无限变长|

> 没有长度声明词的character等效于character(1)
> 如果不带长度说明词使用character varying，那么该类型接受任何长度的串。这是一个PostgreSQL的扩展
> PostgreSQL提供text类型，它可以存储任何长度的串。尽管类型text不是SQL标准，但是许多其它 SQL 数据库系统也有它

### 字节类型

| 名字 | 存储尺寸 | 描述 |
|---|---|---|
| bytea | 1或4字节外加真正的二进制串 | 变长二进制串 |

- 十六进制格式

整个串以序列\x开头

### 位串类型

|类型|描述|
|---|---|
|bit varying(n), varbit(n)|有限制的变长|
|bit(n)|定长, 精准匹配长度n|

> 写一个没有长度的bit等效于bit(1)，没有长度的bit varying意味着没有长度限制。
> 一个位串值对于每8位的组需要一个字节，外加总共5个或8个字节，这取决于串的长度

### 枚举类型

- 枚举类型可以使用CREATE TYPE命令创建

```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
```

> 枚举标签是大小写敏感的
> 一个枚举值在磁盘上占据4个字节

#### 排序

> 一个枚举类型的值的排序是该类型被创建时所列出的值的顺序。枚举类型的所有标准的比较操作符以及相关聚集函数都被支持

### 日期/时间类型

|名字 | 存储尺寸 | 描述 | 最小值 | 最大值 | 解析度|
|---|---|---|---|---|---|
|timestamp | 8字节 | 时间戳(无时区) | 4713 BC | 294276 AD | 1微秒|
|timestamp with time zone | 8字节 | 时间戳(有时区) | 4713 BC | 294276 AD | 1微秒|
|date | 4字节 | 日期 | 4713 BC | 5874897 AD | 1日 |
|time | 8字节 | 一天中的时间(无时区) | 00:00:00 | 24:00:00 | 1微秒|
|time with time zone | 12字节 | 一天中的时间(有时区) |00:00:00+1459 | 24:00:00-1459 | 1微秒|
|interval | 16字节 | 时间间隔 | -178000000年 | 178000000年 | 1微秒|

> timestamptz是timestamp with time zone的一种简写，这是一种PostgreSQL的扩展

#### 特殊值

|输入串 | 合法类型 | 描述|
|---|---|---|
|epoch |date, timestamp | 1970-01-01 00:00:00+00（Unix系统时间0）|
|infinity | date, timestamp | 比任何其他时间戳都晚|
|-infinity | date, timestamp | 比任何其他时间戳都早|
|now | date, time, timestamp | 当前事务的开始时间|
|today | date, timestamp | 今日午夜 (00:00)|
|tomorrow | date, timestamp | 明日午夜 (00:00)|
|yesterday | date, timestamp | 昨日午夜 (00:00)|
|allballs | time | 00:00:00.00 UTC|

### 数组

- PostgreSQL允许一个表中的列定义为变长多维数组。可以创建任何内建或用户定义的基类、枚举类型、组合类型或者域的数组

#### 定义

```sql
<type>[];

<type> ARRAY;
```

> 不会强制维度和尺寸

#### 切片

---

## 数据定义

操作对象:

- [模式](#模式)
- [表](#表)
- [视图](#视图)
- [索引](#索引)

### 模式

一个数据库包含一个或多个命名模式，但不能嵌套，模式中包含着表。模式还包含其他类型的命名对象，包括数据类型、函数和操作符。

使用模式的原因：

- 允许多个用户使用一个数据库并且不会互相干扰
- 将数据库对象组织成逻辑组以便更容易管理
- 第三方应用的对象可以放在独立的模式中，这样它们就不会与其他对象的名称发生冲突

#### 创建/删除模式

```sql
CREATE SCHEMA <模式名>;

CREATE SCHEMA [模式名] AUTHORIZATION <用户名>; -- 模式名默认是用户名

DROP SCHEMA [模式名] [, 模式名]... [CASCADE | RESTRICT];
```

#### 使用对象

`[数据库.][模式名.][对象名]`

对于操作符，`SELECT 3 OPERATOR(pg_catalog.+) 4;`

#### 公共模式与模式搜索路径

- 对象前不指定模式, 默认为public, sql标准没有public模式的概念

- 查看模式搜索路径, 不同环境可能有差异

```sql
SHOW search_path;
```

- 修改搜索路径

```sql
SET search_path TO myschema, public;
```

#### 模式权限

- 在默认情况下，所有人都拥有在public模式上的CREATE和USAGE权限
- 默认情况下，用户不能访问不属于他们的模式中的任何对象。要允许这种行为，模式的拥有者必须在该模式上授予USAGE权限
- 一个用户也可以被允许在其他某人的模式中创建对象。要允许这种行为，模式上的CREATE权限必须被授予

### 表

#### 创建/删除表

```sql
CREATE TABLE <表名> (
    <列名> <数据类型> [DEFAULT <默认值>] [列级约束]
    [, <列名> <数据类型> [DEFAULT <默认值>] [列级约束]]...
    [,表级约束]
);

DROP TABLE <表名> [, <表名>] [RESTRICT | CASCADE];
```

#### 生成列

- 生成列有两种:存储列(STORED)和虚拟列(VIRTUAL)(虚拟列未实现)
- 建立一个生成列，在 CREATE TABLE中使用 GENERATED ALWAYS AS 子句
- 生成列则在行每次改变时进行更新，并且不能被取代

```sql
CREATE TABLE people (
...,
height_cm numeric,
height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```

> 生成列不能被直接写入. 在INSERT 或 UPDATE 命令中, 不能为生成列指定值, 但是可以指定关键字DEFAULT

生成列和涉及生成列的表的定义有几个限制:

- 生成表达式只能使用不可变函数，并且不能使用子查询或以任何方式引用当前行以外的任何内容
- 生成表达式不能引用另一个生成列
- 生成表达式不能引用系统表，除了 tableoid
- 生成列不能具有列默认或标识定义
- 生成列不能是分区键的一部分
- 对于继承:
  - 如果父列是生成的列，则子列也必须也是使用相同的表达式生成的列。 在子列的定义中，不再使用GENERATED子句，因为它将从父列复制过来
  - 在进行多重继承的情况下，如果一个父列是生成的列，那么所有父列都必须是生成的列，并且具有相同的表达式
  - 如果父列不是生成的列，子列可以定义是否为生成的列

使用生成列的其他注意事项：

- 生成列保留着有别于其下层的基础列的访问权限。因此，可以对其进行排列以便于从生成列中读取特定的角色，而不是从下层基础列
- 从概念上讲，生成列在BEFORE 触发器运行后更新。 因此，BEFORE 触发器中的基础列所做的变更将反映在生成列中。 但相反，不允许访问BEFORE 触发器中的生成列

#### 默认值

- 默认值可以是一个表达式，它将在任何需要插入默认值的时候被实时计算（不是表创建时）

#### 约束

- 分为列约束和表约束
- 可以有多个约束，顺序无关
- 可以给约束以约束名，CONSTRAINT <约束名> (非空似乎不行)

- 检查约束
- 非空约束
- 唯一约束
- 主键约束
- 外键约束
- 排他约束

##### 检查约束

- 列级或表级

```sql
CREATE TABLE <表名> (
... CHECK (<布尔表达式>)
);
```

> 需要注意的是，一个检查约束在其检查表达式值为真或空值时被满足

##### 非空约束

列级

```sql
CREATE TABLE <表名> (
... NOT NULL
);
```

##### 唯一约束

- 列级与表级
- 两个空值被认为是不同的

列级

```sql
CREATE TABLE <表名> (
... UNIQUE
);
```

表级

```sql
CREATE TABLE example (
... UNIQUE (<列名>[, 列名]...)
);
```

> 增加一个唯一约束会在约束中列出的列或列组上自动创建一个唯一B-tree索引。只覆盖某些行的唯一性限制不能被写为一个唯一约束，但可以通过创建一个唯一的部分索引来强制这种限制

##### 主键约束

- 列级与表级
- 唯一且非空
- 一个表最多只能有一个主键

列级

```sql
CREATE TABLE <表名> (
... PRIMARY KEY
);
```

表级

```sql
CREATE TABLE example (
... PRIMARY KEY (<列名>[, 列名]...)
);
```

> 增加一个主键将自动在主键中列出的列或列组上创建一个唯一B-tree索引。并且会强制这些列被标记为NOT NULL

##### 外键约束

- 列级与表级

列级

```sql
CREATE TABLE orders (
... REFERENCES <被引用表> [(<被引用表主键列表>)]
);
```

表级

```sql
CREATE TABLE t1 (
... FOREIGN KEY (<引用表外键列表>) REFERENCES <被引用表> [(<被引用表主键列表>)]
);
```

###### 删除/更新动作

- ON DELETE / ON UPDATE

> 限制删除或者级联删除是两种最常见的选项。RESTRICT阻止删除一个被引用的行。NO ACTION表示在约束被检查时如果有任何引用行存在，则会抛出一个错误，这是我们没有指定任何东西时的默认行为（这两种选择的本质不同在于NO ACTION允许检查被推迟到事务的最后，而RESTRICT则不会）。CASCADE指定当一个被引用行被删除后，引用它的行也应该被自动删除。还有其他两种选项：SET NULL和SET DEFAULT。这些将导致在被引用行被删除后，引用行中的引用列被置为空值或它们的默认值。注意这些并不会是我们免于遵守任何约束。例如，如果一个动作指定了SET DEFAULT，但是默认值不满足外键约束，操作将会失败。与ON DELETE相似，同样有ON UPDATE可以用在一个被引用列被修改（更新）的情况，可选的动作相同。在这种情况下，CASCADE意味着被引用列的更新值应该被复制到引用行中。

##### 排他约束

- 排他约束保证如果将任何两行的指定列或表达式使用指定操作符进行比较，至少其中一个操作符比较将会返回否或空值

> 增加一个排他约束将在约束声明所指定的类型上自动创建索引

#### 修改语句

- 增加列
- 移除列
- 增加约束
- 移除约束
- 修改默认值
- 修改列数据类型
- 重命名列
- 重命名表

##### 增加列

```sql
ALTER TABLE <表名> ADD [COLUMN] <列名> <数据类型> [默认值] [约束];
```

##### 移除列

```sql
ALTER TABLE <表名> DROP [COLUMN] <列名> [RESTRICT | CASCADE];
```

##### 增加约束

```sql
ALTER TABLE <表名> ADD <表级约束>;
```

要增加一个不能写成表约束的非空约束，可使用语法：

```sql
ALTER TABLE <表名> ALTER COLUMN <列名> SET NOT NULL;
```

该约束会立即被检查，所以表中的数据必须在约束被增加之前就已经符合约束。

##### 移除约束

```sql
ALTER TABLE <表名> DROP CONSTRAINT <约束名> [RESTRICT | CASCADE];
```

##### 修改列的默认值

```sql
ALTER TABLE <表名> ALTER COLUMN <列名> SET DEFAULT <默认值>;
ALTER TABLE <表名> ALTER COLUMN <列名> DROP DEFAULT;
```

##### 修改列的数据类型

```sql
ALTER TABLE <表名> ALTER [COLUMN] <列名> TYPE <类型>;
```

##### 重命名列

```sql
ALTER TABLE <表名> RENAME [COLUMN] <旧列名> TO <新列名>;
```

##### 重命名表

```sql
ALTER TABLE <旧表名> RENAME TO <新表名>;
```

#### 继承

- 一个表可以从0个或者多个其他表继承

```sql
CREATE TABLE <基表> ();

CREATE TABLE <派生表> () INHERITS (<基表>);
```

- 查询基表默认包括派生表

```sql
SELECT <查询内容> FROM <基表>
```

- 只包括基表

```sql
SELECT <查询内容> FROM ONLY <基表>
```

#### 分区

好处：

- 减少扫描量，提高查询性能
- 便于数据迁移
- pgsql内建支持：范围分区、列表分区、哈希分区

##### 分区表和分区

- 分区有存储空间，是与分区表关联的普通表
- 不可能将常规表转换为分区表，反之亦然。但是，可以将现有的常规或分区表添加为分区表的分区(ATTACH PARTITION)，或从分区表中删除分区，将其转换为独立表(DETACH PARTITION)

##### 声明式分区

1. `PARTITION BY`声明分区表
    CREATE TABLE <分区表> () PARTITION BY RANGE (<分区键>);

2. 创建分区

- 范围分区

```sql
CREATE TABLE <分区> PARTITION OF <分区表>
FOR VALUES FROM <左开> TO <右闭>
[PARTITION BY RANGE (<分区键>)]; -- 子分区
```

##### 分区维护

- 增加分区

```sql
CREATE TABLE <分区> PARTITION OF <分区表>
FOR VALUES FROM <左开> TO <右闭>
TABLESPACE fasttablespace;
```

> 作为一种选择，有时在分区结构之外创建新表更方便，并在以后使其成为适当的分区。

- 移除分区(需要在父表上拿到ACCESS EXCLUSIVE锁)

```sql
DROP TABLE <分区>;
```

- 把分区从分区表中移除，但是保留它作为一个表的访问权(第一种形式需要父表上的ACCESS EXCLUSIVE锁; 在第二种形式中同时添加CONCURRENTLY 限定符，允许detach操作只需要父表上的SHARE UPDATE EXCLUSIVE锁)

```sql
ALTER TABLE <分区表> DETACH PARTITION <分区>;
ALTER TABLE <分区表> DETACH PARTITION <分区> CONCURRENTLY;
```

##### 使用继承的分区

1. 创建“根”表，所有的“子”表都将从它继承。这个表将不包含数据。不要在这个表上定义任何检查约束，除非想让它们应用到所有的子表上。同样，在这个表上定义索引或者唯一约束也没有意义。对于我们的例子来说，根表是最初定义的measurement表
2. 创建数个“子”表，每一个都从根表继承。通常，这些表将不会在从根表继承的列集合之外增加任何列。正如声明性分区那样，这些表就是普通的PostgreSQL表（或者外部表）

    ```sql
    CREATE TABLE measurement_y2006m02 () INHERITS (measurement);
    CREATE TABLE measurement_y2006m03 () INHERITS (measurement);
    ...
    CREATE TABLE measurement_y2007m11 () INHERITS (measurement);
    CREATE TABLE measurement_y2007m12 () INHERITS (measurement);
    CREATE TABLE measurement_y2008m01 () INHERITS (measurement);
    ```

3. 为子表增加不重叠的表约束来定义每个分区允许的键值
4. 对于每个子表，在键列上创建一个索引，以及任何想要的其他索引

    ```sql
    CREATE INDEX measurement_y2006m02_logdate ON measurement_y2006m02 (logdate);
    CREATE INDEX measurement_y2006m03_logdate ON measurement_y2006m03 (logdate);
    CREATE INDEX measurement_y2007m11_logdate ON measurement_y2007m11 (logdate);
    CREATE INDEX measurement_y2007m12_logdate ON measurement_y2007m12 (logdate);
    CREATE INDEX measurement_y2008m01_logdate ON measurement_y2008m01 (logdate);
    ```

5. 我们希望我们的应用能够使用INSERT INTO measurement ...并且数据将被重定向到合适的分区表。我们可以通过为根表附加一个合适的触发器函数来实现这一点。如果数据将只被增加到最后一个分区，我们可以使用一个非常简单的触发器函数
6. 确认constraint_exclusion配置参数在postgresql.conf中没有被禁用，否则将会不必要地访问子表

```sql
ALTER TABLE <派生表> NO INHERIT <基表>; -- 取消继承
```

#### 表权限

- 重新分配所有者

```sql
ALTER TABLE <表名> OWNER TO <所有者>;
```

- 分配权限

```sql
GRANT <权限> ON accounts TO <角色>;
```

- 撤销权限

```sql
REVOKE <权限> ON accounts FROM <角色>;
```

有效的权限：SELECT、INSERT、UPDATE、DELETE、TRUNCATE、REFERENCES、TRIGGER、CREATE、CONNECT、TEMPORARY、EXECUTE、USAGE
所有权限：ALL
所有角色：PUBLIC

#### 行安全性策略

- 所有对该表选择行或者修改行的普通访问都必须被一条行安全性策略所允许（不过，表的拥有者通常不服从行安全性策略）
- 如果表上不存在策略，将使用一条默认的否定策略，即所有的行都不可见或者不能被修改
- 应用在整个表上的操作不服从行安全性，例如TRUNCATE和 REFERENCES

```sql
ALTER TABLE <表名> ENABLE ROW LEVEL SECURITY
```

### 视图

#### 创建/删除视图

```sql
CREATE VIEW <视图名> [(列名列表)] AS -- 列名列表可对查询结果的列部分或全部重命名
<查询结果>
[WITH CHECK OPTION]; -- 在对视图更新时会检查条件

DROP VIEW <视图名> [, <视图名>] [CASCADE | RESTRICT];
```

#### 修改视图

与修改表操作基本相同

### 索引

#### 创建/删除索引

```sql
CREATE INDEX <索引名> ON <表名> (<列名>);

DROP INDEX <索引名>;
```

#### 索引类型

- PostgreSQL提供了多种索引类型：B-tree、Hash、GiST、SP-GiST、GIN 和 BRIN
- 用关键字USING指定

```sql
CREATE INDEX <name> ON <table> USING HASH (<column>);
```

#### 多列索引

- 目前，只有 B-tree、GiST、GIN 和 BRIN 索引类型支持多列索引

#### 唯一索引

- 索引也可以被用来强制列值的唯一性，或者是多个列组合值的唯一性

```sql
CREATE UNIQUE INDEX <name> ON <table> (<column> [, ...]);
```

> 当前，只有B-tree能够被声明为唯一
>
> PostgreSQL会自动为定义了一个唯一约束或主键的表创建一个唯一索引。该索引包含组成主键或唯一约束的所有列（可能是一个多列索引），它也是用于强制这些约束的机制

#### 表达式索引

- 一个索引列并不一定是底层表的一个列，也可以是从表的一列或多列计算而来的一个函数或者标量表达式

---

## 数据操纵

### 插入数据

- 按序插入

```sql
INSERT INTO <表名> VALUES (<值>) [, VALUES (<值>)]...;
```

- 指定插入(可省略部分列)

```sql
INSERT INTO <表名> (<列名>) VALUES (<值>) [, VALUES (<值>)]...;
```

> values列表可替换为查询结果

- 从使用给出的值从左开始填充列，有多少个给出的列值就填充多少个列，其他列的将使用默认值(可以显式地用默认值(DEFAULT)，用于单个的列或者用于整个行)(扩展语法)

```sql
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', DEFAULT);
INSERT INTO products DEFAULT VALUES;
```

### 更新(修改)数据

```sql
UPDATE <表名> SET <列 = 新值>[, 列 = 新值]... [WHERE <条件>]; -- 不含where即为所有行
```

### 删除数据

```sql
DELETE FROM <表名> [WHERE <条件>]; -- 不含where即为所有行 
```

### 从修改的行中返回数据

- INSERT、 UPDATE和DELETE命令都有一个支持这个的可选的 RETURNING子句
- 所允许的RETURNING子句的内容与SELECT命令的输出列表相同。它可以包含命令的目标表的列名，或者包含使用这些列的值表达式

---

## 数据查询

- WHERE在分组和聚集计算之前选取输入行（因此，它控制哪些行进入聚集计算），而HAVING在分组和聚集之后选取分组行

```sql
[WITH with_queries] SELECT <select_list> FROM <table_expression> [sort_specification]
```

[条件表达式](#条件表达式)

CASE WHEN语句

[聚焦函数](#聚焦函数)

count（计数）、sum（和）、avg（均值）、max（最大值）和min（最小值）

[子查询表达式](#子查询表达式)

IN

[模式匹配](#模式匹配)

LIKE

### 别名

#### 选择列表别名

```sql
SELECT <column> [AS] <alias> ...
```

#### 表表达式别名

- 不能够用该表最初的名字引用表
- 表别名主要用于简化符号，当把一个表连接到它自身时必须使用别名

```sql
FROM <table_reference> [AS] <alias>

FROM <table_reference> [AS] <alias> (<column1> [, column2 [, ...]]) - 给列起别名, 如果指定的列别名比表里实际的列少，那么剩下的列就没有被重命名
```

### 选择列表

- 最简单的选择列表类型是*，它发出表表达式生成的所有列。否则，一个选择列表是一个逗号分隔的值表达式的列表

#### DISTINCT

- 空值在这种比较中被认为是相同的

```sql
SELECT [DISTINCT | ALL] <select_list> ... -- 默认是ALL
```

- 可以用任意表达式来判断什么行可以被认为是可区分的(扩展语法)

```sql
SELECT DISTINCT ON (expression [, expression ...]) <select_list> ...
```

> DISTINCT ON子句不是 SQL 标准的一部分， 有时候有人认为它是一个糟糕的风格，因为它的结果是不可判定的。如果有选择的使用GROUP BY和在FROM中的子查询，那么我们可以避免使用这个构造，但是通常它是更方便的候选方法

### 表表达式

#### from子句

- 表引用可以是一个表名或生成表(可以是VALUES列表)
- FROM列表的结果是一个中间的虚拟表，该表可以进行由WHERE、GROUP BY和HAVING子句指定的转换，并最后生成全局的表表达式结果

```sql
FROM <table_reference> [, table_reference [, ...]]
```

#### 连接表

- 所有类型的连接都可以被链在一起或者嵌套：T1和T2都可以是连接表
- 在JOIN子句周围可以使用圆括号来控制连接顺序。如果不使用圆括号，JOIN子句会从左至右嵌套

```sql
T1 <join_type> T2 [join_condition]
```

##### 交叉连接(笛卡儿积)

```sql
T1 CROSS JOIN T2 -- 等效于 T1 INNER JOIN T2 ON TRUE 或者 于FROM T1,T2
```

##### 条件连接

- ON子句接收一个和WHERE子句里用的一样的布尔值表达式
- USING是个缩写符号，它允许你利用特殊的情况：连接的两端都具有相同的连接列名。JOIN USING的输出会废除冗余列, JOIN USING会先列出相同的连接列名，然后先跟上来自T1的剩余列，最后跟上来自T2的剩余列
- ，NATURAL是USING的缩写形式：它形成一个USING列表， 该列表由那些在两个表里都出现了的列名组成。和USING一样，这些列只在输出表里出现一次。如果不存在公共列，NATURAL JOIN的行为将和JOIN ... ON TRUE一样产生交叉集连接

```sql
T1 {[INNER] | {LEFT | RIGHT | FULL} [OUTER]} JOIN T2 ON <boolean_expression>
T1 {[INNER] | {LEFT | RIGHT | FULL} [OUTER]} JOIN T2 USING (<join column list>)
T1 NATURAL {[INNER] | {LEFT | RIGHT | FULL} [OUTER]} JOIN T2
```

##### WHERE子句

- 内连接的连接条件既可以写在WHERE子句也可以写在JOIN子句里
- 对于外部连接而言没有选择：它们必须在FROM子句中完成

```sql
WHERE <search_condition> -- 任意返回一个boolean类型值的值表达式
```

##### GROUP BY和HAVING子句

- 在通过了WHERE过滤器之后，生成的输入表可以使用GROUP BY子句进行分组，然后用HAVING子句删除一些分组行
- 没有在GROUP BY中列出的列都不能被引用，除非在聚集表达式中被引用

```sql
SELECT <select_list>
FROM ...
[WHERE ...]
GROUP BY <grouping_column_reference> [, grouping_column_reference]...
```

> 在严格的 SQL 里，GROUP BY只能对源表的列进行分组，但PostgreSQL把这个扩展为也允许GROUP BY去根据选择列表中的列分组。也允许对值表达式进行分组，而不仅是简单的列名

- 可以用HAVING子句从结果中删除一些组
- HAVING子句中的表达式可以引用分组的表达式和未分组的表达式（后者必须涉及一个聚集函数）

```sql
SELECT <select_list>
FROM ...
[WHERE ...]
GROUP BY ...
HAVING <boolean_expression>
```

### 行排序(ORDER BY)

- sort_expression也可以是列别名或者一个输出列的编号
- 每一个表达式后可以选择性地放置一个ASC或DESC关键词来设置排序方向为升序或降序。ASC顺序是默认值。升序会把较小的值放在前面，而“较小”则由<操作符定义。相似地，降序则由>操作符定义
- NULLS FIRST和NULLS LAST选项将可以被用来决定在排序顺序中。默认情况下，排序时空值被认为比任何非空值都要大

```sql
SELECT <select_list>
FROM <table_expression>
ORDER BY <sort_expression1> [ASC | DESC] [NULLS {FIRST | LAST}]
[, sort_expression2 [ASC | DESC] [NULLS {FIRST | LAST}] ...]
```

### 组合查询(UNION, INTERSECT, EXCEPT)

- 两个查询的结果可以用集合操作并、交、差进行组合
- 如果没有括号，则 UNION 和 EXCEPT 从左到右进行关联，但 INTERSECT 的绑定比这两个运算符更紧密
- 除非声明了ALL，否则所有重复行都被消除

```sql
<query1> UNION [ALL] <query2> -- 并
<query1> INTERSECT [ALL] <query2> -- 交
<query1> EXCEPT [ALL] <query2> -- 差
```

### 截断与偏移(LIMIT和OFFSET)

- 。LIMIT ALL的效果和省略LIMIT子句一样; OFFSET 0的效果和省略OFFSET子句是一样的

```sql
SELECT <select_list>
FROM <table_expression>
[ORDER BY ...]
[LIMIT {<number> | ALL}] [OFFSET <number>]
```

### VALUE列表

- 生成“常量表”，不需要实际在磁盘上创建一个表

### WITH查询（公共表表达式或CTE）

---

## 函数与操作符

### 条件表达式

#### CASE

```sql
CASE WHEN <condition> THEN <result>
[WHEN ...]
[ELSE <result>]
END

CASE <expression>
WHEN <value> THEN <result>
[WHEN ...]
[ELSE <result>]
END
```

#### COALESCE

- COALESCE函数返回它的第一个非空参数的值

```sql
COALESCE(value [, ...])
```

#### NULLIF

- 当value1和value2相等时，NULLIF返回一个空值

```sql
NULLIF(<value1>, <value2>)
```

#### GREATEST和LEAST

- 列表中的 NULL 数值将被忽略。只有所有表达式的结果都是 NULL 的时候，结果才会是 NULL

```sql
GREATEST(value [, ...])

LEAST(value [, ...])
```

> 请注意GREATEST和LEAST都不是 SQL 标准，但却是很常见的扩展。某些其他数据库让它们在任何参数为 NULL 时返回 NULL，而不是在所有参数都为 NULL 时才返回 NULL。

### 子查询表达式

### 模式匹配

### 聚焦函数

### 窗口函数

### 数学函数

### 三角函数

### 随机函数

### 字符串操作符和函数
