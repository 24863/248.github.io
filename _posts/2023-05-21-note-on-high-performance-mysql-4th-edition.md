---
title: "课堂练习"
layout: post
category: note
tags: [Liquid Filters]

excerpt: "All of the standard Liquid filters are supported (see below).

To make common tasks easier, Jekyll even adds a few handy filters of its own, all of which you can find on this page. You can also create your own filters using plugins."
---



# 监控

商业选项: SolarWinds.

开源选项:

* Percona 的 PMM.
* 将慢查询和 performance schema 输出到集中位置, 使用 pt-query-digest 分析

## 可用性

Threads_running 可作为关键指标, 它跟踪当前正运行的查询数量. 如果它快速增长, 说明查询不够快;
如果它超过了 cpu 核数, 可能表明服务器正处于不稳定状态. 它与 max_connections 差值, 可以说明
工作是否过载.

## 查询延迟

## 报错

* lock wait timeout: 若它增加, 可能是主节点上行级锁争用在扩大, 即事务不断重试但仍失败, 这可
  能是无法写入的前兆.
* aborted connections: 客户端与数据库实例间某个访问层出现了问题.
* Connection_errors_xxx
* 只读模式
* too many connections: 连接数超标.

## 连接数

* threads_running/threads_connected 比值低, 可能说明应用打开了很多未使用的连接

## 复制延迟

## I/O 使用率

## 自增键空间

```sql
SELECT
  t.TABLE_SCHEMA AS `schema`,
  t.TABLE_NAME AS `table`,
  t.AUTO_INCREMENT AS `auto_increment`,
  c.DATA_TYPE AS `pk_type`,
  (
    t.AUTO_INCREMENT / (
      CASE DATA_TYPE WHEN 'tinyint' THEN IF(
        COLUMN_TYPE LIKE '%unsigned', 255,
        127
      ) WHEN 'smallint' THEN IF(
        COLUMN_TYPE LIKE '%unsigned', 65535,
        32767
      ) WHEN 'mediumint' THEN IF(
        COLUMN_TYPE LIKE '%unsigned', 16777215,
        8388607
      ) WHEN 'int' THEN IF(
        COLUMN_TYPE LIKE '%unsigned', 4294967295,
        2147483647
      ) WHEN 'bigint' THEN IF(
        COLUMN_TYPE LIKE '%unsigned', 18446744073709551615,
        9223372036854775807
      ) END / 100
    )
  ) AS `max_value`
FROM
  information_schema.TABLES t
  INNER JOIN information_schema.COLUMNS c ON t.TABLE_SCHEMA = c.TABLE_SCHEMA
  AND t.TABLE_NAME = c.TABLE_NAME
WHERE
  t.AUTO_INCREMENT IS NOT NULL
  AND c.COLUMN_KEY = 'PRI'
  AND c.DATA_TYPE LIKE '%int';
```

# Performance Schema

Performance Schema (以下简写为 PS).提供了服务器内部运行的操作上的底层指标.

"程序插桩 (instrument)" 指在 mysql 代码中插入探测代码. setup_instruments 表包含了支持的
插桩列表. 所有插桩的名称由 / 分隔的部件组成, 如 `statement/sql/select`, 最左代表桩类型, 其
余则是从通用到特定的子系统.

"消费者表 (consumer)" 指是插桩发送信息的目的地, 测量结果存在 performance schema 库的多个表
中.

sys Schema 是为了方便使用 performance schema 的, 它全部基于 performance_schema 上的视
图和存储例程.

PS 将数据存在 PERFORMANCE_SCHEMA 引擎的表中, 这个引擎将数据存在内存中. 可通过启动变量来调
整使用的内存大小.

使用 PS 有如下局限性:

* 如果组件没在支持插桩, 也就没法用 PS 监控它了.
* 它只能在开启某个桩之后才开始收集数据, 不能用它查看开启之前的信息.
* 它很难释放自己已经占用了的内存.

早期版本的 PS 实现不够理想, 资源消耗过高, 一般建议关闭; 对于新版建议启用并按需动态地启用插桩和
消费者表, 它可以解决查询性能, 锁定, I/O, 错误等问题.

## 启用

将配置变量 performance_schema 设为 ON 即可启用 PS.

有三个方法启用插桩:

* 将 setup_instruments 表中相应插桩的 ENABLED 列改为 "YES".
* 调用 sys schema 中的 sys.ps_setup_enable_instrument 存储过程.
* 使用 performance-schema-instrument 启动参数.

