---
title: Vulkan
date: 2024-04-16
weight: 2
---

## Coordinate system

The vertex is transformed from the model space to the world space, view space and projection space through the model matrix(M), view matrix(V) and projection matrix(P). Then it is converted to the normal device coordinate (NDC) space through perspective division, and finally to the two-dimensional screen space.

World space and view space have nothing to do with hardware.

The coordinate system of the NDC space is the right hand coordinate system, with the x axis to the right, the y axis down, and the z axis in the screen. The x axis of the screen space is to the right and the y axis is down.

The coordinate system of gltf and glm is the right hand coordinate system. However, glm projection make it left, we need invert elements (1, 1) in a projection matrix.

Matrices are column priority storage, and when used in shaders, you need to multiply the matrix left by a vector, such as P * V * M * x, where x is the vector.

## Mutiple threads

### Reference
- [Vulkan Multi-Threading](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/munich/mschott_vulkan_multi_threading.pdf)

## Synchronization

### Synchronization Within a Device Queue
Vulkan enables us to send command buffers into a queue to process graphics operations. This process is designed to be thread-friendly so we can submit work using command buffers from any CPU thread and they are eventually inserted into the same GPU queue.

Note that our commands can depend on the completion of other commands even within the same queue, and they do not need to be in the same command buffer. Commands are also guaranteed to start in the exact order they were inserted, but because they can run in parallel, there is no guarantee that the commands will complete in that same order.

The in-queue tools for synchronization that ensure these commands wait correctly on their dependencies are pipeline barriers, events, and subpass dependencies.

#### Pipeline barriers
The two types of barriers are:

- Execution barriers
- Memory barriers

We can create either an execution barrier, or an execution barrier and a number of memory barriers of one or more types in a single call.

Here is the pipeline barrier function for reference as we discuss parts of it below:

```cpp
void vkCmdPipelineBarrier(
   VkCommandBuffer                             commandBuffer,
   VkPipelineStageFlags                        srcStageMask,
   VkPipelineStageFlags                        dstStageMask,
   VkDependencyFlags                           dependencyFlags,
   uint32_t                                    memoryBarrierCount,
   const VkMemoryBarrier*                      pMemoryBarriers,
   uint32_t                                    bufferMemoryBarrierCount,
   const VkBufferMemoryBarrier*                pBufferMemoryBarriers,
   uint32_t                                    imageMemoryBarrierCount,
   const VkImageMemoryBarrier*                 pImageMemoryBarriers);
```

*Execution barriers*

When we want to control the flow of commands and enforce the order of execution using pipeline barriers, we can insert a barrier between the Vulkan action commands and specify the prerequisite pipeline stages during which previous commands need to finish before continuing ahead. We can also specify the pipeline stages that should be on hold until after this barrier.

These options are set using the **srcStageMask** and **dstStageMask** parameters of the **vkCmdPipelineBarrier**. Since they are bit flags, we can specify multiple stages in these masks. The **srcStageMask** marks the stages to wait for in previous commands before allowing the stages given in **dstStageMask** to execute in subsequent commands. For execution barriers, the **srcStageMask** is expanded to include logically earlier stages. Likewise, the **dstStageMask** is expanded to include logically later stages. The stages in the **dstStageMask** (and later) will not start until the stages in **srcStageMask** (and earlier) complete. This is sufficient to guard read accesses by stages in **srcStageMask** from write accesses by stages in **dstStageMask** .

To avoid a common pitfall, note that stage mask expansion is not applied to memory barriers defined below.

*Memory barriers*

To increase performance under the hood, Vulkan uses a series of caching mechanisms between the fast L1/L2 cache memory on the CPU and GPU cores and the relatively slow main RAM memory.

When one core writes to memory (to a render target, for example), the updates could still only exist in a cache and not be available or visible to another core ready to work with it. Memory barriers are the tools we can use to ensure that caches are flushed and our memory writes from commands executed before the barrier are available to the pending after-barrier commands. They are also the tool we can use to invalidate caches so that the latest data is visible to the cores that will execute after-barrier commands.

In addition to the pipeline stage masks specified for execution barriers, memory barriers specify both the type of memory accesses to wait for, and the types of accesses that are blocked at the specified pipeline stages. Each memory barrier below contains a source access mask (**srcAccessMask**) and a destination access mask (**dstAccessMask**) to specify that the source accesses (typically writes) by the source stages in previous commands are available and visible to the destination accesses by the destination stages in subsequent commands.

