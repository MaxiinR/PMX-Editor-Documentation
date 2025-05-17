

PMX Editor is a powerful tool for editing models in the PMX format, an extension of the PMD format used in MikuMikuDance (MMD). This documentation provides a comprehensive guide to its features, best practices, and technical details, ensuring both beginners and advanced users can make the most of the editor. 

---

## Introduction

PMX Editor is designed for editing 3D models in the PMX format, offering advanced capabilities compared to the older PMD Editor. It supports extensive customization through plugins and scripting, making it a versatile tool for MMD model creation and modification.

PMX Editor is a specialized 3D modeling tool used primarily for creating models compatible with MikuMikuDance (MMD). It supports the `.pmx` and `.pmd` formats and offers advanced tools for rigging, animation, facial expressions, and physics-based simulations.

Despite its popularity among content creators, **there is a severe lack of developer documentation** regarding plugin development, file structure parsing, or internal mechanics. This document aims to fill that gap by providing technical insights into:

- Plugin development using C#, Python, and SlimDX  
- Bone system architecture and IK handling  
- Facial morph implementation  
- Reverse-engineered PMX format details  
- Existing plugins and community tools  

---

## System Requirements

To ensure optimal performance, the following system requirements are recommended:

- **Operating System:** Windows 10 or newer
- **Processor:** Supports SSE1 or higher
- **Graphics Card:** Compatible with Shader Model 3.0 or higher
- **Architecture:** 64-bit (recommended for large models)

---

## Key Features and Functionalities

### General Information

- **File Management:** PMX models reference external texture files (e.g., `.png` `.jpeg` `.tga` `.dds`). Avoid using Japanese filenames to prevent compatibility issues.
  For spa textures you can also use `.spa`, `.sph`, but for 2025 year  usage of these file extentions are unpopular. 
- **Compatibility:** Be mindful of character limits for paths and filenames, especially when transitioning from PMD to PMX formats. 

### User Interface and Controls

- **TransformView Enhancements:** Intuitive tools like joint pinching for deformation are available, but traditional handle transformations remain the standard for detailed pose design.
- **Selection and Filtering:** Efficient tools for filtering views by bones, rigid bodies, and joints help manage complex models.
- **Camera Settings:** Adjust perspective toggling and rotation center settings for better control.

### Editing Tools

- **Morph Editing:** Improved synchronization and undo functionality for morph edits.
- **Vector Copy Functionality:** Simplifies repetitive coordinate adjustments.
- **UV Mapping:** Complete UV mapping before converting to PMX to avoid compatibility issues.
- **Joint Position Verification:** Basic checks ensure joints are correctly positioned between related rigid bodies.
- Regularly save your work and use automatic backup plugins.
- Use vector copy tools for efficient coordinate operations.
- Complete UV mapping before finalizing PMX conversions.
- Verify joint positions to ensure proper physics simulation.

---


__ переписать 
## Technical Details

The PMX Editor provides a comprehensive toolset for managing bones and weighting in 3D models, crucial for animators and developers working with MMD (MikuMikuDance). Here's an aggregation of insights and tutorials from various sources to help you understand bone setup and weighting processes.

### Bone Setup

To begin with, launching PMX Editor and loading your model is the first step. Under the "Bone" tab, which is also referred to as the BON tab in some versions, users can find a list of bones along with their parameters

This interface allows for the creation, modification, and management of bones essential for animation rigging.

### Rigging Models with Multiple Bones

For complex models that include elements like skirts or wings, it is recommended to set up a detailed bone structure. One useful document outlines physics settings for a 49-bone rig, advising that the first one or two bones of movable parts such as skirts and wings should be configured for bone tracking to ensure stability during animations


### Weighting Process

Weighting is the process of assigning influence of bones over vertices in a model:

1. Load your model and position the elements like hair.
2. Select the necessary tools and navigate to the "Bone" tab.
3. Assign weights to ensure natural movement when the bones are manipulated

### Advanced Weighting Techniques

For more precise control over vertex deformation, consider using SDEF (Spherical Deformation) interpolation. Users can select one or more vertices and change their weight mode to SDEF via the edit/weights menu. The PMX Editor then calculates the centers based on the bone setup, allowing for smoother transitions and more realistic deformations

### Best Practices

