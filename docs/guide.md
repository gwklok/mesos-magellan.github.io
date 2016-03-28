# How to use Magellan

An explanation on how to write and use a module for Magellan to solve your problem.

## Creating a job via the Web UI

* Navigate to the "Create a Job" page by clicking the link on the top-right of the homepage.
* Enter a descriptive indentifying name for the job, (eg. Traveling Sailor).
* Enter how many minutes of CPU time (not real time) you want the job to run for, (eg. 40).
* Enter a URL to a GIT repository or a downloadable tarball (.tar.gz), (eg. git://github.com/mesos-magellan/traveling-sailor).
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