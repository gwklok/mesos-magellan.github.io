#POST /api/job
Create a new job

**Request**
```json
{
	"job_name" : "Solve 99 Problems",
	"job_init_temp" : 200,
	"job_init_cooling_rate" : 0.2,
	"job_iterations_per_temp" : 100000,
	"task_init_temp" : 100,
	"task_init_cooling_rate" : 0.1,
	"task_iterations_per_temp" : 10000,
	"executor_path" : "enrique"
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
	        "job_starting_temp" : 200,
	        "job_cooling_rate" : 0.2,
	        "job_count" : 10,
	        "task_starting_temp" : 100,
	        "task_cooling_rate" : 0.1,
	        "task_count" : 10,
	        "best_location" : "{ \"path\" : \"SUJENAGHWOPQ\" }",
	        "best_energy" : 400.5,
	        "energy_history" : [
				0.2,
				400.5,
				10.122
			]
		},
    	{
	        "job_id" : 47,
	        "job_name" : "Solve All Problems",
	        "job_starting_temp" : 250,
	        "job_cooling_rate" : 0.6,
	        "job_count" : 32,
	        "task_starting_temp" : 80,
	        "task_cooling_rate" : 0.8,
	        "task_count" : 100,
	        "best_location" : "{ \"index\" : 2345093 }",
	        "best_energy" : 12039454,
	        "energy_history" : [
				947345,
				3302,
				12039454
			]
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

**Response (200)**
```json
{
    "job_id" : 42,
    "job_name" : "Solve 99 Problems",
    "job_starting_temp" : 200,
    "job_cooling_rate" : 0.2,
    "job_count" : 10,
    "task_starting_temp" : 100,
    "task_cooling_rate" : 0.1,
    "task_count" : 10,
    "best_location" : "{ \"path\" : \"SUJENAGHWOPQ\" }",
    "best_energy" : 400.5,
    "energy_history" : [
		0.2,
		400.5,
		10.122
	]
}
```

**Response (422)**
```json
{
	"message" : "A parameter is missing"
}
```