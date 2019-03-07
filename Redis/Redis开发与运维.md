# Redis开发与运维读书笔记

## 第五章 持久化

> Redis支持RDB和AOF两种持久化机制

### 5.1 RDB

> RDB持久化是把当前进程数据生成快照保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发。

#### 5.1.1 触发机制

> 手动触发分别对应save和bgsave命令, save命令现已废弃

1. save命令：阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。

2. bgsave命令：Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。

> Redis内部还存在自动触发RDB的持久化机制，例如以下场景： 

1. 使用save相关配置，如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。

2. 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点，更多细节见6.3节介绍的复制原理。

3. 执行debug reload命令重新加载Redis时，也会自动触发save操作。

4. 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave。

#### 5.1.2 bgsave流程说明

![5-1](https://github.com/ossaw/notes/blob/master/Pictures/redis/Redis-05-01.png)

1. 执行bgsave命令，Redis父进程判断当前是否存在正在执行的子进程，如RDB/AOF子进程，如果存在bgsave命令直接返回。 

2. 父进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通过info stats命令查看latest_fork_usec选项，可以获取最近一个fork操作的耗时，单位为微秒。

3. 父进程fork完成后，bgsave命令返回“Background saving started”信息并不再阻塞父进程，可以继续响应其他命令。

4. 子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换。执行lastsave命令可以获取最后一次生成RDB的时间，对应info统计的rdb_last_save_time选项。

5. 进程发送信号给父进程表示完成，父进程更新统计信息，具体见info Persistence下的rdb_*相关选项。

#### 5.1.3 RDB文件的处理

RDB文件保存在dir配置指定的目录下，文件名通过dbfilename配置指定。可以通过执行config set dir{newDir}和config set dbfilename{newFileName}运行期动态执行，当下次运行时RDB文件会保存到新目录, 磁盘写满的情况下可动态执行此命令,在执行bgsave, 同样适用于AOF.

文件压缩: Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启，可以通过参数config set rdbcompression{yes|no}动态修改。

Redis加载损坏RDB文件时,拒绝启动, 可尝试redis-check-dump工具检测获取错误报告

#### 5.1.4 RDB的优缺点

> 优点

1. 恢复数据速度优于AOF方式

2. 适用于全量复制, 灾难恢复等场景

> 缺点

1. 无法做到试试持久化, 在下一次持久化之前宕机会丢失未持久化数据

2. bgsave命令执行成本过高

3. Redis版本演进过程中存在二进制文件版本兼容问题

### 5.2 AOF

AOF（append only file）持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。理解掌握好AOF持久化机制对我们兼顾数据安全性和性能非常有帮助

#### 5.2.1 使用AOF

AOF配置：appendonly yes，默认不开启。AOF文件名通过appendfilename配置设置，默认文件名是appendonly.aof。保存路径同RDB持久化方式一致，通过dir配置指定。

AOF的工作流程操作：命令写入（append）、文件同步（sync）、文件重写（rewrite）、重启加载（load）

1. 所有的写入命令会追加到aof_buf（缓冲区）中。

2. AOF缓冲区根据对应的策略向硬盘做同步操作。

3. 随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩的目的。

4. 当Redis服务器重启时，可以加载AOF文件进行数据恢复。了解AOF工作流程之后，下面针对每个步骤做详细介绍。

![5-2](https://github.com/ossaw/notes/blob/master/Pictures/redis/Redis-05-01.png)

#### 5.2.2 命令写入

> AOF命令写入的内容直接是文本协议格式, 具有很好的兼容性, 可读性

问题: AOF为什么把命令追加到aof_buf中？

Redis使用单线程响应命令，如果每次写AOF文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载。先写入缓冲区aof_buf中，还有另一个好处，Redis可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡。

#### 5.2.3 文件同步

> AOF缓冲区同步策略, 由appendfsync控制, 建议配置everysec.

1. always: 命令写入aof_buf调用fsync同步到aof文件, fsync完成后线程返回, 增加系统负担

2. everysec: 命令写入aof_buf后调用系统write操作, write完成后线程返回, fsync操作有专门的线程每秒执行一次

3. no: 命令写入aof_buf后调用系统write操作, 不做fsync操作, 刷硬盘操作有操作系统完成, 通常周期最长30秒, 会丢失30秒左右的数据

> 系统write和fsync操作 

1. write操作会触发延迟写（delayed write）机制。Linux在内核提供页缓冲区用来提高硬盘IO性能。write操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，例如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失。

2. fsync针对单个文件操作（比如AOF文件），做强制硬盘同步，fsync将阻塞直到写入硬盘完成后返回，保证了数据持久化。

#### 5.2.4 重写机制

> AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程。 

重写后的AOF文件为什么可以变小？

1. 进程内已经超时的数据不再写入文件。

2. 旧的AOF文件含有无效命令，如del key1、hdel key2、srem keys、set a111、set a222等。重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。

3. 多条写命令可以合并为一个，如：lpush list a、lpush list b、lpush listc可以转化为：lpush list a b c。为了防止单条命令过大造成客户端缓冲区溢出，对于list、set、hash、zset等类型操作，以64个元素为界拆分为多条。

4. AOF重写降低了文件占用空间, 更小的AOF文件可以更快地被Redis加载。 

AOF重写过程可以手动触发和自动触发：

* 手动触发：直接调用bgrewriteaof命令。

* 自动触发：根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数确定自动触发时机。

	* auto-aof-rewrite-min-size：表示运行AOF重写时文件最小体积，默认为64MB。

	* auto-aof-rewrite-percentage：代表当前AOF文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值。

重写流程

![5-3](https://github.com/ossaw/notes/blob/master/Pictures/redis/Redis-05-01.png)
