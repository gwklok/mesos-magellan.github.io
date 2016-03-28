# Components

A detailed description of all the components that makes up the Magellan Mesos Framework.

## Web UI (Circumnavigator)

The Web User Interface (UI) allows users to create, manage, and monitor jobs. It is built using jQuery, Bootstrap and pure.js. To connect to the scheduler API, AJAX calls are made to create, modify and retrieve jobs. Bootstrap was used for quick styling and ensuring consistent functionality and appearence across different devices and browsers. pure.js allowed for templating HTML code with scheduler API data.

There are 3 pages to the web UI: Job List (home page), Create a Job, and Job Details. Each page has its own JavaScript file in the js folder, as well as a global script file: script.js, which holds global functions and data. The job list and job details pages auto-refresh periodically to keep the user updated with accurate information. The create a job page validates the user's input, providing feedback on incorrect fields, and redirects the user to the job details page for the job they create.

State-based colour feedback is used throughout the UI. Based on the status of the job, different elements will have different colours or be hidden. The progress bar's appearance differs based on state in colour, animation, and style. When the job is in a state that could allow for completion (RUNNING or PAUSED), the progress bar is animated with moving stripes. When the job is in a state where it is incomplete and will not be able to be completed (STOP or FAILED), the progress bar is no longer animated but is still striped. Once the job is DONE, the progress bar is a solid green. The progress bars have a minimum width to correctly display the progress percentage when the values are low. The action buttons on the job details page are dynamic to the state as well. Their colours correspond to the actions they perform and will be hidden if their action is not valid for the current state.

The job list is ordered from newest to oldest descending with all stored jobs being displayed. The status and progress bars are styled as described above.

## Scheduler REST API (Faleiro API)

### Creating a Job

The Web API receives input as documented on the Web API Schema page. It then validates user input as necessary and forwards the input to an instance of the scheduler framework. To create a job, the API forwards the parameters to the `MagellanFramework::createJob()` method which then returns the created job's ID and returns it to the user making the API request.

### Getting Job Status

When the user requests data about the running jobs, the web API calls the `MagellanFramework::getSimpleJobStatus()` and `MagellanFramework::getSimpleAllJobStatuses()` methods for a specific job and for all jobs, respectively. The method calls provide all the info about a job which is then returned to the user.

### Manage a Job

The Web API is able to pause, resume, and stop a job. The user makes a request and specificies which action they would like to take. The Web API then makes the appropriate method call to the scheduler framework: `MagellanFramework::pauseJob()`, `MagellanFramework::resumeJob()`, and `MagellanFramework::stopJob()`.

# Scheduler Backend (Faleiro)

Faleiro is the code name for our scheduler. Our scheduler is controlled from our web api and is responsible for accepting resource offers from the Mesos master, collecting tasks to be scheduled from active jobs, and choosing and running a subset of these jobs on availble resource offers. 

### Resource Offers
Once the scheduler has registered with the Mesos Master, the Master provides fine grained access to machine resources (such CPU, RAM) availble in the cluster to the scheduler by sending resource offers. Schedulers will then use these resource offers to run tasks. The Master sends resource offers asynchronously to the scheduler by calling `MagellanFramework::resourceOffers()` and the scheduler stores these resource offers in a `BlockingQueue`. These received resource offers are then given to Fenzo inside the framework's main loop (see sections on "Fenzo" and "Framework Main Loop").  

### Fenzo
Fenzo is a java library provided by Netflix that essentially takes care of the bin packing problem for us. In other words, given a list of tasks to run and a list of resource offers, find the most efficient pairings between tasks and resource offers.  Although we could have used a simple, greedy algorithm of pairing the first task in our queue with the first resource offer received, this approach not only scales terribly, but also doesn't take into account the heterogeneous nature of the tasks and resource offers. Each task could require different amounts of system resources to run and mixed constraints. By using fenzo, we can acheive scheduling optimizations and autoscaling the cluster based on usage. 

