# 5.3. 发布的信息

系统核心通过 `sys` 虚拟文件系统（sysfs）将处理器 cache 的信息发布在

`/sys/devices/system/cpu/cpu*/cache`

在 6.2.1 节，我们会看到能用来查询不同 cache 大小的接口。这里重要的是 cache 的拓朴。上面的目录包含了列出 CPU 拥有的不同 cache 信息的子目录（叫做 `index*`）。文件 `type`、`level`、与 `shared_cpu_map` 是在这些目录中与拓朴有关的重要文件。一个 Intel Core 2 QX6700 的信息看起来就如表 5.1。

|  |  | `type` | `level` | `shared_cpu_map` |
| --- | --- | --- | --- | --- |
| `cpu0` | `index0` | Data | 1 | 00000001 |
| `cpu0` | `index1` | Instruction | 1 | 00000001 |
| `cpu0` | `index2` | Unified | 2 | 00000003 |
| `cpu1` | `index0` | Data | 1 | 00000002 |
| `cpu1` | `index1` | Instruction | 1 | 00000002 |
| `cpu1` | `index2` | Unified | 2 | 00000003 |
| `cpu2` | `index0` | Data | 1 | 00000004 |
| `cpu2` | `index1` | Instruction | 1 | 00000004 |
| `cpu2` | `index2` | Unified | 2 | 0000000c |
| `cpu3` | `index0` | Data | 1 | 00000008 |
| `cpu3` | `index1` | Instruction | 1 | 00000008 |
| `cpu3` | `index2` | Unified | 2 | 0000000c |

*表 5.1：Core 2 CPU cache 的 `sysfs` 信息*

这份数据的意义如下：

* 每颗处理器核[^25]拥有三个 cache：L1i、L1d、L2。
* L1d 与 L1i cache 没有被任何其他的处理器核所共享 –– 每颗处理器核有它自己的一组 cache。这是由 `shared_cpu_map` 中的 bit 图（bitmap）只有一个被设置的 bit 所暗示的。
* `cpu0` 与 `cpu1` 的 L2 cache 是共享的，正如 `cpu2` 与 `cpu3` 上的 L2 一样。

若是 CPU 有更多 cache 阶层，也会有更多的 `index*` 目录。

|  |  | `type` | `level` | `shared_cpu_map` |
| --- | --- | --- | --- | --- |
| `cpu0` | `index0` | Data | 1 | 00000001 |
| `cpu0` | `index1` | Instruction | 1 | 00000001 |
| `cpu0` | `index2` | Unified | 2 | 00000001 |
| `cpu1` | `index0` | Data | 1 | 00000002 |
| `cpu1` | `index1` | Instruction | 1 | 00000002 |
| `cpu1` | `index2` | Unified | 2 | 00000002 |
| `cpu2` | `index0` | Data | 1 | 00000004 |
| `cpu2` | `index1` | Instruction | 1 | 00000004 |
| `cpu2` | `index2` | Unified | 2 | 00000004 |
| `cpu3` | `index0` | Data | 1 | 00000008 |
| `cpu3` | `index1` | Instruction | 1 | 00000008 |
| `cpu3` | `index2` | Unified | 2 | 00000008 |
| `cpu4` | `index0` | Data | 1 | 00000010 |
| `cpu4` | `index1` | Instruction | 1 | 00000010 |
| `cpu4` | `index2` | Unified | 2 | 00000010 |
| `cpu5` | `index0` | Data | 1 | 00000020 |
| `cpu5` | `index1` | Instruction | 1 | 00000020 |
| `cpu5` | `index2` | Unified | 2 | 00000020 |
| `cpu6` | `index0` | Data | 1 | 00000040 |
| `cpu6` | `index1` | Instruction | 1 | 00000040 |
| `cpu6` | `index2` | Unified | 2 | 00000040 |
| `cpu7` | `index0` | Data | 1 | 00000080 |
| `cpu7` | `index1` | Instruction | 1 | 00000080 |
| `cpu7` | `index2` | Unified | 2 | 00000080 |

