Download Link: https://assignmentchef.com/product/solved-ttk4255-homework2-feature-detection
<br>
For general information about the assignments, including grading critera and how to get help, consult the assignments.pdf document on BB. To get your assignment approved, you only need to complete 60% (weighting is next to each task). Upload the requested answers and figures as a single PDF. You don’t need to submit your code. You may use any convenient tool to create your report.

<h1>About the assignment</h1>

In this assignment you will learn how to detect some simple image features. Features play an important role in establishing correspondences,whichis a core operation in pose estimation and3D reconstruction. Detecting simple features, such as lines or corners, is also often part of algorithms for detecting more complex objects, such as calibration checkerboards or fiducial markers.

An important class of features are point features. These are specific, well-localized points where the image attains a maximum in some measure of interest. Among the many measures proposed in the literature, a well-known one is the Harris-Stephens measure used in the “Harris corner” detector, which you will implement here.

Unlike point features, which are identified by their local neighborhood, higher-level features tend to require aggregating information from the whole image. The Hough transform (pronounced “huff”) is a general technique that performs this aggregation by voting. Here you will use the Hough transform to detect lines, but it can also be used to detect circles, ellipses and other analytical shapes.

<h1>Relevant reading</h1>

The Harris corner detector and related keypoint detectors is derived in Szeliski 4.1.1. Note that the terms corner, interest point and keypoint are often used interchangeably. To add further confusion, the term corner does not necessarily refer to projections of 3D corners. Instead, it only refers to points with neighborhoods containing strong evidence of variation along two directions. Such points can be present on various types of structures besides corners, such as highly textured surfaces.

The Houghtransform you willimplementis describedin 4.3.2. Note thatthe Houghtransform described in 4.3.2 modifies the original algorithm (which is presented in Lecture 2) to use edge directions, such that each involved point only votes for a single line.

Feature description, matching and tracking (4.1.2-4.1.4) are topics that we will return to later in the course, and are not required reading for this assignment.

The assignments and projects in this course are copyrighted by Simen Haugo (<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="5e363f2b3931702d37333b301e39333f3732703d3133">[email protected]</a>) and may not be copied, redistributed, reused or repurposed without written permission.

Figure 2: In normal form a line is represented by its angle <em>θ </em>and distance to the origin <em>ρ</em>. Any point

(<em>x,y</em>) on the line satisfies the equation <em>ρ </em>= <em>x</em>cos<em>θ </em>+ <em>y </em>sin<em>θ</em>. Note that the <em>y</em>-axis here is pointing downward to match the axes convention of images, so positive <em>θ </em>indicates clockwise rotation.

<h1>Part 1       Theory questions</h1>

<strong>Task 1.1: </strong>(6%) Lines can be represented in different ways. In the standard form <em>y </em>= <em>ax </em>+ <em>b</em>, a line is represented by its slope and intercept (<em>a,b</em>). However, for the Hough transform we prefer to use the normal (polar) form (<em>ρ,θ</em>) where <em>ρ </em>= <em>x</em>cos<em>θ </em>+ <em>y </em>sin<em>θ</em>. Why might the standard form be problematic?

<strong>Task 1.2: </strong>(8%) Consider a square image of size <em>L</em>, with (<em>x,y</em>) ∈ [0<em>,L</em>] × [0<em>,L</em>], and let lines be represented in normal form (<em>ρ,θ</em>). Keeping the angle fixed at each value below, what is the range of possible <em>ρ </em>values attainable by a <em>visible </em>line (a line that intersects the image) with that angle?

(a) <em>θ </em>= 0<sup>◦                         </sup>(b) <em>θ </em>= 180<sup>◦                          </sup>(c) <em>θ </em>= 45<sup>◦                           </sup>(d) <em>θ </em>= −45<sup>◦</sup>

<strong>Task 1.3: </strong>(8%) Several keypoint detectors are based on the auto-correlation matrix, which is defined per-pixel as the outer product of the image derivatives <em>I<sub>x</sub>,I<sub>y</sub></em>, integrated over a weighted neighborhood:

<h2>                                                                            <strong>A</strong><em> ,                                                     </em>(1)</h2>

where <em>w </em>is a weighting function. Szeliski 4.1.1 describes several scalar “corner strength” measures based on the auto-correlation matrix. The measure proposed by Harris and Stephens is

