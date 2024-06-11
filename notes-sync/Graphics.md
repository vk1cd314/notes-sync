```toc
```

# Line drawing algorithm
## DDA Algorithm
- Digital Differential Analyzer. 
- the meat of it $$\boxed{\begin{align}y_{i+1}&=mx_{i+1}+B\\ &=m(x_{i}+\Delta x)+B \\ &= y_{i}+m\Delta x\end{align}}\tag{1}$$
- **if $\Delta x=1$ then $y_{i+1}=y_{i}+m$ one axis has unit change other moves by slope.**
	- We use this as the loop controller. 
- round function $$\boxed{Round(y_{i})=Floor(sign(y_i)\times0.5+y_i)}\tag{2}$$
```python
def sign(x):
	return -1 if x<0 else 1

def DDA(x0, x1, y0, y1):
# seeing what change is bigger 
	dx = x1-x0
	dy = y1-y0
# the bigger change is the loop controller/length
	length = max(abs(dx), abs(dy))
# finding increments along axis
	x_inc = dx / length
	y_inc = dy / length
# initial points
	x = x0
	y = y0

	for i in range(0, length+1):
		drawPoint(int(x+sign(x)*0.5), int(y+sign(y)*0.5))	
		x = x + x_inc
		y = y + y_inc
		
```

#### Simulation practice

### Advantage
- slope indifferent
### Disadvantage
- floating point operations needed for the non loop controller variable and slope/change.
- rounding to integer takes time.

## Midpoint Line drawing
### Find zone
![[linezones]]
```python
def get_zone(x0, y0, x1, y1):
	# Basically finding the sign
    dx= x1-x0 
    dy= y1-y0

    if dx>=0 and dy>=0:
        if dx > dy:
            return 0
        return 1
	
    elif dx>=0 and dy<0:
	    # absolute because the thing is negative.
        if dx > abs(dy):
            return 7
        return 6

    elif dx<0 and dy>=0:
        if abs(dx) > dy :
            return 3
        return 2

    else:
        if abs(dx)>abs(dy):
            return 4
        return 5
```


- integer arithmetic
- Bresenham can be extended to circles
	- but not for conics therefore midpoint concept was introduced
- highly slope dependent
- selection program for slope determination needed

### Derivation thing
-  we start with $$\boxed{\begin{align}
&y=\frac{dy}{dx}x+c\\
&dy.x-dx.y+dx.c = 0\\
&Ax+By+dx.c=0\\
&A = dy \tag{1}\\
&B = -dx \tag{2}\\
&C = dx.c \tag{3}
\end{align}}$$
- we generalize the line equation to $$\boxed{\begin{align}
F(x,y)&=dy.x-dx.y+dx.c\\
&=Ax+By+C
\end{align}\tag{4}}$$
- upto this the thing is general/standard etc. 
- in general next steps are
	1. we compute $F(M)$ where M is the mid point
	2. we define decision variable $d=F(M)$
	3. for each direction that goes next we find $d_{new}=F(M_{dir})$
	4. we subtract $d_{old}$ or $d$ from $d_{new}$ to get an expression for $d_{new}$ in that direction.
	5. we find INITIAL D for starting the loop. 
	6. we find for what sign of d what direction is preferred
		1. One of the length is 1 
		2. the other we check for cases less than 0.5 and greater than 0.5 
		3. the sign of less than indicates axial movement
		4. sign of greater than indicates diagonal movement.
![[midpoint_line_things]]
### derivation for zone 2
1.  $$\begin{align}
&\boxed{d_{m}=F(M)=A(x-\frac{1}{2})+B(y+1)+C}\\
d_{m1}&=A(x-\frac{1}{2})+B(y+2)+C\\
d_{m2}&=A(x-\frac{3}{2})+B(y+2)+C\\
\end{align}$$
2. we have two directions North and North West the increments in these directions are found by$$\boxed{\begin{align}
\Delta N = d_{m1}-d_{m}=B=-dx\tag{5}\\
\Delta NW = d_{m2}-d_{m}=B-A=-dx-dy\tag{6}\\
\end{align}}$$
3. INITIAL D $$\begin{align}
F(x_{0}-\frac{1}{2}, y_{0}+1)&=A(x_{0}-\frac{1}{2})+B(y_{0}+1)+C\\
&=Ax_0+By_0+C-\frac{A}{2}+B\\
&=F(x_0,y_0)\boxed{-\frac{A}{2}+B}\tag{7}
\end{align}$$

