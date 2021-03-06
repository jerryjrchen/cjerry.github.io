---
layout: minimal
title: CS 184 Spring 2018 Final Report
---
# Cloud Simulation

---
<link rel="stylesheet" href="assets/css/main.css?version=1.3"/>

<div id="container">
<canvas id="maincanvas"></canvas>
<div class="controls_container">
<p>
Controls <br/>
<b>WASD:</b> fly<br/>
<b>QE:</b> roll<br/>
<b>Left-Right arrows:</b> yaw<br/>
<b>Up-Down arrows:</b> pitch<br/>
<b>Drag:</b> look
</p>
</div>

<div id="canvas-stats">
</div>
</div>

<button type="button" id="demo" class="btn btn-primary">Show/Hide Demo</button>

<script>
window.addEventListener("keydown", function(e) {
    // space and arrow keys
    if([32, 37, 38, 39, 40].indexOf(e.keyCode) > -1) {
        e.preventDefault();
    }
}, false);

var button = document.getElementById('demo');

button.onclick = function() {
    var div = document.getElementById('container');
    if (div.style.display !== 'none') {
        div.style.display = 'none';
    }
    else {
        div.style.display = 'block';
    }
};

var canvas = document.getElementById('maincanvas');
canvas.style.width ='100%';
canvas.style.height = '480px';
canvas.width  = canvas.offsetWidth;
canvas.height = canvas.offsetHeight;
</script>

<!-- vertex and fragment shader for point cloud -->
<script type="x-shader/x-vertex" id="v_shader">
// we need to retain the alpha values between vertex and frag
attribute vec4 aVertexPosition;

uniform mat4 uModelViewMatrix;
uniform mat4 uProjectionMatrix;

varying float alpha;

void main() {
  gl_Position = uProjectionMatrix * uModelViewMatrix * aVertexPosition;
}

</script>
<script type="x-shader/x-fragment" id="f_shader">

varying float alpha
varying vec3 color

void main() {
  gl_FragColor = vec4(color, alpha)
}
</script>

<script src="assets/js/three.min.js"></script>
<script src="assets/js/FlyControls.js"></script>
<script src="assets/js/Sky.js"></script>
<script src="assets/js/Lensflare.js"></script>
<script src="assets/js/Stats.js"></script>
<script src="assets/js/main.js?version=1.4"></script>

## Summary

For our project, we used OpenGL to accurately render the appearance of
clouds from close up in real time. Additionally, we allow the user to
traverse the scene to observe clouds illuminated from a variety of
different angles. This is achieved through the careful use of cloud textures placed in the scene, which dynamically generate based on the current camera position. Finally, the entire simulation can run on a web browser, sustaining well above 30 fps for a 640x480 window.


## Technical Approach
Unlike meshes that we were rendering in project 3, clouds are a semi-transparent collection of particles made up of suspended gaseous materials. This means that light can penetrate through a cloud, is scattered and absorbed depending on its particles, and moves dynamically with relation to its surrounding particles and external forces.

During the research process, we investigated two ways of dynamically rendering clouds. The one we implemented for this project was the use of imposters, which are cloud textures placed throughout the scene. This is an effective way to create clouds without needing to resort to complex computation. However, there are still challenges associated with this.

After the user passes through the initial clouds, there will be no more clouds to look at. This is not a problem for a static scene, but in our project, we allow the user to freely travel. We solve this problem by keeping track of the user position and the distance towards which we have generated clouds. As the user enters unvisited territory, we generate additional clouds.

Viewing angles for cloud imposters also pose a challenge. As the user moves, it becomes challenging to provide a consistent view of the cloud. This is less of an issue for simple rotation (i.e. rolling) which we are able to handle by looking at the rotation of the camera along the z-axis. Hoever, if the user changes their angle of view of the cloud, the cloud may appear to rotate unnaturally to follow the user's gaze. [This paper on real-time cloud rendering](https://pdfs.semanticscholar.org/a999/d556007e2782c470dd3948b91676f37b7261.pdf) describes a technique for determining a translation error metric which we could use to get a measure of error per cloud. Once the cloud exceeds a certain error threshold, it would be regenerated. Due to time limitations, we were not able to implement this in time, but we believe that the current demo behaves very well when traveling along the default starting direction.

