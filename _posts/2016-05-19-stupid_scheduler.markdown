---
layout: post
title:  "stupid scheduler"
date: 2016-05-19
categories: cs
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 应用需求：</a>
<ul>
<li><a href="#sec-1-1">1.1. 多租户平台必须满足的要求：</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1. Multi-tenancy:</a></li>
<li><a href="#sec-1-1-2">1.1.2. From an administrator’s perspective, multi-tenancy requirements are to</a></li>
<li><a href="#sec-1-1-3">1.1.3. 从用户角度看</a></li>
</ul>
</li>
<li><a href="#sec-1-2">1.2. 支持服务和大数据</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. 服务：</a></li>
<li><a href="#sec-1-2-2">1.2.2. 大数据：</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#sec-2">2. background: 现有的多租户环境下的资源调度算法</a></li>
<li><a href="#sec-3">3. 动机</a></li>
<li><a href="#sec-4">4. 基于竞价方式的资源调度</a>
<ul>
<li><a href="#sec-4-1">4.1. 机制</a></li>
<li><a href="#sec-4-2">4.2. 策略；</a>
<ul>
<li><a href="#sec-4-2-1">4.2.1. 资源分类：</a></li>
<li><a href="#sec-4-2-2">4.2.2. 市场机制：</a></li>
<li><a href="#sec-4-2-3">4.2.3. 伸缩策略：</a></li>
<li><a href="#sec-4-2-4">4.2.4. 支持更多请求：</a></li>
<li><a href="#sec-4-2-5">4.2.5. 提供的接口</a></li>
</ul>
</li>
<li><a href="#sec-4-3">4.3. trade off</a></li>
</ul>
</li>
<li><a href="#sec-5">5. 理论证明：</a></li>
<li><a href="#sec-6">6. bidscheduler实现</a>
<ul>
<li><a href="#sec-6-1">6.1. 两级调度架构：</a></li>
<li><a href="#sec-6-2">6.2. 提供的接口：</a></li>
<li><a href="#sec-6-3">6.3. 实现：</a>
<ul>
<li><a href="#sec-6-3-1">6.3.1. 数据结构：</a></li>
<li><a href="#sec-6-3-2">6.3.2. 数据初始化</a></li>
<li><a href="#sec-6-3-3">6.3.3. allocate方法：</a></li>
<li><a href="#sec-6-3-4">6.3.4. has<sub>reliable</sub><sub>resources</sub></a></li>
<li><a href="#sec-6-3-5">6.3.5. can<sub>preempt</sub><sub>reliable</sub><sub>resources</sub></a></li>
<li><a href="#sec-6-3-6">6.3.6. has<sub>restricted</sub><sub>resources</sub></a></li>
<li><a href="#sec-6-3-7">6.3.7. allocate<sub>task</sub></a></li>
<li><a href="#sec-6-3-8">6.3.8. allocate<sub>task</sub><sub>restricted</sub></a></li>
<li><a href="#sec-6-3-9">6.3.9. bidscheduler.py</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#sec-7">7. 待解决的问题：</a>
<ul>
<li><a href="#sec-7-1">7.1. 架构的重构</a></li>
<li><a href="#sec-7-2">7.2. 性能问题，一秒能够处理多少调度请求</a></li>
<li><a href="#sec-7-3">7.3. 分布式</a></li>
<li><a href="#sec-7-4">7.4. 一致性，容错</a></li>
<li><a href="#sec-7-5">7.5. 调度作业与调度workspace的不同：</a></li>
</ul>
</li>
<li><a href="#sec-8">8. Background or Discussion</a>
<ul>
<li><a href="#sec-8-1">8.1. Dynamic Proportional Share Scheduling in Hadoop</a>
<ul>
<li><a href="#sec-8-1-1">8.1.1. Abstract</a></li>
<li><a href="#sec-8-1-2">8.1.2. Introduction</a></li>
<li><a href="#sec-8-1-3">8.1.3. Hadoop MapReduce:</a></li>
<li><a href="#sec-8-1-4">8.1.4. Design</a></li>
<li><a href="#sec-8-1-5">8.1.5. evaluation</a></li>
<li><a href="#sec-8-1-6">8.1.6. discussion</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# 应用需求：<a id="sec-1" name="sec-1"></a>

