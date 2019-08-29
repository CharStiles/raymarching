# raymarching
![raymarching1](images/figure1.png)
![raymarching1](images/figure2.png)
![raymarching1](images/figure3.png)

This is the code: 

```GLSL

#ifdef GL_ES
precision highp float;
#endif

// These are defined by Kodelife, or whatever environment you are using.
uniform float time;
uniform vec2 resolution;

varying vec3 v_normal;
varying vec2 v_texcoord;

// Define some constants
const int steps = 128; // This is the maximum amount a ray can march.
const float smallNumber = 0.001;
const float maxDist = 10.; // This is the maximum distance a ray can travel.
 
float scene(vec3 position){
    // So this is different from the sphere equation above in that I am
    // splitting the position into its three different positions
    // and adding a 10th of a cos wave to the x position so it oscillates left 
    // to right and a (positive) sin wave to the z position
    // so it will go back and forth.
    float sphere = length(
        vec3(
            position.x + cos(time)/10., 
            position.y, 
            position.z+ sin(time) +1.)
        )-0.5;
    
    // This is different from the ground equation because the UV is only 
    // between -1 and 1 we want more than 1/2pi of a wave per length of the 
    // screen so we multiply the position by a factor of 10 inside the trig 
    // functions. Since sin and cos oscillate between -1 and 1, that would be 
    // the entire height of the screen so we divide by a factor of 10.
    float ground = position.y + sin(position.x * 10.) / 10. 
                              + cos(position.z * 10.) / 10. + 1.;
    
    // We want to return whichever one is closest to the ray, so we return the 
    // minimum distance.
    return min(sphere,ground);
}
vec4 trace (vec3 origin, vec3 direction){
    
    float dist = 0.;
    float totalDistance = 0.;
    vec3 positionOnRay = origin;
    
    for(int i = 0 ; i < steps; i++){
        
        dist = scene(positionOnRay);
        
        // Advance along the ray trajectory the amount that we know the ray
        // can travel without going through an object.
        positionOnRay += dist * direction;
        
        // Total distance is keeping track of how much the ray has traveled
        // thus far.
        totalDistance += dist;
        
        // If we hit an object or are close enough to an object,
        if (dist < smallNumber){
            // return the distance the ray had to travel normalized so be white
            // at the front and black in the back.
            return 1. - (vec4(totalDistance) / maxDist);
 
        }
        
        if (totalDistance > maxDist){
 
            return vec4(0.); // Background color.
        }
    }
    
    return vec4(0.);// Background color.
}
 
// main is a reserved function that is going to be called first
void main(void)
{
    // We are redefining the UV coordinates (aka texcoords) to be 0,0 in the 
    // middle of the screen this is because its easier to work with the camera at
    // (0,0) instead of (0.5,0.5) for the SDFs
    vec2 uv = -1. + 2. * v_texcoord;
    // Unfortunately our screens are not square so we must account for that.
    uv.x *= (resolution.x / resolution.y);
    
    vec3 rayOrigin = vec3(uv, 0.);
    vec3 camOrigin = vec3(0., 0., -1.);
    vec3 direction = camOrigin + rayOrigin;

    // This reserved variable is what we must set the final color to
    gl_FragColor = trace(rayOrigin, direction);
}
```

Resources going forward:

* [Here is an SDF library](http://mercury.sexy/hg_sdf/) from  a demoscene group.


* Again linking the useful function sheet I put together: [Glsl sticker sheet](https://gist.github.com/CharStiles/e6fec016967c6c8fd648aa4b6c0055cc)
* [This ray marching tutorial](https://github.com/ajweeks/RaymarchingWorkshop) he does it a little differently than we did, so it would be good to go over that, but if you want to skip to stuff like simple shading and material go to the half way point.
* If a prompt sounds tempting I propose this: take this smooth min function and combine two different shapes together, and add some movement to the piece! This little exercise will be very rewarding.


```
// polynomial smooth min (k = 0.1);
float sminCubic(float a, float b, float k)
{
 
    float h = max(k-abs(a-b), 0.0);
    return min(a, b) - h*h*h/(6.0*k*k);
}
```

Let me know if you make anything! Even if it's a little thing, let me know we can document it and make a recurser online gallery :D 