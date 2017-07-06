---
hash-prefix: '52fc6639_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: Quick Start
---

* This will become a table of contents (this text will be scraped).
{:toc}

# Quickstart

Welcome to Lugdunum's quickstart tutorial! Here we will show you through a number of examples how to use our 3D engine for your own projects.

As a reminder, please note that classes mentioned here are linked (in blue) to our [external Documentation](https://lugdunum3d.github.io/doc/doxygen/index.html).

We wish you a good reading!

## Building a project

To build the project with Lugdunum, you need a CMakeLists.txt like the following:

``` cmake
cmake_minimum_required(VERSION 3.1)

# project name
project(test_project)

# define the executable
set(EXECUTABLE_NAME "test_project_executable")

# find vulkan
find_package(Vulkan)

# Check only VULKAN_INCLUDE_DIR because the vulkan library is loaded at runtime
if (NOT VULKAN_INCLUDE_DIR)
    message(FATAL_ERROR "Can't find vulkan headers")
endif()

include_directories(${VULKAN_INCLUDE_DIR})

# find fmt
find_package(Fmt)

if (NOT FMT_INCLUDE_DIR)
    if (NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/../../thirdparty/fmt")
        message(FATAL_ERROR "Can't find fmt, call `git submodule update --recursive`")
    endif()

    set(FMT_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../thirdparty/fmt/include)
    message(STATUS "Found Fmt: ${FMT_INCLUDE_DIR}")
endif()

include_directories(${FMT_INCLUDE_DIR})

# find Lugdunum
find_package(LUG REQUIRED ${THIS_DEPENDS})

# source files of your application
set(SRC
    src/Application.cpp
    src/main.cpp
)

# include files of your application
set(INC
    include/Application.hpp
)

# create target
if(LUG_OS_ANDROID)
    add_library(${EXECUTABLE_NAME} SHARED ${SRC} ${INC})

    set(ANDROID_PROJECT_PATH ${CMAKE_SOURCE_DIR}/android/${EXECUTABLE_NAME})
    set(ANDROID_PROJECT_ASSETS ${ANDROID_PROJECT_PATH}/src/main/assets)
    set(ANDROID_PROJECT_SHADERS ${ANDROID_PROJECT_PATH}/src/main/shaders)
else()
    add_executable(${EXECUTABLE_NAME} ${SRC} ${INC})
endif()

lug_add_compile_options(${EXECUTABLE_NAME})

# use lugdunum
include_directories(${LUG_INCLUDE_DIR})
target_link_libraries(${EXECUTABLE_NAME} ${LUG_LIBRARIES})
```

## Creating a window

Let's see how simple it is to create a window with the Lugdunum Engine.

``` cpp
int main() {
    auto window = lug::Window::Window::create(800, 600, "Default Window", lug::Window::Style::Default);

    // ...

    return 0;
}
```

The first and second argument are the size of the window's width and height of the window.
The third argument is the name you wish to give to your application.
The fourth argument is the style of the window. It allows you to choose which decorations and features you wish to enable. You can use any combination of the following styles:

<table>
<colgroup>
<col width="42%" />
<col width="57%" />
</colgroup>
<thead>
<tr class="header">
<th>Style Flag</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>lug::Window::Style::None</code></td>
<td>No decoration at all</td>
</tr>
<tr class="even">
<td><code>lug::Window::Style::Titlebar</code></td>
<td>The window has a titlebar</td>
</tr>
<tr class="odd">
<td><code>lug::Window::Style::Resize</code></td>
<td>The window can be resized and has a maximize button</td>
</tr>
<tr class="even">
<td><code>lug::Window::Style::Close</code></td>
<td>The window has a close button</td>
</tr>
<tr class="odd">
<td><code>lug::Window::Style::Fullscreen</code></td>
<td>The window is shown in fullscreen mode</td>
</tr>
<tr class="even">
<td><code>lug::Window::Style::Default</code></td>
<td>It is a shortcut for <code>Titlebar | Resize | Close</code></td>
</tr>
</tbody>
</table>

## Handling events

Now that the window is created you may want to handle events.
To do this, it is as simple as looping while the window is open and retrieving events that have been received.

``` cpp
int main() {
    auto window = lug::Window::Window::create(800, 600, "Default Window", lug::Window::Style::Default);

    while (window->isOpen()) {
        lug::Window::Event event;
        while (window->pollEvent(event)) {

            // ...        
        }
        // ...
    }
}
```

``` cpp
while (window->isOpen()) {

}
```

This line ensures that our application keeps running while our window is open. If the window's state ever changes then our application ends the loop and ends.

``` cpp
lug::Window::Event event;
while (window->pollEvent(event)) {

}
```

To retrieve events we need a `Event struct` that we pass to `window->pollEvent(...)`.
Each time `pollEvent(...)` is called, the window returns the next available `Event` and discards it from its queue.
If there are no more events left to be handled, then it returns `false`.

### Window events

Here is a simple example where the user detects a [`lug::Window::Event::Type::Close`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Window_1_1Event_1_1Type_1_1Close.html) events and then call the window's `window->close()` function which ends our application by exiting the while loop.

**Note:** Even if you do not care about events, you still need an event loop to ensure that the window works as intended.

``` cpp
int main() {
    auto window = lug::Window::Window::create(800, 600, "Default Window", lug::Window::Style::Default);

    while (window->isOpen()) {
        lug::Window::Event event;
        while (window->pollEvent(event)) {

            if (event.type == lug::Window::Event::Type::Close) {
                window->close();
            }
            
        }
    }
}
```

### Keyboard events

This is pretty much the same example but in addition to detecting window events, the user also detects a keyboard event which leads to the same result.

``` cpp
int main() {
    auto window = lug::Window::Window::create(800, 600, "Default Window", lug::Window::Style::Default);

    while (window->isOpen()) {
        lug::Window::Event event;
        while (window->pollEvent(event)) {

            if (event.type == lug::Window::Event::Type::Close) {
                window->close();
            }
        
            if (event.type == lug::Window::Event::Type::KeyPressed && event.key.code == lug::Window::Keyboard::Key::Escape) {
                window->close();
            }
            
        }
    }
}
```

### Mouse events

Finally here is an example that includes handling mouse events.
In this example, clicking the left mouse button does not do anything, but we are sure you will be able to come up with some creative ways to use this!

``` cpp
int main() {
    auto window = lug::Window::Window::create(800, 600, "Default Window", lug::Window::Style::Default);

    while (window->isOpen()) {
        lug::Window::Event event;
        while (window->pollEvent(event)) {

            if (event.type == lug::Window::Event::Type::Close) {
                window->close();
            }
        
            if (event.type == lug::Window::Event::Type::KeyPressed && event.key.code == lug::Window::Keyboard::Key::Escape) {
                window->close();
            }
            
            if (event.type == lug::Window::Event::Type::ButtonPressed && event.button.code == lug::Window::Mouse::Button::Left) {
            // ...
            }
        }
    }
}
```

## Logger

### Initialization

First, you need to initialize an instance of Logger, with the name of the instance (you can choose any string you want here: it allows you to know where the logs come from in the future).

``` cpp
lug::System::Logger::makeLogger("myLogger");
```

When you have an instance of the logger, you have to attach a handler for each output to which you want to log. For example, if you need the standard output:

``` cpp
LUG_LOG.addHandler(lug::System::Logger::makeHandler<lug::System::Logger::StdoutHandler>("stdout"));
```

If you need to log on Android device:

``` cpp
LUG_LOG.addHandler(lug::System::Logger::makeHandler<lug::System::Logger::LogCatHandler>("logcat"));
```

Below is a table that describes what handlers are available to you and in which circumstances you may use them.

| Class         | Description     |
|---------------|-----------------|
| LogCatHandler | Android         |
| StdoutHandler | Standard output |
| StderrHandler | Error output    |
| FileHandler   | File            |

### Display a message Log

To use the logger, you have to use one of this following methods:

| Display message type  | Example usage                                  |
|-----------------------|------------------------------------------------|
| `debug`               | `LUG_LOG.debug("Debug info: {}", debugValue);` |
| `info`                | `LUG_LOG.info("Starting the app");`            |
| `warn` (warning)      | `LUG_LOG.warn("Something wrong happened");`    |
| `error`               | `LUG_LOG.error("An error occured");`           |
| `fatal` (fatal error) | `LUG_LOG.fatal("Fatal error");`                |
| `assrt` (assert)      | `LUG_LOG.assrt("Assert level logging");`       |

### Modules

The Vulkan API defines many different features (more information [on the Khronos group website](https://www.khronos.org/registry/vulkan/specs/1.0/html/vkspec.html#features-features)) that must be activated at the creation of the device.

Lugdunum abstracts these features with the use of *modules*.

Modules are a bunch of pre-defined set of mandatory features required by the renderer to run specific tasks.

For example, if you want to use tessellation shaders, you have to specify the module `tessellation` in the structure [`InitInfo`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Graphics_1_1InitInfo.html) during the initialization phase.

You can specify two type of modules: `mandatoryModules` and `optionalModules`. The former ensures the initialization fails if any mandatory module is not supported by the device, whereas the later treats them as optionals, as the name implies.

**Note:** You can query which optional modules are active with lug::Graphics::getLoadedOptionalModules()

### Choosing a device

Usually, calling [`Application::init()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html), lets Lugdunum decide which device to use. The engine prioritises a discrete device that supports all the mandatory modules. If the engine is unable to find such a device, the call fails.

Alternatively the user can decide which device he or she wishes to use, as long as the device meets the minimum requirements for all the mandatory modules. For that, you can use two others methods named [`Application::beginInit(int argc, char *argv[])`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) and [`Application::finishInit()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html).

Between the two you can set what we call *preferences*, and choose the device that you want that way.

``` cpp
if (!lug::Core::Application::beginInit(argc, argv)) {
    return false;
}

lug::Graphics::Renderer* renderer = _graphics.getRenderer();
lug::Graphics::Vulkan::Renderer* vkRender = static_cast<lug::Graphics::Vulkan::Renderer*>(renderer);

auto& chosenDevice : vkRender->getPhysicalDeviceInfos();

for (auto& chosenDevice : vkRender->getPhysicalDeviceInfos()) {
    if (...)
    {
        vkRender->getPreferences().device = chosenDevice; 
        break;
    }
}

if (!lug::Core::Application::finishInit()) {
    return false;
}
```

In the example shown above the user retrieves the list of all available device, decides which one they wishes to use and then sets that one as the device to use by default, setting it with the line `vkRender->getPreferences().device = chosenDevice;`.
If the device does not meet the minimum requirements [`Application::finishInit()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) fails.

## Using lug::Core::Application

In order to simplify the use of Lugdunum, the engine provides an abstract class named [`Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) that helps you get started quicker.

**Note:** The use of this class is not mandatory but strongly recommended

The first thing to do is to create your own class that inherits from it:

``` cpp
class Application : public ::lug::Core::Application {
    // ...
}
```

As [`Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) is an abstract class, you have to override two methods as well as the destructor:

``` cpp
void onEvent(const lug::Window::Event& event) override final;
void onFrame(const lug::System::Time& elapsedTime) override final;
~Application() override final;
```

Each time an event is triggered by the system, the method `onEvent()` is called. In this *callback* you can check the `event.type` which is referenced in the enum [`Window::Event::Type`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Window_1_1Event.html)

``` cpp
void Application::onEvent(const lug::Window::Event& event) {
    if (event.type == lug::Window::Event::Type::Close) {
        // ...
    }
}
```

The second method to override is [`Application::onFrame()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html). This method is called each loop, and you can do whatever pleases you in it, e.g. your application's logic. The `elapsedTime` variable contains the elapsed time since the last call to `onFrame()`.

``` cpp
void Application::onFrame(const lug::System::Time& elapsedTime) {
    _rotation += (0.05f * elapsedTime.getMilliseconds<float>());

    float x = 20.0f * cos(lug::Math::Geometry::radians(_rotation));
    float y = 20.0f * sin(lug::Math::Geometry::radians(_rotation));

    if (_rotation > 360.0f) {
        _rotation -= 360.0f;
    }

    auto& renderViews = _graphics.getRenderer()->getWindow()->getRenderViews();

    for (int i = 0; i < 2; ++i) {
        renderViews[i]->getCamera()->setPosition({x, -10.0f, y}, lug::Graphics::Node::TransformSpace::World);
        renderViews[i]->getCamera()->lookAt({0.0f, 0.0f, 0.0f}, {0.0f, 1.0f, 0.0f}, lug::Graphics::Node::TransformSpace::World);
    }
}
```

The constructor of [`Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) takes a structure `Info` defined as follow:

``` cpp
struct Info {
    const char* name;
    Version version;
};
```

You can use the initializer list to make it easier:

``` cpp
lug::Core::Application::Application{{"triangle", {0, 1, 0}}} {
    // ...
}
```

After the constructor phase, you have to call [`Application::init(argc, argv)`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html). This method processes two main initialization steps.

First, it will initialize `lug::Graphics::Graphics _graphics;` with these default values:

``` cpp
lug::Graphics::Graphics::InitInfo _graphicsInitInfo{
    lug::Graphics::Renderer::Type::Vulkan,      // type
    {                                           // rendererInitInfo
        "shaders/"                              // shaders root
    },
    {                                           // mandatoryModules
        lug::Graphics::Module::Type::Core
    },
    {},                                         // optionalModules
};
```

-   [`Renderer::Type`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Renderer_1_1Type.html) is set to `Vulkan` by default, as it the only supported renderer at this moment.
-   [`Module::Type::Core`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Module_1_1Type_1_1Core.html) defines basic default requirements for Vulkan (see [Modules](#modules))

Finally, it will initialize `lug::Graphics::Render::Window* _window{nullptr};` as [`Application`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html) manages itself all the window creation. It uses these default values:

``` cpp
lug::Graphics::Render::Window::InitInfo _renderWindowInitInfo{
    {                               // windowInitInfo
        800,                        // width
        600,                        // height
        "Lugdunum - Default title", // title
        lug::Window::Style::Default // style
    },

    {}                              // renderViewsInitInfo
};
```

You can also manually change these values:

``` cpp
getRenderWindowInfo().windowInitInfo.title = "Foo Bar";
```

**Warning:** You have to change these value before calling [`Application::init(argc, argv)`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Core_1_1Application.html)

### Camera

You use [`createCamera()`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Graphics.html) to create a camera and give it a name.
A [`Camera`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Render_1_1Camera.html) has the following attributes which can be changed via getters and setters. They essentially constitute the frustrum of the camera:

<table>
<colgroup>
<col width="24%" />
<col width="75%" />
</colgroup>
<thead>
<tr class="header">
<th>Attributes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Fov</td>
<td>Field of view</td>
</tr>
<tr class="even">
<td>Near</td>
<td>Near plane</td>
</tr>
<tr class="odd">
<td>Far</td>
<td>Far plane</td>
</tr>
<tr class="even">
<td>ViewMatrix</td>
<td>View matrix (computed from the previous attributes)</td>
</tr>
<tr class="odd">
<td>ProjectionMatrix</td>
<td>Projection Matrix (computed from the previous attributes)</td>
</tr>
</tbody>
</table>

``` cpp
// Create a camera
std::unique_ptr<lug::Graphics::Render::Camera> camera = _graphics.createCamera("camera");
```

### Movable Camera

Once the camera is created, you have to create a node from the scene in order to obtain a movable camera with a position.

``` cpp
// Add camera to scene
{
    std::unique_ptr<lug::Graphics::Scene::MovableCamera> movableCamera = _scene->createMovableCamera("movable camera", camera.get());
    _scene->getRoot()->createSceneNode("movable camera node", std::move(movableCamera));
}
```

### Lights

Our 3D engine has three different types of light:

<table>
<colgroup>
<col width="19%" />
<col width="80%" />
</colgroup>
<thead>
<tr class="header">
<th>Types of light</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Directional</td>
<td>Light that is being emitted from a source that is infinitely far away. All shadows cast by this light are parallel, an ideal choice for simulating sunlight.</td>
</tr>
<tr class="even">
<td>Point</td>
<td>Emits light from a single point, as a real-life bulb. It emits in all directions.</td>
</tr>
<tr class="odd">
<td>Spotlight</td>
<td>Emits light in a cone shape, as a flashlight, or a stage light.</td>
</tr>
</tbody>
</table>

Let us assume that you want to set a [`Directional`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Light_1_1Directional.html) light, here is a sample:

``` cpp
{
    std::unique_ptr<lug::Graphics::Light> light = _scene->createLight("light", lug::Graphics::Light::Light::Type::Directional);
    std::unique_ptr<lug::Graphics::Scene::Node> lightNode = _scene->createSceneNode("light node");

    light->setDiffuse({1.0f, 1.0f, 1.0f});
    static_cast<lug::Graphics::Light::Light*>(light.get())->setDirection({0.0f, 4.0f, 5.0f});

    lightNode->attachMovableObject(std::move(light));
    _scene->getRoot()->attachChild(std::move(lightNode));
}
```

Here is what you need to do, step by step:

1.  First create the light of type `Graphics::Light`, and give as first parameter of `createLight()` the name of the light, and as second parameter the type of light (cf. array above).
2.  Then create a [`lug::Scene::Node`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1Graphics_1_1Scene_1_1Node.html) which will be the movable object in the scene and attach the light to it so you can move the light in the scene.
3.  Now you can set the diffusion of the light.
4.  Next set the direction of the light. Be aware that you have to `static_cast<>` the pointer with the good type of light *(in this case `lug::Graphics::Light::Directional*`)*
5.  Finally attach the light to the scene.

### Handling Time

With Lugdunum, the time is in microseconds, and it is stored in a `int64_t`.
The [`Time`](https://lugdunum3d.github.io/doc/doxygen/classlug_1_1System_1_1Time.html) class represents a time period, the time that elapses between two event.

A Time value can be constructed from a microseconds source.

``` cpp
lug::System::Time time(10000);
```

You can get the time in different formats.

``` cpp
int64_t  timeInMicroseconds = time.getMicroseconds();
float    timeInMilliseconds = time.getMilliseconds();
float    timeInSeconds = time.getSeconds();
```
