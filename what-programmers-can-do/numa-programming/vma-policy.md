# 6.5.4. VMA 策略

要为一个地址范围设置 VMA 策略，必须使用一个不同的接口：

```c
#include <numaif.h>
long mbind(void *start, unsigned long len,
           int mode,
           unsigned long *nodemask,
           unsigned long maxnode,
           unsigned flags);
```

这个接口为地址范围 [`start`, `start` + `len`) 注册一个新的 VMA 策略。由于 memory 管理是在页上操作，所以起始地址必须是对齐页的。`len` 值会被无条件进位至下一个页大小。

`mode` 参数再次指定策略；这个值必须从 6.5.1 节的清单中挑选。与使用 `set_mempolicy` 相同，`nodemask` 参数只会用在某些策略上。它的处理是一样的。

`mbind` 接口的语义取决于 `flags` 参数的值。默认情况下，若是 `flags` 为零，系统呼叫会为这个地址范围设置 VMA 策略。现有的映射不受影响。假如这还不够，目前有三种修改这种行为的标志；它们可以被单独或者一起被选择：

**`MPOL_MF_STRICT`：** 假如并非所有页都在由 `nodemask` 指定的节点上，对 `mbind` 的呼叫将会失败。在这个标志与 `MPOL_MF_MOVE` 和／或 `MPOL_MF_MOVEALL` 一起使用的情况下，呼叫会在任何页无法被移动的时候失败。

**`MPOL_MF_MOVE`：** 系统核心将会试着移动地址范围中、任何分配在一个不在由 `nodemask` 指定的集合中的节点上的页。默认情况下，仅有被目前进程的页表专用的页会被移动。

**`MPOL_MF_MOVEALL`：** 如同 `MPOL_MF_MOVE`，但系统核心会试着移动所有页，而非仅有那些独自被目前进程的页表使用的页。这个操作具有系统层面的影响，因为它也影响其他 –– 可能不是由相同用户所拥有的 –– 进程的 memory 访问。因此 `MPOL_MF_MOVEALL` 是个特权操作（需要 `CAP_NICE` 的能力）。

注意到针对 `MPOL_MF_MOVE` 与 `MPOL_MF_MOVEALL` 的支持仅在 2.6.16 Linux 系统核心中才被加入。

在没有任何标志的情况下呼叫 `mbind`，在任何页真的被分配之前必须为一个新预留的地址范围指定策略的时候是最有用的。

```c
void *p = mmap(NULL, len,
               PROT_READ|PROT_WRITE,
               MAP_ANON, -1, 0);
if (p != MAP_FAILED)
  mbind(p, len, mode, nodemask, maxnode,
        0);
```

这段程序序列保留一段 `len` byte 的地址空间范围，并指定应该使用指涉 `nodemask` 中的 memory 节点的策略 `mode`。除非 `MAP_POPULATE` 标志与 `mmap` 一起使用，否则在 `mbind` 呼叫的时候并不会分配任何 memory，因此新的策略会套用在这个地址空间区域中的所有页。

单独的 `MPOL_MF_STRICT` 标志能用来确定，传给 `mbind` 的 `start` 与 `len` 参数所描述的地址范围中的任何页，是否都被分配在那些由 `nodemask` 指定的那些节点以外的节点上。已分配的页不会被改变。若是所有页都被分配在指定的节点上，那么地址空间区域的 VMA 策略会根据 `mode` 改变。

有时候是需要 memory 的重新平衡的。在这种情况下，可能必须将一个节点上分配的页移到另一个节点上。以设置 `MPOL_MF_MOVE` 呼叫的 `mbind` 会尽最大努力来达成这点。仅有单独被进程的页表树指涉到的页会被考虑是否移动。可能有多个以线程或其他进程的形式共享部页表树的用户。不可能影响碰巧映射相同数据的其他进程。这些页并不共享页表项目。

若是传递给 `mbind` 的 `flags` 参数中设置 `MPOL_MF_STRICT` 与 `MPOL_MF_MOVE` bit，系统核心会试着移动并非分配在指定节点上的所有页。假如这无法做到，这个呼叫将会失败。这种呼叫可能有助于确定是否有个节点（或是一组节点）可以容纳所有的页。可以连续尝试多种组合，直到找到一个合适的节点。

除非执行目前的进程是这台电脑的主要目的，否则 `MPOL_MF_MOVEALL` 的使用是较难以证明为正当的。理由是，即使是出现在多张页表的页也会被移动。这会轻易地以负面的方式影响其他进程。因而应该要谨慎地使用这个操作。
