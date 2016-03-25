#Job Creation to Executor

##Client Starts Job
The client starts a job using the command line interface. Using the CLI, they can specify the parameters for their job. The full list of parameters can be found on the Web API page. After creating a job on the command line client, the request is sent to the Web API using http.

##The Web API
The web api handles the request from the CLI and forwards this request to the MagellanFramework using the function call `MagellanFramework::createJob()`. 

##The Magellan Framework
Internally, the Magellan Framework constructs a MagellanJob object. A MagellanJob, at a high level, represents all data and state corresponding to a problem that the client wants to solve. The MagellanJob breaks this problem into subproblems called tasks which are run on executors. An instance of the Magellan Framework can have multiple instances of MagellanJob with each instance having several tasks associated with it that can be run in parallel.  After creating the MagellanJob object, the framework calls `MagellanJob::start()` which executes the main function of the Job, called `MagellanJob::run()`, in a separate thread. 


A MagellanJob is responsible for partioning the search space of the problem and creating tasks which run on executors and report their results back to the job. MagellanJobs are restricted to creating 10 active tasks at a time to ensure that this system scales well when a lot of jobs are submitted simultaneously. New tasks are submitted to a BlockingQueue called `pendingTasks`. This object holds ready tasks until the MagellanFramework requests them using the method `MagellanJob::getPendingTasks()`. After executing tasks, the results are returned to the Job to be processed using the method, `MagellanJob::processIncomingMessages()`. 

The scheduler sends the following JSON string to the executor as part of the tasks payload: 
```
{
	"uid":
	"location":
	"job_data":
	"task_name":
	"task_seconds":
}
```
As you can see, each task only sends a unique identifier for the task, the starting location for the simulated annealing (SA) algorithm, the name of the SA implementation we want to use to execute the task, how long the task should take and additional parameters passed in by the client as arguments.  The  initial temperature, cooling rate and number of iterations per change in temperature that are detrimental to the SA algorithm are not given and are all chosen on the executor side. See the executor document for details.


The MagellanFramework then uses Fenzo to determine how tasks will be scheduled and then uses the Mesos Driver to schedule the tasks. See the Scheduler docs for more details.

The response from the executor is the following JSON string:
```
{
	"uid":
	"best_location":
	"fitness_score":
}
```
This JSON string cotains the unique identifier for the task, the best solution it discovered, and the fitness or energy score of the best solution. This data is passed to and processed by the MagellanJob from which it originated.  If the energy of this data is better than the global best energy, then we update our global best fitness value and location value. Currently, new tasks start their searches with the global best values but by the final product, we hope to implement a meta annealing scheduler where the scheduler itself performs simulated annealing to choose starting locations for new tasks.