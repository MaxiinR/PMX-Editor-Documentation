

## Table of Contents

1. [Introduction](#introduction "null")

    - [Overview of PMX Editor](#overview-of-pmx-editor "null")
        
    - [Target Audience and Purpose](#target-audience-and-purpose "null")
        
2. [Core Technical Concepts](#core-technical-concepts "null")
    
    - [The PMX File Format](#the-pmx-file-format "null")
        
        - [General Structure and Signature](#general-structure-and-signature "null")
            
        - [Key Data Segments](#key-data-segments "null")
            
        - [Deformation](#deformation-types-bdef-sdef "null") Types (BDEF, SDEF)
            
    - [PMX Editor Architecture](#pmx-editor-architecture "null")
        
        - [Rendering Engine (DirectX via SlimDX)](#rendering-engine-directx-via-slimdx "null")
            
        - [Plugin Subsystem](#plugin-subsystem "null")
            
3. [Plugin Development Environment Setup](#plugin-development-environment-setup "null")
    
    - [System and Software Prerequisites](#system-and-software-prerequisites "null")
        
    - [Essential](#essential-libraries-peplugindll-and-slimdxdll "null") Libraries: `PEPlugin.dll` and `SlimDX.dll`
        
    - [Project Configuration in Visual Studio](#project-configuration-in-visual-studio "null")
        
        - [.NET Framework Version](#net-framework-version "null")
            
        - [C# Language Version](#c-language-version "null")
            
        - [Platform Target (x86/x64)](#platform-target-x86x64 "null")
            
4. [Developing Plugins for PMX Editor](#developing-plugins-for-pmx-editor "null")
    
    - [Plugin Lifecycle and Entry Point (`IPEPlugin`)](#plugin-lifecycle-and-entry-point-ipeplugin "null")
        
    - [Interacting with PMX Editor API](#interacting-with-pmx-editor-api "null")
        
        - [Accessing Model Data (`IPXPmx`, `IPXPmxBuilder`)](#accessing-model-data-ipxpmx-ipxpmxbuilder "null")
            
        - [Manipulating Bones (`IPXBone`)](#manipulating-bones-ipxbone "null")
            
        - [Working with Morphs (`IPXMorph`)](#working-with-morphs-ipxmorph "null")
            
        - [Handling Materials (`IPXMaterial`) and UVs](#handling-materials-ipxmaterial-and-uvs "null")
            
        - [Managing Physics Components (`IPXRigidBody`, `IPXJoint`)](#managing-physics-components-ipxrigidbody-ipxjoint "null")
            
    - [C# Scripting within PMX Editor](#c-scripting-within-pmx-editor "null")
        
5. [Best Practices and Common Pitfalls](#best-practices-and-common-pitfalls "null")
    
    - [API Versioning and Compatibility](#api-versioning-and-compatibility "null")
        
    - [Null Reference Handling](#null-reference-handling "null")
        
    - [Avoiding Modern C# Features](#avoiding-modern-c-features "null")
        
    - [Correct Referencing and Initialization](#correct-referencing-and-initialization "null")
        
    - [Memory Management and Performance](#memory-management-and-performance "null")
        
6. [Troubleshooting Common Development Issues](#troubleshooting-common-development-issues "null")
    
    - [DLL Loading Errors (Blocked Files, Missing Dependencies)](#dll-loading-errors-blocked-files-missing-dependencies "null")
        
    - [Compilation Errors (Type Mismatches, Missing Member Definitions)](#compilation-errors-type-mismatches-missing-member-definitions "null")
        
    - [Runtime Crashes](#runtime-crashes "null")
        
7. [Advanced Topics](#advanced-topics "null")
    
    - [Understanding SDEF Internals (Reverse Engineered)](#understanding-sdef-internals-reverse-engineered "null")
        
    - [Interoperability with Other Tools (e.g., Blender MMD Tools)](#interoperability-with-other-tools-eg-blender-mmd-tools "null")
        
8. [Community and Resources](#community-and-resources "null")
    
    - [Key Repositories and Plugin Examples](#key-repositories-and-plugin-examples "null")
        
    - [Relevant Online Communities](#relevant-online-communities "null")
        
9. [Appendix: Reference Tables](#appendix-reference-tables "null")
    
    - [PMX File Structure Overview](#pmx-file-structure-overview "null")
        
    - [Bone Flags Explained](#bone-flags-explained "null")
        
    - [Key API Interfaces](#key-api-interfaces "null")
        

## 1. Introduction

### Overview of PMX Editor

PMX Editor is a dedicated 3D modeling application for creating and modifying models compatible with MikuMikuDance (MMD). It primarily utilizes the `.pmx` file format, an advanced iteration of the older `.pmd` format, offering enhanced features for rigging, morphing, physics simulation, and material definition. The editor's extensibility via plugins makes it a powerful tool for custom model development.

### Target Audience and Purpose

This documentation is intended for software developers aiming to create plugins or external tools that interact with PMX Editor or the PMX file format. It provides technical insights into the editor's architecture, plugin development methodologies, file format specifics, and best practices, addressing the general lack of official developer-focused documentation.

## 2. Core Technical Concepts

### The PMX File Format

#### General Structure and Signature

The PMX (Polygon Model eXtended) format is the contemporary standard for MMD models (v7.31+).

- **Signature:** Files begin with the ASCII string `"PMX "` (note the trailing space).
    
- **Version:** A `float` indicating the PMX version (e.g., 2.0, 2.1).
    
- **Globals Header:** A fixed-size block defining global parameters:
    
    - Text encoding (UTF-16LE or UTF-8).
        
    - Sizes (in bytes: 1, 2, or 4) for indices related to vertices, textures, materials, bones, morphs, and rigid bodies. This structure facilitates more complex models compared to the PMD format.
        

#### Key Data Segments

A PMX file is composed of distinct data sections, typically including:

1. **Model Information:** Names, comments (often in Japanese and English).
    
2. **Vertex Data:** Positions, normals, UV coordinates (up to 4 sets per vertex), bone weights (BDEF1/2/4, SDEF, QDEF), edge scaling factor.
    
3. **Face (Index) Data:** Defines triangular faces using vertex indices.
    
4. **Texture Table:** A list of paths to external texture files (e.g., `.png`, `.jpg`, `.tga`, `.dds`, `.spa`, `.sph`).
    
5. **Material Data:** Surface properties including diffuse, specular, ambient colors; shininess; edge color/size; texture/sphere/toon map indices; rendering flags.
    
6. **Bone Data:** Hierarchical skeletal structure, including bone names, positions, parent/target indices, IK links, and various behavioral flags.
    
7. **Morph Data:** Defines shape variations (vertex, bone, UV, material, group morphs).
    
8. **Display Frame Data:** Organizes bones and morphs into groups for the editor's UI.
    
9. **Rigid Body Data:** Parameters for physics simulation (shape, mass, friction, restitution, associated bone, collision group/mask).
    
10. **Joint Data:** Constraints connecting rigid bodies for physics.
    

#### Deformation Types (BDEF, SDEF)

PMX supports several skinning methods for vertex deformation by bones:

- **BDEF1:** Influence from a single bone.
    
- **BDEF2:** Linear blend between two bones.
    
- **BDEF4:** Linear blend between up to four bones.
    
- **SDEF (Spherical Deformation):** A more complex algorithm for smoother deformations, especially at joints. It involves additional weight parameters (C, R0, R1 vectors) per vertex for the two influencing bones. The exact SDEF calculation is not officially documented.
    
- **QDEF (Quaternion-based Deformation):** An experimental type, less commonly used.
    

### PMX Editor Architecture

#### Rendering Engine (DirectX via SlimDX)

PMX Editor is a Windows application utilizing the .NET Framework (typically 4.x). Its 3D rendering is powered by DirectX 9.0c, accessed through the **SlimDX** library, which is a .NET wrapper for DirectX APIs. Plugins requiring graphical output or manipulation of 3D data will interact with SlimDX.

#### Plugin Subsystem

The editor features a binary plugin system. Plugins are typically .NET assemblies (DLLs) written in C# that interface with PMX Editor's core functionalities.

- **Plugin Location:** DLLs are placed in the `_plugin\User\` (or similar) subdirectory of the PMX Editor installation.
    
- **API Access:** Plugins interact with the editor via `PEPlugin.dll`, which exposes interfaces for model data manipulation, UI interaction, and other services.
    

## 3. Plugin Development Environment Setup

### System and Software Prerequisites

- **Operating System:** Windows (Windows 10 or newer recommended).
    
- **IDE:** Microsoft Visual Studio (e.g., 2019, 2022) with .NET desktop development workload.
    
- **.NET Framework:** PMX Editor and its plugins target older .NET Framework versions (see below).
    
- **PMX Editor Installation:** Required for accessing core DLLs and testing.
    

### Essential Libraries: `PEPlugin.dll` and `SlimDX.dll`

These are the cornerstone DLLs for plugin development:

- `PEPlugin.dll`: The primary API library for PMX Editor. It contains interfaces and classes for accessing and modifying model data, interacting with the editor's UI, and managing plugin operations.
    
- `SlimDX.dll`: A .NET wrapper for Microsoft DirectX. It's used for 3D graphics operations, vector and matrix math, and any direct rendering tasks.
    

Both DLLs must be referenced in your plugin project. They are found within the PMX Editor's installation directory. It's common practice to copy these to a `lib` folder within your project and reference them from there. Ensure these files are "unblocked" in their Windows properties if downloaded from the internet.

### Project Configuration in Visual Studio

#### .NET Framework Version

- **Target Framework:** Set your Class Library project to **.NET Framework 4.7.2** or **.NET Framework 4.8**. Using newer versions like .NET Core or .NET 5+ will result in incompatible assemblies.
    

#### C# Language Version

- **Language Version:** Set to **C# 7.3** or lower. (Project Properties → Build → Advanced → Language version). PMX Editor's host environment does not support newer C# features (e.g., C# 8.0+ range operators, default interface methods).
    

#### Platform Target (x86/x64)

- **Platform Target:** Typically **x86 (32-bit)**. While 64-bit versions of PMX Editor exist, targeting x86 ensures broader compatibility unless you are specifically developing for an x64 environment and the editor version explicitly supports it. (Project Properties → Build → Platform target).
    

## 4. Developing Plugins for PMX Editor

### Plugin Lifecycle and Entry Point (`IPEPlugin`)

A PMX Editor plugin is a .NET class library (DLL) that implements the `IPEPlugin` interface from `PEPlugin.dll`. Key members of `IPEPlugin`:

- `Name` (string): The display name of the plugin.
    
- `Version` (string): The plugin's version number.
    
- `Description` (string): A brief description of the plugin.
    
- `Run(IPERunArgs args)` (void): The main execution method called when the plugin is invoked from the editor. The `IPERunArgs` argument provides access to the host editor's services and the current model.
    

```
using PEPlugin;
using PEPlugin.Pmx; // For IPXPmx, etc.
using PEPlugin.PE;   // For IPERunArgs, etc.
using System.Windows.Forms; // For MessageBox

public class SamplePlugin : IPEPlugin
{
    public string Name => "Sample Plugin";
    public string Version => "1.0.0";
    public string Description => "A basic PMX Editor plugin.";

    public void Run(IPERunArgs args)
    {
        // Access the host and PMX data
        IPEPluginHost host = args.Host;
        IPXPmx pmx = host.Connector.Pmx;

        if (pmx == null)
        {
            MessageBox.Show("No PMX model is currently loaded.", "Plugin Info", MessageBoxButtons.OK, MessageBoxIcon.Information);
            return;
        }

        MessageBox.Show($"Model '{pmx.ModelInfo.ModelName}' loaded. It has {pmx.Vertex.Count} vertices.", "Plugin Info", MessageBoxButtons.OK, MessageBoxIcon.Information);
        
        // Plugin logic here
    }
}
```

### Interacting with PMX Editor API

#### Accessing Model Data (`IPXPmx`, `IPXPmxBuilder`)

- `IPXPmx`: Represents the currently loaded PMX model. It provides access to collections of vertices, faces, materials, bones, morphs, etc. (e.g., `pmx.Vertex`, `pmx.Bone`). This interface is primarily for reading data.
    
- `IPXPmxBuilder`: Used for constructing new PMX elements (e.g., `builder.Vertex()`, `builder.Bone()`). This is essential when modifying the model or creating new parts. The `IPXPmx` object is typically obtained via `args.Host.Connector.Pmx`. The `IPXPmxBuilder` is often obtained via `args.Host.Connector.PmxBuilder`.
    

#### Manipulating Bones (`IPXBone`)

Bones are accessed via `pmx.Bone` (a list of `IPXBone` objects).

```
// Example: Iterate through bones and print their names
foreach (IPXBone bone in pmx.Bone)
{
    System.Diagnostics.Debug.WriteLine($"Bone: {bone.Name}");
}

// Example: Create a new bone (conceptual, using builder)
// IPXPmxBuilder builder = args.Host.Connector.PmxBuilder;
// IPXBone newBone = builder.Bone();
// newBone.Name = "MyNewBone";
// newBone.Position = new PEPlugin.SDX.V3(0, 1, 0); // Using SlimDX V3 type
// pmx.Bone.Add(newBone); // Modifying collections usually requires updating the view
```

#### Working with Morphs (`IPXMorph`)

Morphs are accessed via `pmx.Morph`. Each `IPXMorph` object has a `MorphType` and a list of offsets.

```
// Example: List vertex morphs
foreach (IPXMorph morph in pmx.Morph)
{
    if (morph.MorphType == MorphType.Vertex)
    {
        System.Diagnostics.Debug.WriteLine($"Vertex Morph: {morph.Name}");
    }
}
```

#### Handling Materials (`IPXMaterial`) and UVs

Materials define surface appearance. `pmx.Material` provides a list of `IPXMaterial`. UVs are part of `IPXVertex` data.

```
// Example: Change diffuse color of the first material
if (pmx.Material.Count > 0)
{
    IPXMaterial firstMaterial = pmx.Material[0];
    firstMaterial.Diffuse = new PEPlugin.SDX.V4(1.0f, 0.0f, 0.0f, 1.0f); // Red color (R,G,B,A)
    // Remember to update the view/model if changes are made
    // args.Host.Connector.View.UpdatePmxView(); or similar
}
```

#### Managing Physics Components (`IPXRigidBody`, `IPXJoint`)

Rigid bodies and joints are accessed via `pmx.RigidBody` and `pmx.Joint` respectively.

### C# Scripting within PMX Editor

Some versions of PMX Editor include a C# scripting window, allowing for quick automation or testing of small code snippets without compiling a full plugin. The API available in the scripting environment is generally the same as for compiled plugins.

## 5. Best Practices and Common Pitfalls

### API Versioning and Compatibility

Be aware that `PEPlugin.dll` might have different versions across various PMX Editor distributions. Plugins compiled against one version might not be compatible with another if there are breaking API changes. Aim for widely distributed PMX Editor versions if possible.

### Null Reference Handling

Always check for `null` before accessing properties or methods of objects obtained from the PMX Editor API, especially `pmx`, `builder`, or individual model elements.

```
IPXMaterial material = builder.Material(); // Assuming builder is IPXPmxBuilder
if (material != null)
{
    material.Name = "NewMaterial";
}
else
{
    // Log error or handle appropriately
    PEPlugin.System.Diagnostics.Debug.WriteLine("Failed to create material instance.");
}
```

### Avoiding Modern C# Features

As stated, PMX Editor's .NET environment is older. Avoid:

- C# 8.0+ syntax (index/range operators `data[1..^0]`, null-coalescing assignments `??=`, etc.).
    
- `init` properties, `record` types.
    
- `Span<T>`, `Memory<T>`.
    
- Complex LINQ expressions if they cause issues (simpler ones are usually fine). Stick to traditional loops for reliability.
    

### Correct Referencing and Initialization

- Ensure `PEPlugin.dll` and `SlimDX.dll` are correctly referenced and accessible at runtime.
    
- Properly initialize API objects using the provided builders or connectors.
    

### Memory Management and Performance

- PMX Editor, especially 32-bit versions, can be memory-intensive with large models.
    
- Undo/Redo operations consume significant memory.
    
- Write efficient code, especially when iterating over large collections (vertices, faces).
    
- Dispose of `IDisposable` objects if you create any that are not managed by the editor (less common with SlimDX in this context, but good practice).
    

## 6. Troubleshooting Common Development Issues

### DLL Loading Errors (Blocked Files, Missing Dependencies)

- **"Operation not supported" / File Blocked:** If DLLs are downloaded, Windows might block them. Right-click the DLL → Properties → Unblock.
    
- **Missing `netstandard.dll` or similar:** Indicates plugin was compiled against an incompatible .NET version (e.g., .NET Core/.NET Standard). Re-target to .NET Framework 4.7.2/4.8.
    
- **`SlimDX.dll` or `PEPlugin.dll` not found:** Ensure these are in the PMX Editor's execution path or alongside your plugin if not correctly referenced.
    

### Compilation Errors (Type Mismatches, Missing Member Definitions)

- **"'IPXMaterial' does not contain a definition for 'FaceCount'"**: This property might not exist or is calculated internally. Consult API examples or use IntelliSense.
    
- **"'IPXVertex' does not contain a definition for 'Def'"**: The property name for deformation type might be different or accessed via a method.
    
- **"Method must declare a body"**: Ensure all non-abstract, non-extern, non-partial methods have an implementation (`{ }`), even if empty.
    
- **Type Mismatches (e.g., `V3` vs. custom vector types):** Always use types from `PEPlugin.SDX` (e.g., `PEPlugin.SDX.V3`, `PEPlugin.SDX.V4`) for vectors and matrices.
    

### Runtime Crashes

- **`NullReferenceException`:** Most common; due to not checking for `null` before using an API object.
    
- **`IndexOutOfRangeException`:** When accessing collections, always check bounds.
    
- **Architecture Mismatch:** Running an x64 plugin in an x86 editor or vice-versa.
    

## 7. Advanced Topics

### Understanding SDEF Internals (Reverse Engineered)

The SDEF (Spherical Deformation) skinning method provides smoother joint deformations. While the PMX specification defines the data structure (weights for two bones, and three `V3` vectors: C, R0, R1), the exact mathematical formula used by MMD and PMX Editor for applying these weights is not officially documented. Community efforts have reverse-engineered plausible formulas, which are crucial for tools that need to accurately replicate MMD's skinning behavior (e.g., exporters to game engines, custom renderers). Researching these community findings is necessary for advanced SDEF manipulation or implementation.

### Interoperability with Other Tools (e.g., Blender MMD Tools)

Many workflows involve moving models between PMX Editor and other 3D software like Blender.

- **MMD Tools for Blender:** A popular Blender addon that supports importing and exporting PMX, PMD, VMD (motion), and VPD (pose) files. Understanding how MMD Tools handles conversions (e.g., SDEF to Blender weight painting, material interpretation) can be beneficial for developers creating complementary tools or plugins.
    
- **Coordinate Systems and Scaling:** Be mindful of differences in coordinate systems (Y-up vs. Z-up) and units when transferring models.
    

## 8. Community and Resources

### Key Repositories and Plugin Examples

- **GitHub:** Search for "PMX Editor Plugin", "MMD Tools", "pymeshio". Many developers share their plugins and PMX-related libraries here.
    
    - `Wampa842/WPlugins`: Collection of PMX Editor plugins.
        
    - `paralleltree/PmXEditorPlugins`: More examples of plugins.
        
    - `ousttrue/pymeshio`: Python library for PMD/PMX I/O.
        
- **BowlRoll:** A Japanese file-sharing site where many MMD assets, including plugins, are distributed.
    

### Relevant Online Communities

- **NicoNicoDouga (and associated blogs):** Historically a central hub for MMD content and technical discussions in Japanese.
    
- **DeviantArt:** Contains MMD communities where users share resources and tutorials.
    
- **VPVP Wiki (Japanese):** A comprehensive wiki for MMD-related information.
    

### 9. Appendix: Reference Tables
   
   
   
PMX File Structure Overview

(Abridged - See full community specifications for byte-level details)
| Segment         | Content Description                                                                                           |
|-----------------|------------------------------------------------------------------------------------ |
| Header          | Signature "PMX ", Version, Globals (text encoding, index sizes)                          |
| Model Info      | Model Name (JP/EN), Comments (JP/EN)                                                          |
| Vertex List     | Count, then array of Vertices (Position, Normal, UV, Extra UVs, Weight Type, Weights, Edge) |
| Face List       | Count, then array of Vertex Indices (3 per face)                                    |
| Texture List    | Count, then array of Texture File Paths (UTF-16 or UTF-8)                           |
| Material List   | Count, then array of Materials (Colors, Tex Indices, Flags, Edge, Toon, Sphere)     |
| Bone List       | Count, then array of Bones (Name, Pos, Parent, IK data, Flags)                      |
| Morph List      | Count, then array of Morphs (Name, Type, Offsets)                                   |
| Display Frame List | Count, then array of Display Frames (Name, Special Flag, Element Groups)          |
| Rigid Body List | Count, then array of Rigid Bodies (Name, Bone, Shape, Physics Params)               |
| Joint List      | Count, then array of Joints (Name, Type, Connected Bodies, Physics Params)          |
Bone Flags Explained

(Interpreted from various community sources; exact bits can be complex)
| Bit (Hex) | Description (Common Interpretation)                                 |
|-----------|---------------------------------------------------------------------|
| 0x0001    | Tail Connection Type (0: Offset, 1: To Bone Index)                  |
| 0x0002    | Rotatable                                                           |
| 0x0004    | Movable / Translatable                                              |
| 0x0008    | Visible                                                             |
| 0x0010    | Controllable / Operable by user                                     |
| 0x0020    | Is IK Bone                                                          |
| 0x0100    | Append Rotation (Rotation is added to parent's rotation)            |
| 0x0200    | Append Translation (Translation is added to parent's translation)   |
| 0x0400    | Fixed Axis (Rotation restricted to a specific axis)                 |
| 0x0800    | Local Transformation (Uses local coordinate system for append)      |
| 0x1000    | Physics After Deform (Bone transformation applied after physics)    |
| 0x2000    | External Parent Deform (Bone transforms with an external parent)    |
Key API Interfaces

(From PEPlugin.dll, namespace PEPlugin and sub-namespaces)
| Interface       | Primary Purpose                                                              |
|-----------------|------------------------------------------------------------------------------|
| IPEPlugin     | Main entry point for plugins. Defines Name, Version, Run method.             |
| IPERunArgs    | Argument passed to IPEPlugin.Run, provides access to IPEPluginHost.      |
| IPEPluginHost | Provides access to editor services like IConnect.                          |
| IConnect      | Connector to obtain IPXPmx (model data) and IPXPmxBuilder.               |
| IPXPmx        | Represents the loaded PMX model; provides access to its components.          |
| IPXPmxBuilder | Factory for creating new PMX model elements (vertices, bones, etc.).         |
| IPXHeader     | Access to PMX file header information.                                       |
| IPXModelInfo  | Access to model name and comments.                                           |
| IPXVertex     | Represents a single vertex and its properties.                               |
| IPXFace       | Represents a single face (triangle of vertex indices).                       |
| IPXMaterial   | Represents a material and its properties.                                    |
| IPXBone       | Represents a bone and its properties.                                        |
| IPXMorph      | Represents a morph and its offsets.                                          |
| IPXRigidBody  | Represents a rigid body for physics.                                         |
| IPXJoint      | Represents a joint (constraint) for physics.                                 |
| IPXView       | Interface to control aspects of the editor's 3D view.                      |
| IPESystem     | Provides system-level information or utilities.                              |

(Note: Specific interface names and availability might vary slightly between PMX Editor versions. Use IntelliSense in Visual Studio for confirmation.)
