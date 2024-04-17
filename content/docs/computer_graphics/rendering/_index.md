---
title: Rendering
date: 2024-04-16
weight: 3
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
