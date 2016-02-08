# Web API Communication With Scheduler

## Creating a Job

The Web API receives input as documented on the Web API Schema page. It then validates user input as necessary and forwards the input to an instance of the scheduler framework. To create a job, the API forwards the parameters to the `MagellanFramework::createJob()` method which then returns the created job's ID and returns it to the user making the API request.

## Getting Job Status

When the user requests data about the running jobs, the web API calls the `MagellanFramework::getJobStatus()` and `MagellanFramework::getAllJobStatuses()` methods for a specific job and for all jobs, respectively. The method calls provide all the info about a job which is then returned to the user.

## Manage a Job

The Web API is able to pause, resume, and stop a job. The user makes a request and specificies which action they would like to take. The Web API then makes the appropriate method call to the scheduler framework: `MagellanFramework::pauseJob()`, `MagellanFramework::resumeJob()`, and `MagellanFramework::stopJob()`.