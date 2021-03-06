## Redis

### Redis工作原理

Redis是一个key-value存储系统，它支持的value类型相对较多，包括string、list、set和zset，这些数据都支持push/pop/add/remove及交并补等操作，而且这些操作都是原子性的，在此基础上，redis支持各种不同方式的排序。

为了保证效率，数据是缓存在内存中的，Redis会周期性的把数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave同步

### Redis持久化-RDB

在Redis运行时，RDB程序将当前内存中的数据库快照保存到磁盘中，当Redis需要重启时，RDB程序会通过重载RDB文件来还原数据库。

#### 保存(rdbSave)

rdbSave负责将内存中的数据库数据以RDB格式保存到磁盘中，如果RDB文件已经存在将会替换已有的RDB文件。保存RDB文件期间会阻塞主进程，这段时间期间将不能处理新的客户端请求，直到保存完成为止。

#### 读取(rdbLoad)

当Redis启动时，会根据配置的持久化模式，决定是否读取RDB文件,并将其中的对象加载到内存中。

### Redis持久化-AOF

以协议文本的方式，将所有对数据库进行的写入命令记录到AOF文件，达到记录数据库状态的目的。

#### AOF的保存

1. 将客户端请求的命令转换为网络协议格式
2. 将协议内容字符串追加到变量server.aof_buf中
3. 当AOF系统达到设定的条件时，会调用aof_fsync(文件描述符号)将数据写入磁盘

#### AOF的读取

1. AOF保存的是数据协议格式的数据，所以只要将AOF中的数据转换为命令，模拟客户端重新执行一遍，就可以还原所有数据库状态。
2. 创建模拟的客户端
3. 读取AOF保存的文本，还原数据为原命令和原参数。然后使用模拟的客户端发出这个命令请求。
4. 继续执行第二步，直到读取完AOF文件

#### AOF重写流程

1. AOF重写完成会向主进程发送一个完成的信号
2. 会将AOF重写缓存中的数据全部写入到文件中
3. 用新的AOF文件，覆盖原有的AOF文件。