In contrast to execution barriers, these access masks only apply to the precise stages set in the stage masks, and are not extended to logically earlier and later stages.

There are three types of memory barriers we can use: global, buffer, and image. Each of these defines the accesses which will be ensured to be available (the source access by the source stage) and the stages and accesses types to which these accesses will be visible (the destination access by the destination stage).

- Global memory barriers are added via the **pMemoryBarriers** parameter and apply to all memory objects.
- Buffer memory barriers are added via the **pBufferMemoryBarriers** parameter and only apply to device memory bound to VkBuffer objects.
- Image memory barriers are added via the **pImageMemoryBarriers** parameter and only apply to device memory bound to VkImage objects.

```cpp

typedef struct VkMemoryBarrier {
   VkStructureType sType;
   const void* pNext;
   VkAccessFlags srcAccessMask;
   VkAccessFlags dstAccessMask;
} VkMemoryBarrier;

typedef struct VkBufferMemoryBarrier {
   VkStructureType sType;
   const void* pNext;
   VkAccessFlags srcAccessMask;
   VkAccessFlags dstAccessMask;
   uint32_t srcQueueFamilyIndex;
   uint32_t dstQueueFamilyIndex;
   VkBuffer buffer;
   VkDeviceSize offset;
   VkDeviceSize size;
} VkBufferMemoryBarrier;
​
typedef struct VkImageMemoryBarrier {
   VkStructureType sType;
   const void* pNext;
   VkAccessFlags srcAccessMask;
   VkAccessFlags dstAccessMask;
   VkImageLayout oldLayout;
   VkImageLayout newLayout;
   uint32_t srcQueueFamilyIndex;
   uint32_t dstQueueFamilyIndex;
   VkImage image;
   VkImageSubresourceRange subresourceRange;
} VkImageMemoryBarrier;

```

The parameters **srcQueueFamilyIndex** and **dstQueueFamilyIndex** are used to support sharing buffers or images among multiple device queues from different queue families, which requires additional care to synchronize memory accesses. For resources created with **VK_SHARING_MODE_EXCLUSIVE**, Queue Ownership Transfer barriers must be executed on both the source and destination queues. These barriers are similar to normal **vkCmdPipelineBarrier** operations, except the source stage and access mask part of the barrier is submitted to the source queue and the destination stage and access mask is submitted to the destination queue. Using **VK_SHARING_MODE_CONCURRENT** when creating buffers and images avoids the needs for these barriers, but usually results in worse performance.

#### Events
Another tool for synchronization in Vulkan is the event, which uses source stage masks and destination stage masks just like pipeline barriers, and can be quite useful when we need to specify and run parallel computation. The key difference between events and pipeline barriers is that event barriers occur in two parts. The first part is setting an event using **vkCmdSetEvent**, and the second is waiting for an event with **vkCmdWaitEvents**. Events synchronize the execution and memory accesses that occur before **vkCmdSetEvent** calls with execution and memory accesses which occur after the **vkCmdWaitEvents** calls; commands that occur between **vkCmdSetEvent** and **vkCmdWaitEvents** are unaffected by the event. While events can be set both from the GPU in command buffers, and from the host, we will only discuss the GPU side event setting here.

An event is set by calling **vkCmdSetEvent** with a **stageMask** parameter that marks the stages to wait for in previous commands before signalling the event. The **vkCmdWaitEvents** call takes nearly identical parameters to the pipeline barrier parameters, with the meaning of the **srcStageMask** altered to be the union of the **stageMask** of all events in **pEvents**. Without memory barriers, the **vkCmdWaitEvents** causes the **dstStageMask** stage execution in subsequent commands to wait until all events in **pEvents** signal. This creates an execution barrier between the **stageMask** stages of each of the given events for commands prior to that **vkCmdSetEvent** with the **dstStageMask** stage in subsequent commands.

When present, the optional memory barriers function very similarly to pipeline barriers. **srcStageMask** and **srcAccessMask** combine to define the memory accesses to be complete, available, and visible to those matching **dstStageMask** and **dstAccessMask** in subsequent commands, creating a memory barrier with those accesses. However, the accesses guaranteed to be available and visible are limited to those matching **stageMask** in commands prior to each **vkCmdSetEvent** as well as being included in **srcStageMask** and **srcAccessMask** in **vkCmdWaitEvents**.

