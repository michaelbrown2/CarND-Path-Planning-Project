# CarND Path Planning Project

[image1]: ./images/NoIncident.png "EKF"


My Path Planning Project began with much of the code that was covered in the Getting Started video just to get the car moving. From there, I went through the process of setting up a cost system for lane changing, analyzing the other vehicles on the road, and ensuring no accidents would / should occur. My control system has successfully run for well over 30 minutes with no accident.  I uploaded a youtube video at https://youtu.be/S-_gFasKYHQ that shows a 10 minute section. Also, below you will see a screenshot of the ~30 minute mark.
![alt text][image1]

The first hurdle to overcome was controlling speed when approaching a vehicle in front of ours.  I have three outcomes when it comes to this. Initially if there are no vehicles in front of us for at least 40 meters, the car will drive at the speed limit. Between 20 and 40 meters, we match the speed of the car in front of us, and if a car cuts us off, we will decrease our speed to 75% of that cars speed until we get our 20 meter cushion again.

```C++
if ( center_closest < 20 )
	{
		target_speed = closest_speed * .75;
	} else if ( center_closest < 40 && closest_speed <= speed_limit )
	{
		target_speed = closest_speed;
	} else
	{
	target_speed = speed_limit;
}
```

Next step was to calculate cost for staying in our current lane. We would PREFER to stay in lane instead of changing lanes, so it's weight starts at 0 and is calculated only by the speed limit - the speed of the closest car in front of us.

```C++
maintainLCost = speed_limit - ( check_speed * 2.23694 );
```

Then we calculate cost of changing lanes.  The same check is done on both lanes, and the closest car ahead of us in that lane is the only one which measurements are used. We also take the speed of the car in front of us and match it through the lane change. Finally, we increment a car counter in that lane, so the program can see if there are no cars and give a good cost to that lane switch

```C++
leftLCCost = 10.0 + speed_limit - ( check_speed * 2.23694 );
	leftLCCost -= .2 * difference;
	left_closest = difference;
	left_lane_change_speed = check_speed * 2.23694;
	leftCarCount += 1;
```

Finally, we use those costs to determine whether or not we should change lanes. We also check to ensure that we have not changed lanes for at least 100 Meters, to make sure we arent just constantly switching lanes and being 'that guy'

```C++
if( leftLCCost < maintainLCost && ( leftLCCost <= rightLCCost || right_blocked )  && !left_blocked && abs( car_s - last_lane_change ) > 100.0 )
	{
		lane -= 1;
		last_lane_change = car_s;
		target_speed = left_lane_change_speed;
	} else if( rightLCCost < maintainLCost && !right_blocked && abs( car_s - last_lane_change ) > 100.0 )
	{
		lane += 1;
		last_lane_change = car_s;
		target_speed = right_lane_change_speed;
	}
```