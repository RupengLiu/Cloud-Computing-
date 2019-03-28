# Cloud-Computing- by coursera

## Basics

Program counter: is an index into the code, which says where the program is executing right now.

## What is a Cloud?
There are two kinds of clouds, a single-site cloud or a geo-distributed or geographically distributed cloud. A single-site cloud, known as a Datacenter often, consists of the following components, of course, it have been source of servers or compute nodes, these are grouped into racks, a rack as a unit of several servers which typically share the same power and often share a top of the rack switch. This top of the rack switches are often connected via a network topology. 

## MapReduce 
Examples :
1. Distributed grep: input: large set of files; output: lines that matches the pattern
2. Reverse Web-link graph: Map: process web log and for each input <source, target>, output <target, source>; reduce: emits <target, list<source>>
3. Count of URL access frequency: input: Log of accessed URL from proxy server; output: for each URL, calculate % total access from that URL. 
4. Sorting: map: quicksort; reduce: merge sort (make sure key are increasing/decreasing)

Inside MapReduce(for the cloud): 
1. Parallelize Map: each map task is independent of each other. All Map output records with same key assigned to same reduce 
2. Transfer data from Map to Reduce: All Map output records with same key assigned to same reduce task; use partitioning function. e.g.hash(key)%number of reducers 
3. Parallelize reduce: each reduce task is independent of the other
4. Implement Storage for Map input, output; Reduce input, output: 
* map input: from distributed file system
* map output to local disk (at Map Node): uses local file system
* reduce input: from (multiple) remote disks; uses local file systems
* reduce output: to distributed file system
* local file system: Linux, etc. distributed file system = GFS (Google File System), HDFS (Hadoop Distributed File System)
<img width="400" alt="Screen Shot 2019-03-26 at 1 04 50 PM" src="https://user-images.githubusercontent.com/29927264/55017889-140d9a00-4fc8-11e9-96f4-a9da76b82871.png">

#### The Yarn Scheduler
* Used in Hadoop 2.x+
* YARN = Yet Another Resource Negotiator
* Treats each server as a collection of containers: container = some CPU + some memory 
* Has 3 main components: <br>
  1. Global Resource Manager (RM): Scheduling
  2. Per-server Node Manager (NM): Daemon and server-specific functions
  3. Per-application (job) Application Master (AM): Container negotiation with RM and NMs; Detecting task failures of that job
<img width="350" alt="Screen Shot 2019-03-26 at 1 28 06 PM" src="https://user-images.githubusercontent.com/29927264/55019324-0dcced00-4fcb-11e9-9490-5603be8b2ac7.png">

### Fault Tolerance 
1. Server Failure: NM heartbeats to RM: If server fails, RM lets all affected AMs know, and AMs take action; NM keeps track of each task running at its server: if task fails while in-progress. mark the task as idle and restart it; AM heartbeats to RM: On failure, RM restarts AM, which then syncs up with its running tasks
2. RM Failure: Use old checkpoints and bring up secondary RM; Hearbeats also used to piggyback container requests， avoid extra messages

### Straggers (slow node)
1. The slowest machine slows the entire job down, due to bad disk, network bandwidth, CPU, memory.
2. Perform backup (replicated) execution of straggler task: task considered done when first replica complete. Called Speculative Execution.

### Locality
Since Cloud has hierarchical topology, GFS/HDFS stores 3 replicas of each chunks: maybe on different racks: 2 on a rack, 1 on a different rack; MapReduce attempts to schedule a map task on: <br>
  1. a machine that contains a replica of corresponding input data, or failing that
  2. on the same rack as a mahcine containing the input, or failing that
  3. anywhere
  
### Summary 
MapReduce uses parallelization + aggregation to schedule applications across cluster; need to deal with failure


## Multicast
Within a particular group of nodes or group of processes, you have a piece of information that you want to send out the entire network where everyone, all the computers or all the nodes on the network want to receive that piece of information. 

The multicast protocol typically sits at the application level, meaning that it does not deal with the underlying network. The application level multicast protocols also talk with the underlying network level, techniques such as IP multicast. IP multicast is what's available in the underlying network itself. This is implemented in routers and switches. It's not necessarily an attractive alternative because even though it's available it may not be enabled in many of the routers and switches, and so, most of the multicast protocols that are deployed out there today, even though some of them may leverage what-IP multicast where it's available, all of them, most of them today are application level, meaning that, they involve processes talking with each other, without really worrying about what's going on in the network underneath.

### The requirements of Multicast protocol: fault tolerance and scalability
fault tolerance: nodes are failure prone, you want recipients receive their infomation
scalablity: overhead of each node won't grow rapidly as number of nodes grows into the thousands

### Tree-based Multicast Protocol
Protocols develop a spanning tree among the nodes or the process in the group. 

This is designed to overcome the huge overhead of the centralized approach multicast (O(n) to send information to all other recipients, high latency and average time), then using balanced tree, the latency will become O(log(n)), also the sender's overhead will become constant since every node has constant number of children. Node failure in tree struture may cause its all decendents failing, use either (ACKs) or (NAKs) to repair multicast not received (sent to the root)

SRM (Scalable Reliable Multicast): use NAKs; adds random delays, and uses exponential backoff to avoid NAK storms: (So receivers when they realize they need to send out a NACK, they don't send out the NACK immediately. They wait for a little bit of time and then they send it out. If they need to send a NACK multiple times,then they might use an exponential backoff, meaning that they, wait for a period of time that doubles every time they wait. Also, the messages that are sent out might also be subject to exponential backoff)

RMTP (Reliable Multicast Transport Protocol): use ACKs; But ACKs only sent to designated receivers, which then re-transmit missing multicasts; 

NACKs and ACKs still grow linearly as the group size increase

## Gossip Protocol
Each Node periodically transmit to b random target using gossip messages (UDP), others do the same after receiving multicast

### Push vs. Pull

Push gossip: once you have a multicast message, you start gossiping about it; gossip a random subset of them, or recently-received ones, or higher priority ones

Pull gossip: Periodically ask and poll a few randomly selected processes for new multicast messages that you haven't received

Hybrid variant: Push-Pull model

### Push based gossip protocol
1. Is lightweight in large groups
2. spreads a multicast quickly
3. is highly fault-tolerant

Analysis: population of (n+1), cantact rate beta, x (uninfected), y (infected)








