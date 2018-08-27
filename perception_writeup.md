## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify).
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.

I took the code developed in the exercises and applied it in the file 'perception.py' to develop the perception pipeline for this project. After taking the 'pcl_msg' ROS message and converting it to a PCL point cloud, I applied Voxel Grid downsampling to simplify the processing of the point cloud. The cloud is then processed with a PassThrough filter to remove extraneous points which cannot include the objects we're looking for. RANSAC plane fitting is then applied to identify inliers and coefficients of the filtered cloud.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.
Extracting inliers and outliers enables the pipeline to further identify objects with Euclidean clustering. First, those inliers and outliers are turned into a statistical outlier filter which, when combined with a standard deviation threshold filter (set to 1 sigma, to start), informs the pipeline's k-nearest neighbors analysis.

Color information is stripped to process only spatial relationships of points. After creating the Euclidean clustering object and setting cluster size and tolerance parameters, we can then extract indices for each of the discovered clusters. The discovered clusters are then assigned colors according to each segmented object and a newly clustered cloud is created and published in ROS messages.

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
The next step is to classify the clusters. We take the points for the clusters from the extracted outliers and compute color histograms from the ROS cloud and concatenate normalized histograms as well to form features. The features can then be applied to feed the prediction and develop labels, ultimately identifying Detected Objects with labels and associated cloud information.

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

Using the list of objects, we can then run the mover! The scene, object and pick and place variables are initialized along with information about the list of desired objects for the robot to pick. The robot loops through the objects in the object list, identifying the label and the centroid representing the location of the object. Objects are assigned to the appropriate dropbox, and the centroid of each object is assigned to the pick_pose so that the arm can navigate to the correct pose to pickup the object. The object's routine also receives the posiion of the appropriate arm and dropbox corresponding to its group.

A yaml dictionary is compiled to contain all these identified labels and poses and and then the robot can try to pick/place routine!

![demo-2](https://user-images.githubusercontent.com/20687560/28748286-9f65680e-7468-11e7-83dc-f1a32380b89c.png)

Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.

This project presented a significant struggle with the simulator environment and VM. I was not able to appropriately execute the code in the VM, because I consistently ran into Gazebo gui crashes. Gazebo crashing, despite what appears to be the correct model path, left me with just the robot itself loaded in RViz, without any items or collision map information.

I would have liked to attempt the extra challenges in the project. This model will run into struggles when introduced to additional complexity with more objects in its environment to classify. It also presents an interesting problem in navigating around obstacles to pick the correct object. I would also have liked to further refine the segmentation and classification pipeline to explore the cluster extraction approach in more detail.
