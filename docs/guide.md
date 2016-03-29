# How to use Magellan

An explanation on how to write and use a module for Magellan to solve your problem.

## Defining Your Problem

Finding a solution to your problem with Magellan is as easy writing one Python class. Well, mostly.

In order to define your problem, you must create a new Python file called `problem.py`. This file needs to contain a class called `Problem` which is an implementation of [`pyrallelsa.Problem`](https://github.com/mesos-magellan/pyrallelsa/blob/master/pyrallelsa/__init__.py#L15). In your implementation of problem, define your:
   * `move` method: This will mutate the state of the solution as it is annealing
   * `energy` method: This determines the energy (also known as fitness) of the current solution. Since we are talking in terms of annealing, lower energy is better! (We are aiming to minimize)
   * `divide` method: The implementation of the divide method is up to your choosing. By definition, the divide method should subdivide the solution space given by your `problem_data` into the given `divisions` deterministically.

And you're done defining your problem! That was easy wasn't it?

## Packaging your Problem for Magellan

Once you have your `problem.py` defined ([see *traveling-sailor*\'s `problem.py` if you need an example](https://github.com/mesos-magellan/traveling-sailor/blob/master/problem.py)), all you need to do is either publish this in a git repository (*traveling-sailor* is the optimal example!) or serve this is a gzip archive.

## Creating a job via the Web UI

* Navigate to the "Create a Job" page by clicking the link on the top-right of the homepage.
* Enter a descriptive indentifying name for the job, (eg. Traveling Sailor).
* Enter how many minutes of CPU time (not real time) you want the job to run for, (eg. 40).
* Enter a URL to a git repository or a downloadable tarball (.tar.gz). The following are all valid examples from *traveling-sailor*. The first two are valid links to a git repository and the last is a static tarball. git repositories are recommended as they can be more efficiently updated if cached on an agent.
   * git://github.com/mesos-magellan/traveling-sailor
   * https://github.com/mesos-magellan/traveling-sailor.git
   * https://github.com/mesos-magellan/traveling-sailor/archive/0.1.0.tar.gz

* Enter any additional data in JSON format you want to be passed on to your module. (Example for traveling-sailor at the end of this section).
* Press the "Create Job" button, you will then be forwarded to the "Job Details" page for your newly created job.

Module JSON data example for traveling-sailor:
```json
{
"cities": {
   "Holmesville": [31.7036111, -82.3208333],
   "Jatipasir": [-8.2509, 113.9975],
   "Tapalinna": [-2.923, 119.1626],
   "Sahout el Ma": [16.9166667, -15.1666667],
   "Isanganaka": [-0.683333, 24.083333],
   "El Yerbanis": [29.45, -108.6],
   "Gun-ob": [9.6289, 124.0517],
   "Chak Seventy-five ML": [31.345362, 71.123454],
   "Bakous": [13.7363889, -16.6080556],
   "Metkow Maly": [50.05, 19.4]
},
"updates_enabled": false,
"start_city": "Bakous"
}
```
