# NoSQL 压力测试

@(技术)[nosql, mongodb, redis, scylla, stress-test]

## 1. 概述

本次压测数据 1000 万, 线程 100 个, 客户端到服务端模式, 工具是 **YCSB**

原因如下:

-  数据 1000 万: Redis 存储在内存, 20 GB 内存仅够存储 1000 万多一点
-  线程 100 个: 模拟 100 个客户端并发, 为了跑满服务端的 CPU
-  客户端到服务端模式: 为了防止请求进程和数据库进程抢占 CPU

主要监测 **吞吐量**, **平均延迟**, **CPU 占用率**

## 2. 环境信息

### 2.1 版本

#### 2.1.1 数据库

| - | - |
| -- | -- |
| **Mongo** | 4.0 |
| **Redis** | 5.0 |
| **Scylla** | 2.3, 3.0rc |

#### 2.1.2 项目环境

| - | - |
| -- | -- |
| **Python** | 2.7.5 |
| **Java** | 1.8 |
| **Maven** | 3.6.0 |
| **YCSB** | 0.15.0 |

### 2.2 硬件信息

### 2.2.1 服务端

| - | - |
| -- | -- |
| **CPU** | Intel(R) Xeon(R) CPU E3-1230 V2 @ 3.30GHz, 4 个逻辑核 |
| **内存** | 20 GB |
| **磁盘** | 200 GB HDD |
| **网络** | 1000Mb/s |
| **操作系统** | CentOS Linux release 7.6.1810 (Core) |
| **内核版本** | 3.10.0-957.1.3.el7.x86_64 |
| **IP** | 192.168.1.23 |

#### 2.2.2 客户端

| - | - |
| -- | -- |
| **CPU** | Intel(R) Xeon(RIntel(R) Xeon(R) CPU E5-1620 v2 @ 3.70GHz, 8 个逻辑核 |
| **内存** | 128 GB |
| **磁盘** | 256 GB SSD, 4 TB Raid 10 HDD |
| **网络** | 1000Mb/s |
| **操作系统** | CentOS Linux release 7.6.1810 (Core) |
| **内核版本** | 3.10.0-862.14.4.el7.x86_64 |

> 为什么客户端配置比服务端好这么多?
> 因为反过来的话, 压测 CPU 跑不满啊


## 3. 测试模板

| Workload | Operations |
| -- | -- |
| A | 50 读, 50% 更新 |
| B | 95% 读, 5% 更新 |
| C | 100% 读 |
| D | 95% 读, 5% 插入 |

## 4. 测试步骤

| 序号 | 步骤 | 示例 |
| -- | -- | -- |
| 1 | 修改 **YCSB/workloads** 下 A,B,C,D 四个模板的数据量为 1000 万 | 修改模板 A: 打开 workloada, 修改为 <br> `recordcount=10000000 operationcount=10000000` <br><br> recordcount 表示测试前插入数据量, operationcount 表示操作数据量 |
| 2 | 测试之前先通过 load 命令插入 1000 万数据 | `python bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://your_username:your_password@192.168.1.23:27017/ycsb?w=0 -threads 100 > mongodb_load.txt 2>&1 &` |
| 3 | 通过 run 命令测试不同模板 | `python bin/ycsb run mongodb -s -P workloads/workloada -p mongodb.url=mongodb://your_username:your_password@192.168.1.23:27017/ycsb?w=0 -threads 100 > mongodb_runA.txt 2>&1 &` |
| 4 | 每个数据库测试完后, 清空数据, 占用内存及缓存 | 清空数据: Mongo 的话直接删除表就好 <br> 清空内存: 重启数据库服务 <br> 清空缓存: `echo 1 > /proc/sys/vm/drop_caches` |

## 5. 测试结果分析

> 图表结果参考 **最终结果比较.xlsx**

### 5.1 吞吐量分析

- 更新较多时, Scylla 的吞吐量较好
- 读取较多时, Redis 的吞吐量较好
- 当有插入时, Redis 的吞吐量明显下降, 可能是因为内存接近占满
- Redis 的吞吐量曲线波动较大, 仅在只读时相对稳定; Scylla 最稳定
- Scylla3 较 2.3 的吞吐量明细下降, 但是从曲线看出 Scylla3 的吞吐量更为稳定
 
### 5.2 平均延迟分析

- Mongo 插入和更新的平均延迟非常低, 插入 41 us, 更新 30 us; 其他数据库插入在 600-1000 us 之间, 更新在 4000-10000 us 之间
- Redis 的读取延迟最低
- Scylla3 的平均延迟均高于 2.3

### 5.3 CPU 占用率分析

> Redis 是单线程的, 最多只会使用一个 CPU

- Redis 基本稳定在 100%, 读取百分比越高, 稳定性越高
- Mongo 虽然也能用满 4 个CPU, 但是波动非常大, 尤其是更新或插入百分比越高时
- Scylla 最为稳定, 稳定在 300% 或 350%
- Scylla3 较 2.3 的 CPU 占用率均有所下降

## 6. 其他结果分析

### 6.1 内存管理

- Mongo 的内存最大只使用 51%
- Redis 的内存管理做的不是很好, 当内存写满时, 还会继续写入, 导致被 OOM Killer 强制关闭
- Scylla 的内存写满时, 约为 91.7%, 会保持在这个数值

> 10% 左右的内存要被系统占用, 所以 90% 就是接近写满了


 







