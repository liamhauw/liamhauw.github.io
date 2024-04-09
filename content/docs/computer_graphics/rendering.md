---
title: Rendering
date: 2024-04-09
weight: 3
---

## Vector transform in vertex shader

Given position, normal and tangent vector and model matirx. 
- To transform the position vector from model space to world space, we need mutiply the postion by the model matrix.
- To transform the normal and tangent vector from model space to world space, we need mutiply the postion by the **inverse transpose** of the model matrix.
