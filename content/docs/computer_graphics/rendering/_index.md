---
title: Rendering
date: 2024-04-16
weight: 2
---

## Vector transform in vertex shader

Given position, normal and tangent vector and model matirx. 
- To transform the position vector from model space to world space, we need mutiply the postion by the model matrix.
- To transform the normal and tangent vector from model space to world space, we need mutiply the postion by the **inverse transpose** of the model matrix.

## Variable rate shading

### Reference
- [Visually Lossless Content and Motion Adaptive Shading in Games](https://leiy.cc/publications/nas/nas-pacmcgit.pdf)

## Best practices

### Reference
- [Arm Developer Documentation](https://developer.arm.com/documentation/101897/latest/)

## Resource
### Real-time rendering
- Real-Time Rendering(4th)
- Foundations of Game Engine Development Volume 2 Rendering
- Physics and Math of Shading
- Moving Frostbite to Pbr
- Real Shading in Unreal Engine 4
- [GAMES202 High-Quality Real-Time Rendering](https://www.bilibili.com/video/BV1YK4y1T7yY)
- [shadertoy](https://www.shadertoy.com)

### Offline rendering
- [pbrt](https://www.pbrt.org)
