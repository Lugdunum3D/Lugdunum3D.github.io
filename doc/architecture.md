---
hash-prefix: 'f502d169_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: Architecture of Lugdunum
---

* This will become a table of contents (this text will be scraped).
{:toc}

# Architecture of Lugdunum

The purpose of this section is to introduce you to the internal operation of our 3D engine. We will first talk about the architecture of the renderer. Then we will describe the sequencing of the engine graphic's loop, how each component of the [`Renderer::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Renderer_1_1Target.html) is interacting with the [`Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) composed of different [`Renderer::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Renderer_1_1View.html). Then, we will discuss the GPU & CPU's side operation. We will explain how each buffer is loaded and used by our engine.

## Renderer Architecture

We decided to be as API independent as possible, i.e. we do not want to be too much dependent on Vulkan itself. This is why we created abstract classes for each type and their Vulkan-equivalent in a separate, API specific directory. This is especially visible in [here](#fig:renderer-classes). Hypothetically speaking, this allows us to be much less dependent on this technology and maybe one day, to derive the implementation for another low-level API, such as D3D12 for example.

The main object of the renderer is the [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html). A [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html) is any surface on which we can render, e.g. a window or an offscreen image.

A [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html) can have multiple [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html)s, each representing a fraction of the [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html), defined by a [`Render::View::Viewport`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View_1_1Viewport.html) and a [`Render::Scissor`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Scissor.html) defined as following:

``` cpp
class Viewport {
public:
    struct {
        float x;
        float y;
    } offset;

    struct {
        float width;
        float height;
    } extent;

    float minDepth;
    float maxDepth;

    inline float getRatio() const;
};

struct Scissor {
    struct {
        float x;
        float y;
    } offset;

    struct {
        float width;
        float height;
    } extent;
};
```

Each of the components of [`Render::View::Viewport`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View_1_1Viewport.html) and [`Render::View::Scissor`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View_1_1Scissor.html) are defined as percentage values (i.e. a float between 0.0 an 1.0), so it has the same appearance on every size of the [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html).

A unique [`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html) can be attached to a single [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html), i.e. we cannot have a [`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html) attached to two different [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html)s.

[`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html)s contain a [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) and have pointer to a [`Scene::Scene`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Scene_1_1Scene.html), which is created by the user, and can be attached to multiple cameras.

Every frame, the [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) is cleared, then filled by the [`Scene::Scene`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Scene_1_1Scene.html) with the objects visible by the [`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html)'s frustrum.

The [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) is finally sent to [`Vulkan::Render::Technique::Technique::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Technique_1_1Technique.html).

<figure id="fig:renderer-classes">
  <img src="assets/09326b6bf0b48a72d90bf07746a51492.svg" width="100%" class="figure"/>
  <figcaption class="caption">Main classes of the renderer</figcaption>
</figure>

In the diagram [here](#fig:renderer-classes), we are representing the main classes of the renderer and their dependencies.

-   Plain line ( ---&gt;): Inheritance
-   Dashed line ( - - - &gt;): Contains an instance of the class with ownership
-   Dotted line ( · · · &gt;): Contains an instance of the class without ownership

The diagram [here](#fig:renderer-view-usage) shows an example of how classes interact with each other:

-   Here we have one [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html), which contains three [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html)s:
    -   The render view A
    -   The render view B
    -   The render view C, which is *disabled*, as each one of these can be enabled and disabled as wished.
-   Both render views A and B each have a camera, and each camera has its own render queue.
-   Cameras are also linked to a scene, and scenes are linked to each camera's render queues.
-   In this particular case, it appears that we have only one scene, so each camera points to the same scene, and the scene points to two render queues.

<figure id="fig:renderer-view-usage">
  <img src="assets/cecb04274b2bd4491033449c2971693a.svg" width="100%" class="figure"/>
  <figcaption class="caption">Example of a possible usage of the render views</figcaption>
</figure>

## Sequence diagrams

In this section will be presented the rendering of a single frame with the help of two sequence diagrams, [here](#fig:seq-rendering-frame-1) and [here](#fig:seq-rendering-frame-2). The second is a subset of the first, as they have been separated to ease readability.

<figure id="fig:seq-rendering-frame-1">
  <img src="assets/2a6585cade20f1b8f2cd0ce92a6d1da1.svg" width="100%" class="figure"/>
  <figcaption class="caption">Rendering of a frame (part. 1)</figcaption>
</figure>

Let us describe this sequence diagram, step by step:

First, `UserApplication` is the user-defined class that inherits from [`lug::Core::Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) and defines the methods `onEvent` and `onFrame`. [`Application::run()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) is called (and must be) by the user like in this example:

``` cpp
int main(int argc, char* argv[]) {
    UserApplication app;
    
    if (!app.init(argc, argv)) {
        return EXIT_FAILURE;
    }
    
    if (!app.run()) {
        return EXIT_FAILURE;
    }
    
    return EXIT_SUCCESS;
}
```

The method [`Core::Application::run()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) is the main loop of the engine which polls the events from the window and renders everything correctly. As expected, we can see that the [`Core::Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) is polling all the events from the [`Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) and sending them to the `UserApplication` through the method [`UserApplication::onEvent(const lug::Window::Event& event)`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html).

Then, [`Core::Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) is calling the method [`Renderer::beginFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Renderer.html) which call itself the method [`Render::Window::beginFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) to notify the [`Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) that we are starting a new frame.

Finally, the user can update the logic of their application in the method [`UserApplication::onFrame(const lug::System::Time& elapsedTime)`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html).

At the end of the frame, the method [`Renderer::endFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Renderer.html) is called and will call the method [`Render::Target::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html) for all [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html) to draw and will finish the frame by calling the method [`Render::Window::endFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) to notify the [`Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Window.html) that we are ending this frame.

<figure id="fig:seq-rendering-frame-2">
  <img src="assets/9cae02fd75e31b202d68339e51f676ea.svg" width="100%" class="figure"/>
  <figcaption class="caption">Rendering of a frame (part. 2)</figcaption>
</figure>

In the method [`Render::Target::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html), the [`Render::Target`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Target.html) is calling the method [`Render::View::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html) for each enabled [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html).

To be rendered, [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html) needs to update its [`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html) which will fetch all the elements in its [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) from the scene with [`Scene::fetchVisibleObjects()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Scene_1_1Scene.html).

So the [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) will contain every elements needed to render the [`Scene::Scene`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Scene_1_1Scene.html), meshes, models, lights, etc.

Then the [`Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1View.html) can call the render technique to draw the the elements in the [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) (e.g. for Vulkan a class inheriting from [`Vulkan::Render::Technique::Technique`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Technique_1_1Technique.html)).

## Vulkan Rendering

### Global

#### GPU Side

The [`Vulkan::Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html) and the [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html)s of Lugdunum are pretty straightforward. For simplicity's sake we have split this process into five steps:

<figure>
  <img src="assets/e101a3845db70d2a4fee1f2c21292848.svg" width="100%" class="figure"/>
  <figcaption class="caption">Swapchain image acquisition and synchronization</figcaption>
</figure>

Each arrow represents a Vulkan semaphore for synchronization purpose.

1.  We get an available image from the swapchain
2.  We change the layout of this image to `VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL`
3.  We render each [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html) in parallel
4.  We change the layout of this image to `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR`
5.  We add the image to the presentation queue of the swapchain.

For steps 2 and 4 we are using one Vulkan command buffer per image in the swapchain. Each of the command buffers are built beforehand, therefore we don't need to rebuild them each frame.
Step 3 is dependent on the render technique used.

#### CPU Side

Since our semaphores are stored in a pool, we let each method ([`beginFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html), [`endFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html), ...) select their own semaphore(s) to use.

##### Steps 1 & 2

The method [`Vulkan::Render::Window::beginFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html) is used to accomplish steps 1 and 2. This method chooses one semaphore to be notified when the next image is available and chooses N semaphores to notify each [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html) when the image has changed layout. (N being the number of [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html) in the [`Vulkan::Render::Window`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html))

##### Step 3

The method [`Vulkan::Render::Window::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html) is used to accomplish step 3. This method uses the N previous semaphores, one for each call to [`Vulkan::Render::View::render()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html). Each [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html) has a semaphore which is signaled when the view has finished rendering. We will explain how the render technique works in the next part.

##### Steps 4 & 5

The method [`Vulkan::Render::Window::endFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Window.html) is used to accomplish steps 4 and 5. This method retrieves all the semaphores from the [`Vulkan::Render::View`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1View.html) and chooses one semaphore to be notified when the image has changed layout.

### Forward render technique

#### GPU Side

<figure>
  <img src="assets/7cfb41a875321b0c31703e00fbbb4c5b.svg" width="100%" class="figure"/>
  <figcaption class="caption">Forward technique</figcaption>
</figure>

The [`Vulkan::Render::Technique::Forward`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Technique_1_1Forward.html) has two different [`Vulkan::Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Queue.html), i.e. one transfer and one graphics.

The transfer [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) is responsible for updating the data of the [`Render::Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html) and [`Light::Light`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Light_1_1Light.html)s, each of which is contained in a uniform buffer [`Vulkan::API::Buffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1API_1_1Buffer.html) which is sent through different [`Vulkan::API::CommandBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1API_1_1CommandBuffer.html)s (i.e. "Command buffer A" and "Command buffer B" in the above schema).
These [`Vulkan::API::CommandBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1API_1_1CommandBuffer.html)s are then sent to the transfer [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html).

Here is the structure of the uniform buffers for the camera and the lights:

``` cpp
// Camera
layout(set = 0, binding = 0) uniform cameraUniform {
    mat4 view;
    mat4 proj;
};

// Directional light
layout(set = 1, binding = 0) uniform lightUniform {
    vec3 lightAmbient;
    vec3 lightDiffuse;
    vec3 lightSpecular;
    vec3 lightDirection;
};

// Point light
layout(set = 1, binding = 0) uniform lightUniform {
    vec3 lightAmbient;
    float lightConstant;
    vec3 lightDiffuse;
    float lightLinear;
    vec3 lightSpecular;
    float lightQuadric;
    vec3 lightPos;
};

// Spot light
layout(set = 1, binding = 0) uniform lightUniform {
    vec3 lightAmbient;
    vec3 lightDiffuse;
    vec3 lightSpecular;
    float lightAngle;
    vec3 lightPosition;
    float lightOuterAngle;
    vec3 lightDirection;
};
```

Each type of light has a different pipeline using different fragment shaders (That's why all the light uniforms are using the same binding point in the above code sample).

To pass the transformation matrix of the objects we are using pushconstant:

``` cpp
layout (push_constant) uniform blockPushConstants {
    mat4 modelTransform;
} pushConstants;
```

The graphics [`Render::Queue`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Queue.html) is responsible for all the rendering.

The "Command buffer C" for the drawing depends on the two command buffers of transfer by means of semaphores at different stages of the pipeline, `VK_PIPELINE_STAGE_VERTEX_INPUT_BIT` for the camera and `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT` for the lights.

#### CPU Side

##### Buffer Pool

The allocation of the uniform buffers is managed by a [`Vulkan::Render::BufferPool`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool.html), one for the camera and one for the lights.

As we do not want to perform lots of allocations, we mitigate this using the pool which will allocate a relatively large chunk of memory on the GPU, that will itself contain many [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html)s.

A [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html) is a portion of a bigger [`Vulkan::API::Buffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1API_1_1Buffer.html) that can be allocated and freed from the pool and bind with a command buffer without worrying about the rest of the [`Vulkan::API::Buffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1API_1_1Buffer.html).

##### Triple buffering

Because we are using triple buffering, we need a way to store data for a specific image. For that we have [`Vulkan::Render::Technique::Forward::FrameData`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1Technique_1_1Forward_1_1FrameData.html) that contains all we need to render one specific frame (command buffers, depth buffer, etc.). To avoid using a command buffer already in use, we are synchronizing their access with a fence.

To share [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html) across frames, e.g. if the camera does not move, we have a way to reuse the same [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html). We associate the [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html) with the object (camera or light), and test at the beginning of the frame if we can use a previous one (if the object has not changed from the update of this [`Vulkan::Render::BufferPool::SubBuffer`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool_1_1SubBuffer.html)).

If it is not possible to use a previously allocated buffer we are allocating a new one from the [`Vulkan::Render::BufferPool`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Vulkan_1_1Render_1_1BufferPool.html).

##### Drawing Command Buffer

Here is the pseudo code that we are using to build the command buffer of drawing:

``` pseudocode
BeginCommandBuffer

# The viewport and scissor are provided by the render view
SetViewport
SetScissor

BeginRenderPass

# We can bind the uniform buffer of the camera early
# It is the same everywhere
BindDescriptorSet(Camera)

# All the lights influencing the rendering (visible to the screen)
Foreach Light
    # Each type of Light has a different pipeline
    BindPipeline(Light)
    
    # We can bind the uniform buffer of the light
    BindDescriptorSet(Light)
    
    # All the objects influenced by the light
    Foreach Object
        # Push the transformation matrix of the Object
        PushConstant(Object)
        
        # We use indexed draw, so we need to bind
        # the index and the vertex buffer of the object
        BindVertexBuffer(Object)
        BindIndexBuffer(Object)
        
        DrawIndexed(Object)
    EndForeach
EndForeach

EndRenderPass

EndCommandBuffer
```
