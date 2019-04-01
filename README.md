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
<img width="350" alt="Screen Shot 2019-03-28 at 10 17 04 AM" src="https://user-images.githubusercontent.com/29927264/55164876-d3865b80-5142-11e9-9e3a-c8d7ccc1e50c.png">

<img width="350" alt="Screen Shot 2019-03-28 at 10 19 43 AM" src="https://user-images.githubusercontent.com/29927264/55165030-0b8d9e80-5143-11e9-87c1-47fdbca3d961.png">

<img width="380" alt="Screen Shot 2019-03-28 at 10 23 19 AM" src="https://user-images.githubusercontent.com/29927264/55165284-88207d00-5143-11e9-8fdd-a8f4863d5b18.png">

#### Fault-tolerance of Push-based gossip protocol
Packet loss: if 50% packet loss (replace b with b/2), to achieve same reliability as 0% packet loss, take twice as many rounds
Node Failure: 50% of nodes fail (replace n with n/2 and b with b/2), take twice as many rounds

If the initial sender dies or early receivers die, the epidemic will die out. But it will be very hard to stop the gossip once it spread several rounds.

### Spread speed between pull and push model
<img width="425" alt="Screen Shot 2019-03-28 at 10 47 40 AM" src="https://user-images.githubusercontent.com/29927264/55167231-f0bd2900-5146-11e9-9df6-3a2b6bda22c7.png">

### Topology-aware Gossip
<img width="500" alt="Screen Shot 2019-03-28 at 11 19 05 AM" src="https://user-images.githubusercontent.com/29927264/55169543-58757300-514b-11e9-8e43-c78db5c11299.png">
 
### Examples use Gossip Protocol
Cassandra key-value store: use gossip for maintaining membership; Usenet NNTP; Sensor networks

## Group Membership List
Used to maintain the list of most or all of the other processes that are currently in your system and that have not yet failed.
One biggest challenge is that this membership protocol has to communicate with the unreliabe communication medium which can drop packets, delay packets quite a bit. 

<img width="425" alt="Screen Shot 2019-03-28 at 11 51 54 AM" src="https://user-images.githubusercontent.com/29927264/55172131-eb181100-514f-11e9-8781-0473ae0be5f4.png">

Three kinds of membership list style: complete; almost-complete; partial-random:

<img width="400" alt="Screen Shot 2019-03-28 at 11 57 48 AM" src="https://user-images.githubusercontent.com/29927264/55172638-bb1d3d80-5150-11e9-9bb6-a9e73587ba06.png">

Use failure detector and dissemination mechanism to deal with process failure: 

<img width="494" alt="Screen Shot 2019-03-28 at 12 02 08 PM" src="https://user-images.githubusercontent.com/29927264/55172999-557d8100-5151-11e9-91d6-6b1689646327.png">

### Distributed failure detectors desirable properties
1. Completeness: each failure is detected 
2. Accuracy: there is no mistaken detection
3. Speed: Time to fisrt detection of a failure
4. Scale: Equal Load on member; Network message load

### Centralized Heartbeating to detect process failure
Every process periodically send heartbeat to a server (Pj), if there is no heartbeat from the server, then it is notified as failed. However, when Pj failed, there will be no gurantee to detect the failure. Also, if the proxy group is large, Pj may overloaded with failure messages. 

### Ring Heartbeating
All processes are organized in a virtual ring. And every process sends heartbeats to at least one of its neighbors.
If you have multiple failures, you may have some failures go undetected. 

### All-to-All heartbeating
Each process sends out heartbeats to all the other processes.
Equal load per member. Also, the protocol is complete. It can detect all process failures. 
However, if there is a slow server, it may delay the heartbeat and mark all other servers as failed. The rate of false positive will be high. 
<img width="590" alt="Screen Shot 2019-03-28 at 1 48 55 PM" src="https://user-images.githubusercontent.com/29927264/55180592-40f4b500-5160-11e9-9a92-d41de0ae4a6d.png">

If the heartbeat has not increased for more than T(fail) seconds, the member is considered failed; And the member is deleted from the list after T(cleanup) seconds. 

In gossip-style failure detection, we shouldn't delete the entry right after it's detected as failed. Since other process may not have deleted that entry and it may be added back. 
<img width="600" alt="Screen Shot 2019-03-28 at 1 57 06 PM" src="https://user-images.githubusercontent.com/29927264/55181126-6c2bd400-5161-11e9-8537-5c1aef3baedf.png">

If T(fail) and T(cleanup) increased, the fasle positive rate decreases, the detection time increases, and bandwidth required is less. 

<img width="492" alt="Screen Shot 2019-03-28 at 2 02 38 PM" src="https://user-images.githubusercontent.com/29927264/55181505-2b808a80-5162-11e9-9bc5-25a548df3e32.png">

### All-to-All heartbeating vs Gossip heartbeating
The All-to-All heartbeating model:
<img width="560" alt="Screen Shot 2019-03-28 at 2 12 21 PM" src="https://user-images.githubusercontent.com/29927264/55182220-97afbe00-5163-11e9-87bc-a47389c94e03.png">

The Gossip heartbeating model:
<img width="560" alt="Screen Shot 2019-03-28 at 2 12 32 PM" src="https://user-images.githubusercontent.com/29927264/55182221-98485480-5163-11e9-8cf3-0f70a44c591e.png">

Noticed that gossip-based heartbeating has a higher load than the all-to-all heartbeating, but it has a slightly better accuracy by using more messages. 