4. Loop controller is y because it is the one that remains unchanged here
5. algo 
```python
def draw_line_2(x0, y0, x1, y1):
    dx = x1 - x0
    dy = y1 - y0
    x = x0
    y = y0
    d = -2 * dx - dy
    del_n = -2 * dx
    del_nw = 2 * (- dy - dx)
    draw_pixel(x, y)
    while (y< y1):
        draw_pixel(x, y)
        if (d < 0):
            d += del_n
            y += 1
        else:
            d += del_nw
            x -= 1
            y += 1
```



# Circle drawing


# Elipse Drawing

# Line Clipping
we clip lines against the viewport. 
## Cohen Sutherland

![[Cohen_sutherland]]
```python
TOP = 8
BOTTOM = 4
RIGHT = 2
LEFT = 1
def get_code(x, y, xmin, xmax, ymin, ymax):
	code = 0
	if x < xmin:
		code |= LEFT # Left bit set
	if x > xmax:
		code |= RIGHT # Right bit set
	if y < ymin:
		code |= BOTTOM # Bottom bit set
	if y > ymax:
		code |= TOP # Top bit set
	return code


def cohen_sutherland_clip(x0, y0, x1, y1, xmin, xmax, ymin, ymax):
	code0 = get_code(x0, y0, xmin, xmax, ymin, ymax)
	code1 = get_code(x1, y1, xmin, xmax, ymin, ymax)
	while True:
		# Trivial acceptance:
		if code0 == 0 and code1 == 0:
			return x0, y0, x1, y1
		

		# Trivial rejection
		if (code0 & code1) != 0:
			return None
		
		  
		
		# Choose endpoint outside window and clip
		
		if code0 != 0:
			code_out = code0
			x = x0
			y = y0
		
		else:
			code_out = code1
			x = x1
			y = y1
		
		  
		
		# Clip against left, right, bottom, or top boundary
		
		if code_out & LEFT: # Left
			x = xmin
			y = y0 + (y1 - y0) * (xmin - x0) / (x1 - x0 if x1 != x0 else 1)
		
		elif code_out & RIGHT: # Right
			x = xmax
			y = y0 + (y1 - y0) * (xmax - x0) / (x1 - x0 if x1 != x0 else 1)
		
		elif code_out & 4: # Bottom
			y = ymin
			x = x0 + (x1 - x0) * (ymin - y0) / (y1 - y0 if y1 != y0 else 1)
		
		elif code_out & 8: # Top
			y = ymax
			x = x0 + (x1 - x0) * (ymax - y0) / (y1 - y0 if y1 != y0 else 1)
		
		  
		
		# Update the clipped endpoint
		
		if code_out == code0:
			x0 = x
			y0 = y
			code0 = get_code(x0, y0, xmin, xmax, ymin, ymax)
		else:
			x1 = x
			y1 = y
			code1 = get_code(x1, y1, xmin, xmax, ymin, ymax)
```
1. Endpoints are checked for trivial acceptance.
2. region checks are done for trivial rejection
3. if not the either we find one outlying point and readjust it as such
	$$\begin{align}
	&\text{x if its a left/right boundary clip y otherwise} \\
	x &= x_{min/max}\ \text{(min/max, based on clip against lower or higher boundary)}\\
	y &= y_{0} + \frac{y_{1} - y_{0}}{x_{1}-x_{0}}\times (x_{min/max}-x_0)\ \text{(same as above)}
\end{align}$$
4. Repeat until entire segement is accepted

## Cyrus Beak
1. we need many loops to get cohen sutherland to get the real segement 
![[cohen_sutherland_ineff]]]
here we need to pass through D->E->F and I->H->G to get to our clipped line
## derivation 
![[cyrus_derive]]
Let,
1. Line on the boundary: $P_t-P_E$
2. normal of the boundary $N$
we know,
$$(P_t-P_E).N=0\tag{1}\text{   (perpendicular)}$$
Parametric equation of a line $$P(t)=P_{0}+t(P_{1}-P_{0})\tag{2}$$
Putting the value of $(2)$ in $(1)$ $$\begin{align}
(P_{0}+t(P_{1}-P_{0})-P_{E}).N&=0\\
t(P_1-P_0).N&=-(P_0-P_E).N\\
\Aboxed{t&=\frac{-(P_0-P_E).N}{(P_1-P_0).N}}
\end{align}$$


# Polygon Clipping
## Sutherland Hodgman 

# Polygon Filling

# Transformations
## Translation

## Rotation

## Scaling

## Shearing

