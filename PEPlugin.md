

# PMX Editor Plugin Development: PEPlugin.dll Technical Reference

This document provides a comprehensive guide to the `PEPlugin.dll`, the core library for developing plugins for PMX Editor. It covers essential interfaces, classes, and best practices for creating, managing, and deploying plugins.

## Table of Contents

* [1. Introduction to PMX Editor Plugins](#1-introduction-to-pmx-editor-plugins)
    * [1.1. What is a Plugin?](#11-what-is-a-plugin) 
    * [1.2. Plugin Capabilities](#12-plugin-capabilities) 
    * [1.3. Development Requirements](#13-development-requirements) 
* [2. PEPlugin: Standard (.NET) Plugins](#2-peplugin-standard-net-plugins)
    * [2.1. Creating a PEPlugin](#21-creating-a-peplugin) 
        * [2.1.1. Project Setup](#211-project-setup) 
        * [2.1.2. Core Implementation (`IPEPlugin` / `PEPluginClass`)](#212-core-implementation-ipeplugin--pepluginclass) 
    * [2.2. Building and Deploying PEPlugins](#22-building-and-deploying-peplugins) 
    * [2.3. Running PEPlugins](#23-running-peplugins) 
    * [2.4. Plugin Development Tips](#24-plugin-development-tips)
        * [2.4.1. Execution Logic (`Run` Method)](#241-execution-logic-run-method) 
        * [2.4.2. Editing PMX Data](#242-editing-pmx-data) 
        * [2.4.3. Using Builders for Object Creation](#243-using-builders-for-object-creation) 
        * [2.4.4. Configuring Plugin Options](#244-configuring-plugin-options) 
        * [2.4.5. Interface Implementation Caution](#245-interface-implementation-caution) 
        * [2.4.6. Understanding PMX Data Structure](#246-understanding-pmx-data-structure) 
        * [2.4.7. Exception Handling](#247-exception-handling) 
        * [2.4.8. Updating Editor Views](#248-updating-editor-views) 
        * [2.4.9. Creating Plugins with Windows Forms](#249-creating-plugins-with-windows-forms) 
        * [2.4.10. Compatibility Notes](#2410-compatibility-notes) 
* [3. C Plugins (Compact Plugins)](#3-c-plugins-compact-plugins)
    * [3.1. What is a C Plugin?](#31-what-is-a-c-plugin) 
    * [3.2. Development Requirements for C Plugins](#32-development-requirements-for-c-plugins) 
    * [3.3. Comparison: PE Plugins vs. C Plugins](#33-comparison-pe-plugins-vs-c-plugins) 
    * [3.4. Important Notes for C Plugin Development](#34-important-notes-for-c-plugin-development)
        * [3.4.1. Registrar Class](#341-registrar-class) 
        * [3.4.2. PMX File Handling](#342-pmx-file-handling) 
        * [3.4.3. PMX Interface (`IPXPmx`) Limitations](#343-pmx-interface-ipxpmx-limitations) 
        * [3.4.4. Primitive Creation](#344-primitive-creation) 
        * [3.4.5. Builder Usage (`PXCBuilder`)](#345-builder-usage-pxcbuilder) 
        * [3.4.6. Builder/Primitive Initialization](#346-builderprimitive-initialization) 
        * [3.4.7. Custom Class Implementation (`IPXCPlugin`, `MarshalByRefObject`)](#347-custom-class-implementation-ipxcplugin-marshalbyrefobject) 
    * [3.5. Advanced Debugging for C Plugins](#35-advanced-debugging-for-c-plugins) 
    * [3.6. View Events in C Plugins](#36-view-events-in-c-plugins) 
    * [3.7. System and View Control in C Plugins](#37-system-and-view-control-in-c-plugins) 
* [4. UI Model Functionality (Primarily with C Plugins)](#4-ui-model-functionality-primarily-with-c-plugins)
    * [4.1. About UI Models](#41-about-ui-models) 
    * [4.2. Creating and Managing UI Models](#42-creating-and-managing-ui-models) 
        * [4.2.1. Creation (`PXCBridge.RegisterUIModel`)](#421-creation-pxcbridgeregisteruimodel) 
        * [4.2.2. Disposal (`IPXUIModel.Release`, `SetAutoRelease`)](#422-disposal-ipxuimodelrelease-setautorelease) 
    * [4.3. UI Model Features and Performance](#43-ui-model-features-and-performance) 
        * [4.3.1. Unsupported Features](#431-unsupported-features) 
        * [4.3.2. Enhanced Features](#432-enhanced-features) 
    * [4.4. UI Model Basic Properties](#44-ui-model-basic-properties) 
    * [4.5. Deformation and Updates](#45-deformation-and-updates) 
    * [4.6. Special Textures (In-Memory Bitmaps)](#46-special-textures-in-memory-bitmaps) 
    * [4.7. UI Model Events](#47-ui-model-events) 
    * [4.8. Event Helper Classes (`PXUIModelHelper`)](#48-event-helper-classes-pxuimodelhelper) 
* [5. Core API Reference (`PEPlugin.dll`)](#5-core-api-reference-peplugindll)
    * [5.1. `IPEPlugin` - Core Plugin Interface](#51-ipeplugin---core-plugin-interface) 
    * [5.2. `IPERunArgs` and `IPEPluginHost` - Host and Runtime Context](#52-iperunargs-and-ipepluginhost---host-and-runtime-context) 
    * [5.3. `IPEFormConnector` - Main Editor Interface](#53-ipeformconnector---main-editor-interface) 
    * [5.4. Color Theme Accessors](#54-color-theme-accessors) 
    * [5.5. `PEPluginClass` - Abstract Base Class](#55-pepluginclass---abstract-base-class) 
    * [5.6. `PEPluginOption` - Startup Behavior](#56-pepluginoption---startup-behavior) 
    * [5.7. `PEStaticBuilder` - Builder Access](#57-pestaticbuilder---builder-access) 
* [6. VMD (Motion Data) Plugin Development](#6-vmd-motion-data-plugin-development)
    * [6.1. Overview of VMD Interfaces](#61-overview-of-vmd-interfaces) 
    * [6.2. `IPEVmd` - Core VMD Interface](#62-ipevmd---core-vmd-interface) 
    * [6.3. Keyframe Types (`IPEVmdBoneKey`, `IPEVmdMorphKey`, etc.)](#63-keyframe-types-ipevmdbonekey-ipevmdmorphkey-etc) 
* [7. View and UI Interaction API](#7-view-and-ui-interaction-api)
    * [7.1. `IPEPMDViewConnector` - Main View Connector](#71-ipepmdviewconnector---main-view-connector) 
    * [7.2. `IPEObjectSelectConnector` - Selection Management](#72-ipeobjectselectconnector---selection-management) 
* [8. Model Component Interfaces (Brief Overview)](#8-model-component-interfaces-brief-overview)
    * [8.1. `IPXModelInfo` and `PXCModelInfo`](#81-ipxmodelinfo-and-pxcmodelinfo) 
    * [8.2. `IPXBody` and `PXCBody` (Rigid Bodies)](#82-ipxbody-and-pxcbody-rigid-bodies) 
    * [8.3. `IPXSoftBody` and `PXCSoftBody`](#83-ipxsoftbody-and-pxcsoftbody) 
    * [8.4. Selection Groups (`IPXSelectBoneGroup`, `IPXSelectMorphGroup`)](#84-selection-groups-ipxselectbonegroup-ipxselectmorphgroup) 
* [9. General Best Practices](#9-general-best-practices) 

---

## 1. Introduction to PMX Editor Plugins

### 1.1. What is a Plugin?
Plugins are external extensions that add new features and functionalities to PMX Editor.

### 1.2. Plugin Capabilities
With plugins, you can:
* Acquire, modify, and update PMX model data.
* Perform partial operations on the main editor and various views like PmxView.
* Launch custom Windows Forms for editing assistance, as .NET functions are generally available.
* *Note: Some functions may have limitations, and specifications can change without notice*.

### 1.3. Development Requirements
To develop plugins, you need an environment capable of creating class libraries targeting **.NET Framework 4.0 or later** (e.g., Microsoft Visual Studio .NET 2010, Microsoft Visual C# 2010 Express Edition). Alternatively, the CSScript plugin allows plugin creation and usage without a dedicated development environment.

---

## 2. PEPlugin: Standard (.NET) Plugins

These are the primary type of plugins developed using the .NET Framework.

### 2.1. Creating a PEPlugin

#### 2.1.1. Project Setup
1.  Create a new project as a **Class Library**.
2.  Add a reference to **`PEPlugin.dll`** from your PMX Editor installation directory.

#### 2.1.2. Core Implementation (`IPEPlugin` / `PEPluginClass`)
Your main plugin class must either:
* Implement the `PEPlugin.IPEPlugin` interface directly.
* Or, more commonly, derive from the `PEPlugin.PEPluginClass` base class and override necessary methods like `Run(IPERunArgs args)`.

The `IPEPlugin` interface requires properties for `Name`, `Version`, `Description`, and `Option`, along with the `Run(IPERunArgs args)` method.

### 2.2. Building and Deploying PEPlugins
1.  Write the necessary logic for your plugin.
2.  Build the project. It's recommended to set the target platform to **AnyCPU**. *Note: x86 target will not work*.
3.  Copy the resulting `*.dll` file into the `_plugin` folder (specifically, the `_plugin\User\` subdirectory is common) of your PMX Editor installation.
4.  Distribute only the DLL file. The `PEPlugin.dll` generated in your output directory during the build is not needed for distribution.
5.  This plugin DLL should be used **only** for creating plugins for PMX Editor.

### 2.3. Running PEPlugins
1.  After placing the DLL in the `_plugin` folder, restart PMX Editor.
2.  The plugin will typically appear under the **[Edit] → [Plugin]** menu, from where it can be executed.
3.  Plugins can also be configured to run automatically at PMX Editor startup.
4.  Menu registration can be customized.
5.  For direct execution without menu registration, use **[Edit] → [Directly Run Plugin DLL].
    * *Note:* Once a .NET plugin DLL is loaded, it remains in memory until PMX Editor is closed. You must restart the editor to reload an updated DLL.

### 2.4. Plugin Development Tips

#### 2.4.1. Execution Logic (`Run` Method)
Implement the core logic of your plugin within the `void Run(IPERunArgs args)` method. Access to the PMX Editor host environment is provided through the `args` parameter (specifically `args.Host`).

#### 2.4.2. Editing PMX Data
To edit PMX model data:
1.  Get a duplicate of the current PMX model state: `IPXPmx pmxData = args.Host.Connector.Pmx.GetCurrentState();`. Editing this `IPXPmx` object does not immediately affect the editor.
2.  Modify the `pmxData` object as needed (e.g., `pmxData.ModelInfo.ModelName = "New Name";`).
3.  Update the PMX Editor with the modified data: `args.Host.Connector.Pmx.Update(pmxData);`. Overloads exist for partial updates (e.g., updating only model info).
4.  Refresh the editor's display: `args.Host.Connector.Form.UpdateList(PEPlugin.Pmd.UpdateObject.Header);`.

#### 2.4.3. Using Builders for Object Creation
You cannot directly instantiate PMX objects using `new`. Instead, use dedicated builder classes.
* PMX-related objects are created via `PEPlugin.PEStaticBuilder.Pmx`. For example, `args.Host.Connector.PmxBuilder.CreateVmd()` or `PXCBuilder` for C Plugins.

#### 2.4.4. Configuring Plugin Options
Plugin options like startup behavior and menu registration are managed via `IPEPluginOption` (interface) and `PEPluginOption` (concrete class).
* **`Bootup`**: (bool) Run plugin on PMX Editor startup.
* **`RegisterMenu`**: (bool) Register the plugin in the menu.
* **`RegisterMenuText`**: (string) Custom text for the menu item; if empty, the plugin's `Name` is used.
Implement the `IPEPlugin.Option` property or set `m_option` in the constructor if deriving from `PEPluginClass`.

```csharp
// Example for PEPluginClass-derived class
public class MyPluginWithOptions : PEPluginClass
{
    public MyPluginWithOptions() : base()
    {
        m_option = new PEPluginOption(false, true, "My Custom Plugin Menu Text"); // Bootup=false, RegisterMenu=true [cite: 1876]
    }
    // ... other overrides ...
}
````

#### 2.4.5. Interface Implementation Caution

It is **strongly recommended not to implement or inherit any PMX Editor interfaces other than `PEPlugin.IPEPlugin`** in your custom classes. Future updates to other interfaces could break your plugin.

#### 2.4.6. Understanding PMX Data Structure

The `IPXPmx` interface provides access to the model's components, which generally correspond to the tabs in the PMX Editor UI. Key properties include:

- `Header` (`IPXHeader`)
    
- `ModelInfo` (`IPXModelInfo`)
    
- `Vertex` (`IList<IPXVertex>`)
    
- `Material` (`IList<IPXMaterial>`)
    
- `Bone` (`IList<IPXBone>`)
    
- `Morph` (`IList<IPXMorph>`)
    
- `Node` (Display Frames) (`IList<IPXNode>`)
    
- `Body` (Rigid Bodies) (`IList<IPXBody>`)
    
- `Joint` (`IList<IPXJoint>`)
    
- `SoftBody` (`IList<IPXSoftBody>`) Refer to the plugin DLL using an object browser for detailed member descriptions.
    

#### 2.4.7. Exception Handling

Exceptions thrown within a plugin **cannot be caught by PMX Editor** and may cause the editor to crash. **Always wrap your plugin's logic, especially in the `Run()` method and any multi-threaded operations, in `try-catch` blocks**.

#### 2.4.8. Updating Editor Views

After modifying PMX data, the editor's views might not update immediately because internal data changes haven't propagated. Use methods like the following to force updates:

- `IPXPmxViewConnector.UpdateModel()`
    
- `IPEFormConnector.UpdateList()`
    
- `IPXPmxViewConnector.UpdateView()`
    

#### 2.4.9. Creating Plugins with Windows Forms

Plugins can create and display custom Windows Forms.

A common pattern for a form-based plugin:

1. Configure the plugin to run at startup (optional).
    
2. Add a Form to your class library.
    
3. In the `Run()` method, differentiate logic for startup calls (`args.IsBootup`) versus menu calls.
    
    - On startup, create and store the form instance.
        
    - On menu click, toggle the form's visibility.
        
4. Prevent the form from fully closing when the user clicks the 'X' button by handling the `FormClosing` event and setting `e.Cancel = true` and `this.Visible = false`.
    
5. Override the plugin's `Dispose()` method to properly close and dispose of the form instance.
    

#### 2.4.10. Compatibility Notes

- The plugin functionality is inherited from the older PMD Editor. View-related components retain legacy compatibility.
    
- Plugins require at least a 64-bit environment. Plugins compiled with "AnyCPU" should continue to work.
    
- If using SlimDX DLLs, setting "Specific Version" to `False` for the DLL reference in your project might help maintain compatibility if PMX Editor updates its bundled SlimDX version.
    

---

## 3. C Plugins (Compact Plugins)

C Plugins offer a more limited but unloadable alternative to standard PEPlugins.

### 3.1. What is a C Plugin?

"C Plugin" stands for **"Compact Plugin"**. It's not a native C/C++ plugin. Unlike PEPlugins, C Plugin DLLs can be unloaded at runtime, a key improvement over standard .NET DLL behavior. However, C Plugins offer fewer functions, focusing on the minimum required for PMX data editing. They use the same `IPXPmx` interface for data editing, with some limitations. Some UI and View events are extended for C Plugins. Installation is the same as for PEPlugins.

### 3.2. Development Requirements for C Plugins

The requirements are the same as for PEPlugins: a .NET Framework 4.0 (or later) compatible class library development environment. Visual Studio 2013 or newer Express Edition is recommended for effective process attachment debugging.

### 3.3. Comparison: PE Plugins vs. C Plugins

|   |   |   |
|---|---|---|
|**Feature**|**C Plugin Details**|**PE Plugin Details**|
|**Functionality**|Limited (basic PMX get/update, selection access, event handling)|Full .NET capabilities, broader API access|
|**PMX Acquisition/Update**|Bulk update only; internal conversion makes it slower. Not for frequent updates.|Supports granular updates, generally faster.|
|**Update Process**|PMX, Form, and View updated at once on PMX update; can be heavy. View-only updates possible.|Separate calls for PMX, Form, View updates.|
|**DLL Updates**|Can be updated while editor runs (via Direct Plugin Execution only).|Requires editor restart to reload DLL.|
|**Debugging**|More efficient due to reload capability without editor restart.|Requires editor restarts for DLL reloading.|
|**Boot Execution**|Not supported.|Supported via `IPEPluginOption`.|
|**Menu Registration**|No specific registration flag; if menu text and plugin name are empty, it won't register.|Configurable via `IPEPluginOption`.|

### 3.4. Important Notes for C Plugin Development

#### 3.4.1. Registrar Class

C Plugins require a specific registrar class: `PXCPlugin.Register` derived from `RegisterBase`. See sample code for details.

#### 3.4.2. PMX File Handling

C Plugins don't use the editor's internal processes for reading/writing external PMX files, so normalization might differ. They cannot handle PMD files or PMX version 1 files.

#### 3.4.3. PMX Interface (`IPXPmx`) Limitations

The `Primitive` property within `IPXPmx` (for primitive creation) is not available in C Plugins.

#### 3.4.4. Primitive Creation

Use the external `PXCPrimitiveBuilder` to create primitives in C Plugins. Example: `PXCBridge.PrimitiveBuilder.AddBox(...)`.

#### 3.4.5. Builder Usage (`PXCBuilder`)

Do not use the general-purpose `PEStaticBuilder`. Use the dedicated `PXCBuilder` instead.

#### 3.4.6. Builder/Primitive Initialization

Builders and primitive tools via `PXCBridge` require initialization through a connector. If deriving from `PXCPluginClass`, `base.Run()` handles this.

#### 3.4.7. Custom Class Implementation (`IPXCPlugin`, `MarshalByRefObject`)

If implementing `IPXCPlugin` directly (not deriving from `PXCPluginClass`), manual initialization of builders is needed, and your class must inherit from `MarshalByRefObject`.

#### 3.4.8. DLL Exclusivity

C Plugin DLLs and PE Plugin DLLs **cannot be shared** and must be built separately. If a PE Plugin with the same name exists, the C Plugin will be ignored.

### 3.5. Advanced Debugging for C Plugins

Use the **[Edit] → [Run Plugin DLL Directly]** feature. Drag-and-drop your C Plugin DLL into the dialog. If it appears, attach Visual Studio (with breakpoints set) to the PMX Editor process for step debugging. Build in **Debug mode**. The **"Unload" button** allows reloading the C Plugin DLL without restarting PMX Editor, significantly speeding up iterative development.

### 3.6. View Events in C Plugins

Create an event connector via `PXCBridge.CreateEventConnector()`, then get an `IPXViewEventListener` by calling `CreateViewEventListener()`. Supported events include mouse events (Click, DoubleClick, Move, Down, Up, Wheel), key events, and model/selection events (ObjectSelected, ModelUpdated, Undo, Redo).

### 3.7. System and View Control in C Plugins

Obtain system and view controllers:

- `PXCBridge.SystemCtrl()` → `IPXSystemControl`
    
- `PXCBridge.ViewCtrl()` → `IPXViewControl` These provide access to functionalities like executing registered plugins and setting camera viewpoints.
    

---

## 4. UI Model Functionality (Primarily with C Plugins)

UI Models allow using PMX models as interactive UI elements, created via C Plugins.

### 4.1. About UI Models

A PMX model (loaded or created by a plugin) can be placed on the View and combined with plugin event controls to function as a UI element. This combination is called a UI Model.

### 4.2. Creating and Managing UI Models

#### 4.2.1. Creation (`PXCBridge.RegisterUIModel`)

Use `PXCBridge.RegisterUIModel()` to create and register a UI Model. The base PMX data must be prepared beforehand. Changes to the original PMX data post-registration are not reflected. The editor manages registered UI Models; updates require dedicated functions.

#### 4.2.2. Disposal (`IPXUIModel.Release`, `SetAutoRelease`)

Call `IPXUIModel.Release()` directly or use `IPXUIModel.SetAutoRelease()` for automatic disposal when the plugin exits. Manual disposal via `Release()` is generally recommended to avoid conflicts if `SetAutoRelease()` was also used.

### 4.3. UI Model Features and Performance

#### 4.3.1. Unsupported Features

- Physics simulation
- Self-shadow
- Ground shadow

#### 4.3.2. Enhanced Features

- Transformation via world matrix
- Billboard modes (full axis or Y-axis fixed)
- Direct material color updates (non-morph)
- Depth (Z) control
- Front-layer rendering
- In-memory texture placement
- Text rendering capability
- Standard bone transformations and morphing still apply.

### 4.4. UI Model Basic Properties

Includes `Visible`, `Light` (lighting enabled/disabled), `Depth` (Z-depth test enabled/disabled), `TopMost` (render in front), and `SetBillboard()` (billboard mode).

### 4.5. Deformation and Updates

Supports initialization for deformation, world matrix transforms, bone transforms (SetBone, SetBoneScale, etc.), morphs (SetMorph), and material color/edge/flag updates. `UpdateTransform()` applies bone/morph deformations. Performance varies: bone/morph changes are high cost; world matrix/material updates are low cost.

### 4.6. Special Textures (In-Memory Bitmaps)

UI Models can use textures from in-memory `Bitmap` data via `SetBitmapTexture()` and `UpdateBitmapTexture()` (supports 32-bit ARGB only). Toon and sphere maps cannot be modified this way. `PXUIModelHelper.TextControl` assists with text rendering onto bitmaps.

### 4.7. UI Model Events

Mouse events (MouseOver, MouseDown, MouseUp, MouseEnter, MouseLeave, MouseDrag, MouseDragEnd, MouseClick, MouseDoubleClick) can be registered on UI Models using an event listener created via `CreateEventListener()`.

### 4.8. Event Helper Classes (`PXUIModelHelper`)

The `PXUIModelHelper` class offers simplified methods for common event handling, like color changes on mouseover (`SetMouseOverColor`) and object movement via mouse drag (`SetMouseDragMove`). It also provides `CreateTextControl()` for text rendering. _Caution: Repeated calls to helper methods might add duplicate event handlers_.

---

## 5. Core API Reference (`PEPlugin.dll`)

This section details some of the most critical interfaces and classes within `PEPlugin.dll`.

### 5.1. `IPEPlugin` - Core Plugin Interface

The entry point for all plugins, defining essential properties and the `Run` method.

- **Properties**: `Name` (string), `Version` (string), `Description` (string).
    
- **Method**: `Run(IPERunArgs args)` (void).
    

### 5.2. `IPERunArgs` and `IPEPluginHost` - Host and Runtime Context

`IPERunArgs` is passed to `Run()`, providing `Host` (`IPEPluginHost`). `IPEPluginHost` gives access to `Connector` (`IPEFormConnector`).

### 5.3. `IPEFormConnector` - Main Editor Interface

Allows control over the editor UI and data model.

- **File Ops**: `InitializePMX()`, `OpenPMXFile(string path)`, `SavePMXFile(string path)`.
    
- **Selection**: `SelectedVertexIndex`, `SelectedBoneIndex`, etc..
    
- **Undo/Redo**: `Undo()`, `Redo()`, `UndoCount`, `RedoCount`.
    
- **View Control**: `Location` (Point), `TopMost` (bool).
    

### 5.4. Color Theme Accessors

Methods like `GetBoneKindColors()`, `GetExpressionCategoryColors()` allow plugins to use editor theme colors.

### 5.5. `PEPluginClass` - Abstract Base Class

A recommended base for plugins, implementing `IPEPlugin` and `IDisposable`. Provides virtual properties for `Name`, `Version`, `Description`, `Option`, and virtual methods `Run(IPERunArgs args)` and `Dispose()`.

### 5.6. `PEPluginOption` - Startup Behavior

Implements `IPEPluginOption` for configuring `Bootup`, `RegisterMenu`, and `RegisterMenuText`.

### 5.7. `PEStaticBuilder` - Builder Access

Provides static properties `Builder` (`IPEBuilder`) and `ShortBuilder` (`IPEShortBuilder`) for accessing global builder instances.

---

## 6. VMD (Motion Data) Plugin Development

`PEPlugin.dll` provides interfaces for working with VMD motion files within the `PEPlugin.Vmd` namespace.

### 6.1. Overview of VMD Interfaces

Interact with bone animation, morphs, camera, lighting, visibility, and IK states.

### 6.2. `IPEVmd` - Core VMD Interface

Represents a VMD file.

- **Properties**: `ModelName` (string), `Bone` (`IList<IPEVmdBoneKey>`), `Morph` (`IList<IPEVmdMorphKey>`), `Camera` (`IList<IPEVmdCameraKey>`), etc..
    
- **Methods**: `FromFile(string path)`, `ToFile(string path, bool trimKeys)`, `NormalizeKeys()`, `TrimKeys()`.
    

### 6.3. Keyframe Types (`IPEVmdBoneKey`, `IPEVmdMorphKey`, etc.)

- `IPEVmdBoneKey`: Represents a bone keyframe with `BoneIndex`, `Translation`, `Rotation`, and interpolation data (`IPEVmdIPL`).
    
- `IPEVmdMorphKey`: Represents a morph keyframe with `MorphIndex` and `Value` (weight).
    
- Other keys include `IPEVmdCameraKey`, `IPEVmdLightKey`, `IPEVmdSelfShadowKey`, `IPEVmdVisibleIKKey`.
    

---

## 7. View and UI Interaction API

Interfaces for interacting with the 3D view and UI components, under `PEPlugin.View`.

### 7.1. `IPEPMDViewConnector` - Main View Connector

Represents the main view window.

- **Methods**: `UpdateView()`, `UpdateModel()`, `UpdateModel_Vertex()`, `UpdateModel_Bone()`.
    

### 7.2. `IPEObjectSelectConnector` - Selection Management

Access and set selected model elements.

- **Properties**: `SelectedVertexIndex`, `SelectedBoneIndex`, `SelectedMaterialIndex`, etc..
    

---

## 8. Model Component Interfaces (Brief Overview)

`PEPlugin.dll` defines numerous interfaces for accessing all parts of a PMX model.

### 8.1. `IPXModelInfo` and `PXCModelInfo`

Represents model metadata like name and comments in Japanese and English. `PXCModelInfo` is the concrete class with cloning support.

### 8.2. `IPXBody` and `PXCBody` (Rigid Bodies)

Define physical properties for rigid bodies, including name, mode, attached bone, shape, size, mass, friction, etc.. `PXCBody` is the concrete, cloneable class.

### 8.3. `IPXSoftBody` and `PXCSoftBody`

Represent deformable physics objects with properties for collision margin, aero model, various coefficients (VCF, DP, DG, LF, PR, VC, DF, MT, CHR), and anchors (`IPXSoftBodyAnchor`).

### 8.4. Selection Groups (`IPXSelectBoneGroup`, `IPXSelectMorphGroup`)

Interfaces for managing collections of selected bones or morphs, with methods like `Clear()`, `Add(int index)`, `AddRange(int[] indices)`, and `Remove(int index)`. `PXSelectBoneGroup` and `PXSelectMorphGroup` are concrete implementations.

---

## 9. General Best Practices

- **Target Framework and Language**: Use .NET Framework 4.0 or later (specifically 4.7.2/4.8 recommended) and C# 7.3 or earlier.
    
- **Null Safety**: Always check for `null` before accessing objects returned by the API (e.g., `args.Host.Connector.Pmx`).
    
- **Index Bounds Checking**: Validate indices before accessing elements in lists or arrays.
    
- **Exception Handling**: Wrap plugin logic in `try-catch` blocks to prevent crashing the editor.
    
- **Updating Views**: Call appropriate `UpdateModel()` or `UpdateView()` methods after modifying data to refresh the UI.
    
- **Resource Management**: Implement `IDisposable` and override `Dispose()` in `PEPluginClass` if your plugin uses unmanaged resources or forms.
    
- **Avoid Direct Instantiation**: Use provided builders (`PEStaticBuilder.Pmx`, `PXCBuilder`) to create model objects.
    
- **DLL Management**: Reference `PEPlugin.dll` and `SlimDX.dll` correctly. Unblock DLLs if downloaded. Deploy to the `_plugin` folder. Restart the editor to load new/updated .NET plugin DLLs.
    
- **Adhere to Interface Guidelines**: Do not implement PMX Editor interfaces other than `IPEPlugin` in your own classes to maintain forward compatibility.
    

This structured documentation should provide a much clearer and more professional overview for developers looking to work with `PEPlugin.dll`.