---
layout: post
title: Linux OS Jitter
comments: true
tags: [software, linux, performance, paper]
categories: [engineering]
---

DRAFT

[Exploratory Study on the Linux OS Jitter](https://www.researchgate.net/publication/258406264_Exploratory_Study_on_the_Linux_OS_Jitter) - Vicente, et al. 2012

At work, we have been looking into problems related to OS jitter. The same benchmark, run on the same machine, with the same data
issueing different results on different runs. Would it be a Java application, I would have had a tendency to quickly attribute it
to JVM hiccups, but the application under test was written in C++. So, I started to look for papers on OS Jitter and will be commenting 
out an interesting one here.

> In this paper, we present the results of an experimental study that quantifies the effects of different sources of OS Jitter in the Linux operating system. We
use the design of experiments (DOE) method to conduct controlled experiments statistically planned. 

The results are interesting and the statistical rigor applied as well. The paper has two main contributions, first to show that processor topology, and especially shared processor caches, have a major role on OS Jitter. Second that, for distributed workloads, the number of computational phases in the algorithm has a quite more important influence than the number of distributed compute nodes.

> HPC cluster-based applications are typically designed to run in a paradigm of parallel processing, where instructions are programmed to be executed in many computational phases [2]. In this approach, after all distributed processes finish a given computational
phase they all synchronize and then start executing the subsequent phase [4]-[6]. Since a new phase only starts after all distributed processes conclude the current phase, synchronizing the computing time of all application processes is critical. The last process that terminates a given phase determines the time length of the phase. So, reducing the runtime variability in each node is a major requirement, given that the occurrence of unexpected delays in a node will spread along other nodes involved in the same computational phase, bringing a longer time to complete the whole task.

From previous work (Kothari et al. 2007) mentioned in the paper, the main source of jitter was the timer interrupt (63%) and the rest was attributed to come from different OS daemons and other hardware interrupts. Since then, the Linux kernel has implemented what is called [_tickless_ kernel](https://en.wikipedia.org/wiki/Tickless_kernel). As a side note, it seems that it was already available on [Solaris since 1999](https://news.ycombinator.com/item?id=13091162).

The first actionable advice for reducing external influences that we can take from the paper is turning off Linux automatic CPU frequency regulation.

>  This is done by recompiling the Linux kernel with the option “CPU Frequency Scaling” (CONFIG_CPU_FREQ) turned off. This feature allows the Linux kernel to change dynamically the processor frequency, affecting the run time length of the application processes.

Three experiments are described in the paper. In all experiments, the test application is running on a single CPU. 

The Experiment #1 plays with five factors, which are turned on and off, called levels (+) and (-) in the paper:

1. **Runlevel**: Defines the OS services loaded during the system initialization. Runlevel 5 means a higher number of services loaded. This is typically the default level for many Linux distributions. Runlevel 1 has a minimum set of services loaded;

2. **Kernel timers**: Used to allow the execution of kernel and user level routines at a given future time;

3. **IRQ**: Hardware interrupts. On level (+) all IRQs are handled by the same processor handling the application. On level (-) no IRQ is handled by it. This is achieved using Linux "SMP IRQ affinity";

4. **Processor affinity**: With this option set (+), no other system process will run on the same CPU where the test application running. All system processes are actually set to run on a specific processor. With the option unset (-), system processes can run on the same CPU where the application is running;

5. **Timer interrupt**: Finally, the timer interrupt is the last factor to be enabled/disabled in this experiment.



<style type="text/css">
    ol { list-style-type: upper-alpha; }
</style>


