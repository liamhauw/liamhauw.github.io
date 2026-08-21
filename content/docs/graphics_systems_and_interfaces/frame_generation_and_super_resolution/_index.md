---
title: Frame generation and super resolution
weight: 6
date: 2026-08-21
---

## Why reconstruction became default

Frame generation and super resolution are now part of the default rendering contract on both PC and mobile. Games no longer try to shade every displayed pixel, or even every displayed frame. They render a cheaper internal stream and reconstruct a denser output that the display can present.

Frame generation (FG) keeps the simulation and shading rate lower: it inserts extra display frames between two rendered frames so motion looks smoother on a high-refresh panel. Super resolution (SR) renders the 3D scene below the panel resolution, then reconstructs a higher-resolution image.

The two techniques solve different costs. FG attacks the remaining time between presented frames after the GPU has already produced a valid image. SR attacks pixel shading, bandwidth, and memory. They are usually stacked: extra frames are interpolated from an upscaled rendered frame, then composed with UI.

Reconstruction does not always see the same inputs. That is the main split in this industry.

- An **in-engine** path uses jitter, color, depth, motion vectors, and history. NVIDIA DLSS, AMD FSR, Intel XeSS, Unreal TSR, Snapdragon GSR, Arm ASR, and MetalFX live here. Quality is high when the engine is honest.
- An **overlay** path treats the swapchain as video. Motion estimation and compensation (MEMC) run on a GPU driver or on a companion display chip, often with no depth and no engine velocity. NVIDIA Smooth Motion, AMD Fluid Motion Frames, and the Game Space toggles on OPPO, vivo, Xiaomi, Huawei, and Honor phones live here. Coverage is high because the game does not have to integrate.

Three constraints explain why both paths shipped.

- Displays grew faster than shading budgets. 4K, 120 Hz, and path tracing on PC, and 1.5K / 2K 120 Hz panels on phones, all ask for more pixels per second than a game can shade natively.
- Temporal reuse is cheaper than native shading. Once a history buffer exists, reconstructing missing pixels is cheaper than shading them. Overlay MEMC reuses the same idea without asking the engine for velocity.
- Latency and power are first-class. Generated frames do not reduce input latency by themselves. Mobile also has a thermal wall that desktop GPUs do not, so phones either slim the shader (SGSR, ASR) or move MEMC off the SoC onto a display chip.

This page surveys that stack as an industry, not as a single vendor feature.

## PC and mobile market map

PC is a GPU SDK war around image quality, path tracing, and multi-frame generation. Mobile is three layers on top of each other: SoC and platform APIs, OEM overlays, and engine defaults.

### PC

Three GPU vendors ship a full reconstruction suite. All three now look similar on paper: frame generation, super resolution, and a low-latency companion.

