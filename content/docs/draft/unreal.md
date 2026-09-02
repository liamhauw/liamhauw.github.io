# Unreal

## 目标
- 熟悉引擎架构
- 熟悉渲染模块的底层设计和原理，如FrameGraph、RHI抽象、数据局部性、并行度、材质管理和shader cache方案等进行深入分析和优化，具备源码级修改能力
- 熟悉 UE5 渲染特性（Nanite、Lumen、VSM、MegaLights 等）

---

## 推荐阅读顺序
一份精选的 **纯英文** 阅读清单，涵盖 UE5 引擎架构相关文章、演讲与文档，重点是渲染器的底层设计。

1. 线程模型 + RHI / RDG（数据如何从 Game Thread 到达 GPU）
2. Mesh Drawing Pipeline + GPUScene（传统网格如何变成绘制命令）
3. 延迟渲染主循环（`FDeferredShadingSceneRenderer`）
4. Nanite → Lumen → VSM → TSR → MegaLights（UE5 特有层）

读源码时，核心目录是：

- `Engine/Source/Runtime/Engine` — Game Thread 场景对象，`UPrimitiveComponent`
- `Engine/Source/Runtime/Renderer` — `FScene`、`FDeferredShadingSceneRenderer`、Nanite / Lumen / VSM
- `Engine/Source/Runtime/RenderCore` — RDG、shader 基础设施
- `Engine/Source/Runtime/RHI` + `D3D12RHI` / `VulkanRHI` — 图形 API 抽象与后端

---

## 1. 官方文档：引擎与渲染器骨架

打开 C++ 之前，先用这些文档把术语对齐。

