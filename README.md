# self-driving-car-path-planning

[//]: # (Image References)
[error]: ./assests/prediction_code_error.PNG
[image1]: ./assests/path_planning.PNG

  ![alt text][image1]

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
At first, let the car move in straight line. We will use 50 waypoints for each path planning. Using the startup code from class. 
**Result**: the ego car moves forward. But it throws errors such as Jerk, out of lane.

Let the car drive within the lane. The solution is so simple. We use Frenet values (s,d), instead
Cartesian coordinates value (x,y). 
**Result**: the ego car drives within the lane. But it throws error such as Jerk, collision.

Now, let’s avoid jerk. Problem is the car speed up to 50MPH in 0.02s (every 20ms car moves to the next point). We want the car to accelerate slowly. 
- Set a certain distance and let the car accelerate slowly within this distance. This avoids jerk. We used 30m as the Frenet s distance. Unfortunately, the car does not drive in straight lane. So We have to find the polynomial of driving track first. And then apply evenly divided x values to polynomial and find the corresponding y values. In addition, we need control the speed within each section. 
- Polynomial calc: use 2 end points from previous path. Add 3 more points those are s+30, s+60, s+90. We set the spline using these 5 points. 
- Using previous path points. This method makes sure the smooth transition. In the code we used 48/49 points from previous waypoints, and only 1 or 2 points are newly generated. 

**Result**: the ego car drives within the lane and jerk warning is gone this time as well. Now it throws collision error.

Let’s solve collision this time. Use Frenet to find out cars that are in the same lane. 
- In 50x0.02xspeed where the car gonna be in the future. s + 50x0.02xs_speed;
- Compare it with our car and if the distance is within 30m, we take a action to slow down our car in order to avoid collision. Let’s say slow down to 30 MPH from 50.

**Result**: Great. The ego car detect car in front and slowed down.  However, in the beginning the ego car still throws Jerk error.  

We need to avoid cold start.
- Set ref_v = 0; 
- And increase ref_v slowly if it is within the speed limit.
- We can update the ref_v within each cycle.

**Result**: Ok. All Jerk errors are gone now. 

Now, I have a general understanding of simulator: how the cars are moving in the simular, how to control car, how to predict other cars etc. In next step, I want the ego car know how to change lane.

I updated the prediction code; Using linear point process model to predict the car s-position in future. If it is within 30m range, we should mark this status and use it in behaviour planning step.
- Find out if a car in front is too close. [car_front_too_close]  
- Find out if it is safe to change lane left.  [left_lane_too_close]
- Find out if it is safe to change lane right. [right_lane_too_close]

I updated the behaviour planning code using three status from above.

**Results**: the car is able to change lange right. The car changed to right most lane, then it is just stuck in the right lane. The car would not go back to center lane. 
If i keep running the car until at about 3.1 miles, the car goes into right most lane, then change lane left and then tries to change lane right again. The car ends up colliding with a car in the right lane. It seems like the car could not detect the car in right lane and turn right anyway. How to avoid this? 

It seems like if car goes into right most lane, it is either not able to come back to center lane or end up collides with other cars. Let’s change the behaviour planning, make the car never goes into the right most lane. 
**Result**: collision happens again at about 1.2 miles. The car change lane left and then change lane right and collides with the car in the right. 

The car thinks it is OK to change lane even though it is not safe to change. It seems like my prediction step has issue. May be the safe distance is too short. Let’s increase the safe distance from 30m to 40m, and try again.
**Result**: did not work. Collision happened as before. 

I am sure something is wrong with my prediction step. I checked the code and found out following error. 

  ![alt text][error]

After updating this error. The car can drive normally. This is terrible error. I learned the lesson hard way. Always remember to print the value when debugging and make sure all printed values are what you expected. 

I want the car to always come back to center lane if is safe to do so. By doing so we will have more options to turn either left or right. This should let car arrive to goal distance in shorter time. 
**Result**: there is no significant improvement. Actually, it took slightly longer time. 

## Next
- More precise prediction can be adapted by adjusting the car speed using Frenet S speed. Currently, we used [car_speed = vx*vx + vy*vy] to calculate the car speed. 
- If the ego car is too close from front car and not safe to change lane, the ego car should goes with speed same with the front car. Currently, we just force the car to slow down.
- Better Frenet to Cartesian coordinate conversion function. The current getXY function is used linear interpolation. It would be better to use polynomial interpolation probably using SPLINE. 
- Better cost function design
- Better process model should be adapted when predicting others cars’ position. In here, we used most simple one, linear point model. More complex models are Nonlinear model, bicycle kinematic model etc.

## Extra Readings
- [Udacity CS373: Programming a Robotic Car Unit 4: Motion](https://www.udacity.com/file?file_key=agpzfnVkYWNpdHl1ckcLEgZDb3Vyc2UiBWNzMzczDAsSCUNvdXJzZVJldiIHZmViMjAxMgwLEgRVbml0GIHQDwwLEgxBdHRhY2hlZEZpbGUYwYUTDA)
- [Path Planning and Collision Avoidance](http://ais.informatik.uni-freiburg.de/teaching/ss10/robotics/slides/16-pathplanning.pdf)
- [Safe Motion Planning for Autonomous Driving](https://wesscholar.wesleyan.edu/cgi/viewcontent.cgi?referer=&httpsredir=1&article=1856&context=etd_hon_theses)
- [Local and Global Path Generation for Autonomous Vehicles Using Splines](http://www.scielo.org.co/pdf/inge/v21n2/v21n2a05.pdf)
- [Medium- Path Planning in Highways for an Autonomous Vehicle](https://medium.com/@mohankarthik/path-planning-in-highways-for-an-autonomous-vehicle-242b91e6387d)
- [Real-time motion planning methods for autonomous on-road driving: State-of-the-art and future research directions](https://www.sciencedirect.com/science/article/pii/S0968090X15003447)
