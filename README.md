Download Link: https://assignmentchef.com/product/solved-computer-vision-lab-exercise-6
<br>
<h1> Chaining &amp; 3 <em>D </em>Reconstruction pipeline</h1>

In a video sequence, a single point is unlikely to remain visible in all the frames. Thus we need to keep track of the points that are present from one frame to the next. We do this by means of a point view matrix (also called observation matrix) which is constructed for all the video frames. It saves the coordinates of the correspondence between two consecutive frames. Later Tomasi-Kanade factorization (introduced in lab 4 for structure from motion) can be performed on the sub-blocks of the point view matrix for a 3D reconstruction. The sub blocks of the point view matrix each represent point correspondences across a particular set of views, you can obtain multiple point clouds from factorizing these sub blocks. Then you can use procrustes analysis to compute transforms to align the point clouds (and perform bundle adjustment if necessary) to obtain the final 3D shape.

<h2>1      Chaining</h2>

Construct point-view matrix with the matches found across image pairs for all consecutive tedy bear images (1-2, 2-3, 3-4, …, 15-16, 16-1). The pointview matrix has views in the rows, and points in the columns: #<em>frames </em>× #<em>matches</em>. You can construct it with the ChainImages.m function which takes as input, a cell array containing matches(each cell contains matches between a pair of images). The steps are elaborated below.

<ol>

 <li>Initialize the point view matrix (The variable ‘PV’) with rows equal to the number of frames you have + 1. (The additional frame is to ease our effort and will be removed later)</li>

 <li>Iterate over the frames saving the matches between each pair of images to the variable ‘newmatches’.</li>

 <li>Initialize the point view matrix with zeros and size equal to</li>

</ol>

1 + <em>numberofframesxthenumberofmatches</em>

between the image pairs. For the first pair, simply add the indices of matched points to the same column of the first two rows of the pointview matrix.

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">if i==1</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">PV(1:2,:) = …</td>

  </tr>

 </tbody>

</table>

<ol start="4">

 <li>Use the intersect function of MATLAB to find points that have been previously found. The indices from the intersection (IA and IB) give the column indices for assigning the correspondences of ‘newmatches’ and ‘prevmatches’ to the next frame (i+1th frame).</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Find already found points using intersection on …PV(i,:) and newmatches</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">[¬, IA, IB] = …</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">PV(i+1,… ) = intersection(… ,… )</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li>Use the setdiff MATLAB function to detect new matches(‘diff’) and their respective indices that are not present in the previous set. For subsequent image frames until the last first frame pair, you will progressively grow the size of your point view matrix. You can ‘grow’ your point-view matrix by simply appending zeros. For example PV =</li>

</ol>

[PV zeros(size<em><sub>x</sub></em>,size<em><sub>y</sub></em>)]. Since you will need to add newly found matches to this matrix the zeros should have columns equal to the number of your new matches(diff) and rows = frames + 1

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Find new matching points that are not in the …previous match set using setdiff.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">[diff, IA] = setdiff(… ,… )</td>

  </tr>

  <tr>

   <td width="59">34</td>

   <td width="429">% Grow the size of the point view matrix each …time you find a new match.</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">start = size(PV,2)+1;</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="429">PV                             = [PV zeros(frames+1, size(diff,2))];</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="429">PV(i, start:end)                           = …</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="429">PV(i+1, start:end) = …</td>

  </tr>

 </tbody>

</table>

Assign the newly found matches to the current view PV(i) and the intersecting points with the previous frame is assigned to the next PV(i+1)

<ol start="6">

 <li>The very last frame-pair, consisting of the last and first frames, requires special treatment. Take the common matches between the first and last frame and move it to the corresponding columns in the first frame and delete the last frame.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Process the last frame-pair. This part is …already completed by TAs.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">% The last frame-pair, consisting of the last and …first frames, requires special treatment.</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">% Move matches between last &amp; 1st frame to their …corresponding columns in</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="429">% the 1st frame, to prevent multiple columns for …the same point.</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">[¬, IA, IB]                                              = intersect(PV(1, :), PV(end, :));</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="429">PV(:, IA(2:end)) = PV(:, IA(2:end)) + PV(:, … IB(2:end)); % skip 1st index (contains zeros)</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="429">PV(:, IB(2:end)) = []; % delete moved points in …last frame</td>

  </tr>

 </tbody>

