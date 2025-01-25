---
title: luka
summary: luka is a a Vulkan API-based real-time rendering engine with modular components including platform, core, base, resource, function, editor, and rendering.
date: 2025-01-25
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
- Configurability of multiple types of punctual lights
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
- Forward shading
- Deferred shading
- Physically based shading
- Mesh shading (DOING)
- Postprocess
  - FXAA
  - MLAA
  - TAA
  - Bilateral filter
  - Edge detect
  - Sharpen
- Shadow (DOING)
- Volumetric and translucency rendering (TODO)

## The future
This project will be open source later.
