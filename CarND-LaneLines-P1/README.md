**Finding Lane Lines on the Road**

### Reflection

### 1. The Pipeline
I used the functions that were provided for grayscale conversion, blurring, Canny filtering and Hough transformation just as in the previous tutorial chapter. I only minimally tuned the Hough filter parameters, but I found that the postprocessing of the detected lines made a much bigger difference in the result than these filtering steps.

I also applied the region of interest masking function provided, but after some experimentation I found it too hard to find a constant region of interest for the masking, so I decided to implement a dynamic region of interest. The algorithm tracks where the interpolated lane markers are converging, and cuts the region of interest based on this convergence point (the _intersectionpoint_ variable).

After the Hough transformation step, my program goes through the detected line segments, and classifies them based on their slope and their position. The endpoints of lines with a plausible slope (between 0.5 and 2) on the left side of the region of interest go into one bucket, and ones with a similar but opposite slope go into another bucket (there is some overlap between the right and left sides to handle slight turns).

The next step is to fit a line (using cv2.fitLine) on the points in each bucket and use these interpolated lines as the two lane markers. A new convergence point is also identified based on the intersection of the two lines (using a simple algorithm from https://stackoverflow.com/questions/20677795/how-do-i-compute-the-intersection-point-of-two-lines for the intersection calculation).

The two lines are drawn on a new image with the draw_lines function and then it is composited with the weighted_img function on top of the original image.




### 2. Shortcomings

This is an extremely naive algorithm, and will have trouble with at least the following scenarios:
- sharp curves (both lane marking dashes, as well as the lanes themselves are not straight, convergence point is moving rapidly)
- steep slopes (causes additional curvature, especially combined with lens distortions toward the edges)
- sharp shadows/bright sunlight (can cause unexpected lines on the image)
- missing or distorted lane markers
- construction work with modified lane markers
- night driving
- cars in front of our vehicle (would create a lot of confusing contours, while also obscuring lane markers)

Overall it is an interesting experiment that helped me to learn a lot of the basics of solving this problem, but is only a tiny first step toward a comprehensive solution.


### 3. Possible improvements

Depending on how the output of the algorithm will be used, it may improve results if temporal (inter-frame) interpolation is added. This would also offer the possibility of hiding transient detection issues by simply dropping the problematic frame and interpolating from the previous frame(s). Obviously it would cause temporal jitter/delay in the output.

The challenge puzzle a the end of the notebook with the sharp turn is indeed way outside of the parameters of this algorithm, and solving it would probably require a slightly different approach (curve interpolation instead of linear, for a start).