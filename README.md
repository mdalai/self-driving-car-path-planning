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

**Why SAFE DISTANCE = 30m**:
car_s moves at the speed of 50mph at most, which is 22 m/s. In every 0.02s car moves to next waypoint in the trajectory. In 0.02s, the car moves 22x0.02 = 0.44 m. We took 50 waypoints in our trajectory generation. So our car will move 0.44x50 = 22 m in general. We add another 8 m as a secure distance between two car, come up with 30m. 

#### Step 2 - Behaviour Planning
Based on the prediction results, we will do behaviour planning in this step.
```
IF (the ego car is too close from the car in front):
	IF (it is safe to change lane LEFT):
		Change Lane Left.
	IF (it is safe to change lane RIGHT):
		Change Lane Right.
	IF (it is NOT safe to change lane):  STAY in the lane, GO slower.

ELSE  (the ego car has no car in in front):
	IF (the ego car goes slower than MAX SPEED): 
		Accelerate. 
	IF (the ego car goes faster than  MAX SPEED): 
		Stay with  MAX SPEED. 
```
Notes:
- To avoid jerk, max acceleration speed is defined as 0.224.
- Max Speed: 49.5 mph.

#### Step 3 - Trajectory Generation
In this step, we will generate trajectory based on the behaviour planning result. There are 4 different types of Motion Planning Algorithms in general. They are 
- Combinatorial Methods
- Potential Fields Method
- Optimal Methods
- Sampling based Trajectory Generating methods. This is the main one we learned in the class.

I take following steps to generate trajectory:
- Prepare 5 points for generating Polynomial
- 2 points are from previous path planner, this makes sure the smooth transition
- 3 points are newly generated, 30m space between each points. These points in the end serves in purpose of smooth transitions
- Fitting  Polynomial using [spline]
- Generate Trajectory using  Polynomial


Notes:
- using Frenet coordinate makes the following lane problem simple. 
- Jerk minimization method need to be adapted.  
- previous waypoints are used in order to make smooth transition 
- Polynomial trajectory generation. We used package - spline
- we will generate 50 waypoints in front of the ego car. The car takes 0.02s to travel from one waypoint to next waypoint. So the planner generates 1 second trajectory.



## Project Problem Solving Approach
