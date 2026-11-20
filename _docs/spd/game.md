---
title: Game platforms
---

# {{ page.title }}

## Vulkan
### Vulkan layer
#### Windows layer
Registry Editor stores the vulkan explicit and implicit layers in the `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Khronos\Vulkan`

#### Android layer
```shell
adb shell settings put global enable_gpu_debug_layers 1
adb shell settings put global gpu_debug_layers [layer name ep.VK_LAYER_KHRONOS_validation]
adb shell settings put global gpu_debug_app [apk name]
adb push [layer so eg.libVkLayer_khronos_validation.so] /data/data/[apk name]
adb shell "settings list global | grep gpu"
```

### Vulkan coordinate system
The vertex is transformed from the model space to the world space, view space and projection space through the model matrix(M), view matrix(V) and projection matrix(P). Then it is converted to the normal device coordinate (NDC) space through perspective division, and finally to the two-dimensional screen space. World space and view space have nothing to do with hardware.

The coordinate system of the NDC space is the right hand coordinate system, with the x axis to the right `[-1, 1]`, the y axis down `[-1, 1]`, and the z axis in the screen `[0, 1]`. The x axis of the screen space is to the right and the y axis is down.

The coordinate system of gltf and glm is the right hand coordinate system by default. However, glm projection make it left, we need invert elements (1, 1) in a projection matrix.

The depth of glm is `[-1, 1]` by default, we need define `GLM_CLIP_CONTROL_ZO_BIT`. The matrices of glm are column priority storage, and when used in shaders, you need to multiply the matrix left by a vector, such as `P * V * M * x`, where x is the vector.

Given position, normal and tangent vector and model matirx.

- To transform the position vector from model space to world space, we need mutiply the postion by the model matrix.
- To transform the normal and tangent vector from model space to world space, we need mutiply the postion by the **inverse transpose** of the model matrix.

### Vulkan mutiple threads
#### Queue
- `vkQueueSubmit` / `vkQueueSubmit2`, `vkQueuePresentKHR`, `vkQueueWaitIdle`, and `vkQueueBindSparse` on the **same** `VkQueue` must be externally synchronized — one mutex, or a single submit thread.
- **Different queues** may be submitted from different threads with no extra lock.
- Submit order across queues does **not** order GPU work; use semaphores.
- `vkQueueSubmit` is expensive — batch many CBs into few submits. Parallelism belongs in recording, not in pounding the same queue.
#### Command Pool
- Not thread-safe: allocate, reset, free, and recording from CBs in that pool must not overlap across threads.
- One pool per worker (often `pool[thread][frame-in-flight]`).
- Reset only after all CBs from that pool have finished on the GPU (fence / timeline semaphore).
- Pool-wide `vkResetCommandPool` is faster; per-CB reset needs `VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT`.
#### Command Buffer
- One CB may be begun, recorded, ended, or reset by only one thread at a time.
- CBs from **different pools** may be recorded in parallel.
- After `vkEndCommandBuffer`, another thread may submit it.
- Do not begin / reset / free a CB while it is pending.
- Typical MT pattern: workers record **secondaries**; the submit thread records a **primary**, `vkCmdExecuteCommands`, then `vkQueueSubmit`.
- Prefer one-time CBs per thread per frame over `SIMULTANEOUS_USE`.

- [Vulkan official website](https://www.vulkan.org)
- [Vulkan tutorial](https://vulkan-tutorial.com)
- [Mastering graphics programming with Vulkan](https://www.amazon.com/Mastering-Graphics-Programming-Vulkan-state/dp/1803244798)
- [NVIDIA Vulkan samples](https://github.com/nvpro-samples/build_all)
- [Vulkan layer factory](https://github.com/LunarG/Vulkan-Layer-Factory)
- [Vulkan loader](https://github.com/KhronosGroup/Vulkan-Loader)
- [Vulkan multi-threading](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/munich/mschott_vulkan_multi_threading.pdf)
- [Understanding Vulkan synchronization](https://www.khronos.org/blog/understanding-vulkan-synchronization)
- [Vulkan timeline semaphores](https://www.khronos.org/blog/vulkan-timeline-semaphores)
- [Yet another blog explaining Vulkan synchronization](https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization)
- [Keeping your GPU fed](https://www.youtube.com/watch?v=iZ3J25qsacA)
- [Vulkan guide](https://github.com/KhronosGroup/Vulkan-Guide/blob/main/chapters/synchronization.adoc)
- [Synchronization examples](https://github.com/KhronosGroup/Vulkan-Docs/wiki/Synchronization-Examples)
- [Zhihu: Vulkan synchronization explanation](https://zhuanlan.zhihu.com/p/350483554)

## OpenGL
- [OpenGL official website](https://www.khronos.org/opengl)
- [Computer graphics through OpenGL from theory to experiments (3rd)](https://www.amazon.com/Computer-Graphics-Through-OpenGL%C2%AE-Experiments/dp/1138612642)
- [OpenGL programming guide](https://www.amazon.com/OpenGL-Programming-Guide-Official-Learning/dp/0321773039)
- [Learn OpenGL](https://learnopengl.com)

## Direct3D12
- [Direct3D12 official website](https://docs.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-graphics)
- [Introduction to 3D game programming with DirectX 12](https://www.amazon.com/Introduction-3D-Game-Programming-DirectX/dp/1942270062)

## Game engines
- [Game engine architecture](https://www.amazon.com/Engine-Architecture-Third-Jason-Gregory/dp/1138035459)
- [GAMES104 modern game engine](https://games-cn.org/games104)
- [Android games dev center](https://developer.android.com/games)
- [Italink personal website](https://italink.github.io/ModernGraphicsEngineGuide)
- [NothingToSay0031 personal website](https://nothingtosay0031.github.io)
- [Unreal source code](https://github.com/EpicGames/UnrealEngine)
- [Unreal offical docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-8-documentation)
- [Nanite a deep dive](https://advances.realtimerendering.com/s2021/Karis_Nanite_SIGGRAPH_Advances_2021_final.pdf)