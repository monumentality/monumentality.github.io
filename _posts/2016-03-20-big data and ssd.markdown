---
layout: post
title:  "bid scheduler"
date: 2016-03-13
categories: research
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 讲道理：大数据处理框架的性能分析 《making sense of performance in data analytics frameworks》</a>
<ul>
<li><a href="#sec-1-1">1.1. NSDI15</a></li>
<li><a href="#sec-1-2">1.2. Abstract</a></li>
<li><a href="#sec-1-3">1.3. Introduction</a>
<ul>
<li><a href="#sec-1-3-1">1.3.1. wide-accepted mantras about the performance of data analytics</a></li>
<li><a href="#sec-1-3-2">1.3.2. two contributions</a></li>
</ul>
</li>
<li><a href="#sec-1-4">1.4. Methodology</a>
<ul>
<li><a href="#sec-1-4-1">1.4.1. workloads</a></li>
<li><a href="#sec-1-4-2">1.4.2. framework architecture</a></li>
<li><a href="#sec-1-4-3">1.4.3. blocked time analysis</a></li>
<li><a href="#sec-1-4-4">1.4.4. cluster setup</a></li>
<li><a href="#sec-1-4-5">1.4.5. production traces</a></li>
</ul>
</li>
<li><a href="#sec-1-5">1.5. How important is disk I/O?</a>
<ul>
<li><a href="#sec-1-5-1">1.5.1. measuring time blocked on disk I/O at four different points in task execution:</a></li>
<li><a href="#sec-1-5-2">1.5.2. summary</a></li>
</ul>
</li>
<li><a href="#sec-1-6">1.6. How important is the network?</a>
<ul>
<li><a href="#sec-1-6-1">1.6.1. why isn't the network more important?</a></li>
<li><a href="#sec-1-6-2">1.6.2. are these results inconsistent with past work?</a></li>
<li><a href="#sec-1-6-3">1.6.3. summary</a></li>
</ul>
</li>
<li><a href="#sec-1-7">1.7. The role of Stragglers</a>
<ul>
<li><a href="#sec-1-7-1">1.7.1. how much do stagglers affect job completion time?</a></li>
<li><a href="#sec-1-7-2">1.7.2. Are these results inconsistent with prior work?</a></li>
<li><a href="#sec-1-7-3">1.7.3. why do stragglers occur?</a></li>
<li><a href="#sec-1-7-4">1.7.4. Improving performance by understanding stragglers</a></li>
<li><a href="#sec-1-7-5">1.7.5. How Does Scale Affect Results?</a></li>
</ul>
</li>
<li><a href="#sec-1-8">1.8. How Does Scale Affect Results?</a></li>
<li><a href="#sec-1-9">1.9. Conclusion</a></li>
</ul>
</li>
<li><a href="#sec-2">2. Tachyon: Reliable, Memory Speed Storage for Cluster Computing Frameworks</a>
<ul>
<li><a href="#sec-2-1">2.1. SoCC 14</a></li>
<li><a href="#sec-2-2">2.2. Abstract</a></li>
<li><a href="#sec-2-3">2.3. introduction</a>
<ul>
<li><a href="#sec-2-3-1">2.3.1. tow challenges:</a></li>
</ul>
</li>
<li><a href="#sec-2-4">2.4. background</a></li>
<li><a href="#sec-2-5">2.5. design overview</a></li>
<li><a href="#sec-2-6">2.6. chekpointing</a></li>
<li><a href="#sec-2-7">2.7. resource allocation</a></li>
<li><a href="#sec-2-8">2.8. implementation</a></li>
<li><a href="#sec-2-9">2.9. evaluation</a></li>
</ul>
</li>
<li><a href="#sec-3">3. Introducing SSDs to the Hadoop MapReduce Framework</a>
<ul>
<li><a href="#sec-3-1">3.1. Cloud Computing 14</a></li>
<li><a href="#sec-3-2">3.2. Abstract</a></li>
<li><a href="#sec-3-3">3.3. Introduction</a>
<ul>
<li><a href="#sec-3-3-1">3.3.1. contributions</a></li>
</ul>
</li>
<li><a href="#sec-3-4">3.4. HDFS characteristics</a>
<ul>
<li><a href="#sec-3-4-1">3.4.1. investigate environment</a></li>
<li><a href="#sec-3-4-2">3.4.2. storage vs Network</a></li>
<li><a href="#sec-3-4-3">3.4.3. Mutiple Readers/Writers</a></li>
</ul>
</li>
<li><a href="#sec-3-5">3.5. MapReduce from a Storage Perspective</a>
<ul>
<li><a href="#sec-3-5-1">3.5.1. Terasort Performance Analysis</a></li>
<li><a href="#sec-3-5-2">3.5.2. Cost Analysis</a></li>
</ul>
</li>
<li><a href="#sec-3-6">3.6. Conclusion</a></li>
</ul>
</li>
<li><a href="#sec-4">4. FB-Tree: A B+-Tree for Flash-Based SSDs</a>
<ul>
<li><a href="#sec-4-1">4.1. IDEAS11 2011</a></li>
<li><a href="#sec-4-2">4.2. Abstract</a></li>
<li><a href="#sec-4-3">4.3. Introduction</a>
<ul>
<li><a href="#sec-4-3-1">4.3.1. key features of SSD</a></li>
<li><a href="#sec-4-3-2">4.3.2. contribution of this paper is two-fold</a></li>
</ul>
</li>
<li><a href="#sec-4-4">4.4. Motivation</a>
<ul>
<li><a href="#sec-4-4-1">4.4.1. SSD's rather similar sequential and random read speeds</a></li>
<li><a href="#sec-4-4-2">4.4.2. the performance of the write workloads has the potential to be improved.</a></li>
</ul>
</li>
<li><a href="#sec-4-5">4.5. Related Work</a></li>
<li><a href="#sec-4-6">4.6. FB-Tree</a>
<ul>
<li><a href="#sec-4-6-1">4.6.1. write-optimized B-tree</a></li>
<li><a href="#sec-4-6-2">4.6.2. storage manager</a></li>
<li><a href="#sec-4-6-3">4.6.3. Buffer Manager</a></li>
<li><a href="#sec-4-6-4">4.6.4. conclusion</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#sec-5">5. Conclusion</a></li>
<li><a href="#sec-6">6. 问题：</a></li>
</ul>
</div>
</div>

