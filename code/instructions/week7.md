# Week 7
Welcome to Week 7 of ROS training exercises! We'll be learning about **bag files** and **computer vision** (CV) topics such as 
**color spaces**, **contours**, and **morphological transformations**. This is going to be a very visual week!

## Using Bag Files
#### What Are Bag Files?
A ROS _bag file_ is used to store multiple topic's message data. Bags are the primary mechanism 
in ROS for data logging and are typically created by [rosbag](http://wiki.ros.org/rosbag), 
which subscribes to one or more ROS topics, and _stores_ the serialized message data in a file as it is 
received. These bag files can also be _played back_ in ROS to the same topics they were recorded from,
or even remapped to new topics. 

<img width="860" height="300" src="https://i.imgur.com/HxoUDXd.png"/>  

#### Why Do We Use Bag Files? 
We often use these bag files to play back the recorded topic data for testing the behavior on new features. 
So a common use case for bag files is while we outside testing some feature on the robot, we often 
want to record and store the data on a number of topics so that later we can play back this recorded 
data for testing and debugging. _Warning:_ Bag files are usually pretty big... Usually 
about 1-3 GB for a single bag file so make sure you have enough room on your system 
before you download / uncompress one. 

#### Running Bag Files
In order to play back a bag file simply do:  
```rosbag play <your bagfile>```   
In order to make the bag file run repeatly simple add the loop flag `-l`:  
```rosbag play <your bagfile> -l```  
You can also press `[Space]` in the terminal running the bag file in order to pause/play the recording. 
Remember to run `roscore` before playing the bag file!

The bag files on our repo are **compressed** so **make sure** to 
**uncompress** them before running them.   
You can compress all bag file in a folder by doing:  
```rosbag compress *.bag```  
And uncompress all of them by doing:  
```rosbag decompress *.bag```  

You can find out if a bag file is uncompressed by doing:  
```rosbag info <your bagfile>```   
And seeing `compression: none` 

_Note_: Make sure that `git lfs` is installed (instructions in _Get Started_) when doing 
this exercise or else you may have issues 
uncompressing and running the bag file.  

## Image Visualization 
Just like how we used `rviz` for visualization of the world in previous exercises, now we
are going to use `rqt_image_view` for displaying images. This just a simple GUI for looking
at what image is being published on any topic. The command for running it is:  
```rqt_image_view```   

<a href="https://imgur.com/TH2EAS3"><img src="https://i.imgur.com/TH2EAS3.png" title="source: imgur.com" /></a>

 Practice working with bag files by running the 
 [start_light bag file](../igvc_training_exercises/bag/start_light.bag) by first
  uncompressing it and then playing it on loop while using `rqt_image_view` for visualization.
 

## OpenCV Library 
**OpenCV** (Open Source Computer Vision Library) is an open source computer vision library frequently used for
real-time perception. The library has more than 2500 optimized algorithms, which includes a comprehensive set of 
both classic and state-of-the-art computer vision algorithms. These algorithms can be used to detect and 
recognize faces, identify objects, classify human actions in videos, track camera movements, track 
moving objects, extract 3D models of objects, and much more!


## Mat  
Mat is just the datatype used to describe images in OpenCV and it consists of a header and a data matrix.
In order to converts between ROS Image messages and OpenCV Mat images, we are going to use `cv_bridge`:
```c++
// Two new important includes for CV
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>

void img_callback(const sensor_msgs::ImageConstPtr &msg) {
    cv_ptr = cv_bridge::toCvCopy(msg, "bgr8");   
    cv::Mat frame = cv_ptr->image;
}
```
The reason we are using `sensor_msgs::ImageConstPtr` instead of `sensor_msgs::Image` is because 
we want to avoid accidentally modifying the original data as well as not copy it multiple times. 

To publish a Mat image just do:
```c++
    cv_ptr->image = img;
    cv_ptr->encoding = "mono8";      //mono8 for BINARY, bgr8 for COLORED
    img_publisher_variable.publish(cv_ptr->toImageMsg();
```

## Kernel
A kernel is simply a small matrix used perform convolution of an image. It is commonly used for blurring, 
sharpening, embossing, edge detection, and much more. The most simple type of kernel is an 
identity kernel:
<p align="center">
    <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/5bf6623ca763ba780b471a565eb1b06cd14b445c">
</p>

which just takes the value at the kernel's "origin" (the center element for odd-sided kernels) and puts it on 
the output image. As you might have noticed below, the input image is padded with enough extra zeros around 
the border so the kernel never "hanging off" the edge of input matrix. 

<img width="860" height="300" src="https://i.stack.imgur.com/uEoXw.gif" alt>  

For now, the least you need to know about kernels is that are used by many computer vision 
algorithms and many CV functions will
simply require you to declare the kernel size (usually around 3 to 9). 

Here is an extra [resource](http://setosa.io/ev/image-kernels/) for understanding kernels!

## Blurring 
**Motivation**  
Smoothing is commonly used with edge detection since most edge-detection algorithms are highly sensitive to noisy 
environments. Blurring before edge detection aims to reduce the level of noise in the image, which improves 
the result of the subsequent edge-detection algorithm:   

<img width="860" height="200" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Noise_Comparison.JPG/300px-Noise_Comparison.JPG" alt=""/>

Often, the first step in a computer vision algorithm is to blur the image since this will smooth out 
small inconsistencies, "noise" in the camera sensor. A widely used type of blurring is 
[Gaussian Blurring](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#gaabe8c836e97159a9193fb0b11ac52cf1). 
To perform a gaussian blur, just run 
```c++
cv::GaussianBlur(frame, frame, cv::Size(3,3), 0, 0)
``` 
where in this case the kernel size is 3. 

## HSV
A color space is just a way of encoding a color with numbers. BGR is the default color space for OpenCV, 
where each color is a blue, green, and red component. Hue, saturation, and value (HSV) is another 
common color space where each pixel is described by it base color (hue or tint) in terms of their shade 
(saturation or amount of gray) and their brightness value. In HSV, colors of each hue are arranged in a 
radial slice, around a central axis of neutral colors which ranges from black at the bottom to white at the top. 
![](https://miro.medium.com/max/1700/1*W30TLUP9avQwyyLfwu7WYA.jpeg)
Working with HSV is very useful and convenient when tackling perception problems like if you wanted to reliably 
check if a pixel is roughly a hue of purple. It would definitely be possible to do using the RGB color space, 
but it would be kinda tough if the lighting conditions weren't
fixed since you might have to specify multiple ranges; however, with HSV is it relatively simple. 

The way to convert an BGR image to HSV is by doing:  
```c++
cv::cvtColor(frame, hsv_frame, cv::COLOR_BGR2HSV);
```

## Thresholding
Thresholding is pretty simple. Given an image, check if each pixel is within a bound or not. 
In the image below, numbers are displayed so that the number's value corresponds
it is brightness. Then through binary thresholding, each pixel will a value greater than 0 is set 
to 255 (The max brightness). 
![](https://www.learnopencv.com/wp-content/uploads/2015/02/opencv-threshold-tutorial.jpg)
We can also check if each pixel is within in a certain range by using `cv::inRange()`.
A common usage of this function is for color thresholding like turning a 3-channel HSV image 
(where each pixel has 3 values 0-255) into a 1-channel binary image (where each pixel 
has 1 value, true or false). For example, if we wanted to find the color blue/green in 
an image by checking if each pixel was within an pre-defined HSV range. This can be done 
simply by doing:
```c++
cv::inRange(hsv_frame, cv::Scalar(20, 120, 120), cv::Scalar(100, 255, 255), green_found);
```

## Morphological Transformations
Morphological operations are a set of operations that process images based on shapes.
The most basic morphological operations are: Erosion and Dilation. They 
have a wide array of uses:
- Removing noise
- Isolation of individual elements and joining disparate elements in an image.
- Finding of intensity bumps or holes in an image  

Here the kernel is defined as:
```c++
cv::Mat kernel(int x, int y) {
    return cv::getStructuringElement(cv::MORPH_RECT, cv::Size(x, y));
}
```
Which just means that the return kernel is a rectangle of _x_ by _y_. 

**Dilate:**  
The dilatation makes the object in white bigger.    
`cv::dilate(frame_binary, frame_binary, kernel(3, 3));`  
**Erode:**  
The erosion makes the object in white smaller.  
`cv::erode(frame_binary, frame_binary, kernel(3, 3));`  
**Opening:**  
Opening is just another name of erosion followed by dilation. It is 
useful in removing noise, as we explained above.  
`cv::morphologyEx(frame_binary, frame_binary, cv::MORPH_OPEN, kernel(3, 3));`  
**Closing:**  
Closing is reverse of Opening, Dilation followed by Erosion. It is useful 
in closing small holes inside the foreground objects, or small black points on the object.  
`cv::morphologyEx(frame_binary, frame_binary, cv::MORPH_CLOSE, kernel(3, 3));`  
<table style="width:100%">
   <tr>
    <th><img src="https://docs.opencv.org/trunk/j.png" alt></th>
    <th><img src="https://docs.opencv.org/trunk/erosion.png" alt></th>
    <th><img src="https://docs.opencv.org/trunk/dilation.png" alt></th>
    <th><img src="https://docs.opencv.org/trunk/opening.png" alt></th>
    <th><img src="https://docs.opencv.org/trunk/closing.png" alt></th>
  </tr>
    <tr>
       <th>Original</th>
       <th>Erode</th>
       <th>Dilate</th>
       <th>Opening</th>
       <th>Closing</th>
    </tr>
</table>


## Contours 
Contours can be explained simply as a list of points that 
represents the edge of a shape.
Contours are a useful tool for shape analysis and object detection and recognition.
<img width="660" height="200" src="https://lh5.googleusercontent.com/EZxifb4wmL0Kwfte3awn5mtkLKeHR1G94K3iG4JJdlwExu4FnglC3euH8zwWuM6LSs682i0yL2_GhgN7V0LXS14HMM49YkVtLcJwxDzL-CQ_jqVwoQYF4zJZPAX5HbvuvKU5vA28" alt=""/>  
In order to find the contours in a image, simply do:
```c++
vector<vector<cv::Point>> contours;
findContours(frame_binary, contours, cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```
**Area:**  
In order to find the area of a contour do:  
```double area = cv::contourArea(contour, false);```
  
**Perimeter:**  
In order to find the perimeter of a contour do:  
```double perimeter = cv::arcLength(contour, true);```
  
**Circularity:**  
In order to find the circularity of a contour do:  
```double circularity = 4 * M_PI * (area / (perimeter * perimeter));```  
since   
<img src="https://latex.codecogs.com/gif.latex?\text{Circularity}&space;=&space;\frac{4\pi&space;Area}{(Perimeter)^2}" title="\text{Circularity} = \frac{4\pi Area}{(Perimeter)^2}" />  

# Exercise 
Now you have everything you need to write a start light detector in ROS! Write your node in 
[src/week7/main.cpp](../igvc_training_exercises/src/week7/main.cpp). Debug using 
`rqt_image_viewer`.     
1. Subscribe to `/camera/image` and write a callback function taking in a `sensor_msgs::ImageConstPtr`.
Also, make a publisher for `/event/race_started` of type `std_msgs::Bool` for if the race has begun
2. Color threshold the image for green and red (both states of the start light)
3. Remove small noise and connect related components by using morphological transformations
4. Check which state the start light is on by checking if there exist a sufficiently large
circular shape in the thresholded images. 
[Hint](#spoiler "check if there exist a contour with 
an area larger than a specified area threshold and a circularity larger than a specified circularity threshold")
5. Check if the red light is on, then if the red light turns off and the green light turns on
within the next 1 second, publish that the race has started.

## Extensions:
<details>
<summary>1. <bold>Line Detector</bold> </summary>  
 
Write a node in a new file (make sure to update the CMakeLists.txt) 
which detects the track lines in the [drag race](../igvc_training_exercises/bag/drag_race.bag) bag file. 

In this exercise, we are going to use the Laplacian operator which is just a fancy way of 
saying we are taking the second derivative over the image using a kernel. This operator
pretty much finds the gradient of the image and is useful for detect edges. It can be applied
to an image by doing: 
```c++
cv::Laplacian(frame_gray, laplacian, CV_16S, 3, 1, 0, cv::BORDER_DEFAULT);
```

Recommended Steps:
1. Blurred image 
2. Convert to greyscale 
3. Apply the Laplacian operator to the greyscale image
4. Threshold the Laplacian output within a range to make a binary image
[Hint](#spoiler "Use a threshold range around -200 and -10 for line detection")  
5. Block the areas that we are not interested in, like the sky, with a black fill rectangle 
covering the top half of the image 
[Hint](#spoiler "cv::rectangle(img, cv::Point(0, 0), cv::Point(img.cols,  img.rows/2), 0, CV_FILLED);")  
6. Remove noise by ignoring any contours smaller than a certain area 
7. Publish binary image 
</details>

# Summary

And that's it for this week!  

We learnt about:
- [Bag Files](#using-bag-files)
    + Stores messages published from ROS topics 
    + Great for logging data on test days 
    + Visualize image messages using `rqt_image_view`
- [OpenCV](#opencv-library)
    + Very helpful for address computer vision problems
    + Use Mat datatype to storing images 
- [Image Processing](#kernel)
    + Commonly, a kernel is used in CV algorithms for applying some effect throughout 
    an image   
    + Blurring is frequency done after obtaining an image in order to smooth the image and
    remove camera noise. 
- [Color Thresholding](#hsv)
    + Easily done by converting RGB image to HSV, then performing thresholding within
    a range. 
- [Morphological Transformations](#morphological-transformations)
    + Used often to remove noise and to connect close features in a binary images
- [Contours](#contours)
    + Consists of the edge points of a shape
    + Has a lot a nice associated functions for finding useful feature information
     (area, perimeter, circularity, and a couple more)