同理, 也有三个方启用消费者表:

* setup_consumers 表
* sys.ps_setup_enable_consumer 或 sys.ps_setup_disable_consumer
* peprformance-schema-consumer 启动参数.

对于特定 "对象" (包括: EVENT, FUNCTION, PROCEDURE, ENABLE, TRIGGER) 的监控配置可通过
setup_objects 表完成;

对于*后台线程*的监控配置可通过 setup_threads 表完成; 对于*用户线程*则可通过 setup_actors
表.

## 优化 SQL

相关桩:

    Instrument class        Description
    ---------------------------------------------------------------------------
    statement/sql           SQL statements, such as SELECT or CREATE TABLE
    statement/sp            Stored procedures control
    statement/scheduler     Event scheduler
    statement/com           Commands, such as quit, KILL, DROP DATABASE, or Binlog Dump.
                            Some are not available for users and are called
                            by the mysqld process itself.
    statement/abstract      Class of four commands: clone, Query, new_packet, and relay_log

对于常规 SQL 语句, 要关注以下输出列:

    Column                      Desc
    ------------------------------------------------------------------------------------
    CREATED_TMP_DISK_TABLES     The query created this number of disk-based temporary
                                tables. You have two options to resolve this issue:
                                optimize the query or increase maximum size for in-
                                memory temporary tables.
    CREATED_TMP_TABLES          The query created this number of memory-based
                                temporary tables. Use of in-memory temporary tables is
                                not bad per se. However, if the underlying table grows,
                                they may be converted into disk-based tables. It is good
                                to be prepared for such situations in advance.
    SELECT_FULL_JOIN            The JOIN performed a full table scan because there is
                                no good index to resolve the query otherwise. You need
                                to reconsider your indexes unless the table is very
                                small.
    SELECT_FULL_RANGE_JOIN      If the JOIN used a range search of the referenced
                                table.
    SELECT_RANGE                If the JOIN used a range search to resolve rows in the
                                first table. This is usually not a big issue.
    SELECT_RANGE_CHECK          If the JOIN is without indexes, which checks for keys
                                after each row. This is a very bad symptom, and you
                                need to reconsider your table indexes if this value is
                                greater than zero.
    SELECT_SCAN                 If the JOIN did a full scan of the first table. This is an
                                issue if the table is large.
    SORT_MERGE_PASSES           The number of merge passes that the sort has to
                                perform. If the value is greater than zero and the query
                                performance is slow, you may need to increase
                                sort_buffer_size.
    SORT_RANGE                  If the sort was done using ranges.
    SORT_ROWS                   The number of sorted rows. Compare with the value of
                                the returned rows. If the number of sorted rows is
                                higher, you may need to optimize your query.
    SORT_SCAN                   If the sort was done by scanning a table. This is a very
                                bad sign unless you purposely select all rows from the
                                table without using an index.
    NO_INDEX_USED               No index was used to resolve the query.
    NO_GOOD_INDEX_USED          Index used to resolve the query is not the best. You
                                need to reconsider your indexes if this value is greater
                                than zero.

可以使用上述列与 0 比较, 筛出想要优化的语句, 如

    # 没有使用合适索引
    WHERE NO_INDEX_USED > 0 OR NO_GOOD_INDEX_USED > 0;
    # 创建了临时表
    WHERE CREATED_TMP_TABLES > 0 OR CREATED_TMP_DISK_TABLES > 0;
    # 返回了错误
    WHERE ERRORS > 0;
    # 时间超过 5s
    WHERE TIMER_WAIT > 5000000000;

sys schema 相关视图:

    statement_analysis
    statement_with_(errors_or_warnings|full_table_scans)
    statement_with_(runtimes_in_95th_percentile|sorting|temp_tables)

## 预处理语句
## 存储例程
## 语句剖析

启用 'stage/%' 模式的桩, 然后查看 events_stages_* 表, 可以用来找出 "查询执行的哪个阶段花费
了非常长时间".

注意, 只有通信服务模块支持, 引擎不支持剖析.

## 读写性能

读写比例:

```sql
SELECT EVENT_NAME, COUNT(EVENT_NAME)
FROM events_statements_history_long
GROUP BY EVENT_NAME;
```

语句延迟:

```sql
SELECT EVENT_NAME, COUNT(EVENT_NAME),
SUM(LOCK_TIME/1000000) AS latency_ms
FROM events_statements_history
GROUP BY EVENT_NAME ORDER BY latency_ms DESC;
```

## 元数据锁

