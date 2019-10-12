# 一、 基础知识
### 1. 数据类型

   1. 整形
   
       * TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT 分别使用 8, 16, 24, 32, 64 位存储空间，一般情况下越小的列越好。INT(11) 中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。
       
   2. 浮点
    
       * FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。FLOAT 和 DOUBLE 为浮点类型，DECIMAL 为高精度小数类型。CPU 原生支持浮点运算，但是不支持 DECIMAl 类型的计算，因此 DECIMAL 的计算比浮点类型需要更高的代价。
       
   3. 字符串
   
      * 主要有 CHAR 和 VARCHAR 两种类型，一种是定长的，一种是变长的。在进行存储和检索时，会保留 VARCHAR 末尾的空格，而会删除 CHAR 末尾的空格。

# 二、 数据库架构

1. 存储模块（文件系统）
   
2. 程序模块（管理存储的数据）

3. 影响程序运行的瓶颈是IO

# 三、索引

1. 什么信息能成为索引

   1. 能把信息限定在一定范围内的字段
   
      1. 主键、唯一键、普通键等，只要能让数据具备一定区分性的字段
      
2. 实现索引的数据结构

   * 二叉查找树、B树、主流B+树、Hash及   mysql不支持BitMap
   
3. 密集索引和稀疏索引区别

4. 如何定位并优化慢查询Sql

   1. 根据慢日志定位慢查询sql
   
   2. 使用explain等工具分析sql
   
   3. 修改sql或者尽量让sql走索引
   
# 四、锁和事务   

1. MyISAM与InnoDB关于锁方面的区别

   1. MyISAM默认用表级锁，不支持行级锁，不支持事务，不支持外键；InnoDB默认用行级锁，也支持表级锁，支持事务，支持外键
   
   2. MyISAM引擎
   
      1. 当表进行select的时候，会自动给整张表上一个表级的读锁，并且阻塞其他session对表的更新，即当读锁未被释放，另一个session要对表加写锁，就会被阻塞
      
      2. 当表进行增删改的时候，会自动给整张表上一个表级的写锁
      
      3. 读锁（共享锁）
      
         1. 先上读锁的同时，另一个session也可以对表加读锁，即上共享锁时，依然支持上共享锁，在一个session进行范围查询的时候，另一个session依然能对表进行读操作
         
         2. 但是当一个session给表上了读锁，另一个session要进行写操作就会被阻塞了，尽管要写的行不是正在读的那些行，因为他锁住的是整张表，而不是正在读的行
         
         3. select也可以像写锁一样上一个排它锁：select * from table_name **for update**，这样一个session上读锁时，別的session就上不了读锁
         
      4. 写锁（排它锁）
      
         1. 一个session上写锁的时候，另一个session不能进行读写操作，不能上读写锁，会被阻塞
      
      5. 显式给表加锁
      
         1. lock tables table_name read/write
         
         2. unlock tables
         
      6. 总结
      
         1. 共享锁：上共享锁之后，依然支持上共享锁，不支持上排它锁
         
         2. 排它锁：上排它锁之后，不支持上其他锁
   

2. 数据库事务四大特性(ACID)

   1. 原子性（Atomic）：事务包含的所有操作要么全执行，要么全失败回滚
   
   2. 一致性（Consistency）：事务应确保数据库的状态从一个一致的状态变为另一个一致的状态。AB账户加起来2000，不管A转账多少给B，事务结束后AB加起来还是2000
   
      * 一致状态的含义：数据库的数据应满足完整性约束
      
   3. **隔离性（Isolation）：** 多个事务并发执行时，一个事务的执行不应该影响其他事务的执行
   
   4. 持久性（Durability）：一个事务一旦提交，他对数据库的修改应该永久保存在数据库中

