# Weighted Voronoi Stippling


Weighted Voronoi Stippling (Secord, 2002) is a technique used to find a collection of points ("generators") from an image, in a similar fashion to what the art movement of Pointillism did. This is particularly useful in the context of TSP-Art (https://www2.oberlin.edu/math/faculty/bosch/tspart-page.html).

![Explanation](/explanation.png "Secord, 2002")
Image from Secord's algorithm*

This jupyter-notebook implements Secords's algorithm for Weighted Voronoi Stippling.

### Voronoi Diagrams

Let's consider a region of space $\Omega$ and a density function $\rho: \Omega \to \mathbb{R}^+$. A *Voronoi Diagram* can be pictured as a "tiling" of $\Omega$: we have $N$ points (*generators*), which define $N$ regions in $\Omega$; we say that a point $x\in\Omega$ belongs to region $i$ if the i-th generator is the closest one to $x$.

#### Centroidal Voronoi Diagram
We say that a Voronoi Diagram is *centroidal* if, for each generator $i$, $i$ is also the center of mass of its region. Formally, if $A$ is the region of generator $i$, the centroid $C_i$ is:

$C_i=\int_A \rho(x)x dx$

### An algorithm for generating centroidal diagrams

We can follow this procedure to start from a set of generators and get to a Centroidal Voronoi Diagram.

1. Start from $N$ generators 
2. **While** the generators do not coincide with the centroids:
    - Compute Voronoi Diagrams
    - Compute centroids $C_i$
    - Move each generator to its centroid
    
The initial generators are technically not important, but we can help the algorithm by starting from a reasonable configuration, e.g. from a Dithering algorithm.

#### Computation of the edges and centroid
Practically speaking, we want to compute the centroids starting from the picture of a painting. In order to do that, we can do the following:
1. Start from the RGB representation of an image and move to a grayscale, which therefore acts as a function $f$ which maps every pixel to the interval $[0,1]$, 0 being black and 1 being white. 
2. Intuitively, we want a higher concentration of points where the image is darker; therefore we can define the density function as $\rho=1-f$: the darker the image, the higher the density
3. We can use the *scipy.spatial.Voronoi* library to compute the edges of the Voronoi cells

Now for the centroid.
If the cell extends in the vertical range $[y_1,y_2]$, we can separate the integral as $\int_{y_1}^{y_2}\int_{x(y_1)}^{x(y_2)}\rho(x,y)dxdy=\int_{y_1}^{y_2}P(x,y)|_{x_1(y)}^{x_2(y)}dy$, with $P(x,y)=\int_0^x \rho(s,y)ds$. This allows for a more confortable computation of the area of a voronoi cell, since we can simply calculate $P(x)$ once for every pixel.

The calculation of the $y$ coordinate of a centroid can be tackled similarly. We have
$\int_{y_1}^{y_2}y\int_{x_1(y)}^{x_2(y)}\rho(x,y)dxdy = \int_{y_1}^{y_2}y P(x,y)|_{x_1(y)}^{x_2(y)}dy$

Finally, the calculation of the $x$ coordinate for the centroid can be performed via integration by parts. We have $\int_{y_1}^{y_2}\int_{x_1(y)}^{x_2(y)}x\rho(x,y)dxdy=\int_{y_1}^{y_2}dy [xP(x,y)|^{x_2(y)}{x_1(y)}-\int{x_1(y)}^{x_2(y)}P(x,y)dx]:=\int_{y_1}^{y_2}[xP-Q]^{x_2(y)}{x_1(y)}dx$ with $Q=\int_{0}^{x}P(s,y)ds$

In other words, we have to compute the values of $P$ and $Q$ across all the pixels and then carry on the sum.


### Bibliography

https://www.cs.ubc.ca/labs/imager/tr/2002/secord2002b/secord.2002b.pdf
