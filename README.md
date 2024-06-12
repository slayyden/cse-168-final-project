# CSE 168 Final Project: ReSTIR Direct Lighting
![cover](https://github.com/slayyden/cse-168-final-project/assets/26509702/ad28f383-a3f6-4b86-975c-62e9442e1f26)
This project implements ReSTIR direct lighting. 
It supports the following features
+ Streaming RIS (unbiased)
+ Spatial reuse (biased)
  - kernel is a square of arbitrary size
  - number of spatial neighbors chosen is customizable
  - any number of reuse passes can be done
+ MIS with BSDF and NEE samples (unbiased)
+ Aribtrary number of samples per pixel 

Implementation Details 
+ The target function does not include a visibility term, but a visibility pass is done before reservoirs are finalized
+ To convert the vector valued BRDF function to a scalar to be used in a target function, the channel values are averaged
+ Spatial reuse is done with uniform MIS weights, which does lead to bias

# streaming RIS 
Streaming RIS a way to better sample lights. 
A number of candidate light samples are chosen (a light sample is not just a light, but a point on the surface of a light). 
We put those samples in a reservoir and weigh them based on a target function. 
The reservoir will give us a "good sample" out of the ones we put in. 
The samples reservoirs spit out are approximately selected as if their PDF were proportional to the target function.
So, if we choose the target function to be something close to the outgoing light, such as unshadowed light contribution, 
we get samples distributed similarly to unshadowed light contribution even though there is no way to directly 
generate samples from a pdf that is proportional to unshadowed light contribution.

Preliminary result: 

<img width="249" alt="image" src="https://github.com/slayyden/cse-168-final-project/assets/26509702/8920ef1b-d148-4e02-b986-7cc2f2369f8f">

# Spatial Reuse 
If we store a reservoir at each pixel, we can choose to combine each pixel's sample with its neighboring pixels' samples in a reservoir. 
When done carefully, this leads to a massive amplification of the number of samples that can contribute to a pixel. 
The algorithm is deceptively simple, but a lot of "computational plumbing" had to be done to implement it. 
If you store reservoirs at pixels, spatial reuse can only work for primary rays, which means you need to treat primary rays as an edge case when evaluating a path. 
Additionally, just storing a reservoir as described in the literature is not enough. 
You must also store extra information to reconstruct a path after sample reuse is done. 
I ended up storing every primary ray as well and using its direction and intersectin position to generate the rest of the path. 

# Experiment 1: Cornell Box (1 spp)
### 1 candidate sample per pixel: 

![cornellBRDF_1candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/e86dcd5c-77c4-47b2-a061-57d45082614d)

### 32 candidates per pixel: 

![cornellBRDF_32candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/eed033c8-2e0f-4322-95f7-627eb8a065a0)\

Increasing the number of candidate samples makes the border of the shadow softer. 

### 32 candidates sample per pixel + spatial reuse (1 pass, kernel_radius = 15, num_neighbors = 1): 

![cornellBRDF_spatialreuse](https://github.com/slayyden/cse-168-final-project/assets/26509702/292bbdc2-3df4-45df-b602-02a747fc49d0)

Spatial reuse does not do much here. 
# Multiple Importance Sampling (1 spp): 
### 1 candidate per pixel: 
![mis_1candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/da1f2856-8d63-4eab-be1c-759051b1f541)

This is unbiased.

### 32 candidates per pixel:
![mis_32candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/732843f8-561c-4ef6-b02c-e2c9e4515911)

### Spatial reuse introduces bias here (1 pass, kernel_radius = 15, num_neighbors = 1): 
![mis_spatialreuse](https://github.com/slayyden/cse-168-final-project/assets/26509702/18ffb32e-ced3-4686-8f45-1d4fb9f6098a)


# Experiment 3: A room (21 area lights)
### 1 candidate per pixel: 
![room_1candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/a155c011-af9a-4d72-8bfd-28c07f8da884)

### 32 candidates per pixel: 
![room_32candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/b415bf6c-b7d4-4350-9eec-a6cece6342a7)
The details on the monkey become actually readable. 

### With Spatial reuse:
![room_spatialreuse](https://github.com/slayyden/cse-168-final-project/assets/26509702/9464a740-4d3e-470f-9f62-19159faa3969)
Some patches on the monkey clear up.

# Experiment 4: A sculpture with many lights 
I sculpted this character in Blender. Here's what it looks like in Blender.
### Blender Reference
![maven_blender_reference](https://github.com/slayyden/cse-168-final-project/assets/26509702/7293a4ef-0b23-42be-821e-9cc22f706cee)
- Default principled BSDF
- Standard view transform
### Blender struggles to render the scene at 1 sample
![maven_blender_1spp](https://github.com/slayyden/cse-168-final-project/assets/26509702/3e502782-e60c-4fc5-82b7-7f46dfdef7ca)
### With one candidate in my renderer (2127 ms):
![maven_1candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/34210cf6-a791-42d1-abdc-ad1c2c14067b)

### With 32 canddiates (21321 ms):
![maven_32candidate](https://github.com/slayyden/cse-168-final-project/assets/26509702/a4d42d18-f118-4e86-8409-500fb1a9d54f)
### 32 candidates + Spatial reuse (22991 ms):
![maven_spatialreuse](https://github.com/slayyden/cse-168-final-project/assets/26509702/c2a4c7ed-84b3-4a5b-a919-7f41562b64c6)
- once again spatial reuse doesn't do much :P
### With 1 canddiate, but 10spp (20011 ms):
![maven_1candidate_10spp](https://github.com/slayyden/cse-168-final-project/assets/26509702/fdac8adf-0826-4d1d-82e1-319b34afe07f)
 - even at equal time, a we have more noise than we do with ReSTIR
 - in particular, ReSTIR resolves the eyes very well in compoarison

# Bonus: Blender File Importer 
I wrote a script using Blender's Python API to import Blender scenes into the class scenefile format. 
It supports 
- Polygonal meshes with any location, rotation, and scale
  - Polygons may have an arbitrary number of sides
- Rectangular area lights
- Diffuse, specular, and roughness parameters for Principled BSDF materials (Principled is Blender's primary physically based BSDF model)