启用 wait/lock/meta-data/sql/mdl 桩.

## 内存使用

memory 类的桩.

## 变量

三个相关表: global_variables, session_variables, variables_by_thread.

## 常见错误

表: events_errors_summary_global_by_error.

# OS 和硬件

最常见的瓶颈是 CPU 耗尽. I/O 饱和也会发生, 但比 CPU 耗尽少得多. 配置大内存的主要原因不是为了
在内存中保存数据, 而是为了避免磁盘 I/O.

## RAID

RAID 代替不了备份.

应明智的使用 RAID 缓存: 用于读操作通常是浪费 (因为 linux 和数据库服务器都有更大的缓存), 通常
将其用于写操作. 但除非有备用电池单元 (BBU), 否则不应启用写缓存.

## 网络配置

大多数时候, 默认值就行, 只在发生异常情况时才更改它们:

DNS 过程可能很慢, 建议启用 skip_name_resolve.

可能需要将本地端口范围以及请求队列调大:

    $ echo 1024 65535 > /proc/sys/net/ipv4/ip_local_port_range
    $ echo 4096 > /proc/sys/net/ipv4/tcp_max_sync_backlog

## 文件系统

大多数 FS 表现相近, *单纯*为性能而寻找 FS 实际是一种干扰.

最好使用日志型 FS, 如 ext4, XFS 或 ZFS, 否则系统崩溃后检查 FS 可能需要很长时间. ext4 在
特定内核版本中存在性能问题, 但它是一个可以接受的选项. 通常建议使用 XFS.

对于 ext4, 日志级别可设置为 3 (在 /etc/fstab 挂载选项中设置); 最好禁用记录访问时间:

    /dev/sda2 /usr/lib/mysql ext3 noatime,nodiratime,data=writeback 0 1

对于磁盘队列调度器, noop 和 deadline 的差别很小, 最重要的是**不要使用 CFQ**, 它会导致非常糟
的响应时间.

## 内存和交换

确保更快内存访问的最佳方法之一, 仍然是**用 tcmalloc/jemalloc 替换内置的 glibc**.

建议完全**不使用交换空间**. 并将 swappiness 设置为 0:

    $ echo 0 > /proc/sys/vm/swappiness

大多数时候都希望 vmstat 的 si,so 值为 0, 绝对不希望看到它超过 10.

强烈建议识别所有关键进程 (如 ssh, mysql), 主动调整 OOM Killer 分值, 防止它们被首先终止.

## 工具

系统自带的:

* vmstat
* iostat
* mpstat 查看 CPU 统计数据
* perf

第三方工具:

* dstat, collectl
* blktrace 检查 I/O 使用情况
* Percona 的 pt-diskstats 是 iostat 的替代品

# 服务器设置

为 mysql 创建合适的配置文件是一不迂回的过程.

建议:

* 仅专注于优化*峰值*工作负载, 在 "足够好" 的时候就停止优化.
* 仅正确的配置*基本配置* (多数情况下只有少数设置是重要的), 将更多时间放在 schema 优化, 索
  引和查询设计上.
* 应该根据**工作负载, 数据和应用需求**来配置, 并且每次更改了查询和 schema 后可能需要重新评估配置.
* 应该使用版本控制跟踪配置变更.

不建议:

* 永远不应该盲目相信网络上的所谓的最佳配置, 或使用别人的调优脚本.
* "按比率调优", 如 "InnoDB 缓冲池命中率应高于某个百分比". 这种建议通常是错误的.

## 配置文件

找到配置文件:

    $ which mysqld
    /usr/sbin/mysqld
    $ /usr/sbin/mysqld --verbose --help | grep -A 1 'Default options'
    /eetc/mysql/my.cnf ~/.my.cnf /usr/etc/my.cnff

配置文件采用 INI 格式. 配置的作用域有以下三种, 每个具体配置项可能有多种作用域:

* 服务器 (全局)
* 会话 (每连接)
* 基于每个对象的

除了配置文件, 很多变量也可以用 SET 动态更改:

    SET sort_buffer_size = <value>;
    SET GLOBAL sort_buffer_size = <value>;
    SET @@sort_buffer_size := <value>;
    SET @@session.sort_buffer_size := <value>;
    SET @@global.sort_buffer_size := <value>;

一般来说, 使用 SET 的更改重启后失效, 但 v8 引入了 "持久化系统变量" 功能, 来进行持久化:

    SET PERSIST ...

在每次更改后, 应检查 SHOW GLOBAL VARIABLES 确保其生效了.