*表 5.2：Opteron CPU cache 的 `sysfs` 信息*

对于一个四槽、双核的 Opteron 机器，cache 信息看起来如表 5.2。可以看出这些处理器也有三种 cache ：L1i、L1d、L2。没有处理器核共享任何阶层的 cache。这个系统有趣的部分在于处理器拓朴。少了这个额外信息，就无法理解 cache 数据。`sys` 文件系统将这个信息摆在下面这个文件

`/sys/devices/system/cpu/cpu*/topology`

表 5.3 显示了在 SMP Opteron 机器的这个阶层里头的令人感兴趣的文件。

|  | `physical_<br>package_id` | `core_id` | `core_<br>siblings` | `thread_<br>siblings` |
| --- | --- | --- | --- | --- |
| `cpu0` | 0 | 0 | 00000003 | 00000001 |
| `cpu1` | 0 | 1 | 00000003 | 00000002 |
| `cpu2` | 1 | 0 | 0000000c | 00000004 |
| `cpu3` | 1 | 1 | 0000000c | 00000008 |
| `cpu4` | 2 | 0 | 00000030 | 00000010 |
| `cpu5` | 2 | 1 | 00000030 | 00000020 |
| `cpu6` | 3 | 0 | 000000c0 | 00000040 |
| `cpu7` | 3 | 1 | 000000c0 | 00000080 |

*表 5.3：Opteron CPU 拓朴的 `sysfs` 信息*

将表 5.2 与 5.3 摆在一起，我们可以发现

* 没有 CPU 拥有 HT（`thread_siblings` bit 图有一个 bit 被设置）、
* 这个系统实际上共有四个处理器（`physical_package_id` 0 到 3）、
* 每个处理器有两颗核、以及
* 没有处理器核共享任何 cache。

这正好与较早期的 Opteron 一致。

目前为止提供的数据中完全缺少的是，有关这台机器上的 NUMA 性质的信息。任何 SMP Opteron 机器都是一台 NUMA 机器。为了这份数据，我们必须看看在 NUMA 机器上存在的 `sys` 文件系统的另一个部分，即下面的阶层中

`/sys/devices/system/node`

这个目录包含在系统上的每个 NUMA 节点的子目录。在特定节点的目录中有许多文件。在前两张表中描述的 Opteron 机器的重要文件与它们的内容显示在表 5.4。

|  | `cpumap` | `distance` |
| --- | --- | --- |
| `node0` | 00000003 | 10 20 20 20 |
| `node0` | 0000000c | 20 10 20 20 |
| `node2` | 00000030 | 20 20 10 20 |
| `node3` | 000000c0 | 20 20 20 10 |

*表 5.4：Opteron 节点的 `sysfs` 信息*

这个信息将所有的一切连系在一起；现在我们有个机器架构的完整轮廓了。我们已经知道机器拥有四个处理器。每个处理器构成它自己的节点，可以从 `node*` 目录的 `cpumap` 文件中的值里头设置的 bit 看出来。在那些目录的 `distance` 文件包含一组值，一个值代表一个节点，表示在各个节点上访问 memory 的成本。在这个例子中，所有本地 memory 访问的成本为 10，所有对任何其他节点的远程访问的成本为 20。[^26]这表示，即使处理器被组织成一个二维超立方体（见图 5.1），在没有直接连接的处理器之间访问也没有比较贵。成本的相对值应该能用来作为访问时间的实际差距的估计。所有这些信息的准确性是另一个问题了。


[^25]: `cpu0` 到 `cpu3` 为处理器核的相关信息来自于另一个将会简短介绍的地方。

[^26]: 顺带一提，这是不正确的。这个 ACPI 信息明显是错的，因为 –– 虽然用到的处理器拥有三条连贯的超传输链接 –– 至少一个处理器必须被连接到一个南桥上。至少一对节点必须因此有比较大的距离。