## 多租户平台必须满足的要求：<a id="sec-1-1" name="sec-1-1"></a>

### Multi-tenancy:<a id="sec-1-1-1" name="sec-1-1-1"></a>

Multi-tenancy is the ability of a single instance of software to serve multiple tenants. A tenant is a group of users that have the same view of the system. Hadoop, as an enterprise data hub, naturally demands multi-tenancy. Creating different instances of Hadoop for various users or functions is not acceptable as it makes it harder to share data across departments and creates silos.

多租户是使用某个软件的一个实例来服务多个租户。对系统拥有某一相同视图的一组用户是一个租户。通常租户可以是企业内部的部门或用户

### From an administrator’s perspective, multi-tenancy requirements are to<a id="sec-1-1-2" name="sec-1-1-2"></a>

Ensure SLAs are met 满足每个用户的服务层协议

Guarantee isolation 提供资源隔离

Enforce quotas      约束每个用户的配额

Establish security and delegation 确立安全和委派

Ensure low cost operations and simpler manageability 低运维成本和简化管理

### 从用户角度看<a id="sec-1-1-3" name="sec-1-1-3"></a>

即满足用户的服务层协议

## 支持服务和大数据<a id="sec-1-2" name="sec-1-2"></a>

### 服务：<a id="sec-1-2-1" name="sec-1-2-1"></a>

web服务的sla非常严格，云计算提供商都声称其提供的计算资源，cpu，内存是隔离的

web服务所需资源，下限由应用程序设计决定，随并发访问量线性增长，既需要预留资源，还需要能弹性伸缩

### 大数据：<a id="sec-1-2-2" name="sec-1-2-2"></a>

分布式应用，总的能够使用的资源上限由数据集大小决定，每个container的大小由应用程序的设计决定

hadoop和yarn和mesos和kubernetes的做法都是：
用户提交请求时，指定所需的每个container大小，需要多少个container

# background: 现有的多租户环境下的资源调度算法<a id="sec-2" name="sec-2"></a>

yarn:优先级，队列，分层级的优先级列队，最小最大公平算法

mesos：DRF 主要资源公平算法

yarn对多租户的支持更多，mesos更侧重公平，缺少多租户支持

# 动机<a id="sec-3" name="sec-3"></a>

公平算法，所运行作业本身的价值不同；租户设定优先级，没有一个全知全能的管理员来设定优先级。不能有效的配置资源，不能有效地管理动态资源池。资源利用率较低。

基于自由市场的经济模型实现一个调度算法，依靠市场的资源配置功能实现计算资源的分配，使最高估价的作业最先得到资源。

当运行在支持资源弹性伸缩的公有云，私有云，混合云环境下时，还可以依据系统中的资源需求量和出价，调整资源供给，达到均衡价格

另外，通过把资源分为可靠资源和不可靠资源，cgroup动态调整资源配额的技术，在充分利用空闲资源，满足更多资源请求的同时，确保不中断正在运行的作业，只降低这些作业的性能。

通过以上机制，此资源调度框实现了利用市场机制配置资源，动态调整资源供给达到价格均衡，并且提升了集群的资源利用率，满足更多的资源请求。

# 基于竞价方式的资源调度<a id="sec-4" name="sec-4"></a>

## 机制<a id="sec-4-1" name="sec-4-1"></a>

平台资源数量是有限的，发给每个租户虚拟币，租户使用虚拟币来对资源竞标，出价高者获得资源

## 策略；<a id="sec-4-2" name="sec-4-2"></a>

### 资源分类：<a id="sec-4-2-1" name="sec-4-2-1"></a>

可靠资源，受限制资源
可靠资源和受限制资源都是通过cgroup的cpu和内存的上限控制
受限制资源受cgroup的软限制控制，当系统资源不足的时候，优先释放受软限制的资源

### 市场机制：<a id="sec-4-2-2" name="sec-4-2-2"></a>

把集群初始化后所有的可用资源作为可靠资源，
有空闲资源时，所有请求都获得可靠资源
当可靠资源分配完毕后，对于一个新的资源请求a，
如果a出价高于已经分配到可靠资源的请求b，把b转换为不可靠资源，a获得可靠资源；如果a出价低于所有已获得可靠资源的请求，检查系统是否有空闲资源（系统资源利用率低），如果有，就分给a不可靠资源。