# 讲道理：大数据处理框架的性能分析 《making sense of performance in data analytics frameworks》<a id="sec-1" name="sec-1"></a>

## NSDI15<a id="sec-1-1" name="sec-1-1"></a>

## Abstract<a id="sec-1-2" name="sec-1-2"></a>

There has been much research devoted to improving the performance of data analytics frameworks, but comparatively little effort has been spent systematically identifying the performance bottlenecks of these systems.

Contrary to our expectations, we find that (i) CPU (and not I/O) is often the bottleneck, (ii)improving network performance can improve job completion time by a median of at most 2%, and (iii) the causes of most stragglers can be identified.

## Introduction<a id="sec-1-3" name="sec-1-3"></a>

### wide-accepted mantras about the performance of data analytics<a id="sec-1-3-1" name="sec-1-3-1"></a>

The network is a bottleneck

The disk is a bottleneck.

Straggler tasks significantly prolong job completion times and have largely unknown underlying causes.

### two contributions<a id="sec-1-3-2" name="sec-1-3-2"></a>

1.  blocked time analysis

    Blocked time analysis uses extensive white-box logging to measure how long each task spends blocked on a given resource.
    
    Taken alone, these per-task measurements allow us to understand straggler causes by correlating slow tasks with long blocked times
    
    Taken together, the per-task measurements for a particular job allow us to simulate how long the job would have taken to complete if the disk or network were infinitely fast, which provides an upper bound on the benefit of optimizing network or disk performance.

2.  using blocked time analysis to understand Spark’s performance on two industry benchmarks and one production workload

    Network optimizations can only reduce job completion time by a median of at most 2%.
    
    Optimizing or eliminating disk accesses can only reduce job completion time by a median of at most 19%.
    
    Optimizing stragglers can only reduce job completion time by a median of at most 10%, and in 75%of queries, we can identify the cause of more than 60% of stragglers.
    
    Blocked-time analysis illustrates that the two leading causes of Spark stragglers are Java’s garbage collection and time to transfer data to and from disk.

## Methodology<a id="sec-1-4" name="sec-1-4"></a>

### workloads<a id="sec-1-4-1" name="sec-1-4-1"></a>

### framework architecture<a id="sec-1-4-2" name="sec-1-4-2"></a>

### blocked time analysis<a id="sec-1-4-3" name="sec-1-4-3"></a>

