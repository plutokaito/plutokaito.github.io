---
title: "缓存 (番外)- 记一次 Redis 的使用空间的优化"
date: 2021-09-11
tags: ["缓存", "redis"]
description : "该篇文章主要是记录一次 Redis 的内存空间的优化"
---
前段时间公司某台非核心功能的 Redis 实例一直跑满内存。影响了部分功能的正常使用。所以这是一个算是比较重要的优化吧。在这里做一次记录。

这次我主要分为几个步骤去解决。
## 确定 redis 的使用版本和过期策略是什么。
### 确认版本
使用命令 `redis-cli -h ${redis-host}` 链接到 Redis 服务上，进入命令行，然后使用命令 `info server` 查看当前 Redis 版本，因为早些版本的 redis 是不支持像 `memory` 这样命令的。 memory 命令的支持是从 `4.0.0` 版本以后的。 `debug` 命令有些服务商也是禁用的。所以有时候也会出现 `(error) ERR unknown command 'debug'` 这种报错。由于我们使用的是 云服务商 `3.2.10` 版本的 Redis，以上命令都不支持。在需要优化的 Redis 上使用 `info server`， 显示如下：
```shell
> info server
# Server
redis_version:3.2.10
...
```
 ### 确认使用内存和过期策略
 使用命令 `info memory` 命令，便能看到相应的使用内存信息和过期策略；需要注意的是该命令是在 `>= 2.4`版本可以使用的。如果使用的 <2.4 版本该命令可能不存在，那么使用 info 命令即可；info 命令是在 redis 1.0.0 以后就可以使用了。 以下是优化过后的显示。
 ```shell
 > info memory
 # Memory
 ...
used_memory_human:6.03G
...
used_memory_rss_human:6.83G
...
used_memory_peak_human:8.83G
...
maxmemory_human:13.07G
maxmemory_policy:volatile-lru
mem_fragmentation_ratio:1.13
```
这里摘了几个比较重要的参数：
- `used_memory_human` : 用户可读的 Redis 已经使用的内存
- `used_memory_rss_human` : 用户可读的 Redis 向操作系统申请的内存
- `used_memory_peak_human` : 用户可读的 Redis 使用峰值
- `maxmemory_human` : 用户可读的 Redis 的设置的最大内存
- `maxmemory_policy` : 当用户达到最大的内存时，使用的数据的淘汰策略是什么。 这里使用的 在设置了过期时间的键空间中，优先移除最近没有使用的 key。 如果该键没有设置过期时间则该键永远不过期。这里在后续的内容中会讲述。
- `mem_fragmentation_ratio` : 内存碎片率。 该值在 `1 ~ 1.5` 之间是正常的。`<1` 表示没有足够的物理内存可以使用了，会导致 Redis 的一部分数据会进入 Swap 区进行置换， 之后 Redis 访问 Swap 区的中的数据时，延迟会变大，性能下降。`>1.5` 时，说明内存碎片超过了 50%， 就需要采取一些措施来降低内存碎片了。

由此，我推断是程序中是否有缓存没有设置过期时间导致了使用的内存越来越大？

## 查看代码是否设置了过期时间
在上述的推断中，我查询了代码，结果发现某个每日任务完成情况的 key 并没有设置过期时间，导致了内存越来越大。 在代码中确定了该业务是每天生成的 key，删除之前的 key 并不会影响当天的任务情况，也不会有其他业务去查询之前完成的情况。还需要组长和其他组员确认这块的逻辑没有其他模块的使用。所以我在这里设置成每天过期了。由于是每日任务，所以建 key 的时候末尾会以当天日期为主，这样是为了方便查询，可能每个公司的不太一样。修改完代码后，只能确保之后的 key 的不会越来越多，而已经在内存中的还是很多。这里虽然知道该 key 会越来越多，但是不知道当前的 key 占用的内存情况如何。
为了搞清楚，这些 key 使用的内存情况，我引入了 [`redis-rdb-tools`](https://github.com/sripathikrishnan/redis-rdb-tools) 工具。

## 使用 rdb-tools 分析 key 的使用情况
由于之前确认了哪些 key 没有设置过期时间，导致了内存越来越大，这里为了确认这些 keys 使用的内存到底有多大, 这里使用了工具 [`redis-rdb-tools`](https://github.com/sripathikrishnan/redis-rdb-tools)。 按照介绍安装完成后，使用命令
```shell
# redis-memory-for-key -s ${redis-host} ${redis-key}
```
来查看当前单个 key 的使用大小。为了确认 keys 是否是 bigkey，使用命令：`redis-cli -h ${redis-host} --bigkeys -i ${time-space}` 查看当前 key 是否属于 bigkey， 如果不是 bigkey， 可以进行以下操作。如果是，统计和删除的是否请用 scan 命令。因为 keys 的时间复杂度为 O(n), 如果是大量的 keys 的话，将会导致 redis 阻塞。
使用命令
```shell
redis-cli -h  ${redis-host} -p ${reds-port} keys "${redis-key-pattern}" | wc -l
```
来统计能匹配的 keys 的个数。结合上述的大小和个数来估算出大致的使用量。经过分析后这些任务大致使用掉了 `2~3` 个G。这是一个昂贵的数值。为了干掉这些内存，我使用了命令
```shell
 redis-cli -h ${redis-host} keys "${redis-key-pattern}" | xargs redis-cli -h ${redis-host} -n 0 del
```
删除相应的 key。这里采用的比较粗暴的方法去删除 keys， 由于单个匹配模式下，每个 key 的大小在 200 左右。所以采用这种方式。


# 总结
以上便是我优化 Redis 的整个步骤，过程中还有可以优化的地方，比如 keys 的命令的不合理使用，删除 keys 的操作需要在测试环境进行回归测试，需要测试人员的配合等。经过这次的优化，我对 Redis 的理解比之前有了更深的理解。