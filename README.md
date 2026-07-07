# 每个程序员都应该了解的内存知识

本文翻译自 [Ulrich Drepper](https://de.wikipedia.org/wiki/Ulrich_Drepper) 于 2007 年撰写的论文《[What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)》(版次: 1.0)，原文共 114 页。

随着 CPU 核心（core）变得更快、数量也更多，内存访问已经成为大多数程序的限制因素，并且在未来一段时间内仍将如此。
硬件设计者已经提出越来越复杂的内存处理与加速技术，例如 CPU 缓存；但如果没有程序员的配合，这些技术无法发挥最佳效果。
遗憾的是，对于大多数程序员来说，计算机内存子系统与 CPU 缓存的结构以及使用成本仍然并不清楚。
本文解释现代商用硬件所使用的内存子系统结构，说明 CPU 缓存为何被发展出来、它们如何工作，以及程序应该如何利用它们来达到最佳性能。

## 翻译信息
繁体译者: [Chi-En Wu](https://github.com/jason2506), [Jim Huang](https://github.com/jserv)
> [!INFO]
> 关于繁体中文翻译内容的修正、改进建议，和贡献，请访问 [sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw)

简体转译：[LLQuant](https://github.com/llquant)，使用 GPT 5.5 转译，并使用 Opus 4.8 校验转译结果。  
项目地址：[llquant/what-every-programmer-should-know-about-memory](https://github.com/llquant/what-every-programmer-should-know-about-memory.git) 
