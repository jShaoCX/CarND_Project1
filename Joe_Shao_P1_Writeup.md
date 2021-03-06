#**Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[imageRoadSection]: ./ResultImages/roadSection.jpg "Road Section"
[imageMask]: ./ResultImages/maskSection.jpg "Mask Section"
[imageRemoveColor]: ./ResultImages/RemoveColor.jpg "Removed Color"
[imageRemoveColor_small]: ./ResultImages/removingRoadSection_small.jpg "Removed Color"
[videoYellow]: ./FinalVideos/yellow.mp4
[videoWhite]: ./FinalVideos/white.mp4
[videoChallenge]: ./FinalVideos/extra.mp4

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

 My pipeline consisted of 5 steps that made use of predefined functions in the helper functions section and 2 additional steps in an attempt to resolve the challenge video. The two additional steps were not necessary for the two non-challenge videos and in fact made the non-challenge videos line-drawing less smooth, but it did allow the challenge video’s lane-line drawing to not bug out as wildly.

 The first step was converting the image to grayscale. This is followed by my two custom steps which flatten non-lane-line edges located in the road and outside the road. I will explain later in this write-up section. I perform Gaussian blur on the processed image with a 5x5 kernel and then perform Canny edge detection on the gray-scale, blurred image. I set the values of the Canny edge algorithm to a lower threshold of 75 and a upper threshold of 150. I then create a mask that only contains a triangle of the lower half of the image: bottom left corner to center of the image to bottom right corner. The binary mask only contains the edges found from the Canny edge detection within the triangle. I then apply a Hough transform on the image with rho =3 px, theta = 2 deg, threshold = 75 px, min_line_length = 75 px and max_line_gap=125 px. The result is an image with only the two lane lines and that is overlaid onto the original image.

 The draw_lines method had to be modified substantially with my current pipeline because it draws as many lines as are detected by the Hough transform. In order to create only two solid lines, I iterated through the found lines and binned them according to their slope range (slopes of 0.2-0.5 were binned together and same with 0.6-0.9). I make sure to exclude any slopes of absolute value less than 0.35 because I figured that a horizontal lane line should not be considered, though I realize this may not be an assumption I’m allowed to make. I then added up all of the distances of these lines in each bin to determine the most common slope. My theory is that the slope of the lane lines should accumulate the most distance because there should be more solid lines found along the white and yellow. After locating the most common positive slope (left lane line) and the most common negative slope (right lane line), I take all of the points within that bin and fit a single line that reaches from the bottom of the image (ysize) to the middle of the image (ysize/2).

 The above pipeline worked fine on the first two videos even with some minor adjustments to the parameters of the image processing algorithms but it failed on the challenge so I added two more steps that I hoped would disregard some of the artifacts in the road, mainly that concrete-colored bridge. Though the additional steps did not help as much as I thought, they were meant to scan the road ahead and set all pixel values that were definitely road to an average pixel value. So when the scan of a small piece of road ahead contains parts of tan concrete instead of dark asphalt (or both concrete and asphalt), the algorithm would go through every pixel the bottom half of the image (the road part) and set both the concrete and asphalt pixels to a standard grey value (128). I figured that this would eliminate any edges in the middle of the road that would create spurious lines in the Hough lines section. I also included a section that would look at the left edge of the road and add those pixel values to the dictionary of “pixels to set to average value”. The image below shows the section of the road I am referring to:

 ![alt text][imageRoadSection]
 
 The resulting road pixel averaging looks like the image below: 
 
  ![alt text][imageRemoveColor_small]
 
 
 Ultimately, the flattening of road artifact pixels and image edge pixels did not help the algorithm as much as I thought it would and it made the pipeline much slower because it had to scan half of each frame’s pixels. Though, it did prevent some of the frames from drawing lines to follow the shadows created by the center divider and the tree. Furthermore, when constraining the slope of the lines to be no less than absolute value of 0.4, the challenge video actually succeeded, though mainly by simply omitting the lines from that particular frame and not actually resolving the image in that frame. The videos in the FinalVideos folder of th repository show the lines cut off before intersecting each other.

![alt text][videoWhite]

