# MySQL

## 数据库基础

### 数据库的优势

**数据保存在内存：**

*   优点：存取速度快

*   缺点：数据不能永久保存


**数据保存在文件：**

*   优点：数据永久保存

*   缺点：1）速度比内存操作慢，频繁的IO操作。2）查询数据不方便


**数据保存在数据库：**

*   数据永久保存
*   使用SQL语句，查询方便效率高。
*   管理数据方便

### 三大范式

-   第一范式：每个列都不可以再拆分。
-   第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。
-   第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

在设计数据库结构的时候，要尽量遵守三范式，如果不遵守，必须有足够的理由，比如性能。

>   事实上我们经常会为了性能而妥协数据库的设计。

### MySQL的权限表

MySQL服务器通过权限表来**控制用户对数据库的访问**，权限表存放在mysql数据库里，由mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。

1.  user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。
2.  db权限表：记录各个帐号在各个数据库上的操作权限。

3.  table_priv权限表：记录数据表级的操作权限。

4.  columns_priv权限表：记录数据列级的操作权限。

5.  host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。

## 数据类型

1、**整数类型**，包括TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示1字节、2字节、3字节、4字节、8字节整数。任何整数类型都可以加上UNSIGNED属性，表示数据是无符号的，即非负整数。

长度：整数类型可以被指定长度，例如：INT(11)表示长度为11的INT类型。**长度在大多数场景是没有意义的，它不会限制值的合法范围，只会影响显示字符的个数**，而且需要和UNSIGNED ZEROFILL属性配合使用才有意义。

例子，假定类型设定为INT(5)，属性为UNSIGNED ZEROFILL，如果用户插入的数据为12的话，那么数据库实际存储数据为00012。

2、**实数类型**，包括FLOAT、DOUBLE、DECIMAL。

-   DECIMAL可以用于存储比BIGINT还大的整型，能存储精确的小数。
-   而FLOAT和DOUBLE是有取值范围的，并支持使用标准的浮点进行近似计算。

计算时FLOAT和DOUBLE相比DECIMAL效率更高一些，DECIMAL你可以理解成是用字符串进行处理。

3、**字符串类型**，包括VARCHAR、CHAR、TEXT、BLOB

-   VARCHAR用于存储**可变长字符串**，它比定长类型更节省空间。
-   VARCHAR使**用额外1或2个字节存储字符串长度**。列长度小于255字节时，使用1字节表示，否则使用2字节表示。
-   VARCHAR存储的内容超出设置的长度时，**内容会被截断**。
-   CHAR是定长的，根据定义的字符串长度分配足够的空间。
-   CHAR会根据需要使用空格进行填充，方便比较。
-   CHAR适合存储很短的字符串，或者所有值都接近同一个长度。
-   CHAR存储的内容超出设置的长度时，内容同样会被截断。

使用策略：

-   对于经常变更的数据来说，CHAR比VARCHAR更好，因为CHAR不容易产生碎片。
-   对于非常短的列，CHAR比VARCHAR在存储空间上更有效率。
-   使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。
-   尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。

4、**枚举类型**（ENUM），把不重复的数据存储为一个预定义的集合。

-   有时可以使用ENUM代替常用的字符串类型。
-   ENUM存储非常紧凑，会把列表值压缩到一个或两个字节。
-   ENUM在内部存储时，其实存的是整数。
-   尽量避免使用数字作为ENUM枚举的常量，因为容易混乱。
-   排序是按照内部存储的整数

5、**日期和时间类型**，尽量使用timestamp，空间效率高于datetime，

-   用整数保存时间戳通常不方便处理。
-   如果需要存储微秒，可以使用bigint存储。

## 存储引擎

存储引擎：MySQL中的数据、索引以及其他对象是如何存储的，是一套文件系统的实现。常用的存储引擎有以下：

### MyISAM与InnoDB

-   Innodb引擎：Innodb引擎提供了对数据库ACID事务的支持。并且还提供了行级锁和外键的约束。它的设计的目标就是处理大数据容量的数据库系统。
-   MyIASM引擎(原本Mysql的默认引擎)：不提供事务的支持，也不支持行级锁和外键。
-   MEMORY引擎：所有的数据都在内存中，数据的处理速度快，但是安全性不高。

#### 区别

1.  InnoDB索引是**聚簇索引**，MyISAM索引是**非聚簇索引**。
2.  InnoDB的主键索引的叶子节点存储着行数据，因此主键索引非常高效。
3.  MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。
4.  InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。

#### InnoDB引擎的4大特性

