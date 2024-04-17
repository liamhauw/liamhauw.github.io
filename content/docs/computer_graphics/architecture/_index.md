---
title: Architecture
date: 2024-04-16
weight: 1
---

## Desktop
### Immediate mode rendering
Desktop GPUs are based on Immediate mode rendering (IMR) architecture.

![imr](image/imr.jpg)

### Reference
- Nvidia
  - [Nvidia developer](https://developer.nvidia.com/)
  - [Geforce graphics architectures introduction](https://zhuanlan.zhihu.com/p/403345668)
- AMD
  - [AMD developer](https://www.amd.com/en/developer.html)

## Mobile

### Tile-based deferred rendering
Mobile GPUs are based on Tile-based Deferred Rendering (TBDR) architecture. TBDR combines two complementary architectural features to provide the very highest levels of efficiency and performance:
- Tile-based rendering
- Deferred rendering

![tbdr-qualcomm](image/tbdr-qualcomm.png "Qualcomm TBDR")

![tbdr-imagination](image/tbdr-imagination.jpg "Imagination TBDR")

### Tile-based rendering
The Tile-based rendering architecture splits the screen into a number of ‘tiles’, which are then processed individually (in parallel to other tiles). Since the GPU only needs to work on a subset of the complete scene data at any given time, this data (such as colour and depth buffers) is small enough to be stored in internal GPU memory, significantly reducing the required number of accesses to system level memory. This results in lower energy and bandwidth consumption and also higher performance.

### Deferred rendering
Deferred rendering uses method (Early Z rejection from Qualcomm, Hidden Surface Removal from Imagination) which defers all texturing and shading operations until the visibility of each pixel in the tile is known – only the pixels that will actually be seen by the end user consume processing resources. This means that unnecessary processing of hidden pixels is eliminated, which further ensures the lowest possible bandwidth usage and number of processing cycles per frame, resulting in the highest performance levels and the lowest power consumption.

### Reference
- Qualcomm
  - [Qualcomm developer](https://developer.qualcomm.com/)
  - [Snapdragon Game Toolkit](https://developer.qualcomm.com/sites/default/files/docs/adreno-gpu/snapdragon-game-toolkit/index.html)
- Arn
  - [Arm developer](https://developer.arm.com/)
- Imagination
  - [Imagination developer](https://developer.imaginationtech.com/)
  - [PowerVR graphics architectures](https://www.imaginationtech.com/products/gpu/graphics-architecture/)
