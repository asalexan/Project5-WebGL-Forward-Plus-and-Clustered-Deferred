WebGL Forward+ and Clustered Deferred Shading
======================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 5**

* Ashley Alexander-Lee
* Tested on: Windows 10, i9-11900H @ 2.50GHz 22GB, RTX 3070 

### Live Online

[![](img/thumb.png)](http://TODO.github.io/Project5-WebGL-Forward-Plus-and-Clustered-Deferred)

### Demo Video/GIF

![Frustum View](img/frustum_vis2.gif)]

![Header](img/header1.png)

### Description
The purpose of this project was to improve the rendering time for WebGL scenes that have many light/material combinations using Forward+ rendering and Clustered Deferred rendering. With simple forward rendering, every light/material combination must be taken into account for multiple fragments. For both Forward+ and Clustered Deferred rendering, the idea is that you can split the viewing frustum into clusters and sort the lights into said clusters. Then, during the shading stage, you only need to perform lighting calculations for the lights contained in the cluster the fragment belongs to. Clustered Deferred takes this a step farther. Instead of doing the shading calculations in the fragment shader, it saves certain fragment attributes to a gbuffer, which is ported to a post-process shader that can perform the lighting calculations. The benefit of this method is that it does not have to perform costly calculations for multiple fragments per pixel -- it only needs to perform these calculations for the final pixel color. 

### Approach

#### Cluster Creation
First, I needed to split the viewing frustum into clusters. I wrote a `Subfrustum` class that would handle these calculations and that would, given a wireframe object, display the frustums and light positions. A subfrustum contains six `SubfrustumPlane`s, which, in turn, contain four points and a three.js plane. The `SubfrustumPlane` has the ability to:

* calculate whether a given light intersects a plane
* calculate whether a given light is inside a plane (if the vector between the light position and the plane faces away from the normal)

The parent `Subfrustum` class takes in six values to create the frustum: `minX`, `minY`, `minZ`, `maxX`, `maxY`, `maxZ`. It organizes these values into points and projects those points based off of the method describe in this [blog post](https://dev.theomader.com/frustum-splits/). It also has a function to determine if a light sphere overlaps a frustum, which checks if either 1) the sphere overlaps one of the bounded planes or 2) the sphere lies inside of the frustum (i.e. it's inside all of the frustum planes). This intersection method was guided by this [stack overflow post](https://stackoverflow.com/questions/37512308/choice-of-sphere-frustum-overlap-test) enumerating the different ways a sphere could overlap a frustum. 

I also have methods in these classes for displaying the subfrustums and the surrounding lights. Below, you can see a visualization of a 15 x 15 x 15 frustum -- the blue crosses represent light positions, and the red highlights represent the frustums that are intersected by a light. 

![Frustum Visualization](img/frustum_vis1.gif)

#### Forward+
I created a new texture to contain the coordinates of each cluster (4 * 8 elements). In the fragment shader, I unpack the cluster points from the texture and create a `Cluster` which is composed of six `Plane`s, as per my earlier method. I then perform a similar method as above to ascertain whether the fragment position is inside the cluster. If so, I return the cluster ID. The cluster ID is then used to unpack the texture containing the lights per cluster. I iterate over the number of lights in that cluster and perform the lighting calculations with just those lights. 

### Performance

### Credits

TODO: frustum slice credit
TODO: frustum intersection credit
* [Three.js](https://github.com/mrdoob/three.js) by [@mrdoob](https://github.com/mrdoob) and contributors
* [stats.js](https://github.com/mrdoob/stats.js) by [@mrdoob](https://github.com/mrdoob) and contributors
* [webgl-debug](https://github.com/KhronosGroup/WebGLDeveloperTools) by Khronos Group Inc.
* [glMatrix](https://github.com/toji/gl-matrix) by [@toji](https://github.com/toji) and contributors
* [minimal-gltf-loader](https://github.com/shrekshao/minimal-gltf-loader) by [@shrekshao](https://github.com/shrekshao)

### Bloopers