**3. 事务隔离级别以及各级别下的并发访问问题**

   1. 事务并发访问引起的问题
   
      1. **更新丢失--READ UNCOMMITTED：** READ UNCOMMITTED事务隔离级别以上避免(mysql所有事务隔离级别在数据库层面上均可避免)--任何操作都**不会加锁**，数据库一般都不会用
      
      2. **脏读--READ-COMMITTED：** 一个事务读到另一个事务未提交的更新数据，可以在READ-COMMITTED事务隔离级别以上避免--数据的读取都是不加锁的，但是数据的写入、修改和删除是需要加锁的
      
         1. 查看当前隔离级别：select @@tx_isolation
         
         2. 成因：把两个session都设置为最低的事务隔离级别：set session transaction isolation level **read uncommitted**，都开启事务：start transaction，session1把字段从1更新为2但不提交，并查询，session2查询，结果和session1查询的一样都是2，这时session1回滚，字段变回原来的1，但是session2仍按字段是2的值去更新并提交，这就发生了脏读
         
         3. 解决：把两个session都设置为set session transaction isolation level **read committed**，都开启事务：start transaction，session1更新字段为2但不提交，并查询，session2查询，结果字段不是session1更新后的2，而是原来的1，这时session1回滚，字段变回原来的2，这时就算session2仍按字段是1的值去更新并提交，也不是脏读了
         
      3. **不可重复读--REPEATABLE-READ：** 事务A多次读取同一数据，事务B在事务A读取数据的过程中，对数据进行了更新并提交，导致事务A多次读取同一数据的结果不一致，REPEATABLE-READ事务隔离级别以上可避免--读取数据时加行级读锁
      
         1. 成因：把两个session都设置为set session transaction isolation level read committed，都开启事务，从session1的角度看，先读一次字段，值是1，再切换到session2的角度，把字段更新为2并提交，再切换回session1，再读一次字段，发现值是2，和上次的数据不一致，这就发生了不可重复读
         
         2. 解决：把两个session都设置为set session transaction isolation level **repeatable read(默认事务隔离级别)**，都开启事务：start transaction，从session1的角度看，先读一次字段，值是1，再切换到session2的角度，把字段更新为2并提交，再切换回session1，再读一次字段，发现值还是1，即尽管session2做出修改并提交了之后，session1读到的值还是原来未提交的值，这就避免了不可重复读的情况。但是这个时候session1对字段做出修改，是在字段最新的值2之上进行修改的，这也不会导致数据不一致的情况
         
      4. **幻读--SERIALIZABLE：** 事务A读取与搜索条件相匹配的若干行，事务B以插入或删除行等方式来修改事务A的结果集，导致事务A看起来像出现幻觉一样，SERIALIZABLE事务隔离级别可避免--读加共享锁，写加排他锁，读写互斥
      
         1. 成因：把两个session都设置为set session transaction isolation level read committed，都开启事务，session1使用当前读的方式查询一下，并加了一个共享锁，查出来20条记录，紧接着session2添加一条数据，发现在这个级别下，session2可以操作成功，提交，回到session1，给表的某个字段全部更新为'a'，这时发现，竟然有21行受影响，本来是对20条数据进行更新，现在变成了21条，这就出现了幻读
         
            * ps：因为mysql在repeatable read级别下也可以避免幻读，所以要用read committed进行测试
         
         2. 解决：把两个session都设置为set session transaction isolation level **serializable(最高事务隔离级别)**，都开启事务：start transaction，重复以上操作，从session1的角度查询一下，查出来20条，再切换到session2，添加一条数据，这时发现被阻塞了，需要等session1提交或回滚之后才能执行插入操作，这是因为serializable级别下，所有操作都会加锁，这就避免了幻读的发生
         
      5. 归纳 
         1. 不可重复读和幻读的区别：
            1. 在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别 ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。
            
            2. 不可重复读的和幻读很容易混淆，不可重复读侧重于**修改**，幻读侧重于**新增或删除**。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表。
         
         2. 事务隔离级别越高，安全性更高，但是串行化也更高，应该根据具体业务需要选择合适的事务隔离级别
         
         3. 默认隔离级别
         
            1. mysql：repeatable read
            
            2. Oracle：read committed
            
         4. 归纳一下四种事务隔离机制是如何用锁解决各种并发访问的问题的？
         

4. InnoDB可重复读隔离级别下如何避免幻读

5. RC、RR级别下的InnoDB的非阻塞读如何实现

# 五、优化   
