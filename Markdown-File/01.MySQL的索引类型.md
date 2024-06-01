## MySQL的索引类型

- 从存储结构上来划分：

  - Btree索引（B+树和B-树）
  - 哈希索引
  - full-index全文索引
  - RTree，一般不会使用，仅支持geometry数据类型，优势在于范围查找，效率较低，通常使用ElasticSearch代替。

- 从应用层次来划分：

  - 主键索引：加速查询 + 列值唯一（不可以有NULL）+ 表中只有一个
    - 在InnoDB的表中，如果没有显示的指定表的主键，InnoDB会自动先检查表中是否有唯一索引且不允许存在null值的字段，如果有，则选择该字段为默认的主键。否则，会自动创建一个6byte的自增主键。

  - 普通索引：一个索引只包含单个列，一个表可以有多个单列索引。加速查询
  - 唯一索引：加速查询 + 列值唯一（可以有NULL）
  - 覆盖索引：一个索引包含（或者说覆盖）所有需要查询的字段值
    - 例如表中有三个字段id,name,age。此时id有primary key，name设置普通索引。
    - Select age from table where name = 'xxx';
    - 这种情况下，如果没有覆盖索引，首先在name索引拿到主键id之后，要到primary key中去回表拿到数据。
    - 如果是覆盖索引，此时覆盖索引index(name, age)，一次查询就可以获取对应的age并返回。

  - 联合索引：一个索引包含多个列，专门用于组合搜索，效率大于索引合并
  - 全文索引：对文本的内容进行分词，进行搜索。目前只有CHAR, VARCHAR, TEXT列上可以创建全文索引，一般不会使用，效率较低。通常使用ElasticSearch代替。
  - 前缀索引：只适用于字符串类型的数据。前缀索引是对文本的前几个字符创建索引，相比普通索引建立的数据更小，因为只取前几个字符。

- 按照物理存储方式划分：

  - 聚集索引：表记录的排列顺序和索引的排列顺序一致。
    - 聚集索引不是一种索引类型，是一种存储结构。索引的键和数据是存放在一起的，在拿到键的时候，可以直接获取到对应的数据。并且叶子结点之间也是相互连接的，可以顺序的找到后续的键值。
    - MySQL的InnoDB默认的索引结构就是聚集索引。
  - 非聚集索引：表记录的排列顺序和索引的排列顺序不一致。
    - 相比于InnoDB的聚集索引，MyISAM引擎将索引进行单独保存。MYD存放的是数据文件，MYI存放的是索引文件。
    - 首先要在MYI文件中查询索引和对应MYD文件的数据指针地址，然后再在MYD文件中根据指针地址找到对应的数据返回。

  

## 什么情况下使用索引

- 为了快速查找匹配WHERE条件的行
- 为了从考虑的条件中消除行
- 如果表有一个联合索引，任何一个索引的最左前缀可以通过使用优化器来查找行
- 查询中与其他表关联的字，字段常常建立了外键关系。
- 查询中统计或分组统计的字段





## 创建索引

- 使用ALTER TABLE来创建索引

  - 主键索引（PRIMARY KEY）

    ~~~sql
    ALTER TABLE table_name ADD PRIMARY KEY (column);
    ~~~

  - 唯一索引（UNIQUE）

    ~~~sql
    ALTER TABLE table_name ADD UNIQUE (column);
    ~~~

  - 普通索引

    ~~~sql
    ALTER TABLE table_name ADD INDEX index_name (column);
    ~~~

  - 全文索引（FULLTEXT）

    ~~~sql
    ALTER TABLE table_name ADD FULLTEXT (column);
    ~~~

  - 多列索引（联合索引）

    ~~~sql
    ALTER TABLE table_name ADD INDEX index_name(column1, column2,...);
    ~~~

  - 前缀索引

    ~~~sql
    ALTER TABLE table_name ADD INDEX index_name(column(length));
    ~~~

    

- 使用CREATE INDEX来创建索引

  - 不能创建主键索引

  - 唯一索引（UNIQUE）

    ~~~sql
    CREATE UNIQUE INDEX index_name on table_name(column);
    ~~~

  - 普通索引

    ~~~sql
    CREATE INDEX index_name on table_name(column);
    ~~~

  - 全文索引（FULLTEXT）

    ~~~sql
    CREATE FULLTEXT INDEX index_name on table_name(column);
    ~~~

  - 多列索引（联合索引）

    ~~~sql
    CREATE INDEX index_name on table_name(column1, column2, ...);
    ~~~

    

## 删除索引

~~~sql
DROP INDEX index_name ON table_name;

ALTER TABLE table_name DROP INDEX index_name;

ALTER TABLE table_name DROP PRIMARY KEY;
~~~



## 查看索引

- 具体操作：

  ```sql
  show index from table_name;
  
  show keys from table_name;
  ```

- 执行结果：

  ![image-20240601110119834](/Users/mrwhynot/Library/Application Support/typora-user-images/image-20240601110119834.png)

- 参数：
  - Table，表名称
  - Non_unique，如果索引不能包括重复值，为0。可以包含重复值，为1。
    - 主键索引不包含重复值，为0。
    - 其他列分别为联合索引和FULLTEXT，所以可以有重复值。
  - Key_name，索引的名称
  - Seq_in_index，索引中的列序列号，从1开始
    - 例如index4_union_name_pwd_age包含了三个列，name,password,age，索引这三个列对应的值为1,2,3
  - Column_name，列名
  - Collation
    - 列以什么方式存储在索引中，A为升序，NULL无分类。
  - Cardinality
    - 索引中唯一值的数目预估值。通过运行ANALYZE TABLE或者myisamchk -a可以更新。基数越大，进行联合时，MySQL使用该索引的机会就越大。
  - Sub_part
    - 如果列只是部分被编入索引，则为被编入索引的字符的数目。如果整列被编入，为NULL
  - Packed
    - 指示关键字如何被压缩，如果没有被压缩，则为NULL
  - Null
    - 如果列含有NULL，为YES。没有则为NULL
  - Index_type
    - 索引底层使用的方法（BTREE，FULLTEXT，HASH，RTREE）