</table>

<ol start="7">

 <li>Find indices of non-zero elements in the first and last rows of the pointview matrix using the MATLAB function find. You need to copy the non zero elements from the last row which are not in the first row to the first row. Here you can use the not ismember function ismember</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Copy the non zero elements from the last row … which are not in the first row to the first row.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">nonzerolast = find(PV(end, :));</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">nonzerofirst = find(PV(1, :));</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="429">nomember                           = ¬ismember(nonzerolast, …nonzerofirst);</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">nonzerolast = nonzerolast(nomember);</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="429">tocopy                                   = PV(:, nonzerolast);</td>

  </tr>

 </tbody>

</table>

<ol start="8">

 <li>To make the whole process ’circular’, we move points indicated by the ‘tocopy’ variable to the first row.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Place these points at the very beiginning of PV</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">PV(:, nonzerolast) = [];</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">PV                                                      = [tocopy PV];</td>

  </tr>

  <tr>

   <td width="59">45</td>

   <td width="429">% Copy extra row elements from last row to 1st …row and delete the last row</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="429">PV(1 ,1:size(tocopy, 2)) = PV(end, 1:size(tocopy, …2));</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="429">PV                                                                     = PV(1:frames,:);</td>

  </tr>

 </tbody>

</table>

We provide on BrightSpace an example of a point-view matrix for your references.

<h2>2        3<em>D </em>Reconstruction Pipeline</h2>

In this section you are provided with the 3<em>D </em>reconstruction pipeline that you have to follow to obtain your final reconstructed 3<em>D </em>object from the provided images. The pipeline uses the following set of steps:

<ol>

 <li>Given a directory with images: first read those images, then detect interest points and extract SIFT features (Lab 3). Then use these features to compute matches between all consecutive images, as well as between the last and image. The matches are found using the 8-point algorithm (Lab 5). Write these matches in a cell array.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">[C,D, Matches]=ransacmatch(directory);</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">save(‘Matches.mat’, ‘Matches’);</td>

  </tr>

 </tbody>

</table>

<ol start="2">

 <li>Then you have to create a point-view matrix as explained in the above ”Chaining” section.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="91">1</td>

   <td width="397">PV = chainImages(Matches);</td>

  </tr>

 </tbody>

</table>

<ol start="3">

 <li>You have to loop over images by taking 3 consecutive images at a time and using the corresponding point-view matrix block to reconstruct the 3<em>D </em> From these chained matches you have to select only the column indexes that do not have non-zero entries. You also have to check that the number of correspondences in all views is larger than 8:</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">for iBegin = 1:n-(numFrames – 1)</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">iEnd = …</td>

  </tr>

  <tr>

   <td width="59">34</td>

   <td width="429">% Select frames from PC to form a block</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">block = …</td>

  </tr>

  <tr>

   <td width="59">67</td>

   <td width="429">% Select columns from the block that do not …have any zeros</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="429">colInds = find(…);</td>

  </tr>

  <tr>

   <td width="59">910</td>

   <td width="429">% Check the number of visible points in all views</td>

  </tr>

  <tr>

   <td width="59">11</td>

   <td width="429">numPoints = size(colInds, 2);</td>

  </tr>

  <tr>

   <td width="59">12</td>

   <td width="429">if numPoints <em>&lt; </em>8</td>

  </tr>

  <tr>

   <td width="59">13</td>

   <td width="429">continue</td>

  </tr>

  <tr>

   <td width="59">14</td>

   <td width="429">end</td>

  </tr>

  <tr>

   <td width="59">1516</td>

   <td width="429">…</td>

  </tr>

 </tbody>

</table>