1.  instrumentation

    1.  logging
    
        Obtaining the measurements shown in Figure 1(b) required significant improvements to the instrumentation in Spark and in HDFS.While some of the instrumentation required was already available in Spark, our detailed performance analysis revealed that existing logging was often incorrect or incomplete [33–36, 38, 44]. Where necessary, we fixed existing logging and pushed the changes upstream to Spark.
    
    2.  cross validation
    
        We found that cross validationwas crucial to validating that our measurements were correct. In addition to instrumentation for blocked time, we also added instrumentation about the CPU, network, and disk utilization on the machine while the task was running (per-task utilization cannot be measured in Spark, because all tasks run in a single process). Utilization measurements allowed us to cross validate blocked times; for example, by ensuring that when tasks spent little time blocked on I/O, CPU utilization was correspondingly high.
    
    3.  no side effect
    
        As part of adding instrumentation, we measured Spark’s performance before and after the instrumentation was added to ensure the instrumentation did not add to job completion time.

2.  simulation

### cluster setup<a id="sec-1-4-4" name="sec-1-4-4"></a>

### production traces<a id="sec-1-4-5" name="sec-1-4-5"></a>

## How important is disk I/O?<a id="sec-1-5" name="sec-1-5"></a>

### measuring time blocked on disk I/O at four different points in task execution:<a id="sec-1-5-1" name="sec-1-5-1"></a>

1.  Reading input data stored on-disk

    (this only applies for the on-disk workloads; in-memory workloads read input from memory).

2.  Writing shuffle data to disk.

    Spark writes all shuffle data to disk, even when input data is read from memory.

3.  Reading shuffle data from a remote disk.

    This time includes both disk time (to read data from disk) and network time (to send the data over the network).Network and disk use is tightly coupled and thus challenging to measure separately; we measure the total time as an upper bound on the improvement from optimizing disk performance.

4.  Writing output data to local disk and two remote disks

    (this only applies for the on-disk workloads). Again, the time to write data to remote disks includes network time as well; we measure both the network and disk time, making our results an upper bound on the improvement from optimizing disk.

### summary<a id="sec-1-5-2" name="sec-1-5-2"></a>

We found that job runtime cannot improve by more than 19% as a result of optimizing disk I/O. To shed more light on this measurement,we compared resource utilization while tasks were running, and found that CPU utilization is typically close to 100% whereas median disk utilization is at most 25%.

One reason for the relatively high use of CPU by the analytics workloads we studied is deserialization and compression

Serialization and compression formats will inevitably evolve in the future, rendering the numbers presented in this paper obsolete.

## How important is the network?<a id="sec-1-6" name="sec-1-6"></a>

Our blocked time instrumentation for the network included time to read shuffle data over the network, and for on-diskworkloads, the time to write output data to one local machine and two remote machines.

Both of these times include disk use as well as network use, because disk and network are interlaced in a manner that makes them difficult to measure separately. As a result, 2% represents an upper bound on the possible improvement from network optimizations.

### why isn't the network more important?<a id="sec-1-6-1" name="sec-1-6-1"></a>

One reason that network performance is relatively unimportant is that the amount of data sent over the network is often much less than the data transferred to disk, because analytics queries often shuffle and output much less data than they read.

### are these results inconsistent with past work?<a id="sec-1-6-2" name="sec-1-6-2"></a>

1.  overestimate

    We instrumented Hadoop to log time spent writing output data and ran the big data benchmark (using Hive to convert SQL queries to Map Reduce jobs) and compared the result from the detailed instrumentation to the estimation previously used. Unfortunately, as shown in Figure 11, the previously used metric significantly overestimates time spent writing output data, meaning that the importance of the network was significantly overestimated.

2.  inefficiencies in hadoop

    A second problem with past estimations of the importance of the network is that they have conflated inefficiencies in Hadoop with network performance problems.

### summary<a id="sec-1-6-3" name="sec-1-6-3"></a>

One reason network performance has little effect on job completion time is that the data transferred over the network is a subset of data transferred to disk, so jobs bottleneck on the disk before bottlenecking on the network.

## The role of Stragglers<a id="sec-1-7" name="sec-1-7"></a>

### how much do stagglers affect job completion time?<a id="sec-1-7-1" name="sec-1-7-1"></a>

the median improvement from eliminating stragglers is 5-10% for the big data benchmark and TPC-DS workloads, and lower for the productionworkloads, which had fewer stragglers.

### Are these results inconsistent with prior work?<a id="sec-1-7-2" name="sec-1-7-2"></a>

