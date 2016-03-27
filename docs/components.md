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

When the user requests data about the running jobs, the web API calls the `MagellanFramework::getJobStatus()` and `MagellanFramework::getAllJobStatuses()` methods for a specific job and for all jobs, respectively. The method calls provide all the info about a job which is then returned to the user.

### Manage a Job

The Web API is able to pause, resume, and stop a job. The user makes a request and specificies which action they would like to take. The Web API then makes the appropriate method call to the scheduler framework: `MagellanFramework::pauseJob()`, `MagellanFramework::resumeJob()`, and `MagellanFramework::stopJob()`.

## Scheduler Backend (Faleiro)

Faleiro is the code name for our scheduler. Our scheduler is controlled from our web api and is responsible for accepting resource offers from the Mesos master, collecting tasks to be scheduled from active jobs, and choosing and running a subset of these jobs on availble resource offers. 

### Using Resource Offers
Mesos master sends resource offers to the scheduler by calling `MagellanFramework::resourceOffers()` which stores the resource offers in a `BlockingQueue` which is accessed by Fenzo later.  


### Picking Parameters for New Tasks
The scheduler tells each executor the starting parameters for its simulated annealing algorithm. These parameters are formatted as a json string and include information such as the starting location to start the search, data passed in by the client using the command line client, and the name of the specific task we want to import on the executor side (see documentation on executor). The full json list is shown in the Job to Executor document. , and the length of time that we want the executor to run for. The  Initial temperature, cooling rate and number of iterations per change in temperature are all chosen on the executor side.


### Fenzo
Fenzo is a library provided by Netflix that manages the scheduling and resource managment of tasks we want to execute. On each run of the MagellanFramework main loop, the framework invokes Fenzo by calling `TaskScheduler::scheduleOnce()` and providing it with a list of resource offers and tasks. The resource offers are taken from the queue where they were placed by `MagellanFramework::resourceOffers()`. The list of tasks to run is attained by calling `MagellanJob::getPendingTasks()`. If resource offers are not used by fenzo, it keeps hold of them and are used on the next call to `TaskScheduler::scheduleOnce()`.

### Framework Main Loop
The main loop of the MagellanFramework is run in a separate thread which calls the function `MagellanFramework::runFramework()`. This function first loops through the list of running jobs in the system and calls `MagellanJob::getPendingTasks()` on each job to get a list of tasks it wants to schedule. It then gives this list to Fenzo by calling `TaskScheduler::scheduleOnce()` and receives a mapping between resource offers and tasks. If some of the tasks are not selected to be scheduled by Fenzo, then the remaining tasks are retained by the Framework and are submitted to Fenzo again during the next iteration. 


After receiving the scheduling suggestions by Fenzo,the the framework then prepares each task by creating a `TaskInfo` object which contains information such as the slave ID and task data. Tasks are scheduled on a per Host (node) basis. Hosts are specified using the `VMAssignmentResult` object and can offer multiple resource offers (called `leases` in the code). A list of all TaskInfo object for each host is created in an object called `taskInfos`. Additionally, a list of all the resource offer ids that will be used on the particular host are created and stored in a list called `offerIDs`. Once the `taskInfos` and `offerIDs` objects are populated, they are submitted to the mesos driver using the method, `MesosSchedulerDriver::launchTasks()`. The mesos driver is responsible for getting the tasks to their corresponding executor.