1.  插入缓冲（insert buffer)
2.  二次写(double write)
3.  自适应哈希索引(ahi)
4.  预读(read ahead)

#### 存储引擎选择

如果没有特别的需求，使用默认的Innodb即可。

-   MyISAM：以**读写插入为主**的应用程序，比如博客系统、新闻门户网站。
-   Innodb：**更新（删除）操作频率也高，或者要保证数据的完整性；并发量高，支持事务和外键**。比如OA自动化办公系统。

## 视图

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。对视图的操作和对普通表的操作一样。

视图具有如下好处：

-   简化复杂的 SQL 操作，比如复杂的连接；
-   只使用实际表的一部分数据；
-   通过只给用户访问视图的权限，保证数据的安全性；
-   更改数据格式和表示。

## 索引

### 简介

索引是一种特殊的文件(InnoDB数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。

索引是一种数据结构。数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询、更新数据库表中数据。索引的实现通常使用B树及其变种B+树。

更通俗的说，索引就相当于目录。为了方便查找书中的内容，通过对内容建立索引形成目录。索引是一个文件，它是要占据物理空间的。

### 基本原理

索引用来快速地寻找那些具有特定值的记录。如果没有索引，一般来说执行查询时遍历整张表。

索引的原理很简单，就是把无序的数据变成有序的查询

1.  把创建了索引的列的内容进行排序
2.  对排序结果生成**倒排表**
3.  在倒排表内容上拼上数据地址链
4.  在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

### 优缺点

**索引的优点：**

可以大大加快数据的检索速度，这也是创建索引的最主要的原因。

通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

**索引的缺点：**

时间方面：创建索引和维护索引要耗费时间，具体地，当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，会降低增/改/删的执行效率；

空间方面：索引需要占物理空间。

### 索引覆盖

如果要查询的字段都建立过索引，那么引擎会直接在索引表中查询而不会访问原始数据（否则只要有一个字段没有建立索引就会做全表扫描），这叫索引覆盖。

因此我们需要**尽可能的在select后只写必要的查询字段，以增加索引覆盖的几率**。

>   这里值得注意的是不要想着为每个字段建立索引，因为优先使用索引的优势就在于其体积小。

### 索引类型

*   **主键索引**: 数据列不允许重复，不允许为NULL，一个表只能有一个主键。

*   **唯一索引**: 数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引。

    >   可以通过 ALTER TABLE table_name ADD UNIQUE (column)；创建唯一索引
    >
    >   可以通过 ALTER TABLE table_name ADD UNIQUE (column1,column2)；创建唯一组合索引

*   **普通索引**: 基本的索引类型，没有唯一性的限制，允许为NULL值。

    >   可以通过ALTER TABLE table_name ADD INDEX index_name (column); 创建普通索引
    >
    >   可以通过ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3)；创建组合索引

*   **全文索引**：是目前搜索引擎使用的一种关键技术。

    >   可以通过ALTER TABLE table_name ADD FULLTEXT (column); 创建全文索引

### 索引的数据结构

索引的数据结构和具体存储引擎的实现有关，在MySQL中使用较多的索引有Hash索引，B+树索引等，而我们经常使用的InnoDB存储引擎的默认索引实现为：**B+树索引**。对于哈希索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景，建议选择BTree索引。

MySQL通过存储引擎取数据，基本上90%的人用的就是InnoDB了，按照实现方式分，InnoDB的索引类型目前只有两种：BTREE（B树）索引和HASH索引。

#### B树索引

B树索引是Mysql数据库中使用最频繁的索引类型，基本所有存储引擎都支持BTree索引。通常我们说的索引不出意外指的就是（B树）索引（实际是用B+树实现的，因为在查看表索引时，mysql一律打印BTREE，所以简称为B树索引）

**BTree算法**

BTree是最常用的mysql数据库索引算法，也是mysql默认的算法。因为它不仅可以被用在=,>,>=,<,<=和between这些比较操作符上，而且还可以用于like操作符，只要它的查询条件是一个不以通配符开头的常量。

**查询方式：**

主键索引区：PI(关联保存的时数据的地址)按主键查询,

普通索引区：si(关联的id的地址,然后再到达上面的地址)。所以按主键查询,速度最快

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gf6pgg6jb9j30ma0b9755.jpg" alt="B+树" style="zoom:67%;" />

**B+树性质：**