### why do stragglers occur?<a id="sec-1-7-3" name="sec-1-7-3"></a>

Our instrumentation allows us to describe the cause of more than 60% of stragglers in 75% of the queries we ran.

common patterns are that garbage collection can cause most of the stragglers for some queries, and many stragglers can be attributed to long times spent reading to or writing from disk (this is not inconsistent with our earlier results showing a 19% median improvement from eliminating disk: the fact that some straggler tasks are caused by long times spent)

### Improving performance by understanding stragglers<a id="sec-1-7-4" name="sec-1-7-4"></a>

Understanding the root cause behind stragglers provides ways to improve performance by mitigating the underlying cause. In our early experiments, investigating straggler causes led us to find that the default file system on the EC2 instances we used, ext3, performs poorly for workloads with large numbers of parallel reads and writes, leading to stragglers. By changing the filesystem to ext4, we fixed stragglers and reduced median task time, reducing query runtime for queries in the big data benchmark by 17−58%

### How Does Scale Affect Results?<a id="sec-1-7-5" name="sec-1-7-5"></a>

## How Does Scale Affect Results?<a id="sec-1-8" name="sec-1-8"></a>

similar results

## Conclusion<a id="sec-1-9" name="sec-1-9"></a>

This paper undertook a detailed performance study of three workloads, and found that for those workloads, jobs are often bottlenecked on CPU and not I/O, network performance has little impact on job completion time, and many straggler causes can be identified and fixed.

# Tachyon: Reliable, Memory Speed Storage for Cluster Computing Frameworks<a id="sec-2" name="sec-2"></a>

## SoCC 14<a id="sec-2-1" name="sec-2-1"></a>

## Abstract<a id="sec-2-2" name="sec-2-2"></a>

While caching today improves read workloads, writes are either network or disk bound, as replication is used for fault-tolerance. Tachyon eliminates this bottleneck by pushing lineage, a well-known technique, into the storage layer.

Our evaluation shows that Tachyon outperforms in-memory HDFS by 110x for writes.It also improves the end-to-end latency of a realistic
workflow by 4x.

## introduction<a id="sec-2-3" name="sec-2-3"></a>

As the performance of many of these systems is I/O bound, traditional means of improving their speed is to cache data into memory.

To improve write performance, we present Tachyon, an in-memory storage system that achieves high throughput writes and reads, without compromising fault-tolerance.

### tow challenges:<a id="sec-2-3-1" name="sec-2-3-1"></a>

1.  bounding the recomputation cost for a long-running storage system.

    checkpoiont + linage, no replication
    
    Tachyon bounds the data recomputation cost, thus addressing the first challenge, by continuously checkpointing files asynchronously in the background. To select which files to checkpoint and when, we propose a novel algorithm, called the Edge algorithm, that provides an upper bound on the recomputation cost regardless of the workload’s access pattern.

2.  how to allocate resources for recomputations

    Tachyon provides resource allocation schemes that respect job priorities under two common cluster allocation models: strict priority and
    weighted fair sharing [31, 52].

## background<a id="sec-2-4" name="sec-2-4"></a>

## design overview<a id="sec-2-5" name="sec-2-5"></a>

## chekpointing<a id="sec-2-6" name="sec-2-6"></a>

## resource allocation<a id="sec-2-7" name="sec-2-7"></a>

## implementation<a id="sec-2-8" name="sec-2-8"></a>

## evaluation<a id="sec-2-9" name="sec-2-9"></a>

# Introducing SSDs to the Hadoop MapReduce Framework<a id="sec-3" name="sec-3"></a>

## Cloud Computing 14<a id="sec-3-1" name="sec-3-1"></a>

## Abstract<a id="sec-3-2" name="sec-3-2"></a>

This paper compares SSD performance to HDD performance within a Hadoop MapReduce framework. It identifies extensible best practices that can exploit SSD benefits within Hadoop frameworks when combined with high network bandwidth and increased parallel storage access.

Terasort benchmark results demonstrate that SSDs presently deliver significant costeffectiveness when they store intermediate Hadoop data, leaving HDDs to store Hadoop Distributed File System (HDFS) source data.

## Introduction<a id="sec-3-3" name="sec-3-3"></a>

While many options and strategies can improve MapReduce performance, introducing SSDs is a particularly attractive strategy because SSDs typically exhibit superior performance - lower latency, higher IO rates, and higher throughput - than HDDs.

