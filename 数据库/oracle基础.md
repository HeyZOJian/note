## 数据库和文件系统的区别

1. 数据库有恢复机制
2. 数据库有事务功能，能保证数据的一致性
3. 数据库有约束功能，能保证数据的完整性
4. 数据库有SQL语句，能高效地对数据库进行操作

## oracle数据库的组成

### 1. 物理结构

1. Data File：包含全部数据库数据。
2. Control File：记录数据库的物理结构。数据库名，  数据库数据文件和日志文件的名字和位置，  数据库建立日期。 
3. UNDO File：主要用于事务回滚操作。
4. Online redo log files：收集数据库日志。日志的主要功能是记录对数据所作的修改，所以对数据库作的全部修改是记录在日志中。
5. Parameter File：参数文件记录了Oracle数据库的基本参数信息，主要包括数据库名、控制文件所在路径、进程等
6. Password File：保存超级管理的密码，其他密码存储在data file中。
7. Archive log File：是redo log file的拷贝

### 2. 内存结构

1. PGA：是一块私有内存区域，仅供当前发起用户使用。用户登录后的session和权限信息会保存在这里。
2. SGA：
   1. Shared pool：用于缓存被执行的SQL语句和被使用的数据定义
   2. Database buffer cache：在数据高速缓冲区中存放着Oracle系统使用过的数据块（即用户的高速缓冲区），当把数据写入数据库时，它以数据块为单位进行读写，当数据高速缓冲区填满时，则系统自动去掉一些不常被用户访问的数据。如果用户要查的数据不在数据高速缓冲区时，Oracle自动从磁盘中去读取。
   3. Redo log buffer cache：缓存对于数据块的所有修改。



## Background Processes

1. PMON:Cleans up after failed processs.
2. SMON：负责多个系统级的清理工作
   1. 执行实例恢复。 
   2. 恢复因文件读或表空间offline错误导致终断的事务。 
   3. 清理临时的未使用的段。 
   4. 通过表空间的字典管理合并连续空闲的区间。
3. CKPT：触发时将脏数据写入数据文件，保证内存和数据文件的同步。降低实际崩溃时恢复的时间。



## Read Consistency

1. SCN  (System change  number)： 这个数字相当于oracle数据库的一个自己 的时钟，用于记录数据改变的时刻。
2. 数据库进行读操作时，当读到大于当前操作的SCN时，先回滚到小于等于当前SCN的数据

## Recovery

1. Rolling forward to recover data that has not been recorded in the datafiles, yet has been recorded in the online redo log, including the contents of rollback segments. This is called *cache recovery*.（向前回滚来恢复尚未被记录到datafiles里，但是记录到online redo log里的数据，包括回滚的segment的内容，这叫缓存恢复）
2. Opening the database. Instead of waiting for all transactions to be rolled back before making the database available, Oracle allows the database to be opened as soon as cache recovery is complete. Any data that is not locked by unrecovered transactions is immediately available.（打开数据库。Oracle允许在缓存恢复完成后打开数据库，而不是等待所有事务在数据库可用之前回滚。未被未恢复的事务锁定的任何数据都立即可用。）
3. Marking all transactions system-wide that were active at the time of failure as DEAD and marking the rollback segments containing these transactions as PARTLY AVAILABLE.（将所有在失效时活动的事务标记为已死亡，并将包含这些事务的回滚段标记为部分可用。）
4. Rolling back dead transactions as part of SMON recovery. This is called *transaction recovery*.（将死区事务作为SMON恢复的一部分进行回滚。这叫做事务恢复。）
5. Resolving any pending distributed transactions undergoing a two-phase commit at the time of the instance failure.（解决在实例失败时进行两阶段提交的未决分布式事务。）
6. As new transactions encounter rows locked by dead transactions, they can automatically roll back the dead transaction to release the locks. If you are using Fast-Start Recovery, just the data block is immediately rolled back, as opposed to the entire transaction.（当新事务遇到死事务锁定的行时，它们可以自动回滚死事务释放锁。如果使用快速启动恢复，则数据块立即回滚，而不是整个事务。）

## SQL Statement Processing Step

1. Parse the Statement
2. Bind and Parallelize
3. Execute
4. Return data

 

## SQL Transformer



## Cost-Based Optimizer



## Join Methods

1. Nested Loops Join：
   1. 工作方式：是从一张表中读取数据，访问另一张表（通常是索引）来做匹配，nested loops适用的场合是当一个关联表比较小的时候，效率会更高。
   2. 适用于外层循环小，内存循环条件列有序
   3. 对于被连接的数据子集较小的情况，嵌套循环连接是个较好的选择
   4. 一般用在连接的表中有索引，并且索引选择性较好的时候.
2. Sort Merge Join：
   1. 工作方式：是先将关联表的关联列各自做排序，然后从各自的排序表中抽取数据，到另一个排序表中做匹配， 通常来讲，能够使用merge join的地方，hash join都可以发挥更好的性能。
   2. 适用于输入两端都有序
3. Hash Join：
   1. 工作方式：是将一个表（通常是小一点的那个表）做hash运算，将列数据存储到hash列表中，从另一个表中抽取记录，做hash运算，到hash 列表中找到相应的值，做匹配。
   2. 数据量大，且没有索引
   3. 做大数据集连接时的常用方式
4. Cartesian Join：尽量避免



## Index

1. B-tree
2. B+-tree
3. Full Table Scan
4. Index Scan
5. Index Type
   1. Unique and nonunique indexes
   2. Composite indexes
6. When to use indexes?
   1. Keys frequently used in search or query expressions
   2. Keys used to join tables
   3. High-selectivity keys
   4. Foreign keys



## Execution Plan

1. The execution plan of a SQL statement is composed of small building blocks called row sources for serial execution plans.一条查询语句在Oracle中的执行过程或访问路径的描述
2. The combination of row sources for a statement is called the execution plan.
3. By using parent-child relationships, the execution plan can be displayed in a tree-like structure (text or graphical).
4. 常用列字段解释：
   1. 基数（Rows）：Oracle估计的当前操作的返回结果集行数
   2. 字节（Bytes）：执行该步骤后返回的字节数
   3. 耗费（COST）、CPU耗费：Oracle估计的该步骤的执行成本，用于说明SQL执行的代价，理论上越小越好（该值可能与实际有出入）
   4. 时间（Time）：Oracle估计的当前操作所需的时间
5. 执行顺序：根据缩进来判断，类似树的后序遍历
6. 