-   n棵子tree的节点包含n个关键字，不用来保存数据而是保存数据的索引。
-   所有的叶子结点中包含了全部关键字的信息，及指向含这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。
-   所有的非终端结点可以看成是索引部分，结点中仅含其子树中的最大（或最小）关键字。
-   B+ 树中，数据对象的插入和删除仅在叶节点上进行。
-   B+树有2个头指针，一个是树的根节点，一个是最小关键码的叶节点。

**B树和B+树的区别**

1.  在B树中，你可以将键和值存放在内部节点和叶子节点；但在B+树中，内部节点都是键，没有值，叶子节点同时存放键和值。

2.  B+树的叶子节点有一条链相连，而B树的叶子节点各自独立。

**使用B树的好处**

B树可以在内部节点同时存储键和值，因此，把频繁访问的数据放在靠近根节点的地方将会大大提高热点数据的查询效率。这种特性使得B树在**特定数据重复多次查询**的场景中更加高效。

**使用B+树的好处**

由于B+树的内部节点只存放键，不存放值，因此，一次读取，可以在内存页中获取更多的键，有利于**更快地缩小查找范围**。B+树的叶节点由一条链相连，因此，当需要进行一次全数据遍历的时候，B+树只需要使用`O(logN)`时间找到最小的一个节点，然后通过链进行`O(N)`的顺序遍历即可。而B树则需要对树的每一层进行遍历，这会需要更多的内存置换次数，因此也就需要花费更多的时间。

**数据库为什么使用B+树而不是B树**

1.  B树只适合随机检索，而B+树同时支持随机检索和顺序检索

2.  B+树空间利用率更高，可减少I/O次数，磁盘读写代价更低

    一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗。B+树的内部结点并没有指向关键字具体信息的指针，只是作为索引使用，其内部结点比B树小，盘块能容纳的结点中关键字数量更多，一次性读入内存中可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素；

3.  B+树的查询效率更加稳定

    B树搜索有可能会在非叶子结点结束，越靠近根节点的记录查找时间越短，只要找到关键字即可确定记录的存在，其性能等价于在关键字全集内做一次二分查找。而在B+树中，顺序检索比较明显，随机检索时，任何关键字的查找都必须走一条从根节点到叶节点的路，所有关键字的查找路径长度相同，导致每一个关键字的查询效率相当。

4.  B树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。B+树的叶子节点使用指针顺序连接在一起，只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作。

5.  增删文件（节点）时，效率更高

    因为B+树的叶子节点包含所有关键字，并以有序的链表结构存储，这样可很好提高增删效率。

#### 哈希索引

哈希索引类似于数据结构中简单实现的HASH表（散列表）一样，当我们在mysql中用哈希索引时，主要就是通过Hash算法（常见的Hash算法有直接定址法、平方取中法、折叠法、除数取余法、随机数法），将数据库字段数据转换成定长的Hash值，与这条数据的行指针一并存入Hash表的对应位置；如果发生Hash碰撞（两个不同关键字的Hash值相同），则在对应Hash键下以链表形式存储。

**Hash算法**

Hash Hash索引只能用于**对等比较**，例如=,<=>（相当于!=）操作符。由于是一次定位数据，不像BTree索引需要从根节点到枝节点，最后才能访问到页节点这样多次IO访问，所以**检索效率远高于BTree索引**。

**Hash索引和B+树所有有什么区别或者说优劣呢?**

首先要知道Hash索引和B+树索引的底层实现原理：

hash索引底层就是hash表，进行查找时，调用一次hash函数就可以获取到相应的键值，之后进行回表查询获得实际数据。B+树底层实现是多路平衡查找树。对于每一次的查询都是从根节点出发，查找到叶子节点方可以获得所查键值，然后根据查询判断是否需要回表查询数据。

那么可以看出他们有以下的不同：

**hash索引进行等值查询更快(一般情况下)，但是却无法进行范围查询。**

因为在hash索引中经过hash函数建立索引之后，索引的顺序与原顺序无法保持一致，不能支持范围查询。而B+树的的所有节点皆遵循(左节点小于父节点，右节点大于父节点，多叉树也类似)，天然支持范围。

1.  hash索引不支持使用索引进行排序，原理同上。
2.  hash索引不支持模糊查询以及多列索引的最左前缀匹配。原理也是因为hash函数的不可预测。AAAA和AAAAB的索引没有相关性。
3.  hash索引任何时候都避免不了回表查询数据，而B+树在符合某些条件(聚簇索引，覆盖索引等)的时候可以只通过索引完成查询。
4.  hash索引虽然在等值查询上较快，但是不稳定。性能不可预测，当某个键值存在大量重复的时候，发生hash碰撞，此时效率可能极差。而B+树的查询效率比较稳定，对于所有的查询都是从根节点到叶子节点，且树的高度较低。

