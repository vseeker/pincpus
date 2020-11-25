# pincpus
Script for automatic pinning of ceph osd daemons to numa node via cgroups

Usage:

1) put prz-pincpus.conf into /etc/ and edit to your liking
2) run pincpus :-)





# ref 

https://www.spinics.net/lists/ceph-users/msg19811.html

```
# 考虑项

1) not moving the processes and threads between cores that much (better cache utilization)
2) aligning the processes with memory on NUMA systems (that means all modern dual socket systems) - you don’t want your OSD running on CPU1 with memory allocated to CPU2
3) the same goes for other resources like NICs or storage controllers - but that’s less important and not always practical to do
4) you can limit the scheduling domain on linux if you limit the cpuset for your OSDs (I’m not sure how important this is, just best practice)
5) you can easily limit memory or CPU usage, set priority, with much greater granularity than without cgroups
6) if you have HyperThreading enabled you get the most gain when the workloads on the threads are dissimiliar - so to have the higher throughput you have to pin OSD to thread1 and KVM to thread2 on the same core. We’re not doing that because latency and performance of the core can vary depending on what the other thread is doing. But it might be useful to someone.


Some workloads exhibit >100% performance gain when everything aligns in a NUMA system, compared to a SMP mode on the same hardware. You likely won’t notice it on light workloads, as the interconnects (QPI) are very fast and there’s a lot of bandwidth, but for stuff like big OLAP databases or other data-manipulation workloads there’s a huge difference. And with CEPH being CPU hungy and memory intensive, we’re seeing some big gains here just by co-locating the memory with the processes….
Could you elaborate a it on this?  I'm interested to learn in what situation memory locality helps Ceph to what extend.



```
http://xiaqunfeng.cc/2017/05/11/cpu-top/

1. 为Ceph进程预留足够多的CPU和内存资源，防止影响性能或产生OOM。尤其是高性能环境中并不能完全满足Ceph进程的开销，在高性能场景（全SSD）下，每个OSD进程可能需要高达6GHz的CPU和4GB以上的内存。

单个 nvme ssd osd 分配 2个cpu 6g 内存

缓存盘osd 分配 1个cpu 1g 内存


2. 执行 cpu-topology.sh
保证osd调用到同一个core上的概率最小 
Logical processor只是OS内部看到的，实际上两个Processor还是位于同一个Core上，所以频繁的调度仍可能导致资源竞争，影响性能。