### 伸缩策略：<a id="sec-4-2-3" name="sec-4-2-3"></a>

当系统中平均资源价格高于扩容代价时，系统进行扩容
当系统中平均资源价格低于收缩收益时，系统进行收缩

### 支持更多请求：<a id="sec-4-2-4" name="sec-4-2-4"></a>

虽然系统的能分配的可靠资源是有限的，但是很多分配出去的资源没有被充分利用，这些资源作为不可靠资源可以被其他出价更低的资源请求使用。当原来申请了可靠资源但是没有使用的作业开始使用这些资源时，使用不可靠资源的请求就会被降低资源使用量，释放出的资源被自动分给使用可靠资源的应用程序。

### 提供的接口<a id="sec-4-2-5" name="sec-4-2-5"></a>

allocate(job<sub>allocation</sub><sub>request</sub>)
release(allocation)
change<sub>bidprice</sub>(allocation)

## trade off<a id="sec-4-3" name="sec-4-3"></a>

用户使用更复杂，用户需要权衡每个作业的重要性，选择为每个作业给出合适的竞价
用户需要关注整个平台的竞价排名，确保自己的作业按照预期完成

# 理论证明：<a id="sec-5" name="sec-5"></a>

之前的计算资源价值最大的证明方法不好，因为已经被市场理论所证明，没有必要再重述

使用需求和供给模型来简述系统的合理性，相关论文都不需要太多理论证明

# bidscheduler实现<a id="sec-6" name="sec-6"></a>

## 两级调度架构：<a id="sec-6-1" name="sec-6-1"></a>

bidscheduler作为一个通用的调度框架，应当采用资源调度和任务调度解耦合的架构。

由于bidscheduler考虑多租户，考虑作业价值的不同，不关注绝对公平，因此与yarn更类似，采用了类似yarn的两级调度架构。

bidscheduler作为 rasource manager， 之上可以运行多种application master, 我们的系统可以看做是一个 workspace master。之后将尝试，支持更多的计算框架，比如spark，比如崔嵬之前的跑作业的计算框架。

为了之后可以支持更多的计算框架，借鉴沿用了yarn的很多概念。

vclustermgr可看做是一个application master的实现。一个mapreduce job包含多个task, 类似于一个workspace包含多个容器。因此，还是沿用了yarn中的job和task的概念

存在一个问题：resource manager分给mapreducerMaster一个容器，就启动一个task，不需要等到所有容器都分配够了才启动执行；那么，一个workspace包含多个容器，是all or nothing 还是分开处理

## 提供的接口：<a id="sec-6-2" name="sec-6-2"></a>

allocate(jobAllocationRequest)
release(allocationid)

vclustermgr中create<sub>cluster函数调用allocation</sub>

    # call bidscheduler.allocate, get resources
    jobAllocationRequest = {
        jobid: clusterid,
        userid: json.loads(user_info)["userid"],
        tasks: clustersize,
        resourcesPerTask: containersize,
        bidprice: bidprice
    }
    import bidscheduler
    job_allocations = bidscheduler.allocate(jobAllocationRequest)

vclustermgr中调用release<sub>allocation</sub>

## 实现：<a id="sec-6-3" name="sec-6-3"></a>

### 数据结构：<a id="sec-6-3-1" name="sec-6-3-1"></a>

把分配出去的资源分为两类：
可靠资源，即受到隔离保护的资源，使用cgroup的cpu和memeory保护
受限制资源：使用cgroup的软限制分配的资源，当系统cpu或者内存不足时，将会减少这些容器的资源

    class AllocationOfTask(object):
        __slots__ = 'uuid','userid','jobid','taskid','resources','bidprice','type'
    
    class AllocationOfMachine(object):
        __slots__ = 'machineid',"resources","reliable_resources_allocation_summary",
                    'reliable_allocation','curr_usage', 'restricted_allocation'

在httprest的main函数中调用以下init方法：

### 数据初始化<a id="sec-6-3-2" name="sec-6-3-2"></a>

    allocations = {}
    nodemanager
    def init_allocations():
        global allocations
        global nodemanager
        machines = nodemanager.get_allnodes()
        for machine in machines:
            allocation = AllocationOfMachine()
            allocation.machineid = machine
            allocation.resources = 100
            allocation.reliable_resources_allocation_summary = 0
            allocation.reliable_allocation = []
            allocation.restricted_allocation = []
    
            allocations[machine] = allocation

