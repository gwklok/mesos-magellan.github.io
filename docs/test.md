# Testing

## Unit Testing

### Scheduler

The scheduler has a suite of jUnit tests that are run on every push to GitHub and after building locally.

## Performance Testing

### Stress Test

Create many jobs over time

### Soak Test

Run a few long jobs

### Spike Test

Create many jobs at once

### Solution Comparisons

#### Distributed Simulated Annealing vs. Bruteforce

#### Distributed Simulated Annealing vs. Locally Serial

#### Distributed Simulated Annealing vs. Locally Parallel

## Accuracy Testing

To prove the accuracy of Magellan and Travelling Sailor, we ran a bruteforce solution to solving TSP for 10 cities. Magellan was able to find the same best fitness as the bruteforce solution. The fitness was measured as the geodesic diestance between the cities for a round trip back to the starting city. The best fitness found was `28919.86893 km` and the data used was:

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