### contributions<a id="sec-3-3-1" name="sec-3-3-1"></a>

We investigate and compare HDFS performance using SSDs and HDDs in different configurations

We examine Hadoop workflows and identify various performance bottlenecks within different configurations.

We explore cost-effective MapReduce configurations.

## HDFS characteristics<a id="sec-3-4" name="sec-3-4"></a>

### investigate environment<a id="sec-3-4-1" name="sec-3-4-1"></a>

1.  Raw Storage Device Performance

    We used the fio benchmark with the following options: (1) queue depth = 32, (2) 128KB block size, (3) sync I/O, and (4) direct I/O. System engineers typically allow enough queue depth since block device drivers significantly delay overall I/O performance with small queue depth. We found that configurations with queue depth more than 32 or with block size more than 128KB marginally changed the result.

2.  TestDFSIO

    Test Distributed File System I/O (TestDFSIO, here after DFSIO) is a widely-used, industry-standard benchmark that enables HDFS performance benchmarks. It distributes map tasks that read/write complete dummy data files on Datanodes for HDFS. Each map task reads the complete local file and writes a few statistical output lines. Reduce tasks simply gather the statistics for output

### storage vs Network<a id="sec-3-4-2" name="sec-3-4-2"></a>

We performed HDFS performance measurements using 1Gbit Ethernet links, 20Gbit links constructed by combining two 10Gbit Ethernet links via link aggregation (channel bonding), HDDs, and, SSDs. This enabled us to vary storage devices and network bandwidth to determine how various SSD and high network bandwidth combinations could improve throughput.

Employing more HDDs or SSDs can utilize the potential of high performance networks. Consequently, matching network bandwidth and I/O throughput is key to build cost-efficient system.

1.  Network bandwidth requirement for high performance storage systems

    KT = ND
    
    0.5KB >= D(N-1)

### Mutiple Readers/Writers<a id="sec-3-4-3" name="sec-3-4-3"></a>

Unlike when using HDDs, increasing the concurrent thread count improves HDFS performance when using SSDs because random SSD read performance approximates sequential read performance (the highest possible performance).

## MapReduce from a Storage Perspective<a id="sec-3-5" name="sec-3-5"></a>

The place to write intermediate data is configurable. We call the local storage keeping intermediate data intermediate storage.

Compressing map output is often used to reduce the amount of data stored in the intermediate storage (and transferred through the cluster network)

### Terasort Performance Analysis<a id="sec-3-5-1" name="sec-3-5-1"></a>

Key Terasort performance observations are:

(1) Map phase performance depends on the performance of HDFS read throughput (primarily sequential).

(2) Shuffle phase performance mainly depends on the random I/O performance when reading map output and writing intermediate data. To explicitly identify performance-degradation sources, we profiled CPU and I/O utilizations during investigations, depicted in Figure 8 and 9.

(3) Reduce phase performance depends more on the random read performance of the storage keeping intermediate data than on the HDFS storage media performance.

### Cost Analysis<a id="sec-3-5-2" name="sec-3-5-2"></a>

The previous section’s Terasort evaluation shows that introducing more DRAM or an SSD to Datanodes as intermediate data storage significantly reduces shuffle phase execution time. When also deploying SSDs as permanent HDFS storage, Hadoop map and reduce phase performance considerably improves.

However, SSD cost-per-bit is presently not inexpensive enough to replace all HDDs. Nevertheless, it is still rapidly decreasing. To estimate Datanode cost-effectiveness, we considered both Datanode configuration cost and Datanode performance.

## Conclusion<a id="sec-3-6" name="sec-3-6"></a>

Sufficient network bandwidth and more I/O parallelism to utilize the maximum SSD performance are critical to the performance of HDFS and Hadoop MapReduce framework.

Specifically, SSDs help when they are used as temporary storage for intermediate data which consists of heavy random and concurrent I/O. We observed that using SSDs for temporary storage and HDDs for permanent storage for HDFS is the most cost effective configuration today.

# FB-Tree: A B+-Tree for Flash-Based SSDs<a id="sec-4" name="sec-4"></a>

## IDEAS11 2011<a id="sec-4-1" name="sec-4-1"></a>

Proceedings of the 15th Symposium on International Database Engineering & Applications

## Abstract<a id="sec-4-2" name="sec-4-2"></a>

ash-based SSDs (Solid-State Drives) have become a mainstream alternative to magnetic disks for database servers.