### allocate方法：<a id="sec-6-3-3" name="sec-6-3-3"></a>

接受job的调度请求，为job中包含的每一个task分配资源，然后把所有的资源打包返回：
选择可靠资源剩余最多的机器，检查是否可以分配可靠资源，如果可以即分配
否则，选择使用率最低的机器，分配受限制资源

    def allocate(job_allocation_request):
        global allocations
        job_allocation_response = []
        sorted(allocations,lambda x: x.reliable_resources_allocation_summary )
    
        # 先从可靠资源最多的机器分配资源
        for i in range(job_allocation_request['tasks_count']):
            task_allocation_request = {
                userid: job_allocation_request['userid'],
                jobid: job_allocation_request['jobid'],
                taskid: i,
                bidprice: job_allocation_request['bidprice'],
                resources: job_allocation_request['resources'],
            }
            if( has_reliable_resource(allocations[i],task_allocation_request)
                or can_preempt_reliable_resources(allocations[i],task_allocation_request)):
                task_allocation_response = allocate_task(task_allocation_request)
                job_allocation_response.add(task_allocation_response)
            else:
                break
    
        if (job_allocation_response.size == job_allocation_request['taskcount']):
            return job_allocation_response
        else:
            # 选择使用率最低的机器，分配restricted_resources
            global usage_per_machine
            sorted(usage_per_machine, lambda x: x.cpu_utilization, reverse=True)
            for i in range(job_allocation_response.size..job_allocation_request['taskcount']):
                machineid = usage_per_machine[i]['machineid']
                allocation_of_machine = allocations[machineid]
                task_allocation_request = {
                    userid: job_allocation_request['userid'],
                    jobid: job_allocation_request['jobid'],
                    taskid: i,
                    bidprice: job_allocation_request['bidprice'],
                    resources: job_allocation_request['resources']
                }
                task_allocation_response = allocate_restricted(allocation_of_machine,task_allocation_request)
                job_allocation_response.add(task_allocation_response)
    
        return job_allocation_response

### has<sub>reliable</sub><sub>resources</sub><a id="sec-6-3-4" name="sec-6-3-4"></a>

    def has_reliable_resources(allocation_of_machine,task_allocation_request):
        if(task_allocation_request['resource']
           +allocation_of_machine.reliable_resources_allocation_summary
           < allocation_of_machine.resources):
            return True
        else:
            return False

### can<sub>preempt</sub><sub>reliable</sub><sub>resources</sub><a id="sec-6-3-5" name="sec-6-3-5"></a>

    def can_preempt_reliable_resources(taskAllocationRequest):
        to_be_preempted=0
        for a in reliable_allocation:
            if (a.bidprice< task_allocation_request['bidprice']):
                to_be_preempted += a.bidprice
                if to_be_preempted > task_allocation_request['resource']:
                    return True
            else:
                break
            return False

### has<sub>restricted</sub><sub>resources</sub><a id="sec-6-3-6" name="sec-6-3-6"></a>

    def has_restricted_resources(allocation_of_machine,task_allocation_request):
        if(task_allocation_request['resources']
           + curr_usage
           < allocation_of_machine.resources * 0.8):
            return True
        else:
            return False

