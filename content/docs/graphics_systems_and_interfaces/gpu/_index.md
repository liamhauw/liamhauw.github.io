---
title: GPU
weight: 1
date: 2026-01-27
---

## Desktop GPU architecture
### Reference
- [Nvidia developer](https://developer.nvidia.com/)
- [Nvidia Geforce graphics architectures introduction](https://zhuanlan.zhihu.com/p/403345668)
- [AMD developer](https://www.amd.com/en/developer.html)

## Mobile GPU architecture
### Reference
- [Qualcomm developer](https://www.qualcomm.com/developer)
- [Qualcomm Snapdragon game toolkit](https://www.qualcomm.com/developer/snapdragon-game-toolkit)
- [Qualcomm Mobile game development guide](https://zhuanlan.zhihu.com/p/1911434325402026577)
- [Arm developer](https://developer.arm.com/)
- [Arm mobile graphics and gaming](https://developer.arm.com/mobile-graphics-and-gaming)
- [Arm tile-based rendering](https://developer.arm.com/documentation/102662/0100)
- [Arm GPU best practices developer guide](https://developer.arm.com/documentation/101897/latest/)
- [Arm GPUs training series](https://www.youtube.com/playlist?list=PLKjl7IFAwc4QUTejaX2vpIwXstbgf8Ik7)
- [Imaginationtech developer](https://developer.imaginationtech.com/)
- [Imaginationtech PowerVP developer documentation](https://docs.imgtec.com/html/index.html)
- [Imaginationtech PowerVR graphics architectures](https://www.imaginationtech.com/products/gpu/graphics-architecture/)
- [Samsung developer](https://developer.samsung.com/)
- [Samsung galaxy game development](https://developer.samsung.com/galaxy-gamedev)
- [GPU hardware and parallel rasterization](https://gfxcourses.stanford.edu/cs248a/winter24content/media/gpuhardware/18_mobilegpu.pdf)
- [Summary of GPU architecture knowledge for mobile devices](https://zhuanlan.zhihu.com/p/112120206)
- [IMR, TBR, TBDR and some understanding of GPU architecture](https://zhuanlan.zhihu.com/p/259760974)

## Common patterns that disable Early-ZS and HSR
Using discard (or clip) in Shaders
- Impact: Forces Late-ZS Update and typically disables HSR.
- Reason: The GPU cannot determine if a fragment will be written to the depth buffer until the shader finishes execution (since the shader might discard the fragment). To ensure correctness, the GPU must execute the shader first before updating the depth buffer, preventing early optimization.

Writing to gl_FragDepth (Modifying Depth)
- Impact: Forces Late-ZS Test & Update and disables HSR.
- Reason: Early-Z relies on the interpolated depth value before the shader runs. If the shader manually modifies the depth value, the early check becomes invalid. The GPU must wait for the shader to calculate the final depth before performing any depth testing.

Reading from Depth/Stencil Buffer
- Impact: Forces Late-ZS Update and disables HSR.
- Reason: Reading the depth buffer creates a dependency on previous fragments covering the same pixel. To ensure the read value is correct, the GPU must serialize execution and wait for previous fragments to finish writing to the depth buffer, breaking the parallel pipeline required for early optimizations.

Reading from Framebuffer or Pixel Local Storage (PLS)
- Impact: Disables HSR.
- Reason: HSR works by determining the single visible surface before shading. However, if a shader reads the existing color of a pixel (e.g., for programmable blending), it implies a dependency on "what was there before." HSR cannot safely discard covered fragments because their color might be needed by the top-most fragment's shader logic.

Writing to SSBOs or UAVs
- Impact: Defaults to Late-ZS Test & Update (disabling Early-ZS) and disables HSR.
- Reason: By standard, depth testing happens after shading. If a shader writes to a global buffer (SSBO/UAV), that write is a side effect that "should" happen even if the fragment later fails the depth test. Early-Z would prevent the shader from running entirely, missing this side effect.
- Solution: You can force Early-Z back on using layout(early_fragment_tests) in; (GLSL) or [earlydepthstencil] (HLSL) if you don't care about the side effects of occluded fragments.

### Reference
- [Five common patterns that disable Early-ZS and HSR](https://nothingtosay0031.github.io/GPU/EarlyZS)