- NVIDIA [DLSS](https://developer.nvidia.com/rtx/dlss) is the quality and adoption leader. Frame generation started as 2x interpolation on RTX 40, then became Multi Frame Generation on RTX 50, with a dynamic multiplier and a 6x mode that can insert five generated frames per rendered frame. NVIDIA Reflex is required in practice, because generated frames otherwise add display delay. DLSS Super Resolution moved from CNNs to a transformer model, then to a second-generation transformer in DLSS 4.5, available across GeForce RTX GPUs. Ray reconstruction sits beside SR as a neural denoiser for path-traced lighting. Game count is the main moat: hundreds of titles, plus driver-side model overrides.
- AMD [FSR](https://gpuopen.com/amd-fsr-sdk/) split into two generations that still ship together. FSR 3 is analytical, open, and vendor-agnostic: optical-flow frame generation plus temporal upscaling that can run on older AMD, NVIDIA, and Intel GPUs. FSR 4 / Redstone is the ML stack on RDNA 4: Frame Generation 4.x, FSR Upscaling 4.x, Ray Regeneration, and a radiance-caching preview. The SDK falls back to the FSR 3 path on older hardware. AMD's pitch is still breadth: one integration, many GPUs, with the best quality reserved for new Radeon cards.
- Intel [XeSS](https://www.intel.com/content/www/us/en/developer/topic-technology/gamedev/xess.html) is the cross-vendor AI option. XeSS 2 added Frame Generation and Xe Low Latency. XeSS 3 added Multi Frame Generation on Intel hardware, up to three generated frames per rendered frame. XeSS-SR uses XMX matrix hardware on Arc and a DP4a fallback elsewhere. Intel opened XeSS 2 APIs to other vendors, so FG, SR, and XeLL can run on recent NVIDIA and AMD GPUs, with the fastest path remaining on Arc.

Beside the GPU SDKs, engines ship a default that works everywhere. Unreal Engine [Temporal Super Resolution](https://dev.epicgames.com/documentation/en-us/unreal-engine/temporal-super-resolution-in-unreal-engine) (TSR) is a platform-agnostic temporal upscaler. It is the fallback when a vendor plugin is missing, and it is the production path on consoles. Microsoft DirectSR tried to give PC titles one upscaler front end. In practice most shipped games still expose DLSS, FSR, XeSS, and TSR as separate options.

Driver-side interpolation also exists: NVIDIA Smooth Motion, AMD Fluid Motion Frames, and similar overlays. They need no game integration, so they cover older titles, but they cannot see engine motion vectors or UI layers as cleanly as an in-engine SDK. That PC overlay is the closest cousin of OEM Game Space interpolation.

### Mobile

The mobile win is not 4K path tracing. It is sustaining 60 or 120 FPS without cooking the chip, while keeping UI at panel resolution. SoC and platform APIs (Qualcomm, Arm, MediaTek, Apple) expect an engine integration, the same as DLSS or FSR. OEMs (Huawei, Xiaomi, OPPO, vivo, Honor) add a coverage layer on top: the GPU still renders a cheap stream, and a companion visual processor — historically [Pixelworks](https://www.pixelworks.com/) X7, later in-house silicon — runs MEMC interpolation, super resolution, and HDR on the way to the panel. Game Space can turn this on without an engine plugin, so it covers titles that never shipped SGSR or ASR. The cost is overlay FG: weak motion vectors, UI smear, and a whitelist that only later became "all games." Several OEMs now expose a graphics SDK so a title can hand over velocity and UI layers, which is how this layer starts to look like DLSS.

- Qualcomm ships Adreno Frame Motion Engine (AFME) and [Snapdragon Game Super Resolution](https://github.com/SnapdragonStudios/snapdragon-gsr) (SGSR) as part of Snapdragon Elite Gaming. AFME interpolates extra frames so a game can present 120 FPS-class motion from a lower rendered rate. SGSR 1 is a single-pass spatial upscaler with a Lanczos-like filter and sharpening, designed for tile-based GPUs. SGSR 2 is a temporal upscaler aimed at beating TAAU on quality without the compute and bandwidth cost of a desktop FSR 2 port. Qualcomm's constraint is Adreno's tiled architecture: extra full-screen compute passes are expensive, so the first version of SGSR fits in one shader pass.
- Arm ships [Accuracy Super Resolution](https://developer.arm.com/mobile-graphics-and-gaming/arm-accuracy-super-resolution) (ASR), an open temporal upscaler derived from AMD FSR 2 and rewritten for mobile bandwidth. It is API-agnostic (Vulkan, GLES, and others), has Unreal Engine plugins, and has production use in titles such as Fortnite. Arm's pitch is practical: drop internal resolution, keep high-end features enabled, and reduce skin temperature. For older devices, Arm still recommends a spatial fallback. Neural Super Sampling is the follow-on path for SoCs with dedicated neural hardware.
- MediaTek [HyperEngine](https://www.mediatek.com/technology/hyperengine-6) treats frame-rate stability as a system problem: CPU, GPU, and NPU scheduling, a frame-rate smoother, variable rate shading, and adaptive quality. Interpolation and super resolution appear as SoC features rather than as a public game SDK with the same visibility as SGSR or ASR.
- Apple [MetalFX](https://developer.apple.com/documentation/metalfx) is the platform API on iPhone, iPad, and Mac. It ships a frame interpolator across Apple platforms, plus spatial and temporal upscalers, dynamic-resolution inputs, reactivity hints, and a neural denoised upscaler that can consume noisy 1 spp frames. MetalFX is not a PC-style brand war. If a game is on Apple silicon, this is the reconstruction API.
- Huawei is the OEM that looks most like a platform vendor. GPU Turbo X rebuilds the system graphics path for load and thermals. AI MEMC interpolation exists as a system feature. On HarmonyOS, [XEngine Kit](https://developer.huawei.com/consumer/cn/sdk/xengine-kit/) exposes spatial GPU super resolution, spatial AI super resolution, and temporal AI super resolution on Maleoon GPUs, plus adaptive VRS. Because Huawei owns the SoC, the GPU, and the OS, reconstruction can be a developer API on Maleoon rather than only a post-process chip.
- Xiaomi (and Redmi) started on Pixelworks X7, then moved to in-house 狂暴游戏独显 chips D1 and D2. D2 runs concurrent dynamic interpolation and a pretrained AI super-resolution model, with HDR, advertised as working on all games rather than a short whitelist. A distinctive extra is touch-aware interpolation: finger motion is fed into the MEMC pass to reduce broken frames during camera flicks. HyperOS Game Turbo is the user switch. Later Redmi gaming phones push the overlay to 144 Hz or 165 Hz panels.
- OPPO (and OnePlus) ships ColorOS [超帧超画](https://www.coloros.com/topic/coloros-performance/) (super frame and super image). A dedicated render chip interpolates up to 120 FPS from 60 and upscales the 3D buffer, claimed on 100+ titles with no per-game port. HDR conversion sits in the same engine. Later, OPPO's 风驰游戏内核 (Fengchi game kernel) moved part of that work back onto the SoC: it rewrites Android scheduling and the GPU render pipeline so "native-grade" FG and SR can run together on the Adreno path, with less extra latency than a post-GPU display chip. OnePlus Ace 5 Pro was the first to advertise Genshin Impact at 120 FPS and 1080p SR concurrently on that kernel.
- vivo (and iQOO) put the same split on a self-designed Q-series display chip. Q3 is marketed with [游戏插帧](https://developers.vivo.com/product/d/gameFrameInsertion) targeting 120–144 FPS, plus QNSS, a temporal [游戏超分](https://developers.vivo.com/product/d/gameSuperScore) path vivo compares to DLSS 2, and a chip-side ray-tracing assist. vivo also ships a graphics SDK and Unreal/Unity plugins through [VGS](https://developers.vivo.com/) (vivo Gaming Services), so a studio can pick quality or performance mode instead of relying only on the OS overlay.
- Honor kept GPU Turbo X after the split from Huawei and added [幻影引擎](https://www.honor.com/cn/phones/honor-gt-pro/) (Phantom Engine). MagicOS Game Manager exposes frame-rate enhancement (MEMC, either a virtual interpolator or a companion display chip by SKU), 幻影稳帧 (Phantom stable frame), which predicts stutters and inserts virtual frames with async compute, and image-quality enhancement (super resolution). Honor's own motion-estimation, semantic-segmentation, and motion-compensation stack is the interpolator. Interpolation plus super-resolution is sold as a PC-like image-quality engine on 100+ titles.

Engine plugins still decide the high-quality path. Unreal and Unity plugins from Qualcomm, Arm, Apple, vivo, and Huawei are how a title ships reconstruction with real motion vectors. Many games still use simple spatial upscaling or TAAU because temporal SR needs jitter, velocity, and a separate UI pass. OEM Game Space overlays cover the rest, at the cost of treating the framebuffer as video.

### What the two markets do not share

PC reconstruction is an image-quality race at 1440p and 4K, with path tracing as the stress test and 240 Hz monitors as the FG target. Multi-frame generation, neural denoisers, and transformer upscalers are PC-first. The overlay path exists, but it is a compatibility net, not the flagship feature.

Mobile reconstruction is a power race. Tile-based deferred GPUs, smaller caches, and a shared thermal budget with the CPU and NPU make desktop FSR 2 or DLSS ports a poor default. Frame generation is usually 2x to a 90, 120, or 144 Hz panel, not 4x or 6x to a 360 Hz monitor. Input latency and touch prediction are as important as optical flow quality. Spatial one-pass upscalers still matter on mid-range devices.

The OEM flagship stack is the piece PC does not have as a product category. A companion chip plus Game Space gives interpolation and SR to games that never integrated. That is closer to driver FG than to DLSS, until the OEM ships a real SDK (XEngine, VGS) or moves the work back onto the SoC (Fengchi, Maleoon).

Handheld PCs (Steam Deck, ROG Ally, and similar) sit on the PC side of the SDK map but the mobile side of the power map. They run DLSS, FSR, or XeSS, then hit a fan and battery wall that looks more like a phone.

## From samples to displayed frames

Both FG and SR are reconstruction problems. The renderer produces a sparse signal. The reconstruction stage estimates the missing samples without re-shading the scene. Where that stage runs — inside the engine, in the GPU driver, or on a display chip — decides which samples it can see.

### Frame generation

FG estimates a frame at time `t+0.5`, or several frames between `t` and `t+1`.

The in-engine pipeline is:

1. Render frame N and frame N+1 (already upscaled).
2. Build a motion field from engine motion vectors plus an optical-flow pass for pixels the engine does not track.
3. Warp both frames toward the interpolation time and blend them.
4. Detect disocclusion (regions revealed by motion) and fill them from the nearer real frame or from a neural inpainter.
5. Composite UI after interpolation, or warp UI with a separate HUD path.
6. Pace presents so the generated frames are evenly spaced on the display.

Optical flow is the hard part. Engine motion vectors are exact for rigid and skinned meshes, but wrong for reflections, shadows, particles, and water. Desktop FG therefore runs a dense flow network or a hardware flow unit (NVIDIA Optical Flow) on the color buffers. In-engine mobile FG (AFME, MetalFX interpolator) has to be cheaper: more dependence on engine velocities, coarser flow, and often 2x only.

Overlay FG skips step 2's engine velocities. Pixelworks MotionEngine, ColorOS super frame, vivo interpolation, Xiaomi D1/D2, Honor frame-rate enhancement, and PC driver FG all estimate motion from color, the way a TV does. Xiaomi feeds touch events into that estimate so a flick does not tear the warp. Honor runs semantic segmentation so UI and characters can be compensated differently. These tricks help, but they are still guessing.

Multi-frame generation repeats the warp at several timestamps. DLSS 4.5 can go to 6x, XeSS 3 to 4x on Intel, and AMD FG 4 stays at high-quality 2x interpolation on RDNA 4 while keeping FSR 3-style generation for older GPUs. Phone overlays stay near 2x, occasionally stretching a 60 FPS game across a 144 Hz or 165 Hz panel. Higher multipliers make pacing and latency the real problem. If the game simulates at 30 Hz and the display shows 180 Hz, the extra frames are cosmetic. The player still waits for the next real frame before the world and the input sample change. That is why Reflex, Anti-Lag 2, XeLL, and mobile touch prediction are part of the product, not extras.

OPPO's Fengchi path is the mobile attempt to close this gap: generate the extra frame on the SoC with less added delay than a post-GPU chip. Huawei's XEngine path is the other attempt: give the game an API so reconstruction is not a compositor surprise.

Generated frames fail in predictable ways: around sudden camera cuts, first-person weapon motion, fine particle effects, and any HUD drawn into the 3D buffer. Overlay FG fails more often, because it cannot hide the HUD. Debug views that show flow, disocclusion, and UI masks are part of a serious in-engine integration. Game Space has no equivalent, which is why OEMs keep a per-title whitelist even when they advertise "all games."

### Super resolution

Production SR falls into three algorithm families. Each family can ship as an in-engine pass or as an overlay.

**Spatial upscalers** take only the current color buffer. FSR 1, SGSR 1, MetalFX spatial scalers, Huawei's spatial GPU SR, and most Game Space "super image" toggles are in this family. They use an edge-aware filter, often Lanczos or EASU-like, plus a sharpening pass. They need no motion vectors, so integration is easy and latency is low. Quality collapses at large scale factors: thin geometry aliases, and there is no history to recover it.

**Temporal upscalers** jitter the camera with a Halton or similar sequence, render at a lower internal resolution, and accumulate samples into a history buffer. Inputs are color, depth, motion vectors, and exposure. The algorithm reprojects last frame's history, rejects samples that fail a neighborhood test (ghosting protection), and blends in the new jittered color. FSR 2, Unreal TSR, Arm ASR, SGSR 2, MetalFX temporal scalers, and Huawei's temporal AI SR are this family. Quality depends on motion-vector honesty. Particles, transparencies, and animated textures do not have geometry motion, so engines pass a reactive or transparency mask. Overlay SR cannot do this well unless the OEM SDK receives those buffers.

**Neural upscalers** replace hand-written history filters with a trained model. DLSS Super Resolution, FSR Upscaling 4, XeSS-SR, MetalFX's neural path, vivo QNSS, and Xiaomi D2's pretrained model take color — and, when available, depth and velocity — and infer the high-resolution frame. CNNs were the first generation. Transformers, used in later DLSS models, give a wider context window and better edges under motion. These models want matrix hardware: Tensor Cores, XMX, WMMA, a Neural Engine, an NPU, or a display-chip AI block. Without it, vendors either refuse to run or fall back to a cheaper compute path (DP4a for XeSS, FSR 3 for AMD, spatial SR on a phone overlay).

A typical **in-engine** SR integration looks the same across vendors.

1. Jitter the projection each frame.
2. Render opaque geometry at internal resolution, writing color, depth, and velocity.
3. Mark reactive pixels (hair, particles, alpha).
4. Run SR to the output resolution.
5. Render UI, HUD, and text at output resolution so they do not get temporally filtered.

A typical **overlay** SR integration is shorter, and worse.

1. The game renders at whatever resolution it already chose, including UI.
2. The display chip or OS compositor upscales the whole framebuffer.
3. Optional HDR remapping runs on the same chip.

Quality modes on PC (Quality, Balanced, Performance, Ultra Performance) are just scale factors. 1.5x per axis is about 44% of the pixels; 2x is 25%; 3x is about 11%. Temporal and neural methods can look close to native at 1.5x to 2x. Spatial methods, including most phone overlays, should stay near 1.5x.

### Why mobile implementations look different

A 12 nm companion chip exists first for MEMC: it interpolates extra frames off the 3 nm GPU so the SoC can shade less and stay cooler. Super resolution often rides the same chip. The extra hop costs latency. That is the bargain Fengchi and Maleoon are trying to undo by pulling reconstruction back onto the SoC.

Tile-based GPUs keep a tile on-chip, then write it out. A desktop-style SR that reads full-screen history, motion, and color in compute passes can blow the bandwidth budget even if the ALU cost looks fine. Hence SGSR 1's one-pass design, ASR's FSR 2 subset with fewer resources, and the advice to keep a spatial fallback for older phones.

Power is the other limiter. SR that saves 40% of GPU time is useful only if the extra history traffic does not eat the savings. On phones the success metric is often skin temperature and battery minutes at 60 FPS, not a 1% low at 4K.

Resolution policy also differs. PC SR targets a fixed output (1440p or 4K) from a chosen quality mode. Mobile SR often sits under a dynamic-resolution controller: internal size moves every frame to hold a frame-time budget, which is why MetalFX grew dynamically sized inputs. OEM overlay SR does not control that inner size. It upscales whatever the game already produced, so a title that never lowered its render scale gains sharpness, not frame time.

### Shared failure modes

- Bad motion vectors produce ghosting. This is the first integration bug on every in-engine path. Overlay paths invent their own vectors and fail the same way, just without a debug view.
- History rejection that is too aggressive flickers; too weak trails.
- UI interpolated or upscaled with the 3D scene becomes blurry or smeared. Draw it after FG and after SR. Overlay stacks cannot, unless the OEM SDK splits the HUD.
- FG without a latency path feels worse than a locked 60 FPS. Native 120 FPS plus overlay interpolation can also fight the compositor and stutter.
- High FG multipliers on a low rendered frame rate are a smoothness trick, not a simulation upgrade. That is true for DLSS 6x and for a 60-to-165 Hz phone overlay.

## Where the stack is heading

Reconstruction is no longer a post-process. It is becoming the renderer.

The PC trend is a neural frame. Extra frames, super resolution, ray denoising, and now radiance caches or ray regeneration are trained models sitting on matrix hardware. DLSS 4.5, FSR Redstone, XeSS 3, and MetalFX denoising all point the same way: render fewer frames, fewer pixels, and fewer paths, then infer the rest. Multi-frame generation will keep climbing until latency and artifact rate, not GPU throughput, set the cap. Dynamic multipliers that track refresh rate are the practical compromise. Driver overlays will remain for old titles.

The mobile SoC trend is the same math under a power envelope. In-engine frame generation on phones will stay closer to 2x, because touch latency is harsher than mouse latency and because 120 Hz is already the panel ceiling for most devices. Temporal SR is moving from flagship-only to a standard Unreal plugin. Neural super sampling will land where NPUs or GPU matrix units are idle enough to beat a hand-written temporal filter on joules per frame, not just on SSIM.

The mobile OEM trend is a migration, not a new idea. Pixelworks-style companion chips proved that overlay MEMC and SR sell. vivo Q chips and Xiaomi D chips brought the model in-house and added touch-aware flow, AI SR, and HDR. The next step is the one OPPO and Huawei are already advertising: pull reconstruction onto the SoC and expose a developer API, so the overlay is only the compatibility net. Honor's Phantom Engine sits in the middle, mixing compositor virtual frames with a super-image toggle. Coverage ("all games") and quality (engine buffers) will keep sitting in tension.

Four engineering problems are still open.

- **One integration, many backends.** Games do not want four upscaler plugins on PC, and they do not want SGSR, ASR, MetalFX, XEngine, and a Q-chip SDK on mobile either. DirectSR, Streamline, and engine abstractions are the start of a common input contract: jitter, color, depth, velocity, reactive masks. The backend can stay vendor-private. OEM overlays that never ask for those inputs will stay the low-quality default.
- **Honesty about latency.** Display FPS and simulation Hz must be shown as different numbers. Competitive modes will keep FG off or at 2x. Single-player path tracing will keep stacking high-ratio FG and SR. Game Space 120 FPS from a 60 FPS game should be labeled as interpolation.
- **Generated content that the engine does not know about.** Neural frames can invent specular detail and extra motion that the simulation never produced. That is fine for presentation. It is not fine for hit detection, animation, or photo-mode correctness. Overlay MEMC makes the same lie with worse data. The rendered frame remains the ground truth.
- **Where the silicon lives.** Display chips win on coverage and thermals. SoC GPU and NPU paths win on latency and engine inputs. The likely phone is both: an overlay for unadapted titles, and an in-engine path for the ten games that pay for an SDK.

The likely end state is a renderer that shades a coarse, noisy, low-rate signal, and a reconstruction model that emits the framebuffer. PC will get there first because of path tracing and 4K. Mobile will get a thinner version of the same pipeline, optimized for tiles, thermals, and a UI that must stay sharp under a thumb. OEMs will keep shipping the overlay until that in-engine path is as easy to turn on as Game Space.