### allocate<sub>task</sub><a id="sec-6-3-7" name="sec-6-3-7"></a>

    import uuid, bisect
    def allocate_task(allocation_of_machine,task_allocation_request):
        if(has_reliable_resources(request)):
            allocation_of_task = AllocationOfTask()
            allocation_of_task.id = uuid.uuid4()
            allocation_of_task.userid = task_allocation_request['userid']
            allocation_of_task.jobid = task_allocation_request['jobid']
            allocation_of_task.taskid = task_allocation_request['taskid']
            allocation_of_task.bidprice = task_allocation_request['bidprice']
            allocation_of_task.type = 'reliable'
            bisect.insort(allocation_of_machine.reliable_allocation, allocation_of_task, lambda x: x.bidprice)
            return {status:success, allocation:allocation_of_task}
    
        if(can_preempt_reliable_resources(task_allocation_request)):
            can_preempt = 0
            can_preempt_count = 0
            # 把被抢占的可靠资源变成受限制资源
            for i,a in reliableAllocation:
                can_preempt+=a['slots']
                can_preempt_count+=1
                a.type = 'restricted'
                import bisect
                bisect.insort(allocation_of_machine.restricted_allocation,a, lambda x: x.bidprice)
                # to-do 调整这些容器的cgroup设置，使用软限制模式，只能使用空闲资源
    
                if can_preempt>=task_allocation_request['resources']:
                    break
                # 把被抢占的可靠资源从reliable_allocation中删除
            del reliable_allocations[0..can_preempt_count]
    
            allocation_of_task = AllocationOfTask()
            allocation_of_task.id = uuid.uuid4()
            allocation_of_task.userid = task_allocation_request['userid']
            allocation_of_task.jobid = task_allocation_request['jobid']
            allocation_of_task.taskid = task_allocation_request['taskid']
            allocation_of_task.bidprice = task_allocation_request['bidprice']
            allocation_of_task.type = 'reliable'
            bisect.insort(allocation_of_machine.reliable_allocation,AllocationOfTask)
            return {status:success, allocation:allocation}
    
        if(has_restricted_resources(task_allocation_request)):
            allocation_of_task = AllocationOfTask()
            allocation_of_task.id = uuid.uuid4()
            allocation_of_task.userid = task_allocation_request['userid']
            allocation_of_task.jobid = task_allocation_request['jobid']
            allocation_of_task.taskid = task_allocation_request['taskid']
            allocation_of_task.bidprice = task_allocation_request['bidprice']
            allocation_of_task.type = 'restricted'
            bisect.insort(allocation_of_machine.restricted_allocation,AllocationOfTask)
            return {status:'success', allocation:allocation_of_task}
    
        else:
            return {status: 'failed'}

### allocate<sub>task</sub><sub>restricted</sub><a id="sec-6-3-8" name="sec-6-3-8"></a>

    def allocate_task_restricted(allocation_of_machine,task_allocation_request):
        if(has_restricted_resources(task_allocation_request)):
            allocation_of_task = AllocationOfTask()
            allocation_of_task.id = uuid.uuid4()
            allocation_of_task.userid = task_allocation_request['userid']
            allocation_of_task.jobid = task_allocation_request['jobid']
            allocation_of_task.taskid = task_allocation_request['taskid']
            allocation_of_task.bidprice = task_allocation_request['bidprice']
            allocation_of_task.type = 'restricted'
            bisect.insort(allocation_of_machine.restricted_allocation,AllocationOfTask)
            return {status:'success', allocation:allocation_of_task}
    
        else:
            return {status: 'failed'}

### bidscheduler.py<a id="sec-6-3-9" name="sec-6-3-9"></a>

1.  依赖的库

        from monitor import summary_resources, summary_usage, curr_usage
        from monitor import summary_usage_per_user, summary_usage_per_user
        from monitor import curr_usage_per_machine
        import nodemgr

