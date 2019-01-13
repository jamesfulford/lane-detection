# **Finding Lane Lines on the Road**

## James Fulford

---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

To make my pipeline a little clearer, I made a list of functions in the order I would like them to be applied, then used `functools.reduce` to execute them in order, providing the output of one step as the input to the next.

- Convert image to grayscale
- Apply gaussian blur (kernel size: 5) to average out less important features
- Detect lines using Canny function
    - range: 80 to 150, which is close to 1:2 ratio
- Remove data outside trapezoidal region of interest
    - NOTE: If applied earlier, the Canny algorithm would pick up on the edges of the region as lines.
    - The region of interest is a symmetrical trapezoid, horizontally aligned with the image.
    - Height is 39% of the image's height, starting at the bottom of image.
    - Bottom width is 14/16ths of the image, with 1/16th buttressing the region on either side.
    - Top width is 2/16ths of the image, with 7/16ths buttressing the region on either side.
- Handle lines
    - Detect lines using HoughLinesP
    - Group lines by sign of slope
        - Since positive y axis goes downward in images (origin is at top left), intuition is deceptive
            - Left lane line, while appearing to have positive slope, has negative slope
            - Right lane line, while appearing to have negative slope, has positive slope
    - In each group of lines (roughly corresponding to a lane line)
        - filter outlier lines that are either:
            - flatter than the average line for that group
                - The horizontal marker at second 11 in the yellow line video prompted this.
            - too steep (angle between x-axis and line is over 85 degrees)
                - This protects from some divide by 0 cases, but might cause problems with curving lines.
        - average the group's lines into one line
        - extend the line to go from region of interest's height to bottom of the image.
            - note that the line may exit the region of interest by exiting out the slanted sides. However, it will not go higher than 39% of the image's height.


### 2. Identify potential shortcomings with your current pipeline

- The region of interest is static.
    - Given its shape, curves may not play well.
    - When going uphill, it's possible for things beyond the road to be included.
    - When going downhill, the region may not be looking far-enough ahead. This could be particularly bad since downhill usually implies speed, and seeing far ahead is useful for knowing when to stop.
    - Other cars in the region of interest may cause issues.
- Pipeline implicitly expects lane lines to be very visible and well-defined. This may not work well at night or during rainy periods.
- Grouping of lines currently uses the sign of the slope to classify left or right lane lines.
    - It is possible for some right-lane lines to have the same sign slope as left-lane lines due to a curve.
    - It is also possible for non-useful lines on the other side of the region of interest to be classified as part of a lane line they clearly are not a part of (i.e. a stick on the right side of the road may be classified as a left-lane line because of the direction it points).
- Filtering lines
    - Steep lines are filtered out. At some point in a curve, those lines may be the lane line.
    - If 'fake lines' are steeper than the real lane lines, it's likely that the flatter real lines will be filtered out.

An obvious shortcoming is that detecting the lines may not always be sufficient for the problem we wish to solve. Not all roads have well-defined lane lines. Internationally, lane markings may vary. Also, not all lines on the road are indicative of boundaries - some are essentially road signs, some may be markings put in by construction crews and are intended to be ignored. In any case, this is a good place to start. We can improve our solution when we get to those problems, but for now we would focus on getting a full solution first, then improving second.


### 3. Suggest possible improvements to your pipeline

- The degrees of lane lines should form a bi-modal distribution, which should be used to classify lines
    - Should fix divide by 0 problems
    - Should fix some potential issues with curves
- Line classification should also consider location of lines, not just slope/degrees.
- Line filtering should remove outliers using something better than average, since averages are very sensitive to outliers.
- Line filtering should also consider rough position of lines along with slope/degrees
- Line filtering could be stateful, remembering where the lines were in the previous image to start with a good guess on the next image. That info can help determine which lines to ignore.
- The image should be adjusted in brightness to fill the entire 0-255 spectrum to avoid issues with dark images (kernel size may also need to be increased)
- The region of interest's height could adjust based on what appears to be the road horizon (when the lines run out)
