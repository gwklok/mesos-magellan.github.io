#Faleiro
Faleiro is the code name for our scheduler. Our scheduler is controlled from our web api and is responsible for accepting resource offers from the Mesos master, collects tasks to be scheduled from active jobs, and chooses a subset of these jobs to be run on availble resource offers. 

## Jobs and Tasks
Clients submit problems to be solved with our scheduler