因此，**在大多数情况下，直接选择B+树索引可以获得稳定且较好的查询速度，而不需要使用hash索引**。

### 索引设计的原则

-   适合索引的列是出现在where子句中的列，或者连接子句中指定的列
-   基数较小的类，索引效果较差，没有必要在此列建立索引
-   使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间
-   不要过度索引。索引需要额外的磁盘空间，并降低写操作的性能。在修改表内容的时候，索引会进行更新甚至重构，索引列越多，这个时间就会越长。所以只保持需要的索引有利于查询即可。

### 创建索引的原则

索引虽好，但也不是无限制的使用，最好符合一下几个原则

1.  最左前缀匹配原则，组合索引非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如`a = 1 and b = 2 and c > 3 and d = 4`， 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
2.  较频繁作为查询条件的字段才去创建索引

3.  更新频繁字段不适合创建索引

4.  若是不能有效区分数据的列不适合做索引列（如性别，男女未知，最多也就三种，区分度实在太低）

5.  尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a，b)的索引，那么只需要修改原来的索引即可。
6.  定义有外键的数据列一定要建立索引。

7.  对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

8.  对于定义为text、image和bit的数据类型的列不要建立索引。

### 创建索引的三种方式

1.  第一种方式：在执行`CREATE TABLE`时创建索引
2.  第二种方式：使用`ALTER TABLE`命令去增加索引
3.  第三种方式：使用`CREATE INDEX`命令创建

### 删除索引

根据索引名删除普通索引、唯一索引、全文索引：`DROP INDEX 索引的名字 ON 索引的表`

删除主键索引：`alter table 表名 drop primary key`（因为主键只有一个）。这里值得注意的是，如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引）

### 创建索引注意事项

-   非空字段：应该指定列为NOT NULL，除非你想存储NULL。在mysql中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。应该用0、一个特殊的值或者一个空串代替空值；
-   取值离散大的字段：变量各个取值之间的差异程度大的列放到联合索引的前面，可以通过count()函数查看字段的差异值，返回值越大说明字段的唯一值越多字段的离散程度高；
-   索引字段越小越好：数据库的数据存储以页为单位，一页存储的数据越多一次IO操作获取的数据越大，效率越高。

### 使用索引查询一定能提高查询的性能吗？

通常通过索引查询数据比全表扫描要快。但是我们也必须注意到它的代价。

索引需要空间来存储，也需要定期维护， 每当有记录在表中增减或索引列被修改时，索引本身也会被修改。这意味着每条记录的INSERT，DELETE，UPDATE将为此多付出4，5 次的磁盘I/O。因为索引需要额外的存储空间和处理，那些不必要的索引反而会使查询反应时间变慢。使用索引查询不一定能提高查询性能，索引范围查询(INDEX RANGE SCAN)适用于两种情况：

1.  基于一个范围的检索，一般查询返回结果集小于表中记录数的30%
2.  基于非唯一性索引的检索

### **百万级别或以上的数据如何删除？**

关于索引：由于索引需要额外的维护成本，因为索引文件是单独存在的文件,所以当我们对数据的增加,修改,删除,都会产生额外的对索引文件的操作,这些操作需要消耗额外的IO,会降低增/改/删的执行效率。所以，在我们删除数据库百万级别数据的时候，查询MySQL官方手册得知**删除数据的速度和创建的索引数量是成正比的**。

1.  所以我们想要删除百万数据的时候可以**先删除索引**（此时大概耗时三分多钟）

2.  然后**删除其中无用数据**（此过程需要不到两分钟）

3.  删除完成后**重新创建索引**(此时数据较少了)创建索引也非常快，约十分钟左右。

### 前缀索引

语法：index(field(10))，使用字段值的前10个字符建立索引，默认是使用字段的全部内容建立索引。

前提：前缀的标识度高。比如密码就适合建立前缀索引，因为密码几乎各不相同。

实操的难度：在于前缀截取的长度。

我们可以利用`select count(*)/count(distinct left(password,prefixLen));`，通过从调整prefixLen的值（从1自增）查看不同前缀长度的一个平均匹配度，接近1时就可以了（表示一个密码的前prefixLen个字符几乎能确定唯一一条记录）

#### 最左前缀、最左匹配

最左前缀匹配原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配。

比如`a = 1 and b = 2 and c > 3 and d = 4` 如果建立(a,b,c,d)顺序的索引，d是用不到索引的(c已经是范围查询了，d肯定是排不了序了)，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

