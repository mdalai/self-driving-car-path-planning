# self-driving-car-path-planning

## Project Overview
### Project goal 
The goal of this project is to build a path planner that creates smooth, safe trajectories for the car to follow. The highway track has other vehicles, all going different speeds, but approximately obeying the 50 MPH speed limit.
### Solution overview
In general, it takes following 3 steps to solve path planning problem in self driving car technology. 
#### Step 1 - Prediction
In prediction step, we will predict behaviours of cars around the ego car. If we do not want to collide with cars around the ego car, this is critical step. We need to check where other cars are in the highway and where they are going to be in the future.

**For each car in the highway, we need to do followings**:
- find out in which lane this car is.
- predict where this car will be in the future using process model. 
- if the future position of this car is within SAFE DISTANCE, we need to make action in trajectory planning step. 

#### Step 2 - Behaviour Planning
#### Step 3 - Trajectory Generation


## Project Problem Solving Approach
