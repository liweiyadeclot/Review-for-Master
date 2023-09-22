# Chapter 4: Ray Tracing

​	One of the basic tasks of computer graphics is rendering three-dimensional objects: taking a scene composed of many geometric objects arranged in 3D space and computing a 2D image that shows the objects as viewed from a particular viewpoint.

​	Rendering involves considering how each objects contributes to each point, and it can be organized in two general ways. 

​	In **object-order rendering**, each object is considered in turn, and for each object, all the pixels that it influences are found and updated. In **image order rendering** , each pixel is considered in turn, and for each pixel all the objects that influence it are found and the pixel value is computed.

​	==Ray Tracing== is an image-order algorithm for making 3D scenes.

## 4.1 The Basic Ray-Tracing Algorithm

​	A ray tracer works by computing one pixel at a time, and for each pixel, the basic task is to find the object that is seen at that pixel's position in the image. The particular object we want is the one that intersects the **viewing ray** nearest the camera. Once that object is found, a shading computation uses the intersection point, surface normal, and other information(depending on the desired type of rendering) to determine the color of the pixel.

​	A basic ray tracer therefore has three parts:

- ray generation, which computes the origin and direction of each pixel's viewing ray based on the camera geometry.
- ray intersection, which finds the closest object intersecting the viewing ray.
- ==shading==, which computes the pixel color based on the results of ray intersection.

​	The structure of the basic ray tracing program is:

```cpp
for(pixel : Pixels)
{
    compute viewing ray;
    find first object hit by ray and its surface normal vn;
    set pixel color to value computed from hit point, lights, and vn;
}
```

## 4.2 Perspective

​	**linear perspective**, in which 3D objects are projected onto an  **image plane** in such a way that ==straight lines in the scene become straight lines in the image==.

​	The simplest type of projection is **parallel projection**, in which 3D points are mapped t o 2D by moving them along a **projection direction** until they hit the image plane. The view that is produced is determined by the choice of projection direction and image plane. If the image plane is perpendicular to the view direction, the projection is called **orthographic**, which keeps parallel lines parallel and they preserve the size and the shape of planar objects that are parallel to the image plane.

​	The advantages of parallel projection are also its limitations. Objects look smaller as the y get farther away, and as a result, parallel lines receding into the distance do not appear parallel. ==This is because eyes and cameras don't collect light from a single viewing direction; they collect light that passes through a particular view point. We can produce natural-looking views using **perspective projection**: we simply project along lines that pass through a single point, the **viewpoint**, rather than along parallel lines.

## 4.3 Computing Viewing Rays

​	All of out ray-generation methods start from an **orthonormal coordinate frame** known as the **camera frame**, which we'll denote by **e**, for the eye point, or viewpoint, and **u**, **v**, and **w** for three basis vectors. The most common way to construct a basis the camera frame is from the viewpoint, which becomes **e**, the **view direction** which is -**w**, and the **up vector**, which is used to construct a basis that has **v** and **w** in the plane defined by the view direction and the up direction.

![image-20230829154003056](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829154003056.png)

​	For an orthographic view, all the ray will have the direction -**w**.

​	For a perspective view, all the rays have the same origin, at the viewpoint; it is the directions that are different for each pixel. The image plane is no longer positioned at **e**, but rather some distance *d* in front of **e**; this distance is the **image plane distance**, often loosely called the **focal length**. The direction of each ray is defined by the viewpoint and the position of pixel on the image plane.

![image-20230829154025067](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829154025067.png)

## 4.4 Ray-Object Intersection

### 4.4.1 Ray-Sphere Intersection

 参见P87

### 4.4.2 Ray-Triangle Intersection

​	We will present the form that uses **barycentric coordinates** for the parametric plane containing the triangle, because it requires no long-term storage other than the vertices of the triangle.

​	To intersect a ray with a parametric surface, we set up a system of equations where the Cartesian coordinates all match:

![image-20230829155414214](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829155414214.png)

​	Here, we have three equations and three unknowns. In the case where the surface is a parametric plane, the parametric equation is linear and can be written in vector form. If the vertices of the triangle are **a**, **b** and **c**, then the intersection will occur when:

![image-20230829155639694](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829155639694.png)

Solving this equation tells us both t, which locates the intersection point along the ray, and(*β*, *γ*), which locates the intersection point relative to the triangle. The intersection is inside the triangle if and only if *β* > 0, *γ* > 0 and  *β* + *γ* < 1. Otherwise, the ray has hit the plane outside the triangle, so it misses the triangle. If there are no solutions, either the triangle is degenerate or the ray is parallel to the plane containing the triangle.

## 4.5 Shading