=和in可以乱序，比如`a = 1 and b = 2 and c = 3` 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

### 聚簇索引、非聚簇索引

*   聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据

*   非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行。

    MyISAM通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在key buffer命中时，速度慢的原因

澄清一个概念：InnoDB中，在聚簇索引之上创建的索引称之为辅助索引，辅助索引访问数据总是需要二次查找，非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引，辅助索引叶子节点存储的不再是行的物理位置，而是主键值。

**非聚簇索引一定会回表查询吗？**

不一定，这涉及到查询语句所要求的字段是否全部命中了索引，如果全部命中了索引，那么就不必再进行回表查询。

举个简单的例子，假设我们在员工表的年龄上建立了索引，那么当进行`select age from employee where age < 20`的查询时，在索引的叶子节点上，已经包含了age信息，不会再次进行回表查询。

### 联合索引

MySQL可以使用多个字段同时建立一个索引，叫做联合索引。在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

具体原因为:

MySQL使用索引时需要索引有序，假设现在建立了"name，age，school"的联合索引，那么索引的排序为: 先按照name排序，如果name相同，则按照age排序，如果age的值也相等，则按照school进行排序。

当进行查询，此时索引仅仅按照name严格有序，因此必须首先使用name字段进行等值查询，之后对于匹配到的列而言，其按照age字段严格有序，此时可以使用age字段用做索引查找，以此类推。因此在建立联合索引的时候应该注意索引列的顺序，一般情况下，将查询需求频繁或者字段选择性高的列放在前面。此外可以根据特例的查询或者表结构进行单独的调整。

## 事务

### 什么是数据库事务？

事务是一个不可分割的数据库操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。事务是逻辑上的一组操作，要么都执行，要么都不执行。

事务最经典也经常被拿出来说例子就是转账了。

假如小明要给小红转账1000元，这个转账会涉及到两个关键操作就是：将小明的余额减少1000元，将小红的余额增加1000元。万一在这两个操作之间突然出现错误比如银行系统崩溃，导致小明余额减少而小红的余额没有增加，这样就不对了。事务就是保证这两个关键操作要么都成功，要么都要失败。

### 事物的四大特性(ACID)

关系性数据库需要遵循ACID规则，具体内容如下：

-   原子性：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

-   一致性：执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；

-   隔离性：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

-   持久性：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 并发一致性问题

在并发环境下，事务的隔离性很难保证，因此会出现很多并发一致性问题。

#### 丢失修改

T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。

#### 读脏数据

T1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。

#### 不可重复读

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

#### 幻影读

T1 读取某个**范围**的数据，T2 在这个**范围内**插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

### 隔离级别

#### 未提交读

事务中的修改，即使没有提交，对其它事务也是可见的。

#### 提交读

一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

#### 可重复读

保证在同一个事务中多次读取同一数据的结果是一样的。

#### 可串行化

强制事务串行执行，这样多个事务互不干扰，不会出现并发一致性问题。

该隔离级别需要加锁实现，因为要使用加锁机制保证同一时间只有一个事务执行，也就是保证事务串行执行。

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| -------- | ---- | ---------- | ------ |
| 未提交读 | √    | √          | √      |
| 提交读   | ×    | √          | √      |
| 可重复读 | ×    | ×          | √      |
| 可串行化 | ×    | ×          | ×      |

事务隔离机制的实现基于锁机制和并发调度。其中并发调度使用的是MVVC（多版本并发控制），通过保存修改的旧版本信息来支持并发一致性读和回滚等特性。

因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是READ-COMMITTED(读取提交内容)，但是你要知道的是InnoDB 存储引擎默认使用REPEATABLE-READ（可重读），并不会有任何性能损失。

InnoDB 存储引擎在分布式事务的情况下一般会用到SERIALIZABLE(可串行化)隔离级别。

## 锁

当数据库有并发事务的时候，可能会产生数据的不一致，这时候需要一些机制来保证访问的次序，锁机制就是这样的一个机制。

就像酒的房间，如果大家随意进出，就会出现多人抢夺同一个房间的情况，而在房间上装上锁，申请到钥匙的人才可以入住并且将房间锁起来，其他人只有等他使用完毕才可以再次使用。

### MySQL都有哪些锁？

从锁的类别上来讲，有共享锁和排他锁。

共享锁: 又叫做读锁。当用户要进行数据的读取时，对数据加上共享锁。共享锁可以同时加上多个。

