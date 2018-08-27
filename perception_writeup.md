## Project: Perception Pick & Place

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps: Pipeline for filtering and RANSAC plane fitting implemented.

I took the code developed in the exercises and applied it in the file 'perception.py' to develop the perception pipeline for this project. After taking the 'pcl_msg' ROS message and converting it to a PCL point cloud, I applied Voxel Grid downsampling to simplify the processing of the point cloud. The cloud is then processed with a PassThrough filter to remove extraneous points which cannot include the objects we're looking for. RANSAC plane fitting is then applied to identify inliers and coefficients of the filtered cloud.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.
Extracting inliers and outliers enables the pipeline to further identify objects with Euclidean clustering. First, those inliers and outliers are turned into a statistical outlier filter which, when combined with a standard deviation threshold filter (set to 1 sigma, to start), informs the pipeline's k-nearest neighbors analysis.

Color information is stripped to process only spatial relationships of points. After creating the Euclidean clustering object and setting cluster size and tolerance parameters, we can then extract indices for each of the discovered clusters. The discovered clusters are then assigned colors according to each segmented object and a newly clustered cloud is created and published in ROS messages.

#### 2. Complete Exercise 3 Steps: Features extracted and SVM trained.  Object recognition implemented.
The next step is to classify the clusters. We take the points for the clusters from the extracted outliers and compute color histograms from the ROS cloud and concatenate normalized histograms as well to form features. The features can then be applied to feed the prediction and develop labels, ultimately identifying Detected Objects with labels and associated cloud information.

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

Using the list of objects, we can then run the mover! The scene, object and pick and place variables are initialized along with information about the list of desired objects for the robot to pick. The robot loops through the objects in the object list, identifying the label and the centroid representing the location of the object. Objects are assigned to the appropriate dropbox, and the centroid of each object is assigned to the pick_pose so that the arm can navigate to the correct pose to pickup the object. The object's routine also receives the posiion of the appropriate arm and dropbox corresponding to its group.

A yaml dictionary is compiled to contain all these identified labels and poses and and then the robot can try to pick/place routine!

![demo-2](https://user-images.githubusercontent.com/20687560/28748286-9f65680e-7468-11e7-83dc-f1a32380b89c.png)

### Room For Improvement

This project presented a significant struggle with the simulator environment and VM. I was not able to appropriately execute the code in the VM, because I consistently ran into Gazebo gui crashes. Gazebo crashing, despite what appears to be the correct model path, left me with just the robot itself loaded in RViz, without any items or collision map information.

![error1](https://github.com/bradleybaggins/RoboND-Perception-Project/blob/master/Gazebo%20Popup.PNG)
![error2](https://github.com/bradleybaggins/RoboND-Perception-Project/blob/master/Gazebo%20Error%20Message.PNG)

I would have liked to attempt the extra challenges in the project. This model will run into struggles when introduced to additional complexity with more objects in its environment to classify. It also presents an interesting problem in navigating around obstacles to pick the correct object. I would also have liked to further refine the segmentation and classification pipeline to explore the cluster extraction approach in more detail.
