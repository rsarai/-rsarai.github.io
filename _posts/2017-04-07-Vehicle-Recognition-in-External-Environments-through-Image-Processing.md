---
layout: post
title:  Vehicle Recognition in External Environments through Image Processing
categories: []
excerpt: This was the result of my scientific research, made in 2015 for the University of Pernambuco.
---

This was the result of my scientific research, made in 2015 for the University of Pernambuco.
<br>
<hr>

_This research project aims to assist the development of a local system of low cost, independent of external conditions can monitor the flow of vehicles at intersections. Using digital image processing techniques to recognize vehicles and from that, in situations of need, make decisions. In a simplified approach, computational methods are presented to detect vehicles in urban environments. The solutions in this work have been implemented with VisualStudio C++ and using the OpenCV library, to achieve recognition. Solutions are presented for two types of entries: image and video. The result of the algorithms found for reconnaissance vehicles in images are more accessible to the objectives of this research. However it is also proposed a solution using a recognition algorithm in videos._

<br>
There are different approaches in the literature that explore the detection and recognition of vehicles. A comparison can be made between processed objects and objects of an initial database, also is possible to get a result optimizing the subtraction of the scenario and using the symmetric difference between consecutive frames.

There are also hybrid approaches, which in addition to computer vision seek other means to optimize recognition and detection, such as the use of laser-based sensors.

Also can be used techniques of segmentation and selection of regions of interest, techniques for detection and recognition of vehicles and removal of shadows, focusing on approaches using static cameras, aimed at active traffic monitoring. Among the methods for volumetric counting, we can highlight those who bring approaches using static monitoring cameras.

Neural networks are applied for vehicle detection. But there are two major problems with the neural network: that there is no guarantee of reaching the global minimum and a second implies that it defines a data set representative of the real world and that there is no ideal neural network model. Fuzzy Logic measures are used to detect vehicles. The light intensity value is used as a diffuse measure. When the intensity value falls within a certain range, distorted measurements must be used to decide whether it is a vehicle or not.

First I used the Haar feature based on a Cascade Classifier to detect the object. The feature was initially proposed by Paul Viola and improved by Rainer Lienhart. A classifier (ie, a cascade of work powered classifiers with similar Haar characteristics) is formed with a few hundred sample views of a particular object (ie a face or a car), called positive examples, which are sized for The same size (for example, 20x20), and negative examples — arbitrary images of the same size.

After a classifier is trained to be applied in a region of interest (the same size as that used during training). The classifier produces a “1” if the region is likely to show the object (ie face / car), and “0” otherwise. To search the object across the image you can move the search window across the image and check each location using the classifier. The classifier is designed so that it can be easily “resized” in order to be able to locate objects of interest in different sizes, which is more efficient than resizing the image itself. Thus, to find an object of an unknown size in the image the check procedure should be done several times at different scales.

Several classifiers were used for testing. Some proved to be efficient for cars in some specific positions, but none proved to be efficient in normal traffic situations.

![alt text](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_GG3pZUnwX9x0VRKOeZSjcA.png "Result 1")

![alt text](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_PkbR8HdOhDcFM7FGByf0Nw.png "Result 2")

![alt text](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_VdKoHyQhh_auJ1JdW3u-JQ.png "Result 3")


As the results were not completely satisfactory with the classifiers found it was noted the need to train a classifier that would meet the needs of this work. But there was no database for training the classifier. It was necessary to create a database with images of cars in the most varied positions.

The initial idea of ​​the project was to develop a tool to perform the volume counting of vehicles in real time. However, due to the great complexity of developing a general purpose counting project, some simplifications were made in order to make implementation feasible. These simplifications concern the acquisition of images, scenery and objects.

Image Acquisition: It must be performed by reading previously stored video files, because the algorithm does not have sufficient computational efficiency to handle real-time video.

The scenario: It should be preferably static, although the system will be able to deal fairly well with smooth movements, but it is critical that the camera is positioned so that it offers a superior view of the track. This choice of scenario is intended to avoid the occurrence of occlusions and to simplify the process of image segmentation and treatment of objects.

Objects: A generalization of objects was made, that is, all objects are considered vehicles. Since we are using a specific scenario and videos, this is a valid generalization, which allows for a simpler recognition process.

The OpenCV library was used for image manipulation and segmentation. For the manipulation, the basic functions of the library were used, that allow the reading, the access, the writing and the presentation of the images.

Segmentation consists of using techniques that exploit some properties of the image in order to subdivide the image into regions or objects that compose it, depending on the problem to be solved. In this case subdivide each frame of the video into two types of regions: scenery and moving objects. A simple approach to detecting these objects is to subtract a scene image from the current frame.

![Original](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_Emv-ujAp6hqf19nrqcCJqw.png "Original")

![MOG2](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_49skSbH-UFMmw_bzHWM_8A.png "MOG2")

![Binary Image](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_w5b91rJycB6OZcsuI4jlmA.png "Binary Image")

![Result](/images/2017-04-07-Vehicle-Recognition-in-External-Environments-through-Image-Processing/1_mKMqe1JL8G2LzUCWuZAaMA.png "Result")


The results in different environments and videos with higher resolution were not as satisfactory as those presented in Figures 4, 5, 6 and 7.

As an application of this algorithm. It is proposed a composition of this algorithm for traffic control. Such a composition recognizes two-way motion, selects the areas of interest in both, and returns to which there is more traffic, through a pixel count. The count can vary by place, time, car flow and various other situations. The simplest solution would be to map the information and divide it into categories: intense flow, normal flow and little flow, then from this pre-defined information, make the decision to release or withhold traffic.

The system was developed in VisualStudio C ++ and uses the OpenCV library from Intel. The version of OpenCV used was 2.4.10. Experimental results are demonstrated in real-time videos made from a single camera. The tests were done on an Intel Core i5–2.60 GHz notebook with 4,00GB of RAM and Windows 8.1.

