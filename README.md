# ReSTIR 
This project implements ReSTIR direct lighting. 
It supports the following features
+ Streaming RIS
+ Spatial reuse
  - kernel is a square of arbitrary size
  - number of spatial neighbors chosen is customizable
  - any number of reuse passes can be done
+ MIS with BSDF and NEE samples
  - unbiased without spatial reuse
+ Aribtrary number of samples per pixel 

# streaming RIS 
Preliminary result: 

<img width="249" alt="image" src="https://github.com/slayyden/cse-168-final-project/assets/26509702/8920ef1b-d148-4e02-b986-7cc2f2369f8f">

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
### With 1 canddiate, but 10spp (20011):
![maven_1candidate_10spp](https://github.com/slayyden/cse-168-final-project/assets/26509702/fdac8adf-0826-4d1d-82e1-319b34afe07f)
 - even at equal time, a we have more noise than we do with ReSTIR
 - in particular, ReSTIR resolves the eyes very well in compoarison