2.  bidscheduler.py

        from monitor import summary_resources, summary_usage, curr_usage
        from monitor import summary_usage_per_user, summary_usage_per_user
        from monitor import curr_usage_per_machine
        import nodemgr
        
        class AllocationOfTask(object):
            __slots__ = 'uuid','userid','jobid','taskid','resources','bidprice','type'
        
        class AllocationOfMachine(object):
            __slots__ = 'machineid',"resources","reliable_resources_allocation_summary",
                        'reliable_allocation','curr_usage', 'restricted_allocation'
        
        allocations = {}
        nodemanager
        def init_allocations():
            global allocations
            global nodemanager
            machines = nodemanager.get_allnodes()
            for machine in machines:
                allocation = AllocationOfMachine()
                allocation.machineid = machine
                allocation.resources = 100
                allocation.reliable_resources_allocation_summary = 0
                allocation.reliable_allocation = []
                allocation.restricted_allocation = []
        
                allocations[machine] = allocation
        
        def has_reliable_resources(allocation_of_machine,task_allocation_request):
            if(task_allocation_request['resource']
               +allocation_of_machine.reliable_resources_allocation_summary
               < allocation_of_machine.resources):
                return True
            else:
                return False
        
        def can_preempt_reliable_resources(taskAllocationRequest):
            to_be_preempted=0
            for a in reliable_allocation:
                if (a.bidprice< task_allocation_request['bidprice']):
                    to_be_preempted += a.bidprice
                    if to_be_preempted > task_allocation_request['resource']:
                        return True
                else:
                    break
                return False
        
        def has_restricted_resources(allocation_of_machine,task_allocation_request):
            if(task_allocation_request['resources']
               + curr_usage
               < allocation_of_machine.resources * 0.8):
                return True
            else:
                return False
        
        import uuid, bisect
        def allocate_task(allocation_of_machine,task_allocation_request):
            if(has_reliable_resources(request)):
                allocation_of_task = AllocationOfTask()
                allocation_of_task.id = uuid.uuid4()
                allocation_of_task.userid = task_allocation_request['userid']
                allocation_of_task.jobid = task_allocation_request['jobid']
                allocation_of_task.taskid = task_allocation_request['taskid']
                allocation_of_task.bidprice = task_allocation_request['bidprice']
                allocation_of_task.type = 'reliable'
                bisect.insort(allocation_of_machine.reliable_allocation, allocation_of_task, lambda x: x.bidprice)
                return {status:success, allocation:allocation_of_task}
        
            if(can_preempt_reliable_resources(task_allocation_request)):
                can_preempt = 0
                can_preempt_count = 0
                # 把被抢占的可靠资源变成受限制资源
                for i,a in reliableAllocation:
                    can_preempt+=a['slots']
                    can_preempt_count+=1
                    a.type = 'restricted'
                    import bisect
                    bisect.insort(allocation_of_machine.restricted_allocation,a, lambda x: x.bidprice)
                    # to-do 调整这些容器的cgroup设置，使用软限制模式，只能使用空闲资源
        
                    if can_preempt>=task_allocation_request['resources']:
                        break
                    # 把被抢占的可靠资源从reliable_allocation中删除
                del reliable_allocations[0..can_preempt_count]
        
                allocation_of_task = AllocationOfTask()
                allocation_of_task.id = uuid.uuid4()
                allocation_of_task.userid = task_allocation_request['userid']
                allocation_of_task.jobid = task_allocation_request['jobid']
                allocation_of_task.taskid = task_allocation_request['taskid']
                allocation_of_task.bidprice = task_allocation_request['bidprice']
                allocation_of_task.type = 'reliable'
                bisect.insort(allocation_of_machine.reliable_allocation,AllocationOfTask)
                return {status:success, allocation:allocation}
        
            if(has_restricted_resources(task_allocation_request)):
                allocation_of_task = AllocationOfTask()
                allocation_of_task.id = uuid.uuid4()
                allocation_of_task.userid = task_allocation_request['userid']
                allocation_of_task.jobid = task_allocation_request['jobid']
                allocation_of_task.taskid = task_allocation_request['taskid']
                allocation_of_task.bidprice = task_allocation_request['bidprice']
                allocation_of_task.type = 'restricted'
                bisect.insort(allocation_of_machine.restricted_allocation,AllocationOfTask)
                return {status:'success', allocation:allocation_of_task}
        
            else:
                return {status: 'failed'}
        
        def allocate_task_restricted(allocation_of_machine,task_allocation_request):
            if(has_restricted_resources(task_allocation_request)):
                allocation_of_task = AllocationOfTask()
                allocation_of_task.id = uuid.uuid4()
                allocation_of_task.userid = task_allocation_request['userid']
                allocation_of_task.jobid = task_allocation_request['jobid']
                allocation_of_task.taskid = task_allocation_request['taskid']
                allocation_of_task.bidprice = task_allocation_request['bidprice']
                allocation_of_task.type = 'restricted'
                bisect.insort(allocation_of_machine.restricted_allocation,AllocationOfTask)
                return {status:'success', allocation:allocation_of_task}
        
            else:
                return {status: 'failed'}
        
        def allocate(job_allocation_request):
            global allocations
            job_allocation_response = []
            sorted(allocations,lambda x: x.reliable_resources_allocation_summary )
        
            # 先从可靠资源最多的机器分配资源
            for i in range(job_allocation_request['tasks_count']):
                task_allocation_request = {
                    userid: job_allocation_request['userid'],
                    jobid: job_allocation_request['jobid'],
                    taskid: i,
                    bidprice: job_allocation_request['bidprice'],
                    resources: job_allocation_request['resources'],
                }
                if( has_reliable_resource(allocations[i],task_allocation_request)
                    or can_preempt_reliable_resources(allocations[i],task_allocation_request)):
                    task_allocation_response = allocate_task(task_allocation_request)
                    job_allocation_response.add(task_allocation_response)
                else:
                    break
        
            if (job_allocation_response.size == job_allocation_request['taskcount']):
                return job_allocation_response
            else:
                # 选择使用率最低的机器，分配restricted_resources
                global usage_per_machine
                sorted(usage_per_machine, lambda x: x.cpu_utilization, reverse=True)
                for i in range(job_allocation_response.size..job_allocation_request['taskcount']):
                    machineid = usage_per_machine[i]['machineid']
                    allocation_of_machine = allocations[machineid]
                    task_allocation_request = {
                        userid: job_allocation_request['userid'],
                        jobid: job_allocation_request['jobid'],
                        taskid: i,
                        bidprice: job_allocation_request['bidprice'],
                        resources: job_allocation_request['resources']
                    }
                    task_allocation_response = allocate_restricted(allocation_of_machine,task_allocation_request)
                    job_allocation_response.add(task_allocation_response)
        
            return job_allocation_response

