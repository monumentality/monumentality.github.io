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
<li><a href="#sec-1">1. 多租户平台必须满足的要求：</a>
<ul>
<li><a href="#sec-1-1">1.1. Multi-tenancy:</a></li>
<li><a href="#sec-1-2">1.2. From an administrator’s perspective, multi-tenancy requirements are to</a></li>
<li><a href="#sec-1-3">1.3. 从用户角度看</a></li>
</ul>
</li>
<li><a href="#sec-2">2. 现有的多租户环境下的资源调度算法</a></li>
<li><a href="#sec-3">3. 动机</a></li>
<li><a href="#sec-4">4. 与Amazon竞价系统的不同</a></li>
<li><a href="#sec-5">5. 与敏捷开发的比较：</a></li>
<li><a href="#sec-6">6. 基于竞价方式的资源调度</a>
<ul>
<li><a href="#sec-6-1">6.1. 机制</a></li>
<li><a href="#sec-6-2">6.2. trade off</a></li>
<li><a href="#sec-6-3">6.3. 策略；</a>
<ul>
<li><a href="#sec-6-3-1">6.3.1. 资源分类：</a></li>
<li><a href="#sec-6-3-2">6.3.2. 用户只需要设置：</a></li>
<li><a href="#sec-6-3-3">6.3.3. 用户可以实时调整自己的策略</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# 多租户平台必须满足的要求：<a id="sec-1" name="sec-1"></a>

## Multi-tenancy:<a id="sec-1-1" name="sec-1-1"></a>

 Multi-tenancy is the ability of a single instance of software to serve multiple tenants. A tenant is a group of users that have the same view of the system. Hadoop, as an enterprise data hub, naturally demands multi-tenancy. Creating different instances of Hadoop for various users or functions is not acceptable as it makes it harder to share data across departments and creates silos.
多租户是使用某个软件的一个实例来服务多个租户。对系统拥有某一相同视图的一组用户是一个租户。通常租户可以是企业内部的部门或用户

## From an administrator’s perspective, multi-tenancy requirements are to<a id="sec-1-2" name="sec-1-2"></a>

Ensure SLAs are met 满足每个用户的服务层协议
Guarantee isolation 提供资源隔离
Enforce quotas      约束每个用户的配额
Establish security and delegation 确立安全和委派
Ensure low cost operations and simpler manageability 低运维成本和简化管理

## 从用户角度看<a id="sec-1-3" name="sec-1-3"></a>

即满足用户的服务层协议

# 现有的多租户环境下的资源调度算法<a id="sec-2" name="sec-2"></a>

yarn:优先级，队列，分层级的优先级列队，最小最大公平算法
mesos：DRF 主要资源公平算法
yarn对多租户的支持更多，mesos更侧重公平，缺少多租户支持

# 动机<a id="sec-3" name="sec-3"></a>

公平算法，对于高优先级的任务不公平
租户设定优先级，难以控制
实现一个竞价调度算法，使更有价值的作业获得更多资源，实现资源的公平和高效利用，需要引用一些经济理论来证明

# 与Amazon竞价系统的不同<a id="sec-4" name="sec-4"></a>

amazon把资源按照固定价格卖给客户，客户把自己的某些时段的空闲资源放到竞价系统交易
这个调度器则适用于某个组织内部，多部门或用户共享有限资源的环境，目的是实现资源的公平和高效利用，避免恶意竞争，鼓励良性竞争。

# 与敏捷开发的比较：<a id="sec-5" name="sec-5"></a>

敏捷开发方法中存在一个类似的过程：即团队确定了一个迭代周期需要完成的所有ticket之后，团队中所有成员对所有ticket进行竞价，所需工作时最少的程序员将获得该ticket。这个调度器和ticket竞价的策略是一致的，即鼓励团队内部成员的良性竞争，公平高效地利用团队资源。

# 基于竞价方式的资源调度<a id="sec-6" name="sec-6"></a>

## 机制<a id="sec-6-1" name="sec-6-1"></a>

平台资源数量是有限的，发给每个租户虚拟币，租户使用虚拟币来对资源竞标，出价高者获得资源
当资源紧张时，出价更高的租户将抢占出价低的租户使用的资源。由于允许抢占，作业/服务必须支持故障恢复

## trade off<a id="sec-6-2" name="sec-6-2"></a>

用户使用更复杂，用户需要权衡每个作业的重要性，选择为每个作业给出合适的竞价
用户需要关注整个平台的竞价排名，确保自己的作业按照预期完成
用户需要研究整个平台的竞价排名状况，与其他租户竞争，才能获得更多的资源占用；比如选择空闲时刻占据资源，甚至操纵竞价

## 策略；<a id="sec-6-3" name="sec-6-3"></a>

### 资源分类：<a id="sec-6-3-1" name="sec-6-3-1"></a>

空闲资源、可抢占资源、自动匹配最高竞价的不可抢占资源

### 用户只需要设置：<a id="sec-6-3-2" name="sec-6-3-2"></a>

只使用空闲资源
抢占BidHigh区间以下的资源
自动匹配最高竞价，使用所有能抢占的资源

### 用户可以实时调整自己的策略<a id="sec-6-3-3" name="sec-6-3-3"></a>