As with pipeline barriers, stage masks are expanded for execution barriers but not for memory barriers.

By interleaving groups of unrelated work, we can reduce device stall time with events. Assuming the work between the Set/Wait pairs is sufficient to cover the write retirement and cache flushing, the stalls at wait time are minimized, while still guaranteeing synchronized memory accesses.

Between each signalling of an event, the event must be reset with **vkCmdResetEvent**. The reset must be synchronized with both the **vkWaitEvents** that precedes it and the **vkCmdSetEvent** that follows. Please see Events in the Vulkan Specification for full details.

#### Subpass dependencies
One more way to synchronize within the device queue is through subpass dependencies. These are similar to pipeline barriers, but are used specifically to express dependencies between render subpasses and between commands within a renderpass instance and those outside it, either before or after.

These can be a bit tricky because they come with many restrictions. But they can come in handy if we’re working with data across render passes, like when rendering shadows or reflections, or if we need to wait on an external resource or event.

When using subpass dependencies, there are some things we want to keep in mind:

- They contain only a single memory barrier for attachments specified with **srcAccessMask** and **dstAccessMask**.
- Subpasses can wait to complete using pipeline stage flags **srcStageMask** and **dstStageMask**.
- They can only make forward progress, meaning a subpass can wait on earlier stages or the same stage, but cannot depend on later stages in the same render pass.

### Synchronization Across Multiple Device Queues
we can orchestrate synchronization across different queues. The Vulkan API provides two options, each with different purposes: semaphores and fences. Note that Vulkan 1.2 introduced timeline semaphores, which is the new and preferred approach to semaphores going forward. They aren’t widely available yet on mobile, so we'll talk about the original approaches first, then look at how the new timeline semaphores design is able to replace both of the original options.

#### Semaphores
Semaphores are simply signal identifiers that indicate when a batch of commands has been processed. When submitting a queue with **vkQueueSubmit**, we can pass in multiple semaphores as a parameter.

The key to understanding and using semaphores is recognizing that they are for synchronizing solely between GPU tasks, especially across multiple queues, and not for synchronizing between GPU and CPU tasks.

If multiple commands are busy crunching away on their tasks across cores and threads, a semaphore is like an announcement that a team of commands has finished. Semaphores are signaled only after all commands in the batch are complete. They make implicit memory guarantees, so we can access any memory following the semaphore without needing to think about adding memory barriers between them.

#### Fences
Fences are pretty straightforward: while semaphores were built for synchronizing GPU tasks, fences are designed for GPU-to-CPU synchronization.

Fences can attach to a queue submission and allow the application to check a fence status using **vkGetFenceStatus** or to wait for queues to complete using **vkWaitForFences**.

Fences make the same implicit memory guarantee as semaphores, and if we want to present the next frame in a swap buffer, we can use fences to know when to swap and start the render of the next frame.

#### Timeline semaphores
The new timeline semaphore approach comes with various advantages because it’s flexible and works as a superset for both semaphores and fences while allowing signaling between GPU and CPU in both directions. Not only can we wait on semaphores from the application on the CPU, we can even signal a semaphore to the GPU! And while fences only work at the coarse queue submission level, timeline semaphores have finer granularity for more flexibility.

The way it works is very clever: it uses an integer counter, which each semaphore signals to increment upon completion, as a signal timeline.

### Reference
- [Understanding Vulkan Synchronization](https://www.khronos.org/blog/understanding-vulkan-synchronization)
- [Vulkan Timeline Semaphores](https://www.khronos.org/blog/vulkan-timeline-semaphores)
- [Yet another blog explaining Vulkan synchronization](https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/)
- [Keeping your GPU fed](https://www.youtube.com/watch?v=iZ3J25qsacA)
- [Vulkan Guide](https://github.com/KhronosGroup/Vulkan-Guide/blob/main/chapters/synchronization.adoc)
- [Synchronization Examples](https://github.com/KhronosGroup/Vulkan-Docs/wiki/Synchronization-Examples)
- [Vulkan Synchronization explanation](https://zhuanlan.zhihu.com/p/350483554)

## Loader

### Windows layer
Registry Editor stores the vulkan explicit and implicit layers in the Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Khronos\Vulkan

### Reference
- [Vulkan Layer Factory](https://github.com/LunarG/Vulkan-Layer-Factory)
- [Vulkan Loader](https://github.com/KhronosGroup/Vulkan-Loader)