# 待解决的问题：<a id="sec-7" name="sec-7"></a>

## 架构的重构<a id="sec-7-1" name="sec-7-1"></a>

重构成一个独立的调度框架，包括resourcemanager和nodemanager两个独立的进程

## 性能问题，一秒能够处理多少调度请求<a id="sec-7-2" name="sec-7-2"></a>

## 分布式<a id="sec-7-3" name="sec-7-3"></a>

为了提高每秒处理的调度请求，未来将重构为分布式设计，多个resourcemanager协同工作

## 一致性，容错<a id="sec-7-4" name="sec-7-4"></a>

还是需要保存在etcd中
最佳方案是，高可靠的内存数据库

## 调度作业与调度workspace的不同：<a id="sec-7-5" name="sec-7-5"></a>

作业可以排队
创建workspace的请求立即得到回应，为了用户的交互体验，不能排队
之后再考虑是否需要队列

# Background or Discussion<a id="sec-8" name="sec-8"></a>

## Dynamic Proportional Share Scheduling in Hadoop<a id="sec-8-1" name="sec-8-1"></a>

### Abstract<a id="sec-8-1-1" name="sec-8-1-1"></a>

It allows users to control their allocated capacity by adjusting their spending over time.

This simple mechanism allows the scheduler to make more efficient decisions about which jobs and users to prioritize and gives users the tool to optimize and customize their allocations to fit the importance and requirements of their jobs.

We envision our scheduler to be used by deadline or budget optimizing agents on behalf of users.

We show that our scheduler enforces service levels more accurately and also scales to more users with distinct service levels than existing schedulers.

### Introduction<a id="sec-8-1-2" name="sec-8-1-2"></a>

we have developed the Dynamic Priority (DP) scheduler, a novel scheduler that extends the existing FIFO and fair-share schedulers in Hadoop.

这个跟我们的一样，作为hadoop已有的调度框架的一个插件，利用hadoop已经实现的很多接口和方法。

This scheduler plug-in allows users to purchase and bid for capacity or quality of service levels dynamically. This simple mechanism allows the scheduler to make more efficient decisions about which jobs and users to prioritize and gives users the tool to optimize and customize their allocations to fit the importance and requirements of their jobs.

这一点跟我们完全一样

it gives users the incentive to scale back their jobs when demand is high, since the cost of running on a slot is then also more expensive.

这一点并不重要

The capacity allotted, represented by Map and Reduce task slots, is proportional to the spending rate a user is willing to pay for a slot and inversely proportional to the aggregate spending rate of all existing users.

主要不同！他并不是完全的基于用户出价来抢占资源的，而是按照用户出价占总报价的比例来分配资源。而我们的策略是：出价最高的用户可以得到他需要的所有资源。按比例分配并不能使真正重要的任务获得优先执行！不符合经济规律。

### Hadoop MapReduce:<a id="sec-8-1-3" name="sec-8-1-3"></a>

review the current hadoop schedulers

### Design<a id="sec-8-1-4" name="sec-8-1-4"></a>

