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
- Mathematics for 3D game programming and computer graphics(3rd)
- Foundations of game engine development volume 1 mathematics
- Fundamentals of computer graphics(5th)
- Computer graphics principles and practice(3rd)
- Real-time rendering(4th)
- Foundations of game engine development volume 2 rendering
- Physics and math of shading
- Moving Frostbite to Pbr
- Real shading in Unreal Engine 4
- [Siggraph](https://dl.acm.org/conference/siggraph)
- [GAMES: Graphics And Mixed Environment Symposium](https://games-cn.org/)
- [GAMES101 introduction to computer graphics](https://games-cn.org/games001/)
- [GAMES202 high-quality real-time rendering](https://games-cn.org/games202/)
- [Real-time rendering website](https://www.realtimerendering.com/)
- [Shadertoy](https://www.shadertoy.com)