排他锁: 又叫做写锁。当用户要进行数据的写入时，对数据加上排他锁。排他锁只可以加一个，他和其他的排他锁，共享锁都相斥。

用上面的例子来说就是用户的行为有两种，一种是来看房，多个用户一起看房是可以接受的。一种是真正的入住一晚，在这期间，无论是想入住的还是想看房的都不可以。

锁的粒度取决于具体的存储引擎，InnoDB实现了行级锁，页级锁，表级锁。

他们的加锁开销从大到小，并发能力也是从大到小。

### 隔离级别与锁的关系

-   在Read Uncommitted级别下，读取数据不需要加共享锁，这样就不会跟被修改的数据上的排他锁冲突
-   在Read Committed级别下，读操作需要加共享锁，但是在语句执行完以后释放共享锁；
-   在Repeatable Read级别下，读操作需要加共享锁，但是在事务提交之前并不释放共享锁，也就是必须等待事务执行完毕以后才释放共享锁。
-   SERIALIZABLE 是限制性最强的隔离级别，因为该级别锁定整个范围的键，并一直持有锁，直到事务完成。

**按照锁的粒度分数据库锁有哪些？锁机制与InnoDB锁算法**

在关系型数据库中，可以按照锁的粒度把数据库锁分为行级锁(INNODB引擎)、表级锁(MYISAM引擎)和页级锁(BDB引擎 )。

MyISAM和InnoDB存储引擎使用的锁：

1、MyISAM采用表级锁(table-level locking)。

2、InnoDB支持行级锁(row-level locking)和表级锁，默认为行级锁

### 行级锁、表级锁和页级锁

**行级锁** ：行级锁是Mysql中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为共享锁 和 排他锁。

特点：开销大，加锁慢；**会出现死锁**；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

**表级锁** ：表级锁是MySQL中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分MySQL引擎支持。最常使用的MYISAM与INNODB都支持表级锁定。表级锁定分为表共享读锁（共享锁）与表独占写锁（排他锁）。

特点：开销小，加锁快；**不会出现死锁**；锁定粒度大，发出锁冲突的概率最高，并发度最低。

**页级锁** ：页级锁是MySQL中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。

特点：开销和加锁时间界于表锁和行锁之间；**会出现死锁**；锁定粒度界于表锁和行锁之间，并发度一般。

### 死锁

死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方的资源，从而导致恶性循环的现象。

**常见的解决死锁的方法**

1.  如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。
2.  在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；
3.  对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；
4.  用分布式事务锁或者使用乐观锁

### 乐观锁和悲观锁

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

**悲观锁：**假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。

实现方式：使用数据库中的锁机制

**乐观锁：**假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。

实现方式：乐观锁一般会使用版本号机制或CAS算法实现。

**两种锁的使用场景**

从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种.

-   乐观锁适用于写比较少的情况下（多读场景），即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。
-   如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以一般多写的场景下用悲观锁就比较合适。

### CAS

CAS 是 compare and swap 的缩写，即我们所说的比较交换。

CAS 是一种基于锁的操作，而且是乐观锁。在 java 中锁分为乐观锁和悲观锁。悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。而乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加 version 来获取数据，性能较悲观锁有很大的提高。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a 线程获取地址里面的值被b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。

java.util.concurrent.atomic 包下的类大多是使用 CAS 操作来实现的。

**CAS 的会产生什么问题？**

1.  ABA 问题：

    比如说一个线程 one 从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且 two 进行了一些操作变成了 B，然后 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操作发现内存中仍然是 A，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但可能存在潜藏的问题。从 Java1.5 开始 JDK 的 atomic包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。

2.  循环时间长开销大：

    对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 synchronized。

3.  只能保证一个共享变量的原子操作：

    当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁。

## 数据库优化

### 为什么要优化

-   系统的吞吐量瓶颈往往出现在数据库的访问速度上
-   随着应用程序的运行，数据库的中的数据会越来越多，处理时间会相应变慢
-   数据是存放在磁盘上的，读写速度无法和内存相比

优化原则：减少系统瓶颈，减少资源占用，增加系统的反应速度。

### 优化SQL语句

1：对查询进行优化，应尽量避免全表扫描，首先应考虑在where及order by 涉及的列上创建索引。因为：索引对查询的速度有着至关重要的影响。

2：尽量避免在where字句中对字段进行`null`值的判断。否则将会导致引擎放弃使用索引而进行全表扫描。

3：应尽量避免在where子句中使用!=或者是<>操作符号。否则引擎将放弃使用索引，进而进行全表扫描。