<h2>                                                                <em>λ</em><sub>0</sub><em>λ</em><sub>1 </sub>− <em>α</em>(<em>λ</em><sub>0 </sub>+ <em>λ</em><sub>1</sub>)<sup>2 </sup>= det(<strong>A</strong>) − <em>α </em>trace(<strong>A</strong>)<sup>2                                                                                         </sup>(2)</h2>

where <em>λ</em><sub>0</sub><em>,λ</em><sub>1 </sub>are the eigenvalues of <strong>A</strong>. Suppose <em>w </em>is radially symmetric and explain why the HarrisStephens measure is invariant to intensity shifts (<em>I</em>(<strong>x</strong>) → <em>I</em>(<strong>x</strong>) + <em>c</em>) and image rotation. Suppose <em>w </em>is not radially symmetric (e.g. a box blur kernel), is the measure still invariant to these transformations?

<strong>Task 1.4: </strong>(8%) The Harris-Stephens measure is not invariant to contrast changes (<em>I</em>(<strong>x</strong>) → <em>cI</em>(<strong>x</strong>)). However, does this affect which points are detected as corners if the acceptance threshold is specified <em>relatively </em>as a fraction of the highest occurring corner strength (as in Task 3)?

<h1>Part 2        Hough transform</h1>

Here you willimplementthe line detectorin Szeliski 4.3.2 basedon the Houghtransform. The basicidea is to accumulate votes for the presence of specific lines (<em>ρ,θ</em>) in an accumulator array <em>H</em>. Specifically, an edge with location (<em>x<sub>i</sub>,y<sub>i</sub></em>) and orientation <em>θ<sub>i </sub></em>votes for the line (<em>ρ<sub>i</sub>,θ<sub>i</sub></em>),where <em>ρ<sub>i </sub></em>= <em>x<sub>i </sub></em>cos<em>θ<sub>i</sub></em>+<em>y<sub>i </sub></em>sin<em>θ<sub>i</sub></em>. The vote is added by incrementing the corresponding cell in the accumulator array. Because arrays can only store values at a finite set of indices, the <em>ρθ</em>-space must first be quantized into <em>N<sub>ρ </sub></em>× <em>N<sub>θ </sub></em>bins, with a pair of ranges [<em>ρ</em><sub>min</sub><em>,ρ</em><sub>max</sub>] × [<em>θ</em><sub>min</sub><em>,θ</em><sub>max</sub>] mapping the continuous parameters to indices in <em>H</em>:

row = floor<em>,       </em>column = floor

After accumulating votes from all edges, those elements of <em>H </em>with more votes than their immediate neighbors, and a minimum number of votes, are extracted and interpreted as dominant lines.

The task2 script provides code to generate the requested figures on a sample image. The basic edge detector from Homework 1 is also provided, which handles the first step of extracting edges.

<strong>Task 2.1: </strong>(5%) Determine appropriate ranges for <em>θ </em>and <em>ρ </em>based on the input image resolution. Your ranges should ensure that all potentially visible and distinct lines can be represented in the array.

Note: As discussed in Szeliski, using edge directions in the voting gives a line two distinct orientations, depending on the edge direction (whether the image went from dark to bright or vice versa along the line normal). Therefore, the minimal range for <em>θ </em>here is of size 2<em>π</em>. Note also that the ranges provided in the book chapter assume normalized pixel coordinates, but you should use ordinary pixel coordinates.

<strong>Task 2.2: </strong>(25%) Compute the accumulator array as described above and include a figure showing the array for the sample image. Let the horizontal axis of your array represent the angle <em>θ </em>and the vertical axis represent <em>ρ</em>; otherwise the provided code for the figure will be wrong. For now, use a small resolution (<em>N<sub>ρ </sub></em>= <em>N<sub>θ </sub></em>= 200). You should be able to see several bright spots, corresponding to the parameters of the dominant lines in the sample image.

<strong>Task 2.3: </strong>(10%) Use the provided function extract_local_maxima to extract the dominant lines. The function takes a minimum acceptance threshold, specified as a fraction between 0 and 1 of the maximum array value. Set this to 0.2. Convert the row and column indices back to continuous (<em>ρ,θ</em>) quantities and include figures showing the location of local maxima in the accumulator array and the lines drawn back onto the input image. For better results, you can try to increase the accumulator array resolution and possibly adjust the acceptance threshold.