| 文章 | 为什么重要 |
|---|---|
| [Graphics Programming Overview](https://dev.epicgames.com/documentation/unreal-engine/graphics-programming-overview-for-unreal-engine) | 官方总览：Renderer / RHI、Game Thread 与 Render Thread、静态与动态绘制路径。 |
| [Graphics Programming for Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/graphics-programming-for-unreal-engine) | 图形编程文档入口：RDG、Mesh Drawing、并行渲染、shaders。 |
| [Threaded Rendering](https://dev.epicgames.com/documentation/unreal-engine/threaded-rendering-in-unreal-engine) | `ENQUEUE_RENDER_COMMAND`、`FRenderResource`、proxy 与 Game Thread 对象。这是 UE 渲染架构的根基。 |
| [Mesh Drawing Pipeline](https://dev.epicgames.com/documentation/unreal-engine/mesh-drawing-pipeline-in-unreal-engine) | `FMeshBatch` → `FMeshPassProcessor` → `FMeshDrawCommand` → RHI；cached vs dynamic；GPUScene / PrimitiveId。 |
| [Render Dependency Graph](https://dev.epicgames.com/documentation/unreal-engine/render-dependency-graph-in-unreal-engine) | UE5 的 frame-graph 编译器：pass 声明、生命周期、barrier、async compute、transient aliasing。现代 Renderer 代码几乎都写在 RDG 上。 |

UE5 仍建立在这次历史性重构之上：

- [Refactoring the Mesh Drawing Pipeline (GDC 2019, video)](https://www.youtube.com/watch?v=qx1c190aGhs) — Marcus Wassmer。为何废弃 Drawing Policies、为何引入 retained-mode 的 `FMeshDrawCommand`，以及这如何为 GPU-driven rendering 和光线追踪铺路。

---

## 2. 源码级架构分析（C++ 走读）

| 文章 | 深度 | 备注 |
|---|---|---|
| [Unreal Source Explained — Rendering](https://donaldwuid.github.io/unreal_source_explained/main/rendering.html) | ★★★★ | 少见的 **源码 + profiler** 拆解：`UWorld` / `FScene`、proxy、Mesh Draw Commands、GPUScene、RHI。配套 [GitHub](https://github.com/donaldwuid/unreal_source_explained) 与 [overview](https://donaldwuid.github.io/unreal_source_explained/main/main.html)。聚焦 UE4.23 / 移动端，但类图在 UE5 中仍然成立。 |
| [Advanced Graphics Programming in Unreal (Part 1–7)](https://medium.com/@manning.w27/advanced-graphics-programming-in-unreal-part-1-10488f2e17dd) | ★★★★★ | 最好的「写代码学渲染器 C++」系列：RHI / RDG / 线程、`FScene` / `FViewInfo` / `FSceneViewExtension`、Global Shaders、Material Shaders、Mesh Passes。后续：[Part 2](https://medium.com/@manning.w27/advanced-graphics-programming-in-unreal-part-2-abf8237491c1) · [Part 3](https://medium.com/@manning.w27/advanced-graphics-programming-in-unreal-part-3-f37f4d4407d7) · [Part 4](https://medium.com/@manning.w27/advanced-graphics-programming-in-unreal-part-4-c89d6fb98b59)。 |
| [UnrealEngineDeepDive](https://github.com/islamhaqq/UnrealEngineDeepDive) | ★★★ | 模块分层：Core / HAL / RHI / Renderer。适合看清引擎如何堆叠。 |
| [UnrealRDGLearning](https://github.com/Ardaurum/UnrealRDGLearning) | ★★★ | UE5.4 RDG / RHI 学习项目：RHI 与 RDG 术语，在不 fork 引擎的情况下插入 pass。 |
| [RDG notes (staticJPL)](https://github.com/staticJPL/Render-Dependency-Graph-Documentation/blob/main/Render%20Dependency%20Graph%20(RDG).md) | ★★★ | Setup / Compile / Execute，对应到 `FRDGBuilder`。 |
| [Rendering pipeline UE5 (frame graph walkthrough)](http://teres4enko.blogspot.com/2025/03/rendering-pipeline-ue5.html) | ★★★ | 按实际 GPU RDG 顺序走读：PrePass → Nanite VisBuffer → LumenSceneUpdate → BasePass → VSM。适合对照 `stat gpu` / Insights。 |

RHI Thread 的设计意图，由 Epic 工程师亲自回答：

- [What is the RHI thread vs rendering thread?](https://forums.unrealengine.com/t/what-is-the-rhi-thread-what-is-the-difference-between-rhi-thread-and-rendering-thread/20412)

---

## 3. Nanite：虚拟化几何与 GPU-driven rendering

这是 UE5 渲染与经典 UE4 deferred 之间最大的分叉。

### Epic 一手资料（必读）

1. [Nanite: A Deep Dive (SIGGRAPH 2021, PDF)](https://www.advances.realtimerendering.com/s2021/Karis_Nanite_SIGGRAPH_Advances_2021_final.pdf) — Brian Karis / Rune Stubbe / Graham Wihlidal  
   Cluster DAG、层次化剔除、software rasterizer、Visibility Buffer、retained GPU scene。读 Nanite 源码前的第一份文档。

2. [Bringing Nanite to Fortnite Chapter 4](https://www.unrealengine.com/tech-blog/bringing-nanite-to-fortnite-battle-royale-in-chapter-4) — Graham Wihlidal  
   Programmable raster：WPO、masked、two-sided。固定功能的 software rasterizer 如何变成材质驱动。

3. [Nanite GPU-Driven Materials (GDC 2024)](https://www.unrealengine.com/en-US/blog/take-a-deep-dive-into-nanite-gpu-driven-materials)  
   博客 + [slides PDF](https://media.gdcvault.com/gdc2024/Slides/GDC+slide+presentations/Nanite+GPU+Driven+Materials.pdf)  
   Raster binning、programmable raster、shade binning、从 VisBuffer 填充 GBuffer。对应 `NaniteCullRaster` / `NaniteShading`。

### 第三方、偏源码的解读

- [Nanite Deep Dive — Part 1 (Tricky Bits)](https://trickybitsblog.github.io/2024/04/20/nanite.html) — GPU-driven 异步管线、VisBuffer vs GBuffer，传统几何与 Nanite 如何合并进同一张 GBuffer。

源码入口：`Engine/Source/Runtime/Renderer/Private/Nanite/`。一帧里先读 VisBuffer pass，再读材质着色。

---

## 4. Lumen：动态 GI 与反射

1. [Lumen: Real-time GI in UE5 (SIGGRAPH 2022, PDF)](https://advances.realtimerendering.com/s2022/SIGGRAPH2022-Advances-Lumen-Wright%20et%20al.pdf) — Daniel Wright / Krzysztof Narkowicz / Patrick Kelly  
   Screen trace → Software RT（SDF + Surface Cache）→ Hardware RT；card captures；Radiance Cache。Lumen 源码的地图。

2. [UE5 goes all-in on Lumen (Epic tech blog)](https://www.unrealengine.com/en-US/tech-blog/unreal-engine-5-goes-all-in-on-dynamic-global-illumination-with-lumen)  
   为何需要 Surface Cache、Nanite 如何加速 capture、为何 Lumen 依赖 TSR。比 SIGGRAPH 演讲更偏系统全貌。

3. [Official Lumen docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine)

4. [Real-time GI in UE5 (Masaryk University thesis, PDF)](https://is.muni.cz/th/n1qq4/real-time_GI_in_UE5.pdf) — 以论文形式写就的 Lumen 管线。适合当结构化笔记用。

源码入口：`Engine/Source/Runtime/Renderer/Private/Lumen/`。

---

## 5. 阴影、直接光照、后处理

### Virtual Shadow Maps

- [Virtual Shadow Maps in Fortnite Chapter 4](https://www.unrealengine.com/en-US/tech-blog/virtual-shadow-maps-in-fortnite-battle-royale-chapter-4) — Andrew Lauritzen / Ola Olsson。Paging、与 Nanite raster 的耦合、cache invalidation。VSM 的设计文档级别。
- [Virtual Shadow Maps (official docs)](https://dev.epicgames.com/documentation/en-us/unreal-engine/virtual-shadow-maps-in-unreal-engine)

### MegaLights（UE5.5+ 随机直接光照）

- [MegaLights: Stochastic Direct Lighting (SIGGRAPH 2025, PDF)](https://advances.realtimerendering.com/s2025/content/MegaLights_Stochastic_Direct_Lighting_2025.pdf)
- [Advances in Real-Time Rendering 2025 course page](https://advances.realtimerendering.com/s2025/index.html)
- [Official MegaLights docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/megalights-in-unreal-engine)

### Temporal Super Resolution

- [Temporal Super Resolution (Epic PDF)](https://assets-unreal2-epic-prod-us2.s3.dualstack.us-east-1.amazonaws.com/original/4X/6/6/d/66dab26619dbce6c73ad3cf02722e1b1787f0cc9.pdf) — 为何 Nanite / Lumen 需要 TSR；history、disocclusion、在 DoF 之后做 upscale。

### Substrate materials

- [Unreal Engine Substrate (SIGGRAPH 2023, PDF)](https://advances.realtimerendering.com/s2023/2023%20Siggraph%20-%20Substrate.pdf) — 可组合 BSDF、可伸缩 GBuffer，Nanite / Lumen 之后材质如何演进。
- [The Future of Materials in Unreal Engine (GDC 2023, video)](https://www.youtube.com/watch?v=joOIBteSo1w)

---

## 6. 架构与文章对照

```text
Game Thread          UWorld / UPrimitiveComponent
        |  proxy 拷贝
Render Thread        FScene / FPrimitiveSceneProxy / FViewInfo
        |  FDeferredShadingSceneRenderer::Render
RDG                  FRDGBuilder  （整帧 pass graph）
        |  MeshDrawCommand / Nanite compute
RHI                  FRHICommandList
        |  D3D12 / Vulkan / Metal
GPU                  VisBuffer → GBuffer → Lumen / VSM / MegaLights → TSR
```

| 层级 | 先读 |
|---|---|
| 线程 / proxy | Threaded Rendering；Manning Part 2；Unreal Source Explained |
| Mesh 提交 | Mesh Drawing Pipeline；GDC 2019 |
| Frame graph | 官方 RDG；Manning Part 2；RDG notes |
| 几何 | Nanite SIGGRAPH 2021 → Fortnite blog → GDC 2024 materials |
| GI | Lumen SIGGRAPH 2022 + Epic tech blog |
| 阴影 | VSM Fortnite blog |
| 直接光照 | MegaLights SIGGRAPH 2025 |
| 上采样 | TSR PDF |
| 材质 | Substrate SIGGRAPH 2023 |

---

## 7. 最小源码入口

读完文章后从这里跳进去：

- `FRendererModule::BeginRenderingViewFamily` — Game Thread 提交一帧
- `FDeferredShadingSceneRenderer::Render` — 延迟渲染主循环（RDG graph 在这里构建）
- `FSceneRenderer::InitViews` / visibility — 可见性、动态 MeshDrawCommands
- `FGPUScene::Update` — primitive 数据上 GPU
- `AddNanite*` / `Nanite::FCullRasterizeArgs` — VisBuffer
- `FDeferredShadingSceneRenderer::RenderLumen*` — Surface Cache + tracing
- `FVirtualShadowMapArray` — VSM page table 与 raster

---

## 如何使用这份清单

- **只关心 UE5 相对 UE4 的变化：** Karis Nanite 2021 → Wright Lumen 2022 → 官方 RDG → Mesh Drawing Pipeline。
- **准备改 Renderer / 加 pass：** Manning 七篇 + 官方 RDG + Mesh Drawing Pipeline，然后读源码。
- **对照 Insights / `stat gpu` 的 pass 名：** teres4enko 的管线走读 + Fortnite 的 Nanite / VSM 博客。
