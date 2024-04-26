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

## Resource
- [Siggraph](https://dl.acm.org/conference/siggraph)
- [GAMES](https://games-cn.org/)
- [Real-time rendering website](https://www.realtimerendering.com/)
- [Shadertoy](https://www.shadertoy.com)
- [Mathematics for 3D game programming and computer graphics(3rd)](https://www.amazon.com/Mathematics-Programming-Computer-Graphics-Third/dp/1435458869)
- [Foundations of game engine development volume 1 mathematics](https://www.amazon.com/Foundations-Game-Engine-Development-Mathematics/dp/0985811749)
- [Fundamentals of computer graphics(5th)](https://www.amazon.com/Fundamentals-Computer-Graphics-Steve-Marschner/dp/0367505037)
- [Computer graphics principles and practice(3rd)](https://www.amazon.com/Computer-Graphics-Principles-Practice-3rd/dp/0321399528)
- [Real-time rendering(4th)](https://www.amazon.com/Real-Time-Rendering-Fourth-Tomas-Akenine-M%C3%B6ller/dp/1138627003)
- [Foundations of game engine development volume 2 rendering](https://www.amazon.com/Foundations-Game-Engine-Development-Rendering/dp/0985811757)
- [GAMES101 introduction to computer graphics](https://games-cn.org/games001/)
- [GAMES202 high-quality real-time rendering](https://games-cn.org/games202/)
- [Physically based shading in theory and practice](https://blog.selfshadow.com/publications/)