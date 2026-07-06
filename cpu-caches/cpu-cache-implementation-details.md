# 3.3. CPU cache 实现细节

cache 实现者有个麻烦是，在庞大的主 memory 中，每个记忆单元都可能得被 cache。假如一支程序的工作集足够大，这表示有许多为了 cache 中的各个地方打架的主 memory 位置。先前曾经提过，cache 与主 memory 大小的比率为 1 比 1000 的情况并不罕见。

