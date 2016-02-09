#Faleiro
Faleiro is the code name for our scheduler. Our scheduler is controlled from our web api and is responsible for accepting resource offers from the Mesos master, collects tasks to be scheduled from active jobs, and chooses a subset of these jobs to be run on availble resource offers. 

## Using Resource Offers
Mesos master sends resource offers to the scheduler by calling `MagellanFramework::resourceOffers()` which stores is in a `BlockingQueue` which is accessed by Fenzo later. 


## Picking new tasks
 