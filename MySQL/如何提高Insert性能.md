# 如何提高Insert性能

1. **合并多条Insert为一条**  
   例如：`insert into t values(a,b,c),(d,e,f);`  
   主要是因为多条insert合并后日志量（MySQL的binlog和innodb的事务日志)减少了，降低日志刷盘的数据量和频率，从而提高效率。通过合并SQL语句，同时也能减少SQL语句解析的次数，减少网络传输的IO。
2. **修改参数 `bulk_insert_buffer_size`,调大批量插入的缓存。**
3. 设置 `innodb_flush_log_at_tex_commit = 0`  
   相对于 `innodb_flush_log_at_tex_commit = 1`可以十分明显的提升导入速度。  
   `innodb_flush_log_at_tex_commit`参数的解释如下：
   - 0: `log buffer`中的数据将以每秒一次的频率写入到`log file`中，且同时会进行文件系统到磁盘的同步操作，但是每个事务的`commit`并不会触发任何`log buffer` 到`log file` 的刷新或者文件系统到磁盘的刷新操作。
   - 1：在每次事务提交的时候将`log buffer` 中的数据都写入到`log file`,同时也会触发文件系统到磁盘的同步。
   - 2：事务提交会触发`log buffer` 到`log file`的刷新，但并不会触发磁盘文件系统到磁盘的同步。此外，每秒会有一次文件系统到磁盘同步操作
4. **手动使用事务**  
   因为MySQL默认是autocommit的，这样每插入一条数据，都会进行一次commit。所以，为了减少创建事务的消耗，我们可以用手动使用事务。  
   即`START TRANSACTION;`,一般一千条insert提交一次