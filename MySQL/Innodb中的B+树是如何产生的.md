# Innodb中的B+树是如何产生的


首先查询Innodb基本页的大小

```SQL
-- 基本页的大小
show GLOBAL STATUS like 'Innodb_page_size'
```

| Variable_name | Value |
| ----- | ----- |
| Innodb_page_size | 16384 |

**Tips：** 在数据库进行查询的时候，一次会从磁盘里边拿出约16kb的数据，再放到内存里边.

其次，创建一个表并查看表索引

```SQL
Create Table `t1`(
    `a` int primary key,
    `b` int,
    `c` int,
    `d` int,
    `e` varchar(20)
) ENGINE = InnoDB;

show index from t1;
```

此时可发现`Index_type`为`BTREE`，由于B+树是B树的一种，所以此处显示的`BTREE`实际为B+树

接着，给表插入数据，并查看结果
```SQL
insert into t1 values (1,1,1,1, 'a');
insert into t1 values (8,8,3,8, 'h');
insert into t1 values (2,2,2,2, 'b');
insert into t1 values (5,2,3,5, 'e');
insert into t1 values (3,3,2,2, 'c');
insert into t1 values (7,4,5,5, 'g');
insert into t1 values (6,6,4,4, 'f');

select * from t1
```
查询结果如下：

|a|b|c|d|e|
|---|---|---|---|---|
|1|1|1|1|a|
|2|2|2|2|b|
|3|3|2|2|c|
|5|2|3|5|e|
|6|6|4|4|f|
|7|4|5|5|g|
|8|8|3|8|h|

可以发现在查询的时候并没有给查询附加排序语句，但是查询到的却是是按a字段进行排序的后结果，由此可见，在插入过程中，会自动在页的用户数据区域进行升序排序。

如果只是单纯的升序排序，假设把各个数据连起来的数据结构是链表，那我在同一个用户数据区域内要找到我需要的数据则要从头开始一个个遍历，直到我找到为止，无疑这特别影响效率。所以每个页中通常会设一个页目录，假如以三个为一组，将每组开头的地址写入页目录存起来。下次查询只需要根据页目录的值大小跟需要查询的值进行比对就可以获取到对应的地址从而缩小范围。

同理每个页之间也会有一个指针用来存地址，这样在每次搜索的时候只需要根据指针的值的范围来找到对应的地址即可。

而B+树也正是这种树+链表的形式

