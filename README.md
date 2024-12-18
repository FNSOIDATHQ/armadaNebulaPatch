# Nebula Patch for Armada 2

## Catalog
* [Installation](#installation)
* [Quick Coder Development Guide](#quick-coder-development-guide)
* [Quick Common Modder Development Guide](#quick-common-modder-development-guide)
* [Known issues in shader+](#known-issues-in-shader)
* [Credits](#credits)
---

## Installation

Check NebulaPatchReadme.txt in zip for instructions.

## Quick Coder Development Guide

### overall

        Projects were created using codeblocks-20.03 and all code was written in C++.

        Read the comments in code for basic guidance.

### dllLoader

#### 1. compile
        It doesn't matter what compiler is used.
        I compiled the code using GCC, but only needed a little modification to make them work fine in other compilers.

        dll.def specifies the alias of exported function, which makes it possible to override functions under the armada designation.

#### 2. features

        You can add your own dlls to armada through this library.  

        At the moment, all new dlls are loaded after "FleetOpsHook.dll" made by FO mod.  

        The order of loading is specified by "Data/dll/after.list".  

        This file is loaded in order from top to bottom.  

        Each line represents the relative path of a dll to "Data/dll".  

### hook toolkit    

#### 1. load toolkit
The hook toolkit has NOT been separated from shader+.dll yet. I will do the separation and provide static libraries later.

For now, you have a couple of options:
1. Import the functions into your dll using the import table in .def file. Preferably checking for the correct function name in IDA.  
   But this means your dll must be loaded after shader+.dll.

2. Copy the corresponding code into your project.

3. Write the new code directly in shader+.

I admit that none of this is good solution, but I don't have time to improve the repository at the moment.

WARNING: Do NOT try to load shader+.dll dynamically, this will cause the code to be re-injected into the game engine.

#### 2. features
We natively provide the following features:
1. hookJMP  

        rewrites the implementation of the target function to yours, but you can't access the original content of the target function. The entire function must be rewritten.

2. hookVTable  

        Inject your function through a virtual function table. You can get a pointer to the original function from the return value.
    Typically, you need to use this function with [this pointer fetching](#f3).  

3. writeVarToAddress/writeVarToAddressP

        Writes specified variable to a memory address. The version with the P suffix uses a pointer as the destination address.

4. getClassFunctionAddress

        Get the specified member functions of a class pointer from the virtual function table.
        WARNING: The implementation of this function is unchecked.

5. <span id="f3">getThisPtrFromECX/moveVarToECX</span>

        Get variable from the ECX register or store variable into the ECX register.
        This is typically used to get the this pointer of a member function. If you don't do this, you need to fake a class to get it.

        Typical use case:

        UINT thisPtr;
        __asm{
                call getThisPtrFromECX
                mov thisPtr,eax
        }

        equate to:

        UINT thisPtr;
        __asm{
                mov thisPtr,ecx
        }

        should be equivalent to:

        UINT thisPtr=getThisPtrFromECX();

NOTE:  
There is also a common function "hookTrampoline" that has not been implemented.  
For the time being, I'm using the MinHook third-party library as a replacement.  
Check the [TsudaKageyu's repository](https://github.com/TsudaKageyu/minhook) for the usage of this library.  
Check the shader+ repository for license that use this library.  

### shader+        

#### 1. compile
You almost have to use MSVC compiler. It is possible to work on other compilers, but you need to find a way to convert the dx9 lib files to a format supported by new compiler.  
As reference, I am using MSVC 14.16.27023(MSVC 2017).  
        
You will need Windows SDK and DX9.0c SDK.  

The version of the Win SDK doesn't matter, but if you can get the dx9.0 era version. [The problems in the header file](#p1) will be skippable.   
In general, just install the latest Win SDK when installing MSVC 2017. They are all included in Visual Studio Build Tools.  
As reference, I am using Windows Kits 10.0.19041.0.  

I got DX9 SDK from here: [Microsoft DirectX 9.0 SDK](https://archive.org/details/dx9sdk)  
This version of the DX9 SDK includes support from DX8 to 9.0c (and perhaps older versions, I haven't looked closely), which makes it possible to not switch SDK versions multiple times throughout the development process.

<span id="p1">When</span> everything is ready, trying to compile the code prompts some errors.  
This is caused by DX9's older header files, you need to find the "basetsd.h" file from "Windows Kits\YourWindowsVersion\Include\YourSDKVersion\shared" and replace the same name file in DX9 SDK.

#### 2. armada shader pipeline

This giude is not complete, but feel free to ask me. I'll do my best to answer your questions!  =)

#### 3. asm shader guide

Shader files are relocated in "Data/Shaders/dx8".

        Supported vertex shader version:1.0-1.1  
        Supported pixel shader version:1.0-1.4  

Useful reference for shader writing:

All DX8,9 SDK  

        Each version offers different instructional content.

[Direct3D ShaderX Vertex and Pixel Shader Tips and Tricks by Wolfgang F. Engel](https://www.realtimerendering.com/resources/shaderx/Direct3D.ShaderX.Vertex.and.Pixel.Shader.Tips.and.Tricks_Wolfgang.F.Engel_Wordware.Pub_2002.pdf)  

        Good tutorial for DX8 shader.


[ShaderX2: Introductions & Tutorials with DirectX 9 by Wolfgang F. Engel](https://www.realtimerendering.com/resources/shaderx/Introductions_and_Tutorials_with_DirectX_9.pdf)  

        Good tutorial for DX9 shader.

[ASM Shader Reference by Microsoft](https://learn.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx9-graphics-reference-asm)  

        A better place to know overall shader difference than SDK document.

[Dx8 Pixel Shaders by Sim Dietrich](https://developer.download.nvidia.com/assets/gamedev/docs/GDC2K1_DX8_Pixel_Shaders.pdf)

        An introduction to pixel shader 1.1

[1.4 Pixel Shaders by Jason L. Mitchell](https://drivers.amd.com/developer/ATIMeltdown01Pixel.PDF)

        An introduction to pixel shader 1.4. 
        ps.1.4 has many differences between old version, so this pdf is actually important.


[Basic Lighting by Learn OpenGL](https://learnopengl.com/Lighting/Basic-Lighting)  

        Basic knowledge for Phong lighting model.


[Lighting for Games by Kenneth L. Hurley](https://slideplayer.com/slide/6420784/)  
[Per-Pixel Lighting by D. Sim Dietrich Jr.](https://developer.download.nvidia.com/assets/gamedev/docs/GDC2K_PerPixel_Lighting.pdf)  
[Vertex Shader Introduction by Chris Maughan and Matthias Wloka](https://developer.download.nvidia.com/assets/gamedev/docs/NVidiaVertexShadersIntro.pdf)  
[Chapter 13 Pixel Shaders](https://user.xmission.com/~legalize/book/download/13-Pixel%20Shaders.pdf)  

        Sorry, but I really do not know which book is this chapter actually in. 

## Quick Common Modder Development Guide

The core of the work is to place the light sources wisely.  

It's best to use two directional lights in the map.  
This is because I used some tricks in the pixel shader to create a buggy but good looking rendering of multiple light sources.  

        In the current armada it is not possible to compute multiple light sources lighting in pixel shader. 
        Even if tricks are used, keep in mind that the next light source in an area where the previous light source is completely unable to illuminate will not have any effect there.

One light source is equally fine, you'll get a dark blue tint in the result, which will partially make up for the fact that we can't render the colors of the lights yet.  

When saving map, the game will determine the order in which light sources are stored.  
After that one of the two light sources will be used as the main light source. The other light source will have half the lighting effect.  

So once you've determined how many light sources you want to illuminate with, save the map and re-enter. Then rotate the light sources to get nice results.


## Known issues in shader+

### for common modder
1. <span id="issue1">3D letters representing crystals in the editor turn black.</span>
2. Pixel Shader does not handle point light sources correctly.
3. Shader do not calculate light source color.
4. When the camera is close enough to the unit, rim light will disappear.
5. Some of the matrix conversions are likely to be wrong, but in most cases the correct result is obtained.
6. Occasionally get strange results in free camera views.

### for coder
1. The method of adding a new texture input to pixel shader is unknown.
2. function "disablePixelShaderInAlpha" is a wrong patch. It violently fixes the rendering of spaceships in nebulae, but causes [modder issue 1](#issue1).
3. cameraToNode->front likely not to get the desired results.
4. Error prompts are not connected to armada's error report system. When the error message appears, game must be closed manually from the task manager.
5. Vertex shaders cannot define constants in .nvv files. They must be defined by code.

## Credits

Developers of vanilla Armada 2 game  

        Making 3d games in the 2000s was a battle with code.
        Developers today stand and work under the sun of Unity and UE engine, and in the 2000s you needed to fight with DirectX like a cowboy: 
        Thanks to your work to let me get to play games like this!

Developers of Fleet Operations Mod  

        Thank you for your generous work! When you actually do the same thing, you will realize how complicated it is.

Members of the FO Mod community represented by DOCa Cola and Jan_B  

        Without your help, it will be pretty hard to find the right development path. 
        Reverse engineering is like walking in a fog, without navigation lights everything would be in a mess.

All other developers and players who make Armada mods and play Armada games  

        I couldn't have recognized this ancient game without you guys.
        I have also played the vast majority of mods that can run on Fleet Operations 3.2.7 and upon. They are all awesome!

Creators of all references  

        Without your help, doing anything on DX8 would be nearly impossible.

MinHook's developer TsudaKageyu  

        I'm a slacker, thank for your help to let me avoid a rock in front. ;)



Everyone else living in the world  

        Good Morning, and in case I don't see you, good afternoon, good evening, and good night! =)
