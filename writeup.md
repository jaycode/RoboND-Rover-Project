# Project: Search and Sample Returns (write-up) #

![image](./misc/rover_image.jpg)

The goal of this project is to automatically drive a rover to map an area in a simulation.

The rover is equipped with a camera and other motion sensors. With the images gathered from the camera, we do the following steps to each image:

1. Perspective transform: In this step, we convert the perspective view from the rover into a bird-eye view.
2. Color thresholding: We use simple color thresholding to identify road, obstacles (including walls), and rocks.
3. Coordinate transformations: Convert Rover coordinates into world coordinates so we may stich the views into a complete map. We also convert the navigable terrain into polar coordinates that we can use in the rover's navigation.

## Notebook Analysis ##

Each step above was first run in the Notebook to ensure it is working properly. In this section, I present the results with both a test image provided by Udacity and a recorded image I took from the simulator.

### Original image

Test image:

![test1](./misc/test1.png)

Recorded image:

![rec1](./misc/rec1.png)

### Perspective Transform

With test image:

![test2](./misc/test2.png)

With recorded image:

![rec2](./misc/rec2.png)

### Color Thresholding

With test image:

Road:

![test3a](./misc/test3-a.png)

Rock:

![test3b](./misc/test3-b.png)

Obstacle:

![test3c](./misc/test3-c.png)

With recorded image:

Road:

![rec3a](./misc/rec3-a.png)

Rock:

![rec3b](./misc/rec3-b.png)

Obstacle:

![rec3c](./misc/rec3-c.png)

We were able to identify the navigable terrain, rock, and obstacle area correctly.

To threshold the rock, I use a range thresholding between R: 110, G: 110, B: 5 and R: 255, G: 255, and B: 90. That range collects the gold color properly.

For the road and obstacle, I used R: 160, G: 160, and B:160. The system identifies anything above those values as road and below those values as obstacles.

### Coordinate Transformations

With test image:

![test4](./misc/test4.png)

With recorded image:

![rec4](./misc/rec4.png)

The arrow shows a mean direction of the area of interest. In this case, it points to the area where most navigable terrain (road) lies. We use this direction in our rover automatic navigation module.

## Test Run with a video ##

I stiched together results of running `process_image()` with all images I have recorded, then output it into a video. The video is located at `/output/test_mapping.mp4`.

## Autonomous Navigation and Mapping ##

I included the code I have written in the `process_image()` function from the notebook into the `perception_step()` function at the bottom of `perception.py` script.

Udacity has provided a scaffolding code in `decision.py` that allows some basic logic for the rover to explore the area and turn around when faced with a dead-end.

### Improve tendency to explore

The default code has the tendency to explore the same road over and over. I improved the tendency to explore new locations by jpdating the code in `Rover.mode == 'forward'`: Add the mean steering by a random number between 0 and -10. This means the rover will be more likely to pick the right road, but if it was somehow stuck in a circle, it can also pick a different road by random chance.

### Stuck recovery

In addition to Udacity's scaffold code for `decision.py`, In this code, I also added a simple logic to recover from being stuck:

1. Detect the velocity of the rover for the last 20 frames.
2. If the mean velocity is between -0.01 and 0.01, we determine that the rover is stuck.
3. When the rover is stuck, throttle backward until either:
 - The rover reaches a velocity of -0.9 or lower.
 - The rover has been going in reverse for 100 frames.
4. Then reset the `stuck_counter` limit to 100 and initiate "forward" mode. This means the rover will move forward for at least 100 frames before it can think that it is "stuck" again.

### Optimize for fidelity

The initial code had a hard time reaching the required fidelity > 60%. The fidelity kept decreasing over time. I updated the perception code to only update the rover's worldmap when its roll and pitch values are between -1.0 and 1.0.

## Screenshot of the result
![result](./misc/result.png)
