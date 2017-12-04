Mysql数据库按/etc/my.cnf-->/etc/mysql/my.cnf-->~/.my.cnf的顺序来读取配置文件（通过mysql --help|grep 'my.cnf'查看）


OLTP是什么？（原文：一个新的OLTP项目不适用MySQL InnoDB存储引擎将是多么的愚蠢）

### 后台线程
#### 1、MasterThread
将缓冲池中的数据异步刷新到磁盘，保证数据一致性。
	脏页的刷新（1.2.x版本之前）
	合并插入缓冲（Insert Buffer）
	Undo页的回收
#### 2、IO Thread
Innodb大量使用Async IO处理写IO请求
IO Thread负责这些IO请求的回调（Callback）处理
innodb_read_io_threads 设置读IO的数量
innodb_write_io_threads 设置写IO的数量
1个insert buffer thread，1个log thread，4个read thread，4个write thread
读线程的id总是小于写线程
#### 3、Purge Thread
事务提交后undo log就没有用了（事务的原子性），Purge Thread回收已经分配并使用的undo页。
Purge Thread有多个的原因：Purge Thread需要离散地读取undo页，这样能够进一步利用磁盘的随机读取性能。
#### 4、Page Cleaner Thread（1.2.x版本引入的，减轻Master Thread的工作和对于用户查询线程的阻塞）
脏页的刷新

### 内存
#### 1、缓冲池（可以有多个缓冲池）
##### 1）背景
InnoDB存储引擎基于磁盘存储，其中记录按照页的方式管理。（Disk-base Database）
由于磁盘速度与CPU速度差距极大，所以Disk-base Database一般采用缓冲池技术提高数据库整体性能。
缓冲池在内存中，通过内存速度弥补磁盘速度对数据库性能的影响。
##### 2）在数据库中读取页的操作
将从磁盘中读取到的页放入缓冲池中（将页Fix在缓冲池中）
下一次再读取相同的页，先查看该页是否在缓冲池中。是则命中，直接读取；否则读取磁盘上的页。
##### 3）数据库中页的修改
先修改缓冲池中的页，然后以一定频率刷新回磁盘。（不是每次页修改时触发，而是通过Checkpoint机制--为了提高数据库整体性能）

tip：为了使用更多的内存，应使用64位操作系统。
##### 4）缓冲池中的数据页类型
索引页
数据页
undo页
插入缓冲（insert buffer）
自适应hash索引（adatptive hash index）
InnoDB存储的锁信息（lock info）
数据字典信息（data dictionary）

tip:索引页和数据页占很大部分。
