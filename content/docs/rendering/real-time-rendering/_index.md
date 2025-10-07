---
title: Real-time rendering
weight: 1
date: 2025-10-08
---

## Shading basics
### Aliasing and antialiasing
At its heart, ​​aliasing​​ is a type of artifact or distortion that occurs when a signal is sampled at a rate that is too low to accurately represent it. Antialiasing​​ is a set of techniques used to reduce or eliminate the visual artifacts of aliasing.

Here are some of the most common methods:
- MSAA
- FXAA
- MLAA
- SMAA
- TAA

### Reference
- [Experimenting with reconstruction filters for MSAA resolve](https://therealmjp.github.io/posts/msaa-resolve-filters/)
- [FXAA shadertoy](https://www.shadertoy.com/view/ls3GWS)
- [MLAA shadertoy](https://www.shadertoy.com/view/43lBD7)
- [MLAA from 2009 to 2017](http://www.iryoku.com/research-impact-retrospective-mlaa-from-2009-to-2017)
- [MLAA paper](https://www.intel.com/content/dam/develop/external/us/en/documents/z-shape-arm-785403.pdf)
- [Enhanced Subpixel Morphological Antialiasing](https://www.iryoku.com/smaa/)
- [Temporal reprojection antialiasing in INSIDE](http://s3.amazonaws.com/arena-attachments/655504/c5c71c5507f0f8bf344252958254fb7d.pdf?1468341463)
- [High quality temporal supersampling](https://de45xmedrsdbp.cloudfront.net/Resources/files/TemporalAA_small-59732822.pdf)
- [An excursion in temporal supersampling](https://developer.download.nvidia.com/gameworks/events/GDC2016/msalvi_temporal_supersampling.pdf)
- [Temporal AA and the quest for the Holy Trail](https://www.elopezr.com/temporal-aa-and-the-quest-for-the-holy-trail/)
- [Temporal antialiasing starter pack](https://alextardif.com/TAA.html)

## Efficient shading
### Cluster shading
Quickly determine ​​which lights affect which small 3D regions​​ of the scene, so each pixel only processes the lights that actually affect it.

### Reference
- [Mesh shading for Vulkan](https://www.khronos.org/blog/mesh-shading-for-vulkan)
- [Vulkan Subgroup tutorial](https://www.khronos.org/blog/vulkan-subgroup-tutorial)
- [Vulkan Subgroup explained](chrome-extension://kppkpfjckhillkjfhpekeoeobieedbpd/lib/pdfjs/web/viewer.html?file=https%3A%2F%2Fwww.khronos.org%2Fassets%2Fuploads%2Fdevelopers%2Flibrary%2F2018-vulkan-devday%2F06-subgroups.pdf)
- [A primer on efficient rendering algorithms and clustered shading](http://www.aortiz.me/2018/12/21/CG.html)
- [Nvidia mega geometray samples](https://github.com/nvpro-samples/build_all?tab=readme-ov-file#mega-geometry)
- [Niagara renderer](https://github.com/zeux/niagara)

## Global illumination
### Path tracing
Path tracing is a more advanced, physically accurate form of ray tracing that uses randomness to solve the entire problem of light simulation.​​
### Reference
- [Nvidia ray/path tracing samples](https://github.com/nvpro-samples/build_all?tab=readme-ov-file#vulkan-ray-tracing-and-path-tracing-samples)

## Neural rendering
### Neural radiance field and 3D gaussian splatting
​Neural Radiance Fields(NeRF) is a technique that uses a small neural network to represent a 3D scene as a continuous volumetric function.

3D gaussian splatting is a method for representing and rendering 3D scenes using millions of anisotropic 3D Gaussian primitives. Think of it as a sophisticated, intelligent point cloud where each point is an ellipsoid that can be efficiently rendered to create photorealistic novel views.

### Nerual shading
Neural Shading​​ refers to techniques that use neural networks to replace, augment, or enhance traditional shading calculations in computer graphics. Instead of using hand-crafted analytical models (like Phong, Blinn-Phong, or PBR), neural networks learn the relationship between surface properties, lighting, and material appearance directly from data.

### Reference
- [Nerf: Representing scenes as neural radiance fields for view synthesis](https://dl.acm.org/doi/pdf/10.1145/3503250)
- [3D Gaussian splatting for real-time radiance field rendering](https://dl.acm.org/doi/pdf/10.1145/3592433)
- [Nvidia 3DGS sample](https://github.com/nvpro-samples/vk_gaussian_splatting)
- [An introduction to neural shading](https://dl.acm.org/doi/pdf/10.1145/3721241.3733999)
- [RenderFormer](https://microsoft.github.io/renderformer/)

## Resource
- [Mathematics for 3D game programming and computer graphics(3rd)](https://www.amazon.com/Mathematics-Programming-Computer-Graphics-Third/dp/1435458869)
- [Foundations of game engine development volume 1 mathematics](https://www.amazon.com/Foundations-Game-Engine-Development-Mathematics/dp/0985811749)
- [Real-time rendering(4th)](https://www.amazon.com/Real-Time-Rendering-Fourth-Tomas-Akenine-M%C3%B6ller/dp/1138627003)
- [Fundamentals of computer graphics(5th)](https://www.amazon.com/Fundamentals-Computer-Graphics-Steve-Marschner/dp/0367505037)
- [Computer graphics principles and practice(3rd)](https://www.amazon.com/Computer-Graphics-Principles-Practice-3rd/dp/0321399528)
- [Foundations of game engine development volume 2 rendering](https://www.amazon.com/Foundations-Game-Engine-Development-Rendering/dp/0985811757)
- [Real-time rendering website](https://www.realtimerendering.com/)
- [Siggraph](https://dl.acm.org/conference/siggraph)
- [Physically based shading in theory and practice](https://blog.selfshadow.com/publications/)
- [GAMES](https://games-cn.org/)
- [Shadertoy](https://www.shadertoy.com)
- [Nvidia RTX kit](https://developer.nvidia.com/rtx-kit/?sortBy=developer_learning_library%2Fsort%2Ftitle%3Aasc&hitsPerPage=15)