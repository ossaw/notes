# MySQL读书笔记

## 索引

优点：快速检索表中数据。

缺点：索引文件本身也占用空间，增删改会降低表的速度。

---

> 创建索引语法

1. CREATE [UNIQUE] INDEX indexName ON mytable(username(length));

2. ALTER TABLE tableName ADD [UNIQUE] INDEX indexName(columnName);

> 删除索引

DROP INDEX [indexName] ON mytable;

> 查询索引

SHOW INDEX FROM table_name;

> 哪些情况适合创建索引

1. 主键自动建立唯一索引

2. 频繁作为查询条件的字段应该创建索引

3. 查询中与其它表关联的字段，外键关系建立索引

4. 频繁更新的字段不适合建立索引

5. where条件里用不到的字段不要建立索引

6. 在高并发下倾向创建组合索引

7. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度

8. 查询中统计或分组字段

> 哪些情况不适合建索引

1. 表数据太少

2. 经常增删改的表

3. 数据重复且分布均匀的字段

## explain

### explain输出格式

#### id

> select查询序列号, 查询中执行select子句或操作表的顺序.

1. id相同执行顺序由上至下.

2. id不同, 如果是子查询, id序号会递增, id值越大, 优先级越高, 越先被执行.

3. id相同不同, 同时存在.

#### select_type

* SIMPLE, 表示此查询不包含UNION查询或子查询

* PRIMARY, 表示此查询是最外层的查询

* UNION, 表示此查询是UNION的第二或随后的查询

* DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询

* UNION RESULT, UNION的结果

* SUBQUERY, 子查询中的第一个SELECT

* DEPENDENT SUBQUERY: 子查询中的第一个SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.

#### table

> 表示查询涉及的表或衍生表

#### type

> 性能比较

** ALL < index < range ~ index_merge < ref < eq_ref < const < system **

> type常用类型

* system: 表中只有一条数据. 这个类型是特殊的const类型.

* const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const查询速度非常快.

* eq_ref: 此类型通常出现在多表的join查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是=, 查询效率较高.

* ref: 此类型通常出现在多表的join查询, 针对于非唯一或非主键索引, 或者是使用了最左前缀规则索引的查询. 

* range: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在=, <>, >, >=, <, <=, IS NULL, BETWEEN, IN() 操作中.

* index: 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.

* ALL: 表示全表扫描.

#### possible_keys

> possible_keys 表示 MySQL 在查询时, 能够使用到的索引. 注意, 即使有些索引在 possible_keys 中出现, 但是并不表示此索引会真正地被 MySQL 使用到. MySQL 在查询时具体使用了哪些索引, 由 key 字段决定.

#### key

> 此字段是 MySQL 在当前查询时所真正使用到的索引.

#### key_len

> 表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.

表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.
key_len的计算规则如下:

* 字符串

	* char(n): n 字节长度

	* varchar(n): 如果是 utf8 编码, 则是 3 n + 2字节; 如果是 utf8mb4 编码, 则是 4 n + 2 字节.

* 数值类型:

	* TINYINT: 1字节

	* SMALLINT: 2字节

	* MEDIUMINT: 3字节

	* INT: 4字节

	* BIGINT: 8字节

* 时间类型

	* DATE: 3字节
	 
	* TIMESTAMP: 4字节
	 
	* DATETIME: 8字节

* 字段属性: NULL 属性 占用一个字节. 如果一个字段是 NOT NULL 的, 则没有此属性.

#### rows

> rows 也是一个重要的字段. MySQL 查询优化器根据统计信息, 估算 SQL 要查找到结果集需要扫描读取的数据行数. 这个值非常直观显示 SQL 的效率好坏, 原则上 rows 越少越好.

#### Extra

> EXplain 中的很多额外的信息会在 Extra 字段显示, 常见的有:

* Using filesort 当 Extra 中有 Using filesort 时, 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果. 一般有 Using filesort, 都建议优化去掉, 因为这样的查询 CPU 资源消耗大.
