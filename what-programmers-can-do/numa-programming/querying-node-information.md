# 6.5.5. 查询节点信息

`get_mempolicy` 接口能用以查询关于一个给定地址的 NUMA 状态的各种事实。

```c
#include <numaif.h>
long get_mempolicy(int *policy,
             const unsigned long *nmask,
             unsigned long maxnode,
             void *addr, int flags);
```

当 `get_mempolicy` 以 `0` 作为 `flags` 参数调用时，关于地址 `addr` 的策略信息会被存储在由 `policy` 指到的 word、以及由 `nmask` 指到的节点的 bit 掩码中。若是 `addr` 落在一段已经被指定一个 VMA 策略的地址空间范围中，就返回关于这个策略的信息。否则，将会返回关于任务策略或者 –– 必要的话 –– 系统默认策略的信息。

若是 `flags` 中设置 `MPOL_F_NODE`、并且管理 `addr` 的策略为 `MPOL_INTERLEAVE`，那么 `policy` 所指到的 word 为要进行下一次分配的节点索引。这个信息可以潜在地用来设置打算要在新分配的内存上运作的一条线程的亲和性。这可能是实现逼近的一个比较不昂贵的方法，尤其是在线程仍未被建立的情况。

`MPOL_F_ADDR` 标志能用来检索另一个完全不同的数据项目。假如使用这个标志，`policy` 所指到的 word 为已经为包含 `addr` 的页分配内存的内存节点索引。这个信息能用来决定可能的页迁移、决定哪条线程可以最有效率地运作在内存位置上、还有更多更多的事情。

一条线程正在使用的 CPU –– 以及因此用到的内存节点 –– 比起它的内存分配还要更加变化无常。在没有明确请求的情况下，内存页只会在极端的情况下被移动。一条线程能被指派给另一个 CPU，作为重新平衡 CPU 负载的结果。关于当前 CPU 与节点的信息可能因而仅在短期内有效。调度器会试着将线程维持在同一个 CPU 上，甚至可能在相同的核上，以最小化由于冷 cache（cold cache）造成的性能损失。这表示，查看当前 CPU 与节点的信息是有用的；只要避免假设关联性不会改变。

libNUMA 提供两个接口，以查询一段给定虚拟地址空间范围的节点信息：

```c
#include <libNUMA.h>
int NUMA_mem_get_node_idx(void *addr);
int NUMA_mem_get_node_mask(void *addr,
                           size_t size,
                           size_t __destsize,
                           memnode_set_t *dest);
```

`NUMA_mem_get_node_mask` 根据管理策略，在 `dest` 中设置代表所有分配（或者可能分配）范围 [`addr`, `addr`+`size`) 中的页的内存节点的 bit。`NUMA_mem_get_node` 只看地址 `addr`，并返回分配（或者可能分配）这个地址的内存节点的索引。这些接口比 `get_mempolicy` 还容易使用，而且应该是首选。

当前正由一条线程使用的 CPU 可以使用 `sched_getcpu` 来查询（见 6.4.3 节）。使用这个信息，一支程序可以使用来自 libNUMA 的 `NUMA_cpu_to_memnode` 接口来确定 CPU 本地的内存节点（们）：

```c
#include <libNUMA.h>
int NUMA_cpu_to_memnode(size_t cpusetsize,
                        const cpu_set_t *cpuset,
                        size_t memnodesize,
                        memnode_set_t *
                        memnodeset);
```

对这个函数的一次调用会设置所有对应于任何在第二个参数指到的集合中的 CPU 本地的内存节点的 bit。就如同 CPU 信息本身，这个信息直到机器的配置改变（例如，CPU 被移除或新增）时才会是正确的。

`memnode_set_t` 对象中的 bit 能被用在像 `get_mempolicy` 这种低阶函数的调用上。使用 libNUMA 中的其他函数会更加方便。反向映射能通过下述函数取得：

```c
#include <libNUMA.h>
int NUMA_memnode_to_cpu(size_t memnodesize,
                        const memnode_set_t *
                        memnodeset,
                        size_t cpusetsize,
                        cpu_set_t *cpuset)
```

在产生的 `cpuset` 中设置的 bit 为任何在 `memnodeset` 中设置的 bit 所对应的内存节点本地的那些 CPU。对于这两个接口，程序开发者都必须意识到，信息可能随着时间改变（尤其是使用 CPU 热插拔的情况）。在许多情境中，在输入的 bit 集中只会设置单一个 bit，但举例来说，将 `sched_getaffinity` 调用检索到的整个 CPU 集合传递到 `NUMA_cpu_to_memnode`，以确定哪些内存节点可以被线程直接访问到，也是有意义的。