4：应尽量避免在where子句中使用or来连接条件，否则导致放弃使用索引而进行全表扫描。可以使用 union 或者是 union all代替。

5：in 和 not in 也要慎用，否则将会导致全表扫描。in 对于连续的数组，可以使用between ..…and.来代替。

6：where子句参数使用时候需注意

如果在where子句中使用参数，也会导致全表扫描。因为sql只会在运行时才会解析局部变量。**但优化程序不能将访问计划的选择推迟到运行时；必须在编译时候进行选择**。然而，如果在编译时建立访问计划，变量的值还是未知大，因而无法作为索引选择输入项。

7：尽量避免在where子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。

8：尽量避免爱where子句中对字段进行函数操作，这将导致引擎放弃索引，而进行全表扫描。

9：很多时候用exists 代替 in是一个很好的选择。

10：并不是所有索引对查询都有效

sql是根据表中数据进行查询优化的，当索引(索引字段)有大量重复数据的时候，sql查询可能不会去利用索引。如字段 sex中male、female 几乎各一半。那么即使在sex上创建了索引对查询效率也起不了多大作用。

11：索引创建需注意

并非索引创建越多越好。索引固然可以提高相应的查询效率，但是同样会降低insert以及update的效率。因为在insert或是update的时候有可能会重建索引或是修改索引。所以索引怎样创建需要慎重考虑，视情况而定。一个表中索引数量最好不要超过6个。若太多，则需要考虑对一些不常用的列创建索引是否有必要。

### 数据库结构优化

一个好的数据库设计方案对于数据库的性能往往会起到事半功倍的效果。

需要考虑数据冗余、查询和更新的速度、字段的数据类型是否合理等多方面的内容。

**将字段很多的表分解成多个表**

对于字段较多的表，如果有些字段的使用频率很低，可以将这些字段分离出来形成新表。

因为当一个表的数据量很大时，会由于使用频率低的字段的存在而变慢。

**增加中间表**

对于需要经常联合查询的表，可以建立中间表以提高查询效率。

通过建立中间表，将需要通过联合查询的数据插入到中间表中，然后将原来的联合查询改为对中间表的查询。

**增加冗余字段**

设计数据表时应尽量遵循范式理论的规约，尽可能的减少冗余字段，让数据库设计看起来精致、优雅。但是，合理的加入冗余字段可以提高查询速度。

表的规范化程度越高，表和表之间的关系越多，需要连接查询的情况也就越多，性能也就越差。

**注意**：冗余字段的值在一个表中修改了，就要想办法在其他表中更新，否则就会导致数据不一致的问题。

### 数据库单表优化

当MySQL单表记录数过大时，数据库的CRUD性能会明显下降，一些常见的优化措施如下：

-   限定数据的范围：务必禁止不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内；
-   读/写分离：经典的数据库拆分方案，主库负责写，从库负责读；
-   缓存：使用MySQL的缓存，另外对重量级、更新少的数据可以考虑使用应用级别的缓存；
-   通过分库分表的方式进行优化，主要有垂直分表和水平分表

**垂直分区：**

根据数据库里面数据表的相关性进行拆分。例如，用户表中既有用户的登录信息又有用户的基本信息，可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。

垂直拆分的优点：可以使得行数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。

垂直拆分的缺点：主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

**水平分区：**

保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。水平拆分可以支撑非常大的数据量。

水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。

水平拆分可以支持非常大的数据量。需要注意的一点是:分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以**水平拆分最好分库** 。

**水平切分的缺点**

1.  给应用增加复杂度，通常查询时需要多个表名，查询所有数据都需UNION操作
2.  在许多数据库应用中，这种复杂度会超过它带来的优点，查询时会增加读一个索引层的磁盘次数

《Java工程师修炼之道》的作者推荐 **尽量不要对数据进行分片**，因为拆分会带来逻辑、部署、运维的各种复杂度 ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。

**数据库分片的两种常见方案：**

-   客户端代理：分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。当当网的 Sharding-JDBC 、阿里的TDDL是两种比较常用的实现。
-   中间件代理：在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。我们现在谈的 Mycat 、360的Atlas、网易的DDB等等都是这种架构的实现。

### 分页查询的性能优化

Mysql的分页查询十分简单，但是当数据量大的时候一般的分页就吃不消了。

传统分页查询：`select * from table limit M,N`

表示查询表M条数据开始（不包括第M条数据），然后一共查询N条

