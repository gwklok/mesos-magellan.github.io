#POST /api/job
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

# PUT /api/job/{job_id}/status
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

#GET /api/jobs
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

# GET /api/job/{job_id}
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