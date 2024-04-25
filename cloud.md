# Kubernetes


## What is Kubernetes?
It is an open-source container orchestrator
Supports multiple run-time

## What are the four basic objects in Kubernetes and what is the main role?
1. Pod: Basic deployment unit
2. Volume: "Persistent Storage" for a pod
3. Service: a "logical group of pods" that work together
4. namespace: logical slices of Kubernetes cluster 

## What is pod co-scheduling in Kubernetes?

Containers in a pod must be scheduled together in the same node

## Explain the following multi-container models:

Sidecar: add on or helper container
Ambassador: proxy container
Adaptor: standardize and normalize output

## What is etcd's responsability in Kubernetes and why is etcd important for failure recovery?

etcd stores all cluster data, and it is the primary target of backup, K8s cluster can be restored with "etcd"

## Controllers
Job: Run-to-completion
replicaset: stable # of pods
deployment: blue/green deployment, support "roll-back", rolling-upgrade 
DaemonSet: Monitoring and Logging daemon

## Autoscaling
pod autoscaling (pod/container)
cluster autoscaling (vm)

### Pod autoscaling

1. Horizontal pod autoscaler
2. Vertical pod autoscaler 
3. multidimential pod autoscaler 


### Cluster autoscaling
What if your cluster is no longer able to allocate more pods? Typically due to capacity limit
When cluster is no longer able to allocate more pods

## Prometheus
Monitoring + time-series DB

# Serverless computing

## What is serverless computing?
container + timeout


# Why did people make serverless?
Provisioning of coarse-grained resources always have idle portion

No need to manage servers
Only pay for what you use (never pay for idle time and new payment model)
deploy functions, not applications
short-lived, stateless

User <--> API gateway <---> AWS Lambda, google function, azure function <---> DB (cloud)

## What is app timeout?
Max allowed app exec time

## What would be the max duration of application execution time on serverless?
10 min (600s)

## What is cold start and what is warm start?

Cold start: 
1. the first execution of your function
2. find VM (can be running or it can be a newly created one)
3. start container with timeout

3 cases?

1. very first execution
2. no provisioned resources for your cloud function
3. need to add more resources, so will have to cold start

Warm start:
1. Function is already deployed
2. Function codes in memory
3. Handles request before it dies

### Cold start steps

1. Find Host VM
2. Container init
3. function load
4. response

### Warm start steps
1. Code is in MEM
2. just response

## How to minimize cold-start?

send packet enough to warm the container

## What is the price for serverless computing?

0.30 per every 1 million executions

## Good Serverless computing
- true on demand
- Scalability
- Near unlimited resources
- never pay for idle time
- avoid over-provisioning
- no infrastructure management
- hiding underlying infrastructure
## Bad serverless computing
- limited resources and execution time
- vendor lock-in
- stateless
- new technology
- cold start latency
- still overprovisioning

no suitable for scientific simulation, deep learning model

suitable for API GW, web crawler, event driven process

## Three types of data

Structured: data that can be put in a table with a schema 
Unstructured: data that is not organized in a pre-defined manner 
Semi: data that cannot be stored in a RDBMS but has organizational properties


# GFS , HDFS, MapReduce, and Hadoop

## GFS
- made with cheap commodity machines (prone to errors)
- System stores "modest" number of files
- Supporting three Google-specific workloads
    - small random reads
        read small piece from large data
    - large stream reads
        - crawled data processing
    - many large sequential reads
        append search index with new content
    - no random write
        Simplicity in FS design
        Simplicity in Failover/data management
- Concurrent, atomic append
- Stable bandwidth is much more important than low latency

### GFS Architecture
1. Master
    only 1 master, communicates with chunkserver with heartbeat
    What resource at GFS master stores a unique handler (ID/location)

2. Chunk server
    stores data
    partions data called chunks, each chunk being 64MB in size

3. Client
    no client-side caching
    -two requests
        control req to master
        data access req to Chunkserver

Recall chunk in GFS again what are the advantages and disavantages of using the specific large size of chunk?

Pro: Large chunk size == small amount # of chunks, reduced # of operations between master & clients

cons: high overhead, high wasted storage due to internal fragmentation

1. What if a task fails
    task tracker detects the failure
    task tracker tells job tracker
    job tracker reschedules tasks


2. WHat if a data node fails
    both job tracker & name space detect the error
    all tasks on the failed node are rescheduled
    name node replicates the data chunk to another one
3. what if a namenode or job tracker fails?
    entire cluster is down
    single point of failure
    YARN handles this
## Hadoop Architecture:
    - job tracker (master)
        coordinates the execution of jobs
    - Task Tracker (worker daemon)
        controls the execution of map & reduce tasks in slave machines
    - Name Node(master)
    manages the file system, keeps metadata 
    - Data node
        follows the instructions from name node
## Hadoop pros & cons
pro: Fault tolerance, highly scalable, simple programming model
Con: WHat if file is smaller than 64MB, batch processing, Data locality


# Spark
Spark = RDD( resilient Distributed Dataset)
* Creating ROM (read-only-mem)
    - Minimizing page updatre
    - RDD( resilient Distributed Dataset)

Restricted form of distributed shared memory

# Scheduling


Why do you need scheduling?
1. multiple tasks
2. limited resources


Goals:
- Max resource
- max throughput
- min turnaround time
- min waiting time
- min response time

- difficult SLA or deadline satisfaction

## Single processor Schedulign
### FIFO or Shorted Task first: good for batch applications
    - Maintain tasks in a queue in order of arivial
    - when processor is free, deque the head, and process it
    - average completion time = (task 1 + task 2 + task 3)/3
    

    PROS
    - good when all jobs have similar length

    CONS 
    - head of line blocking
### Round Robin: is preferable for iterative applications when users need quick response from the system

### Capacity Scheduler
    - Uses multiple queues
    - Each queue has "soft limit"
    - Resource Elasticity ( a queue is allowed to occupy more of cluster if resources are free)
    - if other queues below the soft limit (capacity limits), need to relase (overly occupied ) resources
    - non-prememptive

### Fair Scheduler
- all jobs get equal share of resources

- divide cluster into pools
- resource divided equally among pools
- pre-emption is allowed

When a pool does not have minimum share, it can take resources away from other pools
Most recently started tasks are killed and rescheduled

Delay scheduling is an improvement over Fair Scheduling

Limitations
- Doesn't support data locality
MapReduce job execution time = 90% Disk I/O + 10% Computation



### Delay Scheduling
- relaxed queueing policy: makes jobs wait for a limited time to find idle machine with ideal data locality

- result: very short wait time (1s - 5s) is enough to get nealry 100% locality
- Data locality!!!


## Cluster Scheduler

### Monolithic Scheduler

- single centralized scheduler

- implement all policies in one code base

PROS
- centralized control, scheduler knows everything
- Optimal scheduling decision

CONS
- single code base
- difficult to add new scheduling policies
- as code base increases, code complexity increases
- Scheduler often becomes bottleneck
    * Scalability, scheduling overhead
- not suitable for large-cluster size

### Statically partionined scheduler
it's similar to the hadoop capacity scheduler, but less flexible

PROS
- can handle multiple frameworks
- Bottleneck from one applciation scheduling will not affect other applications' scheduling

Cons
- Resource fragmentation
- suboptimal resource utilization




