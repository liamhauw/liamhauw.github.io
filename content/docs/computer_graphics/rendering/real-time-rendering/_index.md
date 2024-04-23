---
title: Real-time rendering
weight: 1
date: 2024-04-22
---

## Vector transform in vertex shader

Given position, normal and tangent vector and model matirx. 
- To transform the position vector from model space to world space, we need mutiply the postion by the model matrix.
- To transform the normal and tangent vector from model space to world space, we need mutiply the postion by the **inverse transpose** of the model matrix.

## Variable rate shading

### Reference
- [Visually lossless content and motion adaptive shading in games](https://leiy.cc/publications/nas/nas-pacmcgit.pdf)

## Best practices

### Reference
- [Arm GPU best practices developer guide](https://developer.arm.com/documentation/101897/latest/)

## Resource
- Real-time rendering(4th)
- Foundations of game engine development volume 2 rendering
- Physics and math of shading
- Moving Frostbite to Pbr
- Real shading in Unreal Engine 4
- [GAMES101 introduction to computer graphics](https://www.bilibili.com/video/BV1X7411F744)
- [GAMES202 high-quality real-time rendering](https://www.bilibili.com/video/BV1YK4y1T7yY)
- [Shadertoy](https://www.shadertoy.com)
