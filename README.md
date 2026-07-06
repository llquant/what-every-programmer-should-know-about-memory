# 每个程序员都应该了解的内存知识

本文翻译自 [Ulrich Drepper](https://de.wikipedia.org/wiki/Ulrich_Drepper) 于 2007 年撰写的论文《[What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)》(版次: 1.0)，原文共 114 页。

在 CPU 核 (core) 在速度和数量增长的同时，memory 访问限制着当今大多数程序的效率，甚至未来一段时间也会如此。
尽管硬件设计者已提出日趋复杂的 memory 处理与加速机制 —— 例如 CPU cache —— 但若程序开发者无法善用，仍无法有效发挥硬件作用。
不幸的是，论及电脑的 memory 子系统或 CPU cache 时，无论是其内部的结构，抑或访问成本，对大多程序开发者仍相当陌生。
本文解释用于现代电脑硬件的 memory 子系统的结构、阐述 CPU cache 发展的考量、它们如何运作，以及程序该如何针对 memory 操作调整，从而达到最佳的性能。

## 翻译信息
繁体译者: [Chi-En Wu](https://github.com/jason2506), [Jim Huang](https://github.com/jserv)
> [!INFO]
> 关于繁体中文翻译内容的修正、改进建议，和贡献，请访问 [sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw)

简体转译：[LLQuant](https://github.com/llquant)，使用GPT 5.5转译，并使用Opus 4.8 校验转译结果。  
[llquant/what-every-programmer-should-know-about-memory](https://github.com/llquant/what-every-programmer-should-know-about-memory.git) 