On each run of the MagellanFramework main loop, the framework invokes Fenzo's bin backing functionality by calling `TaskScheduler::scheduleOnce()` and providing it with a list of resource offers received from `MagellanFramework::resourceOffers()` and tasks to schedule which is attained by calling `MagellanJob::getPendingTasks()`. If resource offers are not used by fenzo in the current itertation of the Framework's main loop, it keeps hold of them and are re-used on the next call to `TaskScheduler::scheduleOnce()`.

### Zookeeper
Zookeeper is a service provided by Apache that provides synchronization and maintains configuration information reliably across a distributed cluster. In our system, Zookeeper is responsible for providing high availability (HA) by performing two services: leader election and persisting scheduler state.

You can find more information on the specifics of Zookeeper [here](http://zookeeper.apache.org/doc/r3.3.3/zookeeperProgrammers.html#_introduction) but these are the essentials:

1. Zookeeper stores data in a hierarchal namespace much like a file system. The only difference is that each node can have data as well as having children. You can think of this data model like a tree.
2. A Zookeeper node can be one of two types - ephemeral or persistant. A persistant node maintains its information even after the process that wrote it dies which makes it really useful for persisting scheduler state. An ephemeral node only maintains its data until the process that created it dies and this functionality is utilized during leader election. 
3. Processes can place watches on various nodes within Zookeeper so that they are notified when events such as data is modified or when node existence changes. 	

#### Leader Election
In order to ensure that our cluster is resilient to schedulers crashing, we introduced a system with multiple schedulers. The idea is that one of these schedulers will be the leader and coordinate all the various functions of a scheduler while the other "follower" schedulers will wait in idle until the lead scheduler goes down for any reason . When the leader goes down, the remaining, idle schedulers will undergo the process of leader election by confering among themselves and electing a new scheduler as the leader. This new leader will be responsible for restoring the state of the system to where it was before the first leader crashed. This entails re-enabling the web endpoints to communicate with our web UI, open a new connection with the Mesos master and restarting jobs that were running previously. 

The process of electing a leader is done by levereging Zookeeper's ability to synchronize and coordinate concurrent writes. The steps are as follows:

1. Create an ephemeral node, Z, at path "/election/p_id". The id is a sequential number assigned by zookeeper.
2. Get the children of "/election", denoted as C.
3. If the id of Z is the lowest id present in C, then z is elected the leader.
4. If the id of Z is not the lowest, set a watch on the node whose sequential id is immediately lower in the list of children, C and then go to sleep

If the current leader ,or any of the "follower" scheduler dies, the ephemeral node that it created is deleted with it. This action triggers a watch that one of the other schedulers will pick up. When a watch is triggered in response to a delete, the following sequence occurs:

1. The scheduler that is woken by the watch gets the new list of children, C, of "/election"
2. if the scheduler's id is the smallest in C, then this scheduler becomes the new leader.
3. If the scheduler's id is not the smallest, then set another watch on the node whose id is immediately smaller in C and go to sleep.

### Framework Main Loop
The main loop of the MagellanFramework is run in a separate thread which calls the function `MagellanFramework::runFramework()`. This function first loops through the list of running jobs in the system and calls `MagellanJob::getPendingTasks()` on each job to get a list of tasks it wants to schedule. It then gives this list to Fenzo by calling `TaskScheduler::scheduleOnce()` and receives a mapping between resource offers and tasks. If some of the tasks are not selected to be scheduled by Fenzo, then the remaining tasks are retained by the Framework and are submitted to Fenzo again during the next iteration. 


After receiving the scheduling suggestions by Fenzo,the the framework then prepares each task by creating a `TaskInfo` object which contains information such as the slave ID and task data. Tasks are scheduled on a per Host (node) basis. Hosts are specified using the `VMAssignmentResult` object and can offer multiple resource offers (called `leases` in the code). A list of all TaskInfo object for each host is created in an object called `taskInfos`. Additionally, a list of all the resource offer ids that will be used on the particular host are created and stored in a list called `offerIDs`. Once the `taskInfos` and `offerIDs` objects are populated, they are submitted to the mesos driver using the method, `MesosSchedulerDriver::launchTasks()`. The mesos driver is responsible for getting the tasks to their corresponding executor.