# Mysql 与 InnoDB

**数据库**：物理操作文件系统或其他形式类型的集合

**实例**：MySQL数据库由后台线程以及一个共享内存区组成

数据库与实例往往都是意义对应的，而我们无法直接操作数据库，而是要通过数据库实例来操作数据库文件，可以理解为数据库实例是数据库为上层提供的一个专门用于操作的接口。

## 数据存储

在InnoDB存储引擎中，所有的数据都被逻辑地存放在表空间，表空间（table space）是存储引擎中最高的存储逻辑单位，在表空间的下面又包括段（segment）、区（extent）、页（page）：

![Tablespace-segment-extent-page-row](https://raw.githubusercontent.com/Draveness/Analyze/master/contents/Database/images/mysql/Tablespace-segment-extent-page-row.jpg)

同一个数据库实例的所有表空间都具有相同的页大小；默认情况下，表空间中的页大小都为16KB，当然也可以通过改变 `innodb_page_size`选项对默认大小进行修改，需要注意的是不同的页大小最终也会导致区大小的不同。

![Relation Between Page Size - Extent Size](https://raw.githubusercontent.com/Draveness/Analyze/master/contents/Database/images/mysql/Relation-Between-Page-Size-Extent-Size.png)

上图可以看出，在InnoDB存储引擎中，一个区的大小最小为1MB，页的数量最少为64个。

## 如何存储表

MYSQL使用InnoDB存储表时，会将 **表的定义** 和 **数据索引** 等信息分开存储，其中前者在 `.frm` 文件中，后者存储在 `.ibd` 中。

![frm-and-ibd-file](https://raw.githubusercontent.com/Draveness/Analyze/master/contents/Database/images/mysql/frm-and-ibd-file.jpg)

**.frm文件**

无论在MYSQL中选择了什么引擎，所有的MYSQL表都会在硬盘上创建一个 .frm 文件用来描述表的格式或者说是定义。 .frm 文件的格式在不同的平台上面是相同的。

```sql
CREATE TABLE test_frm (
	column1 CHAR(5),
	column2 INTEGER
);
```

当我们使用上面的代码创建表的时候，会在磁盘上的 datadir 文件夹中创建一个test_frm.frm 的文件，这个文件就包含了表结构相关的信息：

![frm-file-hex](https://raw.githubusercontent.com/Draveness/Analyze/master/contents/Database/images/mysql/frm-file-hex.png)

[官方文档介绍](https://dev.mysql.com/doc/internals/en/frm-file-format.html)

**.ibd文件**

InnoDB 中用于存储数据的文件总共有两部分，一是系统表空间文件，包括 ibdata1、ibdata2 等文件，其中存储了 InnoDB 系统信息和用户数据库表数据和索引，是所有表公用的。

当打开 innodb_file_per_table 选项的时候， .ibd 文件就是每个表独有的表空间，文件存储了当前表的数据和相关的索引数据。