MySQL的limit工作原理就是先读取前面n条记录，然后抛弃前n条，读后面m条想要的，百千条数据的时候，这种查找的方法是全表扫描，找到页码，所以N越大，偏移量越大，性能就越差。

推荐分页查询方法：

1.  **使用 id 限定优化**

    这种方式假设数据表的id是连续递增的，根据查询的页数和查询的记录数可以算出查询的id的范围

    `select * from student where id>=8000000 ORDER BY id desc limit 0,20`

    适用于数据量多的情况(元组数上万)。最好ORDER BY后的列对象是主键或唯一索引，使得ORDER BY操作能利用索引被消除索引扫描，速度会很快。MySQL的排序操作，asc升序，desc降序。

    这种查询方式能够极大地优化查询速度，基本能够在几十毫秒之内完成。限制是只能使用于明确知道id的情况，不过一般建立表的时候，都会添加基本的id字段，这为分页查询带来很多便利。

2.  **利用子查询**

    这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

    `select id from student limit 800000, 20`

    id是主键且是唯一索引，这样只需要查找索引结构，查询速度会非常快，但是这样查询只有id信息，那么如果要得到全列信息，可以使用子查询或者join连接

    `select * from student where id>=(select id from student limit 800000,1) limit 20`

    `select * from student a join(select id from student limit 800000, 20) b ON a.id= b.id`

### 上亿条数据的数据库、查询结果都是千万级别的，怎么优化

1.  第一优化你的sql和索引；
2.  第二加缓存，memcached,redis；
3.  第三以上都做了后，还是慢，就做主从复制或主主复制，读写分离
4.  第四如果以上都做了还是慢，不要想着去做切分，mysql自带分区表，先试试这个，对你的应用是透明的，无需更改代码,但是sql语句是需要针对分区表做优化的，sql条件中要带上分区条件的列，从而使查询定位到少量的分区上，否则就会扫描全部分区，另外分区表还有一些坑，在这里就不多说了；
5.  第五如果以上都做了，那就先做垂直拆分，其实就是根据你模块的耦合度，将一个大的系统分为多个小的系统，也就是分布式系统；
6.  第六才是水平切分，针对数据量大的表，这一步最麻烦，最能考验技术水平，要选择一个合理的sharding key,为了有好的查询效率，表结构也要改动，做一定的冗余，应用也要改，sql中尽量带sharding key，将数据定位到限定的表上去查，而不是扫描全部的表；

mysql数据库一般都是按照这个步骤去演化的，成本也是由低到高

## 一致性哈希

在 Mysql数据库要存储的量达到一个很高的地步的时候，我们会对数据库进行分库分表操作。将单张表的数据切分到多个服务器上去，每个服务器具有相应的库与表，只是表中数据不同。 这样做能够有效的缓解单机数据库的压力和系统的性能瓶颈。

Distributed Hash Table（DHT） 是一种哈希分布方式，其目的是为了克服传统哈希分布在服务器节点数量变化时大量数据迁移的问题。

**基本原理**

将哈希空间 [0, 2^n^-1] 看成一个哈希环，每个服务器节点都配置到哈希环上。每个数据对象通过哈希取模得到哈希值之后，存放到哈希环中顺时针方向第一个大于等于该哈希值的节点上。

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gf6pgeci4vj30fa0fd0te.jpg)

一致性哈希在增加或者删除节点时只会影响到哈希环中相邻的节点，例如下图中新增节点 X，只需要将它前一个节点 C 上的数据重新进行分布即可，对于节点 A、B、D 都没有影响。

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gf6pgf7o3rj30f60fo0ti.jpg)

## MySQL的复制与分离

主从复制：将主数据库中的DDL和DML操作通过二进制日志（BINLOG）传输到从数据库上，然后将这些日志重新执行（重做）；从而使得从数据库的数据与主数据库保持一致。

#### 主从复制

-   主数据库出现问题，可以切换到从数据库。
-   可以进行数据库层面的读写分离。
-   可以在从数据库上进行日常备份。

**MySQL主从复制解决的问题**

-   数据分布：随意开始或停止复制，并在不同地理位置分布数据备份
-   负载均衡：降低单个服务器的压力
-   高可用和故障切换：帮助应用程序避免单点失败
-   升级测试：可以用更高版本的MySQL作为从库

#### 读写分离

读写分离是依赖于主从复制，而主从复制又是为读写分离服务的。

读写分离就是只在mysql主服务器上写，只在mysql从服务器上读。基本原理是让主数据库处理事务性查询，而从数据库处理select查询。数据库复制被用来把事务性查询导致的变更同步到集群中的数据库。
