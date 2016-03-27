# Communication

Description of the communication between the different components of the Magellan Mesos Framework.

## The Scheduler API Schema

The following REST API schema is used by the Web UI to communicate with the scheduler back end.

### POST /api/job
Create a new job

**Request**
```json
{
	"job_name" : "Solve 99 Problems",
	"job_time" : 3600,
	"module_url" : "git@github.com:mesos-magellan/faleiro.git",
	"module_data" : {
		"problem_size" : 99,
		"expected_solution" : 0,
		"anything_i_want" : {
			"all_my_things" : [
				"cat",
				4096,
				"dogs",
				"foo"
			]
		}
	}
}
```

**Response (200)**
```json
{
	"job_id" : 42
}
```

**Response (422)**
```json
{
	"message" : "A parameter is missing"
}
```

**Response (500)**
```json
{
	"message" : "Failed to create job"
}
```

### PUT /api/job/{job_id}/status
Set the status of an existing job

**Parameters**

`{job_id}` : _ID of the job to modify_

**Request**
```json
{
	"status" : "resume" /* or "stop" or "pause" */
}
```

**Response (200)**
```json
{
}
```

**Response (422)**
```json
{
	"message" : "A parameter is missing"
}
```

### GET /api/jobs
Get all the jobs

**Request**
```json
{
}
```

**Response (200)**
```json
{
	[
		{
			"job_id" : 42,
			"job_name" : "Solve 99 Problems",
			"job_starting_time" : 112385801,
			"task_name" : "git@github.com:mesos-magellan/faleiro.git",
			"task_seconds" : 500,
			"current_state" : "RUNNING",
			"best_location" : "{ \"path\" : \"SUJENAGHWOPQ\" }",
			"best_energy" : 400.5,
			"num_finished_tasks" : 91,
			"num_total_tasks" : 99,
			"energy_history" : [
				0.2,
				400.5,
				10.122
			],
			"additional_params" : {
				"start" : {
					"X" : 356,
					"name" : "foo"
				}
			}
		},
		{
			"job_id" : 47,
			"job_name" : "Solve All Problems",
			"job_starting_time" : 112385801,
			"task_name" : "git@github.com:mesos-magellan/faleiro.git",
			"task_seconds" : 600,
			"current_state" : "STOP",
			"best_location" : "{ \"index\" : 2345093 }",
			"best_energy" : 12039454,
			"num_finished_tasks" : 42,
			"num_total_tasks" : 123,
			"energy_history" : [
				947345,
				3302,
				12039454
			],
			"additional_params" : {
				"start" : {
					"X" : 50,
					"name" : "hello world"
				}
			}
		}
	]
}
```

### GET /api/job/{job_id}
Get a specific job by job_id

**Parameters**

`{job_id}` : _ID of the job to retrieve_

**Request**
```json
{
}
```

**Response (200) - No longer running**  
**Response (202) - Job still running**
```json
{
	"job_id" : 47,
	"job_name" : "Solve All Problems",
	"job_starting_time" : 112385801,
	"task_name" : "git@github.com:mesos-magellan/faleiro.git",
	"task_seconds" : 600,
	"current_state" : "STOP",
	"best_location" : "{ \"index\" : 2345093 }",
	"best_energy" : 12039454,
	"num_finished_tasks" : 42,
	"num_total_tasks" : 123,
	"energy_history" : [
		947345,
		3302,
		12039454
	],
	"additional_params" : {
		"start" : {
			"X" : 50,
			"name" : "hello world"
		}
	}
}
```

**Response (422)**
```json
{
	"message" : "A parameter is missing"
}
```

## Job Creation to Executor

### Client Starts Job
The client starts a job using the command line interface. Using the CLI, they can specify the parameters for their job. The full list of parameters can be found on the Web API page. After creating a job on the command line client, the request is sent to the Web API using http.

### The Web API
The web api handles the request from the CLI and forwards this request to the MagellanFramework using the function call `MagellanFramework::createJob()`. 

### The Magellan Framework
Internally, the Magellan Framework constructs a MagellanJob object. A MagellanJob, at a high level, represents all data and state corresponding to a problem that the client wants to solve. The MagellanJob breaks this problem into subproblems called tasks which are run on executors. An instance of the Magellan Framework can have multiple instances of MagellanJob with each instance having several tasks associated with it that can be run in parallel.  After creating the MagellanJob object, the framework calls `MagellanJob::start()` which executes the main function of the Job, called `MagellanJob::run()`, in a separate thread. 


A MagellanJob is responsible for partioning the search space of the problem and creating tasks which run on executors and report their results back to the job. MagellanJobs are restricted to creating 10 active tasks at a time to ensure that this system scales well when a lot of jobs are submitted simultaneously. New tasks are submitted to a BlockingQueue called `pendingTasks`. This object holds ready tasks until the MagellanFramework requests them using the method `MagellanJob::getPendingTasks()`. After executing tasks, the results are returned to the Job to be processed using the method, `MagellanJob::processIncomingMessages()`. 

The scheduler sends the following JSON string to the executor as part of the tasks payload: 
```
{
    # required
    "uid":
    "name":
    "command":
    "problem_data":
    # for 'divisions' command
    "divisions":
    # for 'anneal' command
    "sstates":
    "minutes_per_division":
}
```
As you can see, each task only sends a unique identifier for the task, the starting location for the simulated annealing (SA) algorithm, the name of the SA implementation we want to use to execute the task, how long the task should take and additional parameters passed in by the client as arguments.  The  initial temperature, cooling rate and number of iterations per change in temperature that are detrimental to the SA algorithm are not given and are all chosen on the executor side. See the executor document for details.


The MagellanFramework then uses Fenzo to determine how tasks will be scheduled and then uses the Mesos Driver to schedule the tasks. See the Scheduler docs for more details.

The response from the executor is the following JSON string:
```
{
	"uid":
    # for anneal
	"best_location":
	"fitness_score":
    # for divisions
    "divisions":
}
```
This JSON string cotains the unique identifier for the task, the best solution it discovered, and the fitness or energy score of the best solution. This data is passed to and processed by the MagellanJob from which it originated.  If the energy of this data is better than the global best energy, then we update our global best fitness value and location value. Currently, new tasks start their searches with the global best values but by the final product, we hope to implement a meta annealing scheduler where the scheduler itself performs simulated annealing to choose starting locations for new tasks.

In case of a failed task, the following body will be returned with a TASK_FAILED status:
```
{
    "error": <stacktrace>
}
```