Nevertheless, database systems, designed and optimized for magnetic disks, still do not fully exploit all the benets of the new technology

We propose the FB-tree: a combination of an adapted B+- tree, a storage manager, and a buer manager, all optimized for modern SSDs.\* SSD usage in Facebook.

Together the techniques enable writing to SSDs in relatively large blocks, thus achieving greater overall throughput. This is achieved by the out-of-place writing, whereby every time a modied index node is written, it is written to a new address, clustered with some other nodes that are written together.

While this constantly frees index nodes, the FB-tree does not introduce any garbage-collection overhead, instead relying on naturally occurring free-space segments of sufficient size.

As a consequence, the FB-tree outperforms a regular B+-tree in all scenarios tested. For instance, the throughput of a random workload of 75% updates increases by a factor of three using only two times the space of the B+-tree.

## Introduction<a id="sec-4-3" name="sec-4-3"></a>

At first, NAND  flash was intended for embedded storage, where its small size and low power consumption were the key benets.

In recent years, NAND flash storage has emerged into the world of mainstream storage, where its fast random access and low power consumption are considered its key benets.

In present database systems, most aspects are optimized with respect to the performance characteristics of magnetic disks, and hence these optimizations must be reconsidered [6, 8, 19].

One of the key parts of the data base internals that have to be reconsidered are the index methods and especially the ubiquitous B+-tree index structure.

proposals successfully enforcing a sequential workload of writes, either severely degrade the other properties of the existing B+-tree , or completely redesign the index structure [9, 20], making it difficult to implement and integrate with the existing systems.

### key features of SSD<a id="sec-4-3-1" name="sec-4-3-1"></a>

in this paper, we embrace two key features of SSDs|very fast random reads and the fact that larger writes are more efficient than smaller ones.

### contribution of this paper is two-fold<a id="sec-4-3-2" name="sec-4-3-2"></a>

First, we propose the adaptation of the write-optimized B-tree for SSDs by combining it with the specially designed buffer manager and storage manager. The combination, which we term the FB-tree, enables to improve the performance of any query/update workloads and does so with only slight 34 modications to the well known B+-tree.

Second, we report on extensive experimental study of the FB-tree and the B+-tree on modern SSDs.

## Motivation<a id="sec-4-4" name="sec-4-4"></a>

### SSD's rather similar sequential and random read speeds<a id="sec-4-4-1" name="sec-4-4-1"></a>

Hence, we discard clustering on SSDs. Removing this legacy optimization from the B+-tree indexing scheme opens up for other possible SSD optimizations, as no special placement of pages is required for getting a fast range scan.

### the performance of the write workloads has the potential to be improved.<a id="sec-4-4-2" name="sec-4-4-2"></a>