<strong>Task 2.4: (Optional self-study task – 0%) </strong>The results shown in Fig. 1 were achieved by two further improvements. The first was to implement the optional edge detector improvements described in Homework 1. The second was to modify extract_local_maxima to use a larger neighborhood, in order to suppress smaller nearby maxima. Are you able to get similar results?

Figure 3: Detected Harris corners in a sample image from <a href="http://www.cvlibs.net/publications/Geiger2012ICRA.pdf">(Geiger</a> <a href="http://www.cvlibs.net/publications/Geiger2012ICRA.pdf"><em>et al.</em></a><a href="http://www.cvlibs.net/publications/Geiger2012ICRA.pdf">, 2012)</a><a href="http://www.cvlibs.net/publications/Geiger2012ICRA.pdf">.</a> The checkerboards are used for camera calibration. Some well-known calibration packages localize the interior vertices of the checkerboard patterns using the Harris corner detector. (Today there are more robust methods).

<h1>Part 3        Harris detector</h1>

Here you will implement the Harris corner detector to gain a deeper understanding of its usage and limitations. As described in Szeliski 4.1.1, its basis is the auto-correlation matrix

<h2>                                                                             <strong>A</strong>                                                     (4)</h2>

where <em>I<sub>x </sub></em>and <em>I<sub>y </sub></em>are the horizontal and vertical image derivatives. Conceptually, the quantities, and <em>I<sub>x</sub>I<sub>y </sub></em>are formed at each pixel and locally integrated over a neighborhood weighted by <em>w</em>. A corner strength image can then be computed using one of the measures in Szeliski 4.1.1 and analyzed for local maxima. Using the Harris-Stephens measure mentioned earlier gives the well-known Harris detector.

The task3 script provides code to generate the figures requested in Task 3.1 and 3.4.

<strong>Task 3.1: </strong>(15%) Compute the Harris-Stephens measure for the sample image calibration.jpg.

Include a figure showing the resulting corner strength as a grayscale image.

Image derivatives should be computed by convolving with the partial derivatives of a 2-D Gaussian with standard deviation <em>σ<sub>D </sub></em>(the differentiation scale). The weighting function <em>w </em>should be a 2-D Gaussian with standard deviation <em>σ<sub>I </sub></em>(the integration scale). The hand-out code includes a function derivative_of_gaussian to compute the image derivatives. Use <em>σ<sub>D </sub></em>= 1<em>,σ<sub>I </sub></em>= 3 and <em>α </em>= 0<em>.</em>06.

<strong>Task 3.2: </strong>(5%) What do negative corner strength values indicate?

<strong>Task 3.3: </strong>(5%) Why do some of the checkerboards have a weaker response than others?

<strong>Task 3.4: </strong>(5%) Use the provided function extract_local_maxima to extract strong corners. Set the acceptance threshold to 0.001 of the maximum corner strength. Include a figure showing the extracted corners as a scatter-plot over the sample image.

Figure 4: Test images. Left-to-right: checker1.png, checker2.png, arcs.png.

<strong>Task 3.5: (Optional self-study task – 0%) </strong>The Harris detector is not scale invariant. Thus, given two images taken at different distances from a scene, the detected corners may not be attached to the same world points in both images. When specifying the parameters <em>σ<sub>D </sub></em>and <em>σ<sub>I</sub></em>, we can control what characteristic scale of a corner we are looking for. For example, in the arcs image, should the lower-right structure be considered as a corner or a circular edge?

The answer depends on the application and <em>σ<sub>D </sub></em>and <em>σ<sub>I </sub></em>must be chosen appropriately. The differentiation scale <em>σ<sub>D </sub></em>is used to stabilize gradients in the presence of image noise, and to simulate the smoothing effects of capturing the image at a greater distance. The integration scale <em>σ<sub>I </sub></em>is chosen larger than <em>σ<sub>D</sub></em>, and is used to control the characteristic scale.

Try your Harris detector on the three test images shown above (included in the zip), using different values for <em>σ<sub>D </sub></em>and <em>σ<sub>I</sub></em>, and try to answer the following questions:

<ul>

 <li>Keeping <em>σ<sub>D </sub></em>fixed (e.g. equal to 1) and increasing <em>σ<sub>I </sub></em>(e.g. from 1 to 100), do the detected corners converge to any particular point?</li>

 <li>Are some types of corners more stable in scale than others?</li>

</ul>