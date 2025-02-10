---
title: luka
summary: A Vulkan API-based real-time rendering engine with modular components including platform, core, base, resource, function, rendering and editor.
date: 2025-02-10
authors:
  - admin
tags:
  - Real-time rendering
  - Game engine
---

## Features
- Following the RAII principle
- Task-based multi-threading using enkiTS
- Supporting glTF scenes
- Configurability of directional and punctual lights
- User-defined shaders
- Driving rendering with frame graph
- Asynchronously loading asset
- Supporting GPU-driven rendering with task shader and mesh shader
- Improving load time with spirv cache and pipeline cache
- Automating pipeline layout generation with SPIRV-Cross
- Bindless rendering with descriptor indexing
- Recording commands on multiple threads
- Supporting graphics and compute pipeline
- Analyzing rendering performance with query pool
- User interface using imgui

## Implementation based on engine
- Shading basics
  - Forward shading
  - Deferred shading
  - Mesh shading
  - Fast approximate antialiasing (FXAA)
  - Morphological antialiasing (MLAA)
  - Temporal antialiasing (TAA)
- Shadows
  - Shadow maps
  - Percentage closer soft shadows (PCSS)
  - Variance soft shadow map (VSSM)
- Physically based shading
  - Metallic roughness
- Image space effects
  - Bilateral filter
  - Edge detect
  - Sharpen
- Acceleration algorithms
  - Back face culling
  - View frustum culling
  - Occulusion culling
- Volumetric and translucency rendering
  - Atmosphere

## The future
This project will be open source later.
