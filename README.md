# Cloud-Computing- by coursera

### Basics

Program counter: is an index into the code, which says where the program is executing right now.

### What is a Cloud?
There are two kinds of clouds, a single-site cloud or a geo-distributed or geographically distributed cloud. A single-site cloud, known as a Datacenter often, consists of the following components, of course, it have been source of servers or compute nodes, these are grouped into racks, a rack as a unit of several servers which typically share the same power and often share a top of the rack switch. This top of the rack switches are often connected via a network topology. 

### MapReduce 
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