The primary design goal of our Hadoop task scheduler is to allow capacity distribution across concurrent users to change dynamically based on user preferences.

Traditional priority systems that try to guess user priority are too inaccurate , and unregulated user priorities assume trusted small groups of users.

Our scheduler automates capacity allocation and redistribution in a regulated task slot resource market.

1.  Mechanism

    The core of our design is a proportional share resource allocation mechanism that allows users to purchase or be granted a queue priority budget.
    
    This budget may be used to set spending rates denoting the willingness to pay a certain amount of the budget per Hadoop map or reduce task slot per time unit.
    
    The time unit is configurable, and referred to as allocation interval. It is typically set to somewhere between 10 seconds and 1 minute.
    
    In each allocation interval the scheduler:
    
    &#x2014;aggregates all spending rates s from all current users to calculate the Hadoop cluster price, p,
    
    &#x2014;for all users, allocates (si/p) × c task slots (both mappers and reducers) to user i, where si, is the spending rate of user i, and c is the aggregate slot capacity of the cluster
    
    &#x2014;for all users, deducts si × ui from budget b where ui, is the number of slots used by user i
    
    用户管理员可根据集群资源总量为每个用户分配一定的预算。用户可根据自己的需要动态调整自己的消费率，即每个时间单元内单个slot的价钱。在每个时间单元内，调度器按照以下步骤计算每个用户获得的资源量：
    
    1.  计算所有用户的消费率之和p
    
    2.  对于每个用户分配i，分配si/p\*c 的资源量，si是用户i的消费率，c为hadoop集群中资源总数
    
    3.  对于每个用户i，从其预算中扣除si \* ui，ui为用户正使用的slot数目
    
    The key feature of this mechanism is that it discourages free-riding and gaming by users.
    
    恰好相反，我们想要创造一个自由市场，鼓励博弈。为什么本文作者专门消除这些行为？
    
    The disadvantage is less capacity predictability and more variation in capacity allocated to an application.
    
    This introduces the difficulty of making spending rate decisions to meet the SLA and deadline requirements.
    
    Possible starvation of low-priority (low-spending) tasks can be mitigated by using the standard approach in Hadoop of limiting the time each task is allowed to run on a node.
    
    即使是低优先级的任务仍然不会被饿死。没有必要！
    
    We note that the Dynamic Priority scheduler can easily be configured to mimic the behavior of the other schedulers. If no queues or users have any credits left the scheduler reduces to a FIFO scheduler. If all queues are configured with the same share (spending rate in our case) and the allocation interval is set to a very large value, the scheduler reduces to the behavior of the static fair-share schedulers.
    
    可以退化为FIFO 和Fair Share scheduler

2.  implementation

### evaluation<a id="sec-8-1-5" name="sec-8-1-5"></a>

In the first set, we examine the correlation of spending rates, budgets and performance metrics.

In the second set, we study how accurately and effectively service levels can be supported.

### discussion<a id="sec-8-1-6" name="sec-8-1-6"></a>

Our scheduling approach is closely related to and inspired by economic schedulers,whereby you bid for resources on a market and receive allocations based on various auction mechanisms [18,19,20,17,21,22,23,24].We do not preclude nor require that our scheduler budgets are tied to a real currency. Furthermore, we do not assume that there are competing users who should be given different shares of the resources.

动态优先级调度受到经济学调度器的启发并且和他们很接近，都是用户在一个市场上基于各种各样的拍卖机制竞标资源。动态优先调度器不排除也不阻止用户的预算跟一种虚拟货币或者真实货币绑定。（编者注：其实不能同真实或者虚拟货币挂钩，因为不是自由市场）。

18-24这些论文都是05年以前，还没有hadoop和大数据的时代，主要面向网格计算和HPC。调研一下是否算是与它们重复。

应用了经济模型的调度器一定会引用这篇论文，因此我调研了所有160篇引用这篇论文的其他论文，暂时还没有发现跟我们重复的方案，有十几篇论文可能与我们有关联，可能需要引用，之后再读。

<div id="footnotes">
<h2 class="footnotes">Footnotes: </h2>
<div id="text-footnotes">

<div class="footdef"><sup><a id="fn.1" name="fn.1" class="footnum" href="#fnr.1">1</a></sup> <p>DEFINITION NOT FOUND.</p></div>


</div>
</div>
