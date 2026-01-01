---
title: Vulkan
weight: 2
date: 2025-10-05
---

## Layer

### Windows layer
Registry Editor stores the vulkan explicit and implicit layers in the Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Khronos\Vulkan

### Android layer
```shell
adb shell settings put global enable_gpu_debug_layers 1
adb shell settings put global gpu_debug_layers [layer name ep.VK_LAYER_KHRONOS_validation]
adb shell settings put global gpu_debug_app [apk name]
adb push [layer so eg.libVkLayer_khronos_validation.so] /data/data/[apk name]
adb shell "settings list global | grep gpu"
```

### Reference
- [Vulkan layer factory](https://github.com/LunarG/Vulkan-Layer-Factory)
- [Vulkan loader](https://github.com/KhronosGroup/Vulkan-Loader)

## Coordinate system
The vertex is transformed from the model space to the world space, view space and projection space through the model matrix(M), view matrix(V) and projection matrix(P). Then it is converted to the normal device coordinate (NDC) space through perspective division, and finally to the two-dimensional screen space. World space and view space have nothing to do with hardware.

The coordinate system of the NDC space is the right hand coordinate system, with the x axis to the right [-1, 1], the y axis down [-1, 1], and the z axis in the screen [0, 1]. The x axis of the screen space is to the right and the y axis is down.

The coordinate system of gltf and glm is the right hand coordinate system by default. However, glm projection make it left, we need invert elements (1, 1) in a projection matrix. The depth of glm is [-1, 1] by default, we need define GLM_CLIP_CONTROL_ZO_BIT. The matrices of glm are column priority storage, and when used in shaders, you need to multiply the matrix left by a vector, such as P * V * M * x, where x is the vector.

Given position, normal and tangent vector and model matirx.
- To transform the position vector from model space to world space, we need mutiply the postion by the model matrix.
- To transform the normal and tangent vector from model space to world space, we need mutiply the postion by the **inverse transpose** of the model matrix.

## Mutiple threads
### Reference
- [Vulkan multi-threading](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/munich/mschott_vulkan_multi_threading.pdf)

## Synchronization
### Reference
- [Understanding Vulkan synchronization](https://www.khronos.org/blog/understanding-vulkan-synchronization)
- [Vulkan timeline semaphores](https://www.khronos.org/blog/vulkan-timeline-semaphores)
- [Yet another blog explaining Vulkan synchronization](https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/)
- [Keeping your GPU fed](https://www.youtube.com/watch?v=iZ3J25qsacA)
- [Vulkan guide](https://github.com/KhronosGroup/Vulkan-Guide/blob/main/chapters/synchronization.adoc)
- [Synchronization examples](https://github.com/KhronosGroup/Vulkan-Docs/wiki/Synchronization-Examples)
- [Vulkan synchronization explanation](https://zhuanlan.zhihu.com/p/350483554)

## Resource
- [Vulkan official website](https://www.vulkan.org)
- [Vulkan tutorial](https://vulkan-tutorial.com)
- [Mastering graphics programming with Vulkan](https://www.amazon.com/Mastering-Graphics-Programming-Vulkan-state/dp/1803244798)
- [Nvidia Vulkan samples](https://github.com/nvpro-samples/build_all)