With a workload of random writes of 4{8KB caused by updating the B+-tree nodes in-place, an SSD will perform its absolute worst.

To improve on the write performance, a solution would be to write in larger sizes or ultimately write large sequential writes, and, notably, this has been the main concern when optimizing the B+-tree workloads or building new indexing schemes for flash and SSDs [9, 19, 20].

The SSD incurs somewhat the same trend but on a much smaller scale. The results are, however, encouraging enough to be used for improving the write workload of a B+-tree on an SSD.

With the combined fact that the placement of nodes in a B+-tree structure is no longer restricted by clustering and that performing relatively large writes improves performance.

we propose the FB-tree, which is a B+-tree indexing scheme where nodes are written in an out-of-place fashion also called copy-on-write.

## Related Work<a id="sec-4-5" name="sec-4-5"></a>

## FB-Tree<a id="sec-4-6" name="sec-4-6"></a>

First, we present the write-optimized B-tree , which is theB+-tree variant we choose as a basis of our proposal.

Next, we introduce our algorithms for the storage manager and the buffer manager.

### write-optimized B-tree<a id="sec-4-6-1" name="sec-4-6-1"></a>

1.  original paper

    G. Graefe. Write-Optimized B-Trees. In proc. of VLDB, pp. 672{683, 2004.}

2.  algorithm

    The main idea behind the write-optimized B-tree is borrowed from the log-structured le systems (LFS ), where every piece of data is written in a out-of-place fashion, collecting smaller writes into larger writes.
    
    In a normal LFS, writing out-of-place requires maintaining a mapping between a stable logical address and a frequently changing physical addresse.
    
    While this overhead is required for a le system, not having any control over the overlying data structure, the overhead can be diminished by instead incorporating the mapping into the index structure.
    
    While the write-optimized B-tree enables out-of-place writes, they do not automatically guarantee that a large group of such writes can be collected into a single large write. Careful consideration of the algorithms of the storage manager and the buer manager are necessary.

### storage manager<a id="sec-4-6-2" name="sec-4-6-2"></a>

While for disk-based systems, storage manager main contain algorithms for defragmentation or reorganization of nodes to improve reading performance, this is an unnecessary overhead for SSDs. For modern SSDs, the recommended disk scheduling and le system optimization is \no optimization" [2, 3]."

With this in mind we propose a simple storage manager that incurs no I/O overhead. The idea is to perform relatively large writes as much as possible. Thus, the storage manager manages the free space so that several nodes can be written at a time, claiming a relatively large chunk of free space. The simple policy is to choose the largest naturally occurring free space segment and write as many evicted nodes as possible contiguously in this segment.

One benefit of out-of-place writing is the possibility of exploiting the fact that B-tree nodes are not completely filled (approximately 68% on average ).

As evicted nodes can be of dierent sizes, the group of evicted nodes does not always t perfectly into the largest free space segment, possibly leaving some space in the segment unused. This potentially causes further unnecessary fragmentation, and hence, we propose to use a rened policy, the so-called exact t, which includes searching for a segment of free space that serves as the best t, leaving as little wasted space as possible. Such a segment of free space may not necessarily be the largest one\*\*\* buffer manager.

As the storage manager in FB-tree does not perform any defragmentation of nodes and free space, the amount of free space is especially important for maintaining a high probability of large free space segments

### Buffer Manager<a id="sec-4-6-3" name="sec-4-6-3"></a>

Probably the best-known buering mechanism is the Least-Recently-Used (LRU) buer, which simply evicts the page least recently used.

Treating buffer reads and buffer writes differently is motivated by the asymmetric performance of read and write on flash and SSDs, when reads are faster than writes.

As a consequence, proposals have been made [13, 14] for different flash-friendly replacement policies, the main idea of which is evicting clean pages before dirty pages.

What makes the proposed buffering mechanism significantly different from the other flash-optimized buffer managers, is the ability to evict several dirty pages at a time

### conclusion<a id="sec-4-6-4" name="sec-4-6-4"></a>

In the future, we plan to explore the concurrency algorithms of the FB-tree

# Conclusion<a id="sec-5" name="sec-5"></a>

第一篇论文《大数据处理框架的性能分析》为我们的研究提供了性能分析工具

这几篇论文的性能分析，实验方法我们可以模仿

tachyon的论文原文没有提它对于大数据分析的提升效果，因为理论上和实际上应该提升并不大，tachyon的商业公司的宣传不能信

tachyon把shuffle阶段的中间数据放在内存文件系统，不符合大数据处理追求高性价比的初衷

另一些研究把shuffle阶段的中间数据放在SSD上，性价比更高，很关键！

# 问题：<a id="sec-6" name="sec-6"></a>

为什么spark把shuffle中间数据放到磁盘上？

shuffle阶段的策略和算法还可以优化，有需要sort的，也有不需要sort的；不需要sort的应用直接使用hash table

不能把中间数据放到SSD上就完了，应当使用为SSD优化过的数据结构，例如FB-Tree

集群配置：网络吞吐量和磁盘吞吐量如何最优化配置？

使用本地存储如hdfs和统一外部存储如S3，对于网络吞吐量和性价比的影响

压缩shuffle阶段中间数据对于性能的影响？能否通过压缩提升io密集型应用的性能？

<div id="footnotes">
<h2 class="footnotes">Footnotes: </h2>
<div id="text-footnotes">

<div class="footdef"><sup><a id="fn.1" name="fn.1" class="footnum" href="#fnr.1">1</a></sup> <p>DEFINITION NOT FOUND.</p></div>

<div class="footdef"><sup><a id="fn.2" name="fn.2" class="footnum" href="#fnr.2">2</a></sup> <p>DEFINITION NOT FOUND.</p></div>

<div class="footdef"><sup><a id="fn.3" name="fn.3" class="footnum" href="#fnr.3">3</a></sup> <p>DEFINITION NOT FOUND.</p></div>

<div class="footdef"><sup><a id="fn.4" name="fn.4" class="footnum" href="#fnr.4">4</a></sup> <p>DEFINITION NOT FOUND.</p></div>


</div>
</div>