[White Lane Line Video](https://www.youtube.com/watch?v=5QFGzzh83Ec&feature=youtu.be)

![alt text][videoYellow]

[Yellow Lane Line Video](https://www.youtube.com/watch?v=M8MODgdEyAs&feature=youtu.be)

![alt text][videoChallenge]

[Challenge Video - Jumpy](https://www.youtube.com/watch?v=nUBTyS9HWc8&feature=youtu.be)

###2. Identify potential shortcomings with your current pipeline


 My pipeline does make several assumptions which may not be valid in situations and the additional custom sections I added slowed the pipeline down noticeably, around 3 times slower.

 The sections where I scan a piece of the road ahead and a piece of the image edge to find all pixel values to average out, makes several assumptions and is slow. The assumption is that a trapezoid right in front of the car is a good area to look for just road colors and no lane line colors would be present. If lane line colors are present in that scan block, then the section I added would average out the lane line colors in the image resulting in no lane lines on the road for the canny edge detection to find, hazardous. The image below is an extreme case of this:
 
  ![alt text][imageRemoveColor]
 
 Also, building that dictionary of pixel values to get rid of and then searching through the bottom half of the image for those pixel values creates a very large time difference in the processing of a single frame. It may not be worth having this section.

 In the masking section, I make the assumption that the region of interest is a triangle but I believe that valuable information could be gathered from the rest of the road. A triangle may not be the best option for masking though with the current pipeline, it is necessary to make sure other neighboring lane lines, center dividers and road shoulders are not picked up by the canny edge detection and outnumber the lane line right next to the car. In the image below, the mask helps remove most of the road's shoulder which could potentially create other lines as it does in the challenge image.
 
 ![alt text][imageMask]
 
 The line drawing section made an assumption that all slopes less than absolute value of 0.35 are invalid. This may not be true if the car is making a sharp turn or maybe crossing lane lines. In the case of my draw_lines function, the line would just be omitted. Ideally, the line would still remain and the next closes lane line, the boundary of the next lane, would appear during a lane change. Creating the lines also assumes that there is a positive sloped line and a negative slope line. Again, this would be true for the camera that is right in front of the car, but if the camera were to change position, the threshold of 0.35 could easily be violated and that particular lane line would not appear. For example, changing it to 0.40 would allow the Challenge video to work but cause the yellow lane line video to, intermittently, have no lane lines:

 [Yellow Lane Line - Jumpy due to bad threshold](https://www.youtube.com/watch?v=nqMc9dktbnE&feature=youtu.be)
 
 [Challenge - with threshold, looks less jumpy but is just omitting those lines](https://www.youtube.com/watch?v=7MDdxSi2Uco&feature=youtu.be)

###3. Suggest possible improvements to your pipeline

Additional image processing is necessary even before the grayscale. There must be a way to quickly, per frame, determine the colors that are not part of the lane lines. Originally, I tried to take the RGB values of the pixels of the road section but that resulted in the pipeline being almost 10x slower because it would have to search through all three channels instead of just a single grayscale channel. I also tried to just color threshold at around yellow and above, but when the concrete in the challenge image showed up, the yellow lane line on the right was completely lost. There must be a way to only threshold by the colors of the lane lines and I believe another color scheme such as HSV would have worked to separate out the colors. I've read online that HSV color space separates color intensity from color instead of just separating between red, green and blue. This seems like the way to go in order to avoid the tree shadows covering the yellow lane in the challenge video. 

In terms of tampering with the existing parts of the pipeline, once within an optimal range, there is only so much that canny edge detection or transforming the image into hough space can do. As for binning the lines and averaging them, I think it would be best to not just pick the most common negative slope and most common positive slope. The image processing steps should ensure that the lines resulting from the Hough transform would be the actual lines we are interested in and the two most common (cumulative distance for all of the lines per slope bin) should simply be selected. 

There may also be a smarter masking strategy. If it is possible, we would want to determine what is the end (or furthest) of the road we can see and maske the image from there instead of arbitrarily selecting half or 3/5 of the image. For lane line detection exclusively, I believe that this would be a safe optimization because we are only concerned with what the ground looks like.

Once other cached information is available, it would be best to remember successful runs of the pipeline on the most recent frames and use successful line fits to guide the current calculation. It would be helpful to cache the most recent lane line colors because we assume those are fairly constant on a long stretch of road. Also, caching the road types and colors would be helpful to determine if a patchy road is anything for the edge detection algorithm to be concerned about. If it is not, then the cache can be used to flatten out those sections of the road. 