# Innodb是如何实现事务的

`Innodb`通过`Buffer Pool`,`Log Buffer`,`Redo Log`, `Undo Log`来实现事务，以一个`update`语句为例：
1. `Innodb`在收到一个`update`语句后，会先根据条件找到数据所在的页，并将该页缓存在`Buffer Pool`中
2. 执行`update`语句，修改`Buffer Pool`中的数据，也就是内存中的数据
3. 针对`update`语句生成一个`RedoLog`对象，并存入`Log Buffer`中
4. 针对`update`语句生成`undolog`日志，用于事务回滚
5. 如果事务提交，那么则把`RedoLog`对象进行持久化，后续还有其他机制将`Buffer Pool`中所修改的数据页持久化到磁盘中
6. 如果事务回滚，则利用`undolog`日志进行回滚