<ol start="4">

 <li>Then you have to create a measurement matrix (as the one used in Lab4) from these correspondences points, where the measurement matrix has the size: 2#<em>frames</em>×#<em>matches</em>, and the odd rows correspond to x coordinates and even rows to y coordinates.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Create measurement matrix X with coordinates … instead of indices using the block and the …Coordinates C</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">block = block(:, colInds);</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">X = zeros(2 * numFrames, numPoints);</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="429">for f = 1:numFrames</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">for p = 1:numPoints</td>

  </tr>

  <tr>

   <td width="59">67</td>

   <td width="429">X(2 * f – 1, p) = …</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="429">                           X(2 * f, p)                       = …</td>

  </tr>

  <tr>

   <td width="59">9</td>

   <td width="429">end</td>

  </tr>

  <tr>

   <td width="59">10</td>

   <td width="429">end</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li>The we reconstruct the 3<em>D </em>point clouds for the current 3 images using the structure from motion implementation (from Lab4). Sometimes finding a reliable solution may not be possible between the image matches. In this case the Choleski decomposition will return a non-positive matrix error.</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Estimate 3D coordinates of each block following …Lab 4 “Structure from Motion” to compute the …M and S matrix.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">% Here, an additional output “p” is added to deal …with the non-positive matrix error</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">% Please check the chol function inside sfm.m for …detail.</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="429">[M, S, p] = … % Your structure from motion …implementation for the measurements X</td>

  </tr>

  <tr>

   <td width="59">56</td>

   <td width="429">if ¬p</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="429">% Compose Clouds in the form of (M,S,colInds)</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="429">Clouds(i, &#x1f642; = {M, S, colInds};</td>

  </tr>

  <tr>

   <td width="59">9</td>

   <td width="429">i = i + 1;</td>

  </tr>

  <tr>

   <td width="59">10</td>

   <td width="429">end</td>

  </tr>

 </tbody>

</table>

We only add the current points to the cloud cell array if the Choleski decomposition was successful.

<ol start="6">

 <li>We now need to stitch the obtained point clouds into a single point cloud. We use the point correspondences to find the optimal transformation between shared points in the 3<em>D </em>point clouds. The <em>procrustes </em>analysis is used for this. We start by initializing the merged cloud points with the main view, given by the first set of point clouds.</li>

</ol>

We then find the points that are both in the new cloud and in the already merged cloud. The <em>procrustes </em>analysis needs a minimum of 15 points. We then apply the <em>procrustes </em>analysis to find the transformation between the points. <em>Procrustes</em>(<em>X,Y </em>) finds the linear transformation of the set of points in Y, that best conforms them to the points in X.

<table width="488">

 <tbody>

  <tr>

   <td width="91">1</td>

   <td width="397">% Get the points that are in the merged cloud … and the new cloud by using “intersect” …over indexes</td>

  </tr>

  <tr>

   <td width="91">2</td>

   <td width="397">[sharedInds, ¬, iClouds] = intersect(…)</td>

  </tr>

  <tr>

   <td width="91">3</td>

   <td width="397">sharedPoints                                              = …</td>

  </tr>

  <tr>

   <td width="91">45</td>

   <td width="397">% Find optimal transformation between shared …points using procrustes. The inputs must … be of the size [Nx3].</td>

  </tr>

  <tr>

   <td width="91">6</td>

   <td width="397">[d, Z, T] = procrustes(…’, …’)</td>

  </tr>

 </tbody>

</table>

<ol start="7">

 <li>We then need to select the points that are not yet in the merged cloud using ”setdiff”. We transforms the points with the found transformation using the formula:</li>

</ol>

<em>Z </em>= <em>T.b </em>∗ <em>Y </em>∗ <em>T.T </em>+ <em>T.c.                                     </em>(1)

Where <em>T.c </em>is the translation component, <em>T.T </em>is the orthogonal rotation and reflection component and <em>T.c </em>is the scale component.

<strong>Note: </strong>The matlab function <em>procrustes </em>expects inputs of the form [<em>N </em>× 3], so that means we also need to transpose our points when computing their transformed version.

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% Find the points that are not shared between the …merged cloud and the Clouds{i,:} using …“setdiff” over indexes</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">[iNew, iCloudsNew] = setdiff(..)</td>

  </tr>

  <tr>

   <td width="59">34</td>

   <td width="429">…</td>

  </tr>

  <tr>

   <td width="59">56</td>

   <td width="429">% Transform the new points using: Z = (T.b * Y’ * … T.T + c)’.</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="429">% and store them in the merged cloud, and add …their indexes to merged set</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="429">mergedCloud(:, iNew) = …</td>

  </tr>

  <tr>

   <td width="59">9</td>

   <td width="429">mergedInds                                      = …</td>

  </tr>

 </tbody>

</table>

The obtained points can the be visualized with the function <em>scatter</em>3.