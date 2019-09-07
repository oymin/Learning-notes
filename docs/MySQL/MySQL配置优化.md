# MySQL 配置优化

定义：数据库是基于操作系统的，目前大多数 MySQL 都是安装在 linux 系统之上，所以对于操作系统的一些参数配置也会影响到 MySQL 的性能，下面就列出一些常用的系统配置。

优化包括`操作系统的优化` 及 `MySQL 的优化`

## 1. 操作系统的优化

### 1.1 网络配置优化

网络方面的配置，要修改 `/etc/sysctl.conf`：

1、增加 tcp 支持的队列数

```yml
net.ipv4.tcp_max_syn_backlog = 65535
```

2、减少断开连接时，资源回收(tcp 有连接状态)

```yml
net.ipv4.tcp_max_tw_buckets = 8000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 10
```

说明： TCP 是有连接状态，通过 netstat 查看连接状态，经常会看到 timeout 状态或者 timewait 状态连接，为了加快 timewait 状态的连接回收，就需要调整上面的四个参数，保持 TCP 连接数在一个适当的状态。

### 2. 打开文件数的限制

打开文件数的限制，可以使用 `ulimit –a` 查看目录的各个限制，可以修改 `/etc/security/limits.conf` 文件，增加以下内容以修改打开文件数量的限制（永久生效）

```yml
*Soft nofile 65535
*Hard nofile 65535

# 参数说明
*       表示对所有用户有效
soft    当前系统生效的设置（同一资源soft不能高于hard配置）
hard    系统中能设置的最大值
nofile  限制资源打开文件的最大数目
65535   最大数目值
```

如果一次有效，就要使用 `ulimit –n 65535` 即可。（默认情况是1024）

除此之外最好在 MySQL 服务器上关闭 iptables ，selinux 等防火墙软件。

## 2. MySQL配置参数优化 - MySQL配置文件优化

### 2.1 MySQL 配置文件修改

MySQL 可以通过启动时指定参数和使用配置文件两种方法进行配置，在大多数情况下配置文件位于 `/etc/my.cnf` 或者是 `/etc/mysql/my.cnf` 在 Windows 系统配置文件可以是位于 `C://windows//my.ini` 文件。

MySQL 查找配置文件的顺序可以通过以下方法获得

```bash
mysqld --help --verbose | grep -A 1 'Default options'

# 加载配置文件的顺序
/etc/my.cnf
/etc/mysql/my.cnf
/usr/etc/my.cnf
~/.my.cnf
```

注意：如果存在多个位置存在配置文件，则后面的会覆盖前面的。

### 2.2 MySQL 配置文件 - 常用参数说明

常用参数表：

| 参数 | 说明 |
| ---- | ---- |
| max_connections | MySQL的最大连接数 |
| max_used_connections | 响应的连接数 |
| back_log | MySQL能暂存的连接数量 |
| interactive_timeout | 一个交互连接在被服务器在关闭前等待行动的秒数 |

### max_connections

MySQL 的最大连接数，增加该值增加 mysqld 要求的文件描述符的数量。如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于 MySQL 会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。

数值过小会经常出现 `ERROR 1040: Too many connections` 错误，可以过 `'conn%'` 通配符查看当前状态的连接数量，以定夺该值的大小。

```sql
show variables like 'max_connections' -- 最大连接数

show status like 'max_used_connections' --响应的连接数
```

说明：理想值设置为多大才合适？

`max_used_connections / max_connections * 100% （理想值≈ 85%）`

如果 `max_used_connections` 跟 `max_connections` 相同那么就是 `max_connections` 设置过低或者超过服务器负载上限了，低于10%则设置过大。

#### back_log

MySQL 能暂存的连接数量。当主要 MySQL 线程在一个很短时间内得到非常多的连接请求，这就起作用。如果 MySQL 的连接数据达到 max_connections 时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即 back_log，如果等待连接的数量超过 back_log，将不被授予连接资源。

back_log 值指出在 MySQL 暂时停止回答新请求之前的短时间内有多少个请求可以被存在堆栈中。只有如果期望在一个短时间内有很多连接，你需要增加它，换句话说，这值对到来的 TCP/IP 连接的侦听队列的大小。

当观察你主机进程列表（mysql> show full processlist），发现大量

```sql
| 264084 | unauthenticated user | xxx.xxx.xxx.xxx | NULL | Connect | NULL | login | NULL |
```

的待连接进程时，就要加大 back_log 的值了。默认数值是50，可调优为128，对于 Linux 系统设置范围为小于512的整数。

#### interactive_timeout

一个交互连接在被服务器在关闭前等待行动的秒数。一个交互的客户被定义为对`mysql_real_connect()` 使用 `CLIENT_INTERACTIVE` 选项的客户。

默认数值是28800，可调优为7200。

## 3. 缓冲区变量

### 全局缓冲

参数表：

| 参数 | 说明 |
| ---- | ---- |
| sort_buffer_size        | 每个线程排序缓存区的大小 |
| join_buffer_size        | 每个线程连接缓冲区的大小 |
| read_buffer_size        | 每个线程读缓冲区的大小(要是4k的倍数) |
| read_rnd_buffer_size    | 每个线程索引缓冲区的大小 |
| Innodb_buffer_pool_size | 定义 Innodb 缓存池的大小=[总内存-(每个线程所需的内存*连接数)-系统保留内存] |
| key_buffer_size         | 指定索引缓冲区的大小 (用于 myisam 引擎) |
| query_cache_size        | 使用查询缓冲 |
| record_buffer_size      | 记录缓冲区大小 |
| table_cache             | 表高速缓存的大小 |
| max_heap_table_size     | 最大堆表大小 |
| tmp_table_size          | 临时表大小 |
| thread_cache_size       | 线程池缓存大小 |
| thread_concurrency      | 线程并发 |
| wait_timeout            | 一个请求的最大连接时间 |

#### key_buffer_size

key_buffer_size 指定索引缓冲区的大小，它决定索引处理的速度，尤其是索引读的速度。通过检查状态值 Key_read_requests 和 Key_reads，可以知道 key_buffer_size 设置是否合理。比例 key_reads / key_read_requests 应该尽可能的低，至少是1:100，1:1000更好。

```sql
show variables like 'key_buffer_size';

+-----------------+---------+
| Variable_name   | Value   |
+-----------------+---------+
| key_buffer_size | 8388608 |
+-----------------+---------+
```




