- **Smooth Physics:** When setting up bones for elements like skirts and wings, prioritize stability by configuring initial bones for tracking purposes before adding more complex physics simulations

- **Vertex Control:** Utilize advanced weighting methods like SDEF to enhance realism in vertex deformations, particularly useful in characters with intricate designs or dynamic movements


By following these guidelines and leveraging the capabilities of PMX Editor, developers and animators can achieve highly realistic and fluid animations tailored to their specific needs. Understanding both fundamental and advanced techniques in bone setup and weighting is essential for maximizing the potential of 3D models within MMD environments. 

__ переписать 

---

## Development Environment

### Memory Management

- Undo operations consume significant memory. Use the 64-bit version for large models to avoid 32-bit memory limitations.

---

## Troubleshooting

- **Startup Errors:** Unblock ZIP or DLL files after downloading to prevent errors.
- **High DPI Issues:** Run the editor at 100% scale on high-resolution displays.
- **Memory Crashes:** Regularly save your work due to potential memory-related crashes.

---

## Known Issues and Workarounds

- Legacy UI elements may not work properly on newer Windows versions.
- Block external DLLs before use to prevent security warnings.

---

## Security Considerations

- External texture files (e.g., `.png`) are referenced by the model.
- Avoid using Russian characters in filenames and paths to prevent issues.
- Texture file encryption remains a challenge for web-based implementations.

# PMX Editor (MMD) – Technical Overview

**Architecture & Dependencies:** PMX Editor is a Windows-only model-editing tool (successor to PMD Editor) implemented as a standalone GUI application. It requires the Microsoft .NET Framework (4.5 or newer), the DirectX 9.0c runtime, and the Visual C++ 2010 Redistributable. (One popular English build even notes “Remember: ‘Unblock’ the .zip file before extracting it” to avoid Windows DLL-blocking issues.) The program uses DirectX for 3D rendering (e.g. requiring Shader Model 3.0) and bundles the SlimDX library for GPU access. Both 32-bit and 64-bit executables are provided (the 32-bit build “will NOT work on 64-bit OS” as noted in distribution notes). No formal installer is needed – the app is “install-free” (just unzip the archive). Performance and memory limits align with PMX conventions (PMX v2.0 allows up to ~4 billion vertices vs. 65535 in PMD).