Other aspects of the scene have been added through the three.js library.
* Lighting was done using a three.js hemisphere light. The sun was simulated as a far away mesh.
* Sky and the horizon were added [using a custom sky shader](https://threejs.org/examples/webgl_shaders_sky.html)
* Movement controls were added using [FlyControls](https://threejs.org/examples/misc_controls_fly.html), a three.js module
* The plane mesh was found [here](https://clara.io/view/4446a747-cbc4-4485-bc76-985a32e80251) and is in the public domain


[This paper from Intel](https://software.intel.com/en-us/articles/dynamic-volumetric-cloud-rendering-for-games-on-multi-core-platforms) (dynamic volumetric cloud rendering) mentions a particle method for simulating clouds that we tried in OpenGL (which does support parallelism). It uses "a multithread framework and Intel® Streaming SIMD Extensions (Intel® SSE) instructions" to achieve real time performance. On top of that, the method described also simulates dynamic evolution of clouds, which could be a feasible stretch goal.
However, as we continued using raymarching under shaders with individual point meshes to simulate cloud particles, we found that our threshold for memory and computation was too hard to handle. Most articles either involved lengthy time renders for clouds, or using a mix of mesh and shaders to model a cloud appearance when viewed from below. We intended to use shaders to simulate flying up, down, and between clouds. Other implementations, like the Intel paper referenced above, involved tampering with SIMD instructions and their own GPU programming to speed up calculations; we had Three.js.
The intended idea was to use raymarching in shaders to simulate how clouds would look. In short, we sample a cloud by building up an alpha channel and calculating lighting as we iterate through the cloud from the camera’s perspective. We also need to account for scattering of light within the cloud, from other clouds, and also along the cloud to create the “silver-lining” effect. We also need to account for light absorption with an albedo weight. This seems like a trivial matter, but to account for all this, at each march, we need to account for every particle’s interaction with each other in the cloud.
One approach involved adding a matrix to each fragment shader input that represents each point’s properties in the cloud, but since Three.js runs on the browser, we have finite memory to work with, and realistic simulations would throw a memory leak error. On top of this and having a broken display halfway through the project, the implementation that involved using particle-by-particle cloud simulation ended up being infeasible given the time constraint. As a result, we plan to implement this under next time, provided that the motivation still exists.

Final lessons we learned:
* WebGL (and by extension, OpenGL) is an incredibly powerful graphics API. However, we quickly learned that building something from scratch would be quite time consuming. The use of libraries like three.js greatly accelerated our progress.
* We were lucky that our problem could be easily scaled: i.e. number of clouds could be adjusted. This meant that we had a reasonable way of tuning performance without needing to compromise on our core solution.

## Results

Although performance depends largely on what kind of device you run the demo, we have observed a steady >30 fps performance on a reasonably modern laptop (i.e. Macbook Pro 2014). It's also possible to tweak simulation parameters, such as lowering the number of clouds or lowering the travel speed, to improve the performance.

You can try out a demo at the top of the page. Final report video:
<iframe width="560" height="315" src="https://www.youtube.com/embed/z8-rap6ZWTw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## References

* Dynamic volumetric cloud rendering: <https://software.intel.com/en-us/articles/dynamic-volumetric-cloud-rendering-for-games-on-multi-core-platforms>
* Real-Time Cloud Rendering: <https://pdfs.semanticscholar.org/a999/d556007e2782c470dd3948b91676f37b7261.pdf>
* Plane mesh: <https://clara.io/view/4446a747-cbc4-4485-bc76-985a32e80251>
* Custom sky/sun shader: <https://threejs.org/examples/webgl_shaders_sky.html>
* Flight controls: <https://threejs.org/examples/misc_controls_fly.html>
* Volumetric Cloudscapes: <http://killzone.dl.playstation.net/killzone/horizonzerodawn/presentations/Siggraph15_Schneider_Real-Time_Volumetric_Cloudscapes_of_Horizon_Zero_Dawn.pdf>

## Contributions from each team member

All group members contributed to the initial brainstorming and planning process.

Jerry created all the non-cloud portions of the scene, helped debug some of the imposter viewing angle issues, and prepared the proposal and checkpoint writeups.

Harry researched and attempted to find a way to shade point clouds to simulate cloud particles. Also, he added some cool lensflare to the sun.

Tammy successfully implemented dynamic generation of imposter clouds based on depth, attempted to implement dynamic regeneration based on translation angle, and made the milestone and final videos.
