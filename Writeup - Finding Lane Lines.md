# **Finding Lane Lines on the Road** 


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

Next I will talk about my approach to this challenge.
I added some pictures and videos produced by my code in this notebook. 
I also commented all of the code so this reflection is seen as an addition to that and might be redundant at points.


### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The function defined as PaintLanes(image) takes said image and puts it through multiple steps of processing:

1. Converting to a grayscale image
2. Applying the gaussian filter to smooth out the edges
3. The canny filter gives us a picture with identified edges and is showing these
4. I then define a sector of any given image that is of interest. This allows me to leave out areas and unwanted result for the    later steps (see vertices)
5. I then mask the image showing the canny edges with this area of interest, to only keep these edges, leaving everything else      out/black
6. Using the hough lines transformation I can identify the lines with the respective coordinates
7. Within this hough line operation I use my updated function called "draw_linesIMPROVED" 
8. After tuning all the operations I talked about beforehand to get the result I expected on the test cases, I then proceeded to    optimize the "draw_linesIMPROVED" function. This has beend the point where it I put in most of my work and learned a lot.
   Hence I will write about this in an extra step:
   
##### Improved version of "draw_lines"
At first I tried just gathering the points from the hough transformation and just with some simple min/max tactics grabbing the end points of the lanes. And actually these results weren't that bad. I did have to tune some more of my parameters like adjusting the vertices to accomodate all test pictures given. But I was not really happy about the outcome because lines were just way smaller than the actual visible lane markings or were of a little and so on. 
I also had to implement it in a way that the slope gives away which lane the lines in the array belong to. With this I was able to have a general separation. But I was stil getting some bad results so I went ahead and tried a linear regression to get the best fit line for both sides. The results actually were worse then the simplistic approach. So I went back to the first simple approach but had to think of ways of improving it. Especially on videos I was not happy with my stand at this point. The painted lanes were flickering around (even ranging over to the other side of the lane), being inconsitent and just didn't look on point most of the time. To make my life eassier with debugging and improving I created 3 steps to analyze videos on my own: Take video and write single frames to folder. Loop through these with the pipeline and write the output to a different folder. Take the frames and put them back together as the output video...
With this I was able to analyze single frames any what was going on exactly. So I will go over my steps in this final function now and talk about the though process:

1. Grab global variables and initialize all needed variables
2. Divide the vales from hough to left and right by checking the slope and making sure that the points are in the right general    area of the frame (I was having lines being recognized of other stuff on the road etc.) 
3. If nothing was foun my Arrays would be empty, creating an error, so in that case I append a "0" and set "FoundIt" to False.
4. If nothing was found in that frame (FoundIt=False) I then use the values of the previous frame to continue. This seems to        work well sind the change happening between each frame is not that crucial at these somewhat "slow speeds" of the car. So I      think this assumption works well.
5. I then grab the minimum Value of x within the left lane Array and assume this is my starting point of the lane on the bottom.    Using the same method for the other values I needed.
6. I found that the lanes with gaps for example but also the consistent lanes would sometimes make my lane markings too short. I    wanted them to start at the bottom of the screen and extend at least to value where it would make sense that lanes could        still be detected in the far. So with the slope of the two points I picked to create the line I extended these to the bottom    and towards the top with using the slope and the general function of a line. I only did this if the line did not already        reach these points.
7. This already gave me pretty good output. But still, It did not look too smoth. The lines were flickering between each frame.     Sometimes the slope had been pretty off. So if the original line was pretty small and the slope was a bit off, the extension     of point 6 made it to where I wasn't always keeping a good track of the lane at the beginning and end in many frames.
    So I though that I could implement it to where it did not accept too big changes in between frames.
    So I limited the change of each point in each direction and if it was exceeded in regards to the previous frame, it only         ajusted the lane to that max extent of allowed change.
    With this I was able to get really good output. Even the "challenge" video is handled quite well I think. 

9. Continuing the general pipeline with the updates mentioned above I then merged the found lines with the original image and      returning that image.



### 2. Identify potential shortcomings with your current pipeline


I now implemented a maximum change of 1 pixel per frame for each coordinate. At high speeds (or low camera framerates) and drastically changing lane conditions this could cause problems.
I also do not know how my code performs if a car crosses into my lane. I would like to test that.
As seen if my code is applied to the "challenge" video strong curves make the painted lanes cross right now, because the extension is too big and it is not optimized for that case.
Also with difficult light settings or different camera positions my code might be having trouble. So it is not as generic as I would like it to be with different camera resultions and positions some things like masking might be off.


### 3. Suggest possible improvements to your pipeline

I could also implement it to where the end points of both painted lanes on the top always have to have a certain distance and hence stop this extension that causes the painted lanes to cross.
I would also like to improve it to where the painted lanes are not linear but can also take the shape of a curve and hence creating a more realistic view for the system. 
