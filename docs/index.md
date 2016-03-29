# Magellan

A Mesos framework for Distributed Simulated Annealing.

## Components

### Faleiro (The Scheduler)

Faleiro performs orchestration and assigning tasks to agents. It makes sure your job gets done.

### Enrique (The Executor)

Enrique is responsible for pulling and executing tasks on remote agents. It receives orders from Faleiro.

### Miguel (The Scheduler CLI)

Miguel is a CLI which communicates with Faleiro over its REST API.

### Circumnavigator (Web User Interface)

If you like GUIs, Circumnavigator is the graphical analogue to Miguel.

### Victoria (Development Environment)

Victoria is a development and production environment deployment tool. It can bring up your dev environment and a huge production cluster with a single click (`vagrant up`).

### Traveling Sailor (TSP Solver using Simulated Annealing)

`traveling-sailor` is the official Magellan example Problem.

### PyrallelSA

`pyrallelsa` is the Magellan library that has the Problem definition interface in Python which you can use to define your problem for use with Magellan.