## Reflection

# Transformation Matrix

# Projection
- scale depends on object
- view volume -> left, right, top, bottom, near, far limit of a window display
- ## Two types based on 'd'
	- ### Parallel Projection: 
		- infinite distance of COP and PP
	- ### Perspective:
		- d is finite
- ## Two types based on orientation of PP:
	- ### Orthographic:
		- PP is perpendicular to z-axis
	- ### Oblique:
		- PP not perpendicular. 
# Projection Matrix

![[Simple_Projection_COP_on_origin]]
1. From the figure we get $$\begin{align}
\frac{AB}{BC}&=\frac{AD}{DE}\\
\frac{d}{x'}&=\frac{-z}{x}\\
&\boxed{x'=\frac{x}{-z/d}}\\
similarly,&\\
&\boxed{y'=\frac{y}{-z/d}}
\end{align}$$
2. Putting this into a projection matrix we get $$\begin{vmatrix}x'\\y'\\z'\\1\end{vmatrix}=\begin{vmatrix}1&0&0&0\\0&1&0&0\\0&0&1&0\\0&0&-1/d&0\end{vmatrix}.\begin{vmatrix}x\\y\\z\\1\end{vmatrix}$$
![[Origin_on_projection_plane]]
1. From the figure $$\begin{align*}
\frac{AB}{BC}&=\frac{AD}{DE}\\
\frac{d}{x'}&=\frac{-z+d}{x}\\
\Aboxed{x'&=\frac{x}{1-z/d}}\\
similarly,&\\
\Aboxed{y'&=\frac{y}{1-z/d}}\\
and,&\\
\Aboxed{z'&=0}
\end{align*}$$
2. Projection matrix $$\begin{vmatrix}x'\\y'\\z'\\1\end{vmatrix}=\begin{vmatrix}1&0&0&0\\0&1&0&0\\0&0&0&0\\0&0&-1/d&1\end{vmatrix}.\begin{vmatrix}x\\y\\z\\1\end{vmatrix}$$
![[General_purpose_Matrix]]
1.  consider any parametric point $P(t)$ on the line $P'P$. We can write it as.
   $$P(t)=COP+t(P-COP)\tag{1}$$
   2. **Consider the fact that $COP$ is $\bar{Q}$ away from $<0,0, z_p>$.** we can write:$$COP=\bar{Q}+<0,0,z_{p}>\tag{2}$$
   3. Combining $1$ and $2$ we get: $$P(t)=\begin{vmatrix}\bar{Q}x\\\bar{Q}y\\\bar{Q}z+z_{p}\end{vmatrix}+t\begin{vmatrix}
x-\bar{Q}x\\y-\bar{Q}y\\z-\bar{Q}z-z_p
\end{vmatrix}$$
4. Consider the case at PP $z'=z_{p}$ which gives us:$$\begin{align}
z_p&=z_p+\bar{Q}z+t(z-\bar{Q}z-z_p)\\t&=\frac{-\bar{Q}z}{z-\bar{Q}z-z_{p}}\\
\Aboxed{t&=\frac{1}{-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z}}}
\end{align}$$
5. using t we find all values of $P'$ $$\begin{align}
x'&=\bar{Q}x+t(x-\bar{Q}x)\\ \Aboxed{x' &= \frac{x-z\frac{\bar{Q}x}{\bar{Q}z}+z_{p}\frac{\bar{Q}x}{\bar{Q}z}}{-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z}}}\\
similarly, \\
\Aboxed{y' &= \frac{y-z\frac{\bar{Q}y}{\bar{Q}z}+z_{p}\frac{\bar{Q}y}{\bar{Q}z}}{-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z}}}

\end{align}$$
6. We know $z'=z_{p}$ but to maintain consistency $$\begin{align}
z'&=\frac{z_{p}(-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z})}{-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z}}\\
\Aboxed{z'&=\frac{-z\frac{z_p}{\bar{Q}z}+z_{p}(1+\frac{z_{p}}{-\bar{Q}z})}{-\frac{z}{\bar{Q}z}+1+\frac{z_{p}}{-\bar{Q}z}}}
\end{align}$$
7. Which Finally gives us $$\begin{vmatrix}x'\\y'\\z'\\1\end{vmatrix}=\begin{vmatrix}1&0&-\frac{\bar{Q}x}{\bar{Q}z}&z_{p}\frac{\bar{Q}x}{\bar{Q}z}\\0&1&-\frac{\bar{Q}y}{\bar{Q}z}&z_{p}\frac{\bar{Q}y}{\bar{Q}z}\\0&0&-\frac{z_{p}}{\bar{Q}z}&z_{p}(1+\frac{z_{p}}{-\bar{Q}z})\\0&0&-1/\bar{Q}z&1+\frac{z_{p}}{-\bar{Q}z}\end{vmatrix}.\begin{vmatrix}x\\y\\z\\1\end{vmatrix}$$
8. ![[general_purpose_to_other_matrices]]


# Visible surface determination
- ## Object space techniques:
	- applied before vertices are mapped to pixels
	- back face culling
	- Painters algo
	- BSP trees
- ## Image space techniques
	- applied while the vertices are rasterized
	- z buffering
	- visible surface ray tracing
# Back Face Culling
- Vertices of polygon are oriented in an anti clockwise manner
- when viewed from outside the surface vector $N$ points out
	- Test $N.V$ or simply z component of surface normal
	- if negative cull
- Only in convex surfaces
# painters algo 
- Supports transparency
- draw surfaces back to front
- key issue is order determination
- Does not always work 
	  MC escher cases.
# z-buffering algorithm
- two buffer 
	- Frame and z buffer
- initialize frame buffer to background color
- initialize z buffer to max z
- for each triangle 
	- calculate z for each polygon pixels
	- if z is less than z value in z buffer for the same pixel co-ordinate 
		- update frame and depth buffer
	- else
		- continue
#### Advantages:
1. simple 
2. diversity of primitives. not just polygons
3. complexity
4. no need to calculate intersections
#### disadvantage:
1. extra memory and bandwidth
2. wastes time drawing hidden objects z precision error
3. may have to use point sampling
#### Complexity
1. Time $O(N)$ 
2. Space $O(1)$

# Visible Surface Ray tracing
### Algorithm:
- select a view point and a screen plane
- for every pixel in the screen plane
	- find the ray from eye through the pixels center
	- for each object in the scene
		- if ray hits object
			- if intersection nearest to eyes
				- record intersection point
				- record color of the object at the point
				- set screen plane pixel to the nearest recorded color. 
# Example of intersection with a sphere.
1. Ray is defined as $$\begin{align}
\text{From, }COP=R_{0}=[X_{0}\ Y_{0}\ Z_{0}]\\
\text{To, }Pixel=R_{d}=[X_{d}\ Y_{d}\ Z_{d}]
\end{align}$$
2. a point on the ray $$R_{(t)}=R_{0}+t\times(R_{d}-R_{0})$$ where $t>0$
3. for sphere: $$\begin{align}
\text{Sphere's Center}&= S_{C} = [X_{C}\ Y_{C}\ Z_{C}]\\
\text{Sphere's Surface Points}& = [X_{S}\ Y_{S}\ Z_{S}]\\
\text{Sphere's Radius}&= S_{R}
\end{align}$$
4. Sphere's surface is then an implicit equation $$(X_S-X_C)^2+(Y_S-Y_C)^2+(Z_S-Z_C)^2-S_R^2=0$$
5. we put the sphere points with the ray points $$(X_0+(X_d-X_{0})\times t -X_C)^2+(Y_0+(Y_d-Y_{0})\times t-Y_C)^2+(Z_0+(Z_d-Z_{0})\times t-Z_C)^2-S_R^2=0$$
6. simplify to $$At^2+Bt+C=0$$
7. solve for $t$
8. if $B^2<4AC$:
	1. Ray is not intersecting object
	2. Fill pixel with background color
9. else if $B^2==4AC$:
	1. ray is intersecting boundary
10. Else:
	1. pick t = lower root
	2. lowest t among all intersection is visible.
# Color 
- ### Categories of Color
	- #### Additive color model
		- RGB/RGBA
		- based on transmitting light
	- #### Subtractive Color
		- CMY/CMYK
		- based on reflected light.
- ## Achromatic Color 
	- Intensity/Luminance
	- Gray levels 0 to 1
		- we can distinguish 128 levels of gray
	- Monochrome color: two levels
# Subtractive color and pigments

-  absorb selective wavelengths
	- Cyan: 
		- absorbs red
		- reflects blue and green
	- Yellow:
		- absorbs blue
		- reflects red and green
	- Magenta:
		- absorbs green 
		- reflects blue and red
	- ![[Colors]]
- # HSV/HSB Model
- Hue Saturation and Value.
- cone 
- #### Hue:
	- color portion of the model
	- Red withing 0-60 degrees
	- yellow 61-120
	- green 121-80
	- Cyan 181-240
	- Blue 241-300
	- magenta 301-360
- ### Saturation:
	- amount of gray in a particular color 0 to 100 pc
	- 0 is gray 1 is a primary color
- ### Value (Brightness):
	- intensity of color 
	- 0 is black  100 is brightest and reveals most color
# RGB to HSV conversion
$$\begin{align}
\Delta &= max(R,G,B) - min(R,G,B)\\
\Aboxed{V&=max(R,G,B)}\\
\text{if, } V==0,
\Aboxed{S&=0}\\
else,\ \Aboxed{S&=\frac{\Delta}{V}}\\
if,\ S==0, \Aboxed{H&=undefined}\\
\text{if, max(R,G,B)==R, }\Aboxed{H&=\frac{G-B}{\Delta}\times60}\\
\text{if, H<0 } H+=360\\
\text{if, max(R,G,B)==G, }\Aboxed{H&=\frac{B-R}{\Delta}\times60 + 120}\\
\text{if, max(R,G,B)==B, }\Aboxed{H&=\frac{R-G}{\Delta}\times60+240}\\
\end{align}$$

# HSV to RGB
trick here is Cx and swaps then moves last one begins with a swap
$$
\boxed{\begin{align}
C&=V\times S\\
H'&=\frac{H}{60}\\
X&=C(1-|H'mod\ 2-1|)\\
(R_1,G_1,B_1)&=(0,0,0) \text{ if H' is undefined}\\
&=(C,X,0) \text{ if } 0 \leq H' < 1\\
&=(X,C,0) \text{ if } 1 \leq H' < 2\\
&=(0,C,X) \text{ if } 2 \leq H' < 3\\
&=(0,X,C) \text{ if } 3 \leq H' < 4\\
&=(X,0,C) \text{ if } 4 \leq H' < 5\\
&=(C,0,X) \text{ if } 5 \leq H' < 6\\
m &= V-C\\
\Aboxed{(RGB)&=((R_1+m,G_1+m,B_1+m))}
\end{align}}
$$
# HSL
- closer to how humans perceive color
- Hue, Saturation, Luminas
- L= 0 means black L=1 means white
# RGB to HSL conversion
$$\begin{align}
\Delta &= max(R,G,B) - min(R,G,B)\\
\Aboxed{L&=\frac{max(R,G,B)+min(R,G,B)}{2}}\\
\text{if, } L==0\ or,\ V==1,\ 
\Aboxed{S&=0}\\
else,\ \Aboxed{S&=\frac{\Delta}{1-|2L-1|}}\\
if,\ S==0, \Aboxed{H&=undefined}\\
\text{if, max(R,G,B)==R, }\Aboxed{H&=\frac{G-B}{\Delta}\times60}\\
\text{if, H<0 } H+=360\\
\text{if, max(R,G,B)==G, }\Aboxed{H&=\frac{B-R}{\Delta}\times60 + 120}\\
\text{if, max(R,G,B)==B, }\Aboxed{H&=\frac{R-G}{\Delta}\times60+240}\\
\end{align}$$

# HSL to RGB

$$
\boxed{\begin{align}
C&=(1-|2L-1|)\times S\\
H'&=\frac{H}{60}\\
X&=C(1-|H'mod\ 2-1|)\\
(R_1,G_1,B_1)&=(0,0,0) \text{ if H' is undefined}\\
&=(C,X,0) \text{ if } 0 \leq H' < 1\\
&=(X,C,0) \text{ if } 1 \leq H' < 2\\
&=(0,C,X) \text{ if } 2 \leq H' < 3\\
&=(0,X,C) \text{ if } 3 \leq H' < 4\\
&=(X,0,C) \text{ if } 4 \leq H' < 5\\
&=(C,0,X) \text{ if } 5 \leq H' < 6\\
m &= L-\frac{1}{2}C\\
\Aboxed{(RGB)&=((R_1+m,G_1+m,B_1+m))}
\end{align}}
$$
# Illumination and Shading
1. Lighting is the exposing light on the object 
2. shading is the process of determining the color of a pixel on an object 

## Illumination
process of simulating light and it's interactions. 
	1. Ambient
	2. diffuse
	3. specular
	4. global 
## Shading
determining color of a pixel on a surface based on it's interaction with light. 
#### Illumination Model
 the rule of calculating intensity at a point on the surface of an object for light sources from specific positions and directions.  
#### Surface Rendering method
 The rule of calculating intensity of each pixel based on illumination model.

## Global Illumination model
1. accounts for diffuse reflections 
2. handles direct and indirect lighting effects. 
3. more physically accurate
4. slower 
5. raytracing radiosity photon mapping
6. recursive ray tracing
## Light properties 
1. Color: 
2. Shape
3. direction
### Other properties
1. material
2. geometry
3. absorption 
## Local Light model:
light is divided into two types Ambient and directional.
$$\text{light falling at a point on the surface}=ambient + source$$
$$\text{light reflecting from a point = Ambient reflection + source reflection}$$
$$I_{light}= I_{ambient}+(I_{diffuse}+I_{specular})$$

3 models discussed
1. Lambertian
	1. Purely diffuse surfaces
2. Phong 
	1. adds perceptually bases specular term with lambertian
3. Blinn phong
	1. simplifies specular term

### Light source properties.
1. assume light has one wavelength 
2. Light sources as a 3D point in space
## Positional light:
- specified position of the source at x,y
- If P is a point on the surface and $L_P$ is the position of the light source then $$light\ direction\ vector\ L= (L_p-P)$$
## Directional light
- light source significantly far away can be modeled only a direction vector. 
# Ambient light
- represents the contribution of the light to the general scene. regardless of location of light and object.
- assumed to be falling equally on all direction
- simplified reflected light or ambient light:
	$$\boxed{I_{ambient}=k_{a}\times I_{a}}\tag{1}$$
	$k_{a}=[0..1]$ = surface diffuse reflectivity due to ambient light
# Lambert Reflection Model
- Diffuse surfaces follow Lamberts Cosine Law

> [!NOTE]
> - reflected energy from a small surface area in a particular direction is proportional to cosine of the angle between that direction and surface normal

# Diffuse Reflection
$$\boxed{I_{diffuse}=k_{d}\times I_{s}\times cos\theta = k_{d}\times I_{s}\times (N.L)}$$
where L is the light direction vector.
![[diffuse reflection model]]

# combining Diffuse and Ambient
$$\boxed{I=k_{a}\times I_{a}+k_{d}\times I_{s}\times cos\theta = k_{a}\times I_{a}+ k_{d}\times I_{s}\times (N.L)}$$

# Specular Reflection
- Shiny Highlight
- Follow Snells laws
![[snells_law]]
$$I_{specular}= k_{specular}\times I_{source}\times (cos\phi)^{shininess}=k_{specular}\times I_{source}\times (V.R)^{shininess}$$
# Computing R 
![[Pasted_image_20240611200148.png]]

# Phong Illumination Model
$$\boxed{I= k_{ambient}\times I_{ambient}+ k_{diffuse}\times I_{source}\times (N.L)+k_{specular}\times I_{source}\times (R.V)^{shininess}}$$
k-> surface material reflectivity 
shininess -> specular reflection parameter.
## Blinn Phong Model
- $R.V$ replaced with $N.H$
- $$H=\frac{L+V}{|L+V|}$$
- cheaper to compute 
- more physically correct
# Attenuation
- Energy fall off is proportional to $1/d^2$
- Inverse squared law
- $$f(d)= min(1, \frac{1}{a_0+a_1d+a_2d^2})$$
- a_0 -> constant attenuation a_1 -> linear attenuation a_2 quadratic attenuation.
# FINAL ILLUMINATION MODEL
$$\boxed{I= k_{ambient}\times I_{ambient}+ f(d)\times[k_{diffuse}\times I_{source}\times (N.L)+k_{specular}\times I_{source}\times (R.V)^{shininess}]}$$


# Shading
two types
1. constant
2. smooth 
	1. gouraud shading
	2. phong shading
# constant shading:
- constant intensity or flat shading uses **face normal**
- one color for entire triangle
	- uniform intensity
- fast
- good for some objects
- sudden intensity changes at borders.

# Smooth shading.
- Uses **vertex normal**
- Variable color within a triangle/variable intensity within a polygon
- small number of polygons needed
- gradual intensity changes at borders
![[Screenshot_20240611_201826.png]]
## Gouraud Shading
- Interpolation based shading
- intensity/color calculation using vertex normals
- calculate light at vertices then interpolate the colors as a scan line
- relatively fast only does 3 calculations.
- no sudden intensity changes 
- ![[Pasted_image_20240611202242.png]]
### issue
- may lose highlight
- can't interpolate light areas between two dark vertex normal.
# Phong Shading:
- Interpolate the Normal. 
- True per pixel lighting
- not done by most libraries.


# Curves and surfaces