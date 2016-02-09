#Faleiro
Faleiro is the code name for our scheduler. Our scheduler is controlled from our web api and is responsible for accepting resource offers from the Mesos master, collects tasks to be scheduled from active jobs, and chooses a subset of these jobs to be run on availble resource offers. 

## Using Resource Offers
Mesos master sends resource offers to the scheduler by calling `MagellanFramework::resourceOffers()` which stores it in a `BlockingQueue` which is accessed by Fenzo later.  


#Picking parameters for new tasks


## Fenzo
Fenzo is a library provided by Netflix that manages the scheduling and resource managment of tasks we want to execute. On each run of the MagellanFramework main loop, the framework invokes Fenzo by calling `TaskScheduler::scheduleOnce()` and providing it new resource offers that were placed in a queue by `MagellanFramework::resourceOffers()`, and new tasks that current, running jobs in the system want executed which is retrieved by calling `MagellanJob::getPendingTasks()`. If resource offers are not used by fenzo, it keeps hold of them while tasks that were not chosen for schedling by Fenzo are resubmitted to fenzo on the next call to `TaskScheduler::scheduleOnce()`.

# Framework main loop
The main loop of the MagellanFramework is run in a separate thread which calls the function MagellanFramework::runFramework(). This function first loops through the list of running jobs in the system and calls `MagellanJob::getPendingTasks()` on each job to get a list of tasks it wants to schedule. It then gives this list to Fenzo by calling `TaskScheduler::scheduleOnce()` and it receives a mapping between resource offers and tasks. If some of the tasks are not selected to be scheduled by Fenzo, then the remaining tasks are retained by the Framework and are submitted to Fenzo again during the next iteration. 
After receiving the scheduling suggestions by Fenzo,the the framework then prepares each task by creating a `TaskInfo` object which contains information such as the slave ID and task data. Tasks are scheduled on a per Host (node) basis. Hosts are specified using the `VMAssignmentResult` object and can offer multiple resource offers (called `leases` in the code). A list of all TaskInfo object for each host is created in an object called `taskInfos`. Additionally, a list of all the resource offer ids that will be used on the particular host are created and stored in a list called `offerIDs`. Once the `taskInfos` and `offerIDs` objects are populated, they are submitted to the mesos driver using the method, `MesosSchedulerDriver::launchTasks()`. The mesos driver is then responsible for getting the tasks to their corresponding executor.