**Plugin/System Integration:** PMX Editor has a binary plugin system but no public SDK documentation. By convention, plugins are distributed as DLLs dropped into the `_plugin\User` folder within the PMX Editor directory. (For example, a Japanese tutorial explains that under the PMX Editor folder is a `_plugin` directory containing a `User` subfolder; placing plugin DLLs there “completes installation”. The official readme simply states “Plugins folder: See _plugin/user.path”, which corresponds to that structure.) Plugins are typically written in .NET languages (e.g. C#) to interoperate with the editor’s internals. Community examples (below) are mostly C# DLLs. The editor loads each plugin DLL at runtime; users sometimes must “unblock” downloaded DLLs (or the containing ZIP) to avoid the “operation not supported” errors due to Windows file security.

**PMX File Format (Spec):** The PMX format (Polygon Model eXtended) is the official model format for MMD v7.31+. It begins with a 4-byte ASCII signature `"PMX "` followed by a float version (2.0 or 2.1). A fixed-size “Globals” block (8 bytes for PMX 2.0) then follows, encoding the text string format and index sizes. In particular, one global byte flags whether text is UTF-16LE or UTF-8, and several bytes specify how many bytes are used for indices of vertices, textures, materials, bones, morphs, and rigid bodies (each can be 1, 2, or 4 bytes). After this header come model metadata (Japanese/English names and comments) and counts of vertices, faces, textures, materials, bones, morphs, display frames, rigid bodies, joints, etc. Vertices have positions, normals, UVs (up to 4 sets), and bone weights (supporting up to 4 bones per vertex in PMX 2.0); multiple deform types (BDEF1/2/4, SDEF) and edge flags are encoded as specified by the spec. Material entries include textures/spheres, toon/shadow flags, and other per-material data. In short, PMX adds beyond PMD: unlimited-length text fields, 4-vector additional UVs, real-valued bone weights, SDEF skinning, more flexible material flags, etc. (See e.g. the community “PMX file format” specification for a complete field-by-field breakdown.)

**Plugin/Tool Examples:** A number of third-party plugins and libraries exist for extending PMX Editor or working with PMX files. Notable examples include:

- **Wampa842 – “WPlugins” (GPL‑3.0):** A bundle of PMXE plugins (source on GitHub) providing an OBJ importer/exporter, morph scaling tools, selection storage, etc. Its readme lists features like “Wavefront OBJ importer” (with job file presets) and “Selection Storage” (save/restore named vertex/bone selections).
    
- **paralleltree – PmXEditorPlugins (MIT):** A GitHub repo of simple plugins targeting PMD/PMX Editor. For example, it hosts “ContinuousSelection” (select all faces adjacent to the current vertex set) and “WeightedOffsetBlend” (add vertex offsets weighted by distance). These plugins are DLLs downloaded (often via BowlRoll) and drop into the user plugin folder.
    
- **pymeshio (Python, zlib license):** A Python library that can read/write PMD/PMX and convert between them. For instance, you can load a PMD model, convert it to a PMX model, and write it to disk – e.g. `pymeshio.pmx.writer.write(file, pmx_model)` returns `True` when complete. Such libraries illustrate how PMX structure is parsed (parsing vertices, bones, morphs, etc.) in code.
    
- **Other tools:** The broader MMD community offers many resources. For example, _LegToe-IK Generator_ (Johnwithlenon) adds leg/toe IK bones via a PMXE plugin. Many utilities exist on forums or sites like BowlRoll/DeviantArt (e.g. plugins to add default bones, auto-luminosity morphs, etc.). Blender MMD import/export tools (like MMD Tools) also support PMX.
    

**Community Notes:** Discussion threads and blog posts often clarify usage details. For instance, Japanese sites emphasize the need to install prerequisites (VC++ 2010, DirectX 9.0c) and to disable file blocking. Amenrenet’s DeviantArt page provides a partial English translation of the official README, confirming plugin folder paths and startup options. The VPVP wiki notes that all MMD v7.31+ models use PMX 2.0 by default. Developers writing tools or plugins should consult these specs and examples; for instance, the detailed PMX spec on GitHub and open-source plugin projects provide concrete guidance on fields and APIs.

**Sources:** Official spec and user translations; community wikis; tool/plugin docs and code; tutorials/blogs on PMX Editor usage. Each source is cited above for verification. 



---

## Technological Stack & Libraries for Plugin Development

### Key Technologies Used

| Technology | Description |
|-----------|-------------|
| **SlimDX** | A .NET wrapper for DirectX, used extensively for rendering, texture handling, and graphical effects in plugins like `PMXEPlugin536`. |
| **C#** | Primary language for plugin development due to integration with PEPlugin.dll and native support in PMX Editor. |
| **Python** | Used in plugins like `KKPMX` for automation tasks such as bone cleanup, physics conversion, and mesh optimization. |

### Example: Using SlimDX in a Plugin

```csharp
using SlimDX;
using SlimDX.Direct3D9;

public class TextureMaskPlugin {
    public void MaskImageToEdge(Texture texture, Rectangle edgeRect) {
        // Normalize UV positions and scale edge regions
        // ...
    }
}
```

> **Note**: To compile plugins, you must manually copy `SlimDX.dll` and `PEPlugin.dll` from the PMX Editor installation directory into your project folder.

---

## PMX Editor Architecture: Bones, Bindings, and File Operations

### Bone Management System

The internal bone system in PMX Editor supports:
- **IK Chains**
- **Local vs Global Transform Flags**
- **Parent-Child Inheritance**
- **SDEF Deformations (Sphere Deformation)**

A newly discovered flag — **bit 7** — controls whether an additional transformation is applied locally. This flag corresponds to the "L" button in the UI and is essential when developing plugins like `ArmIKPlus`.

The bone and weight system is central to PMX Editor's functionality:

- **Bone Creation and Manipulation:**
  ```csharp
  var bone = m_bld.Bone();
  bone.Position = new V3(x, y, z);
  pmx.Bone.Add(bone);
  ```
- **Weight Painting:** BDEF4 continuous bone selection and deformation synchronization are supported.
 
### File Handling

- Supports `.pmx` (version 2.0+) and `.pmd`
- Each model includes sections for:
  - Vertices
  - Bones
  - Materials
  - Morphs
  - Rigid bodies and constraints

Example bone structure inside PMX files:
```csharp
struct Bone {
    string Name;
    Vector3 Position;
    int ParentIndex;
    int ChildCount;
    byte[] Flags; // Includes flags like BDEF type, SDEF usage, etc.
}
```

---

## Plugin Development: Code Examples and Tools

### Building Your First Plugin

#### Steps:
1. Install Visual Studio 2019+
2. Create a new Class Library (.dll)
3. Add references to:
   - `PEPlugin.dll`
   - `SlimDX.dll`
4. Implement the plugin interface:
   ```csharp
   public class MyPlugin : IPEPlugin {
       public string Name => "MyPlugin";
       public string Version => "1.0";
       
       public void Run(PEPlugin.ICore core) {
           MessageBox.Show("Hello from PMX Editor!");
       }
   }
   ```
5. Place the compiled `.dll` in:
   ```
   $(PmxEditor_root_dir)\_plugin\User\
   ```


- **Dynamic Plugins:** New Compact Plugins (CPlugins) can be loaded/unloaded at runtime:
  ```csharp
  public class Register : RegisterBase {
      public override string[] ClassNames {
          get { return new string[] { "CPluginTest.Class1" }; }
      }
  }
  ```


- **C# Scripting:** Write custom scripts directly within the editor using C#. This is ideal for automating tasks or extending functionality without needing a full Visual Studio setup.
- **Sample Plugin Code:**
  ```csharp
  foreach (var m in material.Where(x => x.Toon == "toon01.bmp")) {
      m.Toon = "toon03.bmp";
  }
  ```
 
### Open Source Tool: MMD Tools (Blender Plugin)

- Supports import/export of `.pmx`, `.pmd`, `.vmd`, `.vpd`
- Latest version: **4.3.6** (May 8, 2025)
- Features:
  - Physics body editing
  - Material tweaking
  - Support for OpenCC without pip install

GitHub: [https://github.com/UuuNyaa/blender_mmd_tools](https://github.com/UuuNyaa/blender_mmd_tools)


### Material System

Materials define the visual properties of models:

- **Material Properties:**
  ```csharp
  this.txtMaterial_Amb_G = new TextBox(); // Ambient Green component
  this.txtMaterial_Spe_G = new TextBox(); // Specular Green component
  ```

###### Soft Body Physics

Soft body physics simulate flexible objects:

- **Configuration:**
  ```csharp
  this.txtSoftBody_Margin.Text = sb.Margin.ToString();
  this.sbConfig.chkSoftBody_GenerateClusters.Checked = sb.IsGenerateClusters;
  ```

###### Morph System

Morphs allow dynamic changes to models:

- **Offset Management:**
  ```csharp
  this.ledMorphOffset.Location = new Point(182, 286);
  ```

###### Joint System

Joints define constraints between rigid bodies:

- **Limit Configuration:**
  ```csharp
  this.txtJoint_Limit_MovY_L.Text = j.Limit_MoveLow.Y.ToString();
  ```


---
??
## Reverse Engineering the PMX Format

### PMX File Structure Breakdown

| Section | Description |
|--------|-------------|
| Header | Contains format version, encoding, and extra data size |
| Vertex Data | Position, normals, UVs, and bone weights |
| Face Indices | Triangle indices for drawing |
| Bones | Hierarchical structure with transforms and flags |
| Morphs | Face expressions, shape keys, and material morphs |
| Display Frames | UI categories for grouping bones/morphs |
| Rigid Bodies | Physics simulation data |
| Joints | Constraints between rigid bodies |

### Deformation Types Supported

| Type | Description |
|------|-------------|
| **BDEF1** | Single bone influence |
| **BDEF2** | Two bone blending |
| **BDEF4** | Four bone blending |
| **SDEF** | Sphere deformation – requires `C`, `R0`, and `R1` vectors |
| **QDEF** | Quaternion-based deformation |

> ⚠️ **Note**: The exact calculation method for `SDEF` remains undocumented and has been reverse-engineered only partially.

---
?? 
## Bone System Implementation in PMX Editor

### IK Chain Creation with ArmIKPlus

This plugin creates arm IK chains automatically and supports three modes:
- **Type1**: Basic hand positioning
- **Type2**: Object holding (e.g., sword, microphone)
- **Advanced**: Obsolete due to dependency issues

Usage:
1. Load two models (e.g., Honne Dell and Hatsune Miku)
2. Use `OP` function to link bones
3. Result: Right hand follows head movement automatically

> 🧩 Known issue: Newly created IK bones may be invisible until renamed via Batch Name Editor.

### Bone Flag Details

| Bit | Meaning |
|-----|---------|
| Bit 0 | Rotatable |
| Bit 1 | Translatable |
| Bit 2 | Visible |
| Bit 3 | Controllable |
| Bit 4 | Indexed Tail Position |
| Bit 5 | Append Rotation |
| Bit 6 | Append Translation |
| Bit 7 | Local Transformation (undocumented) |

---

## Facial Expressions (Morphs) and Animation Integration

### Morph Types

| Type | Description |
|------|-------------|
| **Vertex Morphs** | Direct vertex position changes |
| **Bone Morphs** | Changes bone transform |
| **UV Morphs** | Modifies texture coordinates |
| **Material Morphs** | Alters material properties |

### Flip Morph Behavior

Flip morphs allow dynamic control over other morphs through weight coefficients. For example:
```python
def apply_flip_morph(morph_name, weight):
    if weight > 0.5:
        activate_morph("Smile")
    else:
        deactivate_morph("Smile")
```

Used heavily in plugins like `KKPMX` to simulate emotional expression transitions.

---
?? 
## Ecosystem Analysis: Plugins, Extensions, and Communities

### Notable Plugins

| Plugin | Purpose | Notes |
|-------|---------|-------|
| **ArmIKPlus** | Auto-generates arm IK chains | Developed by t0r0 on NicoNico |
| **KKPMX** | Converts Koikatsu characters to PMX | Uses Python for automation |
| **PMXEPlugin536** | Texture masking and edge scaling | Requires SlimDX for rendering |

### Community Resources

- **NicoNico Forums** – Japanese-centric but rich with plugin discussions  
- **Modelo Platform** – Hosts open-source PMX models and rigs  
- **GitHub Repositories** – Search for `mmd blender pmx` for integration guides

---

## Projects and Models Created with PMX Editor

### Common Use Cases

- MMD animations (dance, music videos)
- VRChat avatars
- Game character exports
- Custom anime figure creation

### Optimization Techniques

- Use **UV morphs** instead of increasing polygon count
- Apply **SDEF deformations** for smoother skinning
- Export with **MMD Tools** for Blender-based refinement

---

## Appendix: Structured Reference Tables

### File Formats and Structures

| Item | Description |
|------|-------------|
| `.pmx` | Main format used in modern MMD workflows |
| `.pmd` | Legacy format, still supported |
| Vertex Layout | Position, normal, UV, bone IDs, weights |
| Bone Hierarchy | Parent-child relationships, flags, transformations |

### Supported Deformation Types

| Type | Usage Case |
|------|------------|
| BDEF1 | Simple static limbs |
| BDEF2 | Arms/legs with dual influence |
| BDEF4 | Complex joints (shoulders, hips) |
| SDEF | High-quality soft body deformation |
| QDEF | Experimental quaternion-based system |

### Available Plugin APIs

| Interface | Functionality |
|----------|---------------|
| `IPEPlugin` | Core plugin entry point |
| `IPXModel` | Model manipulation |
| `IPXBone` | Bone hierarchy access |
| `IPXMorph` | Expression and morph control |
| `IPXPhysics` | Access to rigid bodies and constraints |

---
?? 
## Conclusion

While **PMX Editor lacks official developer documentation**, this guide provides a comprehensive overview of:
- How to develop plugins using C#, Python, and SlimDX
- Bone system internals and IK chain generation
- Facial morphs and animation systems
- File structure and reverse-engineered PMX specs
- Available tools and active communities

We recommend further research into:
- SDEF vector calculations
- Rigging interoperability with Blender and Unity
- Improving internationalization support
- Developing a standardized SDK for plugin developers


### Performance Optimization

- Parallel processing is implemented for certain operations:
  ```csharp
  Parallel.ForEach(view.GetSelectedVertexIndices(), vx => {
      // Parallel logic
  });
  ```
 