​	 Once the visible surface for a pixel is known, the pixel value is computed by evaluating a **shading model**. 

### 4.5.1 Light Sources

​	Computing shading from a point or directional light source requires certain geometric information, and in a ray tracer, after a viewing ray has been determined to hit the surface, we have all we need to determine these four vectors:

- The shading point **x** can be computed by evaluating the viewing ray at the *t* value of the intersection.
- The surface normal **n** depends on the type of surface(sphere, triangle), and every surface needs to be able to compute its normal at the point where a ray intersect it.
- The light direction **l** is computed from the light source position or direction as part of shading.
- The viewing direction **v** is simply opposite the direction of the viewing ray.

### 4.5.3 Shadows

TO-DO

# Chapter 5: Surface Shading

​	One of the key contributors to the visual impression of three-dimensionality is shading or coloring surfaces in the scene based on their shape and their relationship to other objects in the scene.

​	Shading should be designed to provide the clearest, most accurate impression of 3D shape. The equations used to compute shading are known as a **shading model**, and a range of different shading models have been developed for these different applications. 

## 5.1 Point-like light sources

​	Point-like sources come in two flavors: a point source is small enough to be treated as a point, but is close to the scene and can illuminate different surface differently; a **directional source** is both small enough(relative to its distance) to be treated as point-like and also so far away that it illuminates all surface the same and there is no need to keep track of its location, only its direction.

### 5.1.1 Point source illumination

​	A point light source is described by its position, which is a point in 3D space, and its intensity, which describes the amount of light it produces. A point source can be *isotropic*, meaning the intensity is the same in all directions.

​	For an isotropic point source, it's easy to reason about how much light falls on a surface a certain distance away. Suppose we have a point source that emits one Watt of **radiant power** isotropically, and we place this source at the center of a hollow sphere with one meter radius. All the power from the light falls on the inside surface of the sphere, and it's distributed uniformly over the whole surface area of 4*π*m^2, so the density of radiant power per unit area is 1/(4pi) Watts per square meter. The density is known as **irradiance** and is the right quantity to describe how much light is falling on a surface for the purposes of light reflection.

​	In general case of a source that has power *P* and a receiving sphere of radius *r*, we find the irradiance *E* is:

![image-20230829225854199](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829225854199.png)

​	==The quantity *I = P/(4pi)* is the intensity of the source; it is a property of the source itself that is independent of what surface it's illuminating. The r^-2 factor, often called the inverse square term, describes how irradiance depend on the distance *r* between the source and the surface.==

​	One other important consideration in computing irradiance is the **angle of incidence**--the angle between the surface normal and the direction the light is traveling. In general, when rotated by an angle theta it intercepts an amount of light(radiant power) proportional to cos(theta), and since the area stays the same, the irradiance(which is ==radiant power per unit area==) is proportional to the same factor. This rule is known as **Lambert's cosine low**

​									![image-20230829231007748](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230829231007748.png) 

​	The term cos(theta)/r^2 can be called the **geometry factor** for a point source; it depends on the -==geometric relationship== between source and receiving surface, but not on the specific properties of either one.

## 5.2 Basic reflection models

​	Now we have the ability to compute irradiance, which describes how the object reflects that light. This depends on the material the object is made out of.

### 5.2.1 Lambertian reflection

# Chapter 14 Physics-Based Rendering

## 14.1 Photons

​	To aid our intuition, we will describe radiometry in terms of collections of large number of **photons**, and this section establishes what is meant my a photon in this context. In graphic, a photon is usually just an energy packet that behaves in a way that obeys geometric optics.

​	More precisely, for us a photon is a packet of light that has a position, direction of propagation, and a wavelength λ. A photon also has a speed *c* that depends only on the refractive index *n* of the medium through which it propagates. The frequency *f = c/λ* is also used for light. This is convenient because unlike λ and c, f dose not change when the photon refracts into a medium with a new refractive index. Another invariant measure is the amount of energy *q* carried by a photon, which is given by the following relationship:

![image-20230907202932444](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230907202932444.png)



where *h* = 6.63 * 10^-34 J is Plank's Constant.

## 14.2 Smooth Metals

​	For a smooth metal, light either reflects specularly as described in Section 4.5.4 or is refracted into the surface and then quickly absorbed. The amount of light reflected is determined by the **Fresnel equations**. 

​	Almost all graphics programs use a simple approximation for the Fresnel equations developed by Schlick. For a metal, we typically specify the reflectance at normal incidence R0(λ). The reflectance should vary according to the Fresnel equations.

![image-20230907215035896](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20230907215035896.png)

where theta is the angle between the direction of light propagation and the surface normal.