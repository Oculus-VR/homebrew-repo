# Develop OpenXR apps on macOS with Meta XR Simulator

## Prerequisites

### Vulkan SDK 

Following the [official instruction](https://vulkan.lunarg.com/doc/sdk/latest/mac/getting_started.html) to install Vulkan SDK:
* Download the Vulkan SDK from the Download Site
Open the downloaded vulkansdk-macos-v.w.xx.0.dmg, double click “InstallVulkan” to launch the installer
* During installation, enable **System Global Installation** option, which updates Vulkan Loader and MoltenVK libraries in /usr/local

### Homebrew
Install Homebrew following the instruction on https://brew.sh/ 

### Cmake
Run `brew install cmake` from Terminal

## Use OpenXR on macOS

### Install Meta XR Simulator

Run the following commands to install Meta XR Simulator

```bash
brew update
brew tap Oculus-VR/tap
brew install meta-xr-simulator
```

Disable quarantine of `SIMULATOR.so`, so it can be loaded by another OpenXR app:

```bash
sudo xattr -d com.apple.quarantine /opt/homebrew/Cellar/meta-xr-simulator/64.0.0-alpha.1/SIMULATOR.so
```

It's recommended to set Meta XR Simulator as the active OpenXR Runtime for the system:

```bash
sudo mkdir -p /usr/local/share/openxr/1
sudo ln -s /opt/homebrew/Cellar/meta-xr-simulator/64.0.0-alpha.1/meta_openxr_simulator.json /usr/local/share/openxr/1/active_runtime.json
```

However, it's not necessary. You can also set `XR_RUNTIME_JSON` environment variable before running your OpenXR app. It can be used in absence of, or override the OpenXR runtime set through `active_runtime.json`:

```bash
# Add this line to ~/.zshrc or run it before launch an OpenXR app
export XR_RUNTIME_JSON=/opt/homebrew/Cellar/meta-xr-simulator/64.0.0-alpha.1/meta_openxr_simulator.json
```

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

> Note: please verify if Vulkan SDK is correctly detected when running `cmake`. You should see the following lines in the output:
```
-- Found Vulkan: /usr/local/lib/libvulkan.dylib (found version "1.3.275") found components: glslc glslangValidator
-- Enabling Vulkan support
```
> However, if you see `-- Could NOT find Vulkan (missing: Vulkan_LIBRARY Vulkan_INCLUDE_DIR) (found version "")` in the output, please fix your Vulkan installation before proceed.

### Build and run hello_xr

Following the steps to build and launch **hello_xr** from Xcode:
* Open build/macos/OPENXR.xcodeproj in Xcode
* In the Targets dropdown, choose **hello_xr**
Open the Targets dropdown again, click "Edit Scheme …"
* Set **-g vulkan2** in “Arguments Passed on Launch”
* If the active OpenXR runtime hasn’t been set, set `XR_RUNTIME_JSON` to `/opt/homebrew/Cellar/meta-xr-simulator/64.0.0-alpha.1/meta_openxr_simulator.json` in “Environment Variables”
* Click **Play** button

## More information

Please check the Meta XR Simulator [introduction](https://developer.oculus.com/documentation/native/xrsim-intro/) to obtain further information.