### SWIM Failure detector protocol 
Process pi runs the following protocol, which runs periodically every T prime time units, that's called the Protocol period. The beginning of the Protocol period it picks one other process at random, wa-call that process pj, and sends it a ping message. If the process pj receives a ping messages, it responds back with a ack message immediately. If pi receives this ack then, it does nothing else for the remainder of the Protocol period, it's satisfied. However, if it does not hear back an acknowledgement, which might happen if the acknowledgement is dropped or the original ping is dropped. Then, it tries to ping pj again, but, instead of using the direct path, it uses indirect path. It does this by sending indirect pings to K other randomly selected processes. This, third process is one of them. When it receives this indirect ping, it then sends a direct ping, uh, to pj, which responds back with an acknowledgement, and then, uh, a direct acknowledgement, and then the third process sends an indirect acknowledgement back to pi. If pi receives at least one such indirect acknowledgement, by the end of the protocol, uh, period, uh, then, uh, it is happy and is satisfied. If it does not receive either, direct acknowledgement from the beginning or any indirect acknowledgements then, it marks pj as having failed.

<img width="550" alt="Screen Shot 2019-03-28 at 2 24 04 PM" src="https://user-images.githubusercontent.com/29927264/55182954-2cff8200-5165-11e9-93d0-18d08dff9af2.png">

<img width="400" alt="Screen Shot 2019-03-28 at 2 40 26 PM" src="https://user-images.githubusercontent.com/29927264/55184009-7b158500-5167-11e9-888e-93f422967ea8.png">

<img width="400" alt="Screen Shot 2019-03-28 at 2 44 15 PM" src="https://user-images.githubusercontent.com/29927264/55184216-faa35400-5167-11e9-982a-a77d69f74f76.png">

<img width="391" alt="Screen Shot 2019-03-28 at 3 07 31 PM" src="https://user-images.githubusercontent.com/29927264/55185662-3d1a6000-516b-11e9-99eb-37ed406d64b9.png">
 
#SWIM vs. Heartbeating

<img width="550" alt="Screen Shot 2019-03-28 at 2 37 49 PM" src="https://user-images.githubusercontent.com/29927264/55183834-178b5780-5167-11e9-84b9-ff2f551fad01.png">

### Dissemination and Suspicion
Dissemination options: 
1. Multicast (Hardware / IP): unreliable; multiple simulataneous 
2. Point-to-Point (TCP/UDP): expensive 
3. Zero extra messages: Piggyback on Failure Detector messages

<img width="500" alt="Screen Shot 2019-03-28 at 9 30 43 PM" src="https://user-images.githubusercontent.com/29927264/55203154-d2841700-51a0-11e9-928a-31dcde70b179.png">

### Suspicion mechanism:
 1. False Detections, due to: Perturbed processes, Packet losses, e.g., from congestion
 2. Indirect pinging may not solve the problem: e.g., correlated message losses near pinged host
 3. Key: suspect a process before declaring it as failed in the group
 
<img width="500" alt="Screen Shot 2019-03-28 at 9 46 11 PM" src="https://user-images.githubusercontent.com/29927264/55203765-f0527b80-51a2-11e9-80fd-b23d59090330.png">

<img width="350" alt="Screen Shot 2019-03-28 at 9 56 22 PM" src="https://user-images.githubusercontent.com/29927264/55204164-58ee2800-51a4-11e9-8515-8b5644411699.png">
 
Failures the norm, not the exception in datacenters; Every distributed system uses a failure detector; Many distributed systems uses a membership service;

Ring failure detection underlies(IBM SP2 and many other similar cluster/machines)
Gossip-style failure detection underlies(Amazon EC2/S3)
 
## P2P System
### Napster 
 <img width="400" alt="Screen Shot 2019-04-01 at 4 46 58 PM" src="https://user-images.githubusercontent.com/29927264/55358611-d1533280-549d-11e9-97c4-cf610f82113a.png">

#### Client
1. Connect to Napster server: upload list of music files that you want to share; Server maintains list of <filename, ip_address, portnum> tuples. Server stores no files.
2. Search: send server keywords to search with, server search its list with keywords, server returns a list of hosts <ip_address, portnum> tuples to client, client pings each host in the list to find transfer rates, client fetches file from best host
3. All communication uses TCP, since it's reliable and ordered networking protocol. 
<img width="430" alt="Screen Shot 2019-04-01 at 5 48 49 PM" src="https://user-images.githubusercontent.com/29927264/55361870-6a864700-54a6-11e9-892e-30d08988d484.png">
* server use ternary tree algorithm to search their lists, ternary tree is a sorted dictionary. 
 
#### Join a P2P system
1. send an http request to well-know url for that P2P service
2. Messages routed (after lookup in DNS=Domain Name system) to introducer, a well known server that keeps track of some recently joined nodes in p2p system
3. Introducer initializes new peers' neigjbor table
 
#### Problems for Napster System
* Centralzied server a source of congestion
* Centralized server single point of failure
* No security: plaintext messages and passwords

### Gnutella
* Eliminate the servers
* Client machines search and retrieve amongst themsevlves
* Client act as servers too, called servents
* Originial design underwent several modifications 

<img width="350" alt="Screen Shot 2019-04-01 at 6 09 38 PM" src="https://user-images.githubusercontent.com/29927264/55362872-5859d800-54a9-11e9-86dc-f7d2a457053c.png"> <img width="420" alt="Screen Shot 2019-04-01 at 6 11 23 PM" src="https://user-images.githubusercontent.com/29927264/55362967-93f4a200-54a9-11e9-9ffc-72d27d646c31.png">
 
 
 
 
 
 
 
 



