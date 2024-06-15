# Develop OpenXR apps on macOS with Meta XR Simulator

Current build: *v66.0.0-beta.3*

> **WARNING**: The macOS support of Meta XR Simulator is currently in developer preview. Both the functionality and distribution solution may be subject to changes before production. For feedback and suggestion, please send email to xrsim-mac-feedback@meta.com.
>
> **Apple Silicon Mac** is required. Intel-based Mac is not currently supported.

## Prerequisites

### Vulkan SDK 
Following the [official instruction](https://vulkan.lunarg.com/doc/sdk/latest/mac/getting_started.html) to install Vulkan SDK:
* Download the Vulkan SDK from the Download Site
Open the downloaded *vulkansdk-macos-v.w.xx.0.dmg*, double click “InstallVulkan” to launch the installer
* During installation, enable **System Global Installation** option, which updates Vulkan Loader and MoltenVK libraries in `/usr/local`

### Homebrew
Install Homebrew following the instruction on https://brew.sh/ 

### (Optional) CMake
You would need CMake for building apps using OpenXR SDK. To install CMake, run `brew install cmake` from Terminal.

### (Optional) Android Device Bridge (adb)
To use **Data Forwarding** (which enables you to use the physical Quest Touch Controller with XR Simulator), you need to have `adb` in your `PATH` environment variable. `adb` is a part of Android SDK Platform Tools and can be installed with **Android Studio** from its [official website](https://developer.android.com/studio).

### (Optional) Meta Quest Developer Hub
While not essential, it's recommended to use the **Meta Quest Developer Hub** in conjunction with XR Simulator to manage your Quest headset when using **Data Forwarding**. It allows you to disable the Proximity Sensor and launch the Data Forwarding Server with just a few clicks. You can install it from the [official website](https://developer.oculus.com/meta-quest-developer-hub/).

## Install and Use Meta XR Simulator on macOS

### Installation

Run the following commands to install Meta XR Simulator:

```bash
brew tap Oculus-VR/tap
brew install meta-xr-simulator
```

If Meta XR Simulator is already installed, you may run the following commands to upgrade:

```bash
brew update
brew upgrade meta-xr-simulator
```

After install/upgrade Meta XR Simulator, run the following command to complete the initial setup (as prompted in Homebrew output):

```bash
sudo /opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/post_installation_macos.sh
```

The script will remove the quarantine protection from Meta XR Simulator binaries, and create/modify the symbol link `/usr/local/share/openxr/1/active_runtime.json` to set Meta XR Simulator as the default OpenXR runtime to the system. 

> Note 1: If you want to switch between multiple OpenXR runtimes, on top of modifying the global symbol link, you can also set `XR_RUNTIME_JSON` environment variable before running your OpenXR app. It can be used in absence of, or override the OpenXR runtime set through `active_runtime.json`.

> Note 2: If you forgot which folder has the Meta XR Simulator, you can run `brew info meta-xr-simulator` to obtain that. Homebrew should install it under `/opt/homebrew/Cellar/meta-xr-simulator/` folder, on Apple Silicon Macs.

### Launch

With Meta XR Simulator be setup as the active OpenXR runtime, it will automatically be launched when you are running an OpenXR app. Specifically, you will see its debug window be opened when `xrCreateSession()` is called.

You can either use a game engine that supports OpenXR on Mac, or follow the subsequent sections to build a native C++ app using the OpenXR SDK.

### Synthetic Environment Server

Synthetic Environment Server simulates the physical environment for mixed reality OpenXR apps. By launching it before Meta XR Simulator starts, the mixed reality app can access passthrough, anchor, or scene data through the corresponding OpenXR extensions.

To launch Synthetic Environment Server, use one of the scripts under `/opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/synth_env_server`. 

For example, if you want to use a simulated living room, run
```
/opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/synth_env_server/LaunchLivingRoom.sh
```

You should not launch more than one Synthetic Environment Server at the same time. All Meta XR Simulator instances will share the same synthetic environment, to simulate a co-location multiplayer experience.

### Data Forwarding

Data Forwarding lets you control the Meta XR Simulator with real Meta Quest controllers by connecting a Quest headset to your computer without wearing it. With this feature, you will find it easier to provide complex input to your application running in the simulator.

To install Data Forwarding Server to your Quest, run
```
adb install /opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/data_forwarding_server/XrSimDataForwardingServer.apk
```

Check our [documentation page](https://developer.oculus.com/documentation/unity/xrsim-data-forwarding) for more details.

## Develop OpenXR app using OpenXR SDK

### Download OpenXR-SDK-Source

Clone or download OpenXR SDK from https://github.com/KhronosGroup/OpenXR-SDK-Source

### Generate and open Xcode project

Assuming you already have the latest Xcode installed, you can use the following commands to create the Xcode project:

```bash
mkdir -p build/macos
cd build/macos
cmake -G "Xcode" ../..
```

Please check [BUILD.cs](https://github.com/KhronosGroup/OpenXR-SDK-Source/blob/main/BUILDING.md) of OpenXR SDK for further details.

**Please verify if Vulkan SDK is correctly detected when running `cmake`. You should see the following lines in the output:**
```
-- Found Vulkan: /usr/local/lib/libvulkan.dylib (found version "1.3.275") found components: glslc glslangValidator
-- Enabling Vulkan support
```
However, if you see `-- Could NOT find Vulkan (missing: Vulkan_LIBRARY Vulkan_INCLUDE_DIR) (found version "")` in the output, please fix your Vulkan installation before proceed.

### Build and run hello_xr

Following the steps to build and launch **hello_xr** from Xcode:
* Open build/macos/OPENXR.xcodeproj in Xcode
* In the Targets dropdown, choose **hello_xr**
Open the Targets dropdown again, click "Edit Scheme …"
* Set **-g vulkan2** in “Arguments Passed on Launch”
* If the active OpenXR runtime hasn’t been set, set `XR_RUNTIME_JSON` to `/opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/meta_openxr_simulator.json` in “Environment Variables”
* Click **Play** button
> Note: Although both `XR_KHR_vulkan_enable` and `XR_KHR_vulkan_enable2` are supported by Meta XR Simualtor, The former is not compatible with MoltenVK. Thus, you may get `ERROR_INCOMPATIBLE_DRIVER` if setting the graphics plugin to "vulkan", instead of "vulkan2".

## Develop OpenXR app using Unity

You need a recent Unity editor build (e.g. latest 2022.3 LTS) for Apple Silicon and **OpenXR Plugin** to use Meta XR Simulator on Mac.

Here shows are setup steps to enable OpenXR to a Unity Project:

0. Refer to the instruction above to install **Vulkan SDK** and **Meta XR Simulator**
1. In Unity Package Manager, install **XR Plugin Management** from Unity Registry.
2. Before macOS is supported in a future **OpenXR Plugin** production release, you need to install an expermental version of that. Click "+" and choose "Add package by name". Input **com.unity.xr.openxr** as the name and **1.11.1-exp.1** as the version, and click "Add" button.
3. After the installation of **OpenXR Plugin**, restart Unity editor when prompted for enabling the new Input System.
4. In **Project Settings** / **XR Plug-in Management**, check **OpenXR** in the first (Standalone) and last (Android) panels.
5. Go to the first panel of **XR Plugin-in Management** Settings, click the "Warning" sign besides OpenXR, and click the "Fix All" button.
6. Click "Edit" button of the "at least one interaction profile must be added" message, it brings you to the **OpenXR Settings**, add "Oculus Touch Controller Profile" to "Enabled Interaction Profiles".
    * (Optional) If you haven't set Meta XR Simulator as the active OpenXR Runtime of the system, you can also set **Play Mode OpenXR Runtime** to `/opt/homebrew/Cellar/meta-xr-simulator/__VERSION__/meta_openxr_simulator.json` in this panel.
7. Click **Play** button of the Unity editor. It will launch Meta XR Simulator and open its Debug Window if everything is setup correctly.

You may then install the [Meta XR All-in-one SDK](https://assetstore.unity.com/packages/tools/integration/meta-xr-all-in-one-sdk-269657) (v66 or above), or [Unity XR Interaction Toolkit](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/index.html), to develop your OpenXR app using Unity.

## More information

Please check the Meta XR Simulator [documentation](https://developer.oculus.com/documentation/native/xrsim-intro/) to obtain further information.

## Known issues

* The performance of **Synthetic Environment Server** is much slower on macOS compared to its Windows build. It's an known issue and will be optimized later.
* **Vulkan** is the only graphics API supported on Mac currently. **Metal** graphics API will be enabled through a future Khronos OpenXR update.
* Unity **Oculus XR Plugin** doesn't support using an OpenXR runtime on Mac. If you are developing an Quest app, you may use "Oculus XR Plugin" on Android and "OpenXR Plugin" on Standalone. Meta XR SDK is compatible with both XR Plugins.