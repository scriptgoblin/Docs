# Icon Forge — User Guide

Icon Forge is a Unity Editor tool for rendering prefab icons as transparent PNG files, ready for use in game UI, asset libraries, or sprite atlases.

---

## Getting Started

1. Open the tool via **Window → Icon Forge**.
2. Click **Create New Settings** to create an `IconForgeSettings` asset, or drag an existing one into the **Settings Asset** field.
3. Drag prefabs into the **Prefabs** list to add them.
4. Switch to the **Export** tab, choose an export mode, and click the render button.

---

## Interface Overview

The window is divided into three main tabs — **Main**, **Rendering**, and **Export** — with a live preview at the top.

### Preview

The preview shows a real-time render of the currently selected object against a transparent (checkerboard) background.

| Input | Action |
|---|---|
| Drag | Orbit camera |
| Cmd/Ctrl + Drag | Pan camera |
| Scroll wheel | Zoom in / out |
| Drag (Rendering tab) | Rotate directional light |

---

## Main Tab

### Object Sub-tab

#### Animation

If the selected prefab has an **Animator** or legacy **Animation** component, an Animation section appears at the top of the Object sub-tab.

| Control | Description |
|---|---|
| Clip | Dropdown to select which animation clip to sample |
| Frame | Slider (0–1 normalized time) to scrub to the desired pose. The current frame number and total frame count are shown to the right. |

The selected pose is applied to the preview immediately and is baked into the render when exporting. Each prefab remembers its own clip and frame independently.

#### Object Transform

Expand **Object Transform** to adjust the selected prefab's position, rotation, and scale within the scene. Each field has an **R** button to reset it to its default value.

### Camera Sub-tab

Controls the camera for the currently selected prefab. Camera settings are saved per-entry, so each prefab can have its own angle and distance.

| Setting | Description |
|---|---|
| Orthographic | Switches between perspective and orthographic projection |
| Ortho Size | Size of the orthographic view volume |
| Rotation | Camera orbit angles (X = vertical tilt, Y = horizontal orbit) |
| Distance | Distance from the camera to the focal point |

**Center Camera** — automatically fits the camera to the selected object's bounds.  
**Center All** — fits the camera for every entry at once.  
**Apply Camera to All Entries** — copies the current entry's camera settings to all other entries.

### Prefabs List

- **Drag prefabs** from the Project window onto the list to add them.
- **Drag rows** to reorder.
- Click an entry's name to **select** it for preview and editing.
- Click **Remove** to delete an entry.

---

## Rendering Tab

### Resolution

Sets the render texture size in pixels. Both width and height can be set independently. Values are clamped between 32 and 4096.

### Unlit Mode

When enabled, the directional light is disabled and objects are rendered with their full albedo colour, unaffected by lighting. Useful for flat UI icons.

### Light Settings

Available when Unlit Mode is off. Controls the single directional light used for all renders.

| Setting | Description |
|---|---|
| Color | Tint of the directional light |
| Intensity | Brightness (0–5) |
| Rotation | Euler angles of the light direction |

> You can also drag in the preview area while on the Rendering tab to rotate the light interactively.

---

## Export Tab

### Output Path

The folder where rendered files are saved. Can be a path inside `Assets/` (recommended) or an absolute path on disk. Files saved outside `Assets/` will not appear in the Project window automatically.

Use **Browse** to pick a folder with a dialog.

The Export tab has three sub-tabs — **Individual Icons**, **Spritesheet**, and **Turntable**. The Spritesheet tab is disabled until at least 2 objects are in the list. The Turntable tab is disabled until at least 1 object is in the list.

### Individual Icons

Renders each prefab as its own PNG file.

| Setting | Description |
|---|---|
| File Name | Template for the output file name. Supports tags (see below). |

**Tags for individual icons:**

| Tag | Value |
|---|---|
| `[name]` | Prefab name |
| `[index]` | Zero-padded position in the list |
| `[count]` | Total number of entries |
| `[resolution]` | E.g. `512x512` |
| `[width]` | Render width in pixels |
| `[height]` | Render height in pixels |
| `[date]` | Date in `yyyy-MM-dd` format |
| `[timestamp]` | Time in `HHmmss` format |
| `[preset]` | Name of the settings asset |

**Render Selected** — renders only the currently selected entry.  
**Copy to Clipboard** — copies the current preview render to the system clipboard (macOS and Windows).  
**Render All** — renders every entry and saves the files.

When a file already exists you will be asked whether to replace it or keep both (auto-rename with a numeric suffix). You can apply the choice to all remaining conflicts in one step.

### Spritesheet

Renders all prefabs into a single PNG arranged in a grid.

| Setting | Description |
|---|---|
| Columns | Number of columns in the grid |
| Padding (px) | Gap between sprites and around the edges |
| File Name | Template for the output file name. Supports tags (see below). |
| Auto-Setup Sprite Import | Configures the exported PNG as a Sprite (Multiple) asset with named slices automatically |

**Tags for spritesheet file name:**

| Tag | Value |
|---|---|
| `[count]` | Total number of sprites |
| `[resolution]` | Sheet dimensions, e.g. `2048x1024` |
| `[width]` | Sheet width in pixels |
| `[height]` | Sheet height in pixels |
| `[date]` | Date in `yyyy-MM-dd` format |
| `[timestamp]` | Time in `HHmmss` format |
| `[preset]` | Name of the settings asset |

When **Auto-Setup Sprite Import** is enabled, Icon Forge configures the texture importer immediately after export: it sets the texture type to Sprite (Multiple), assigns each sprite a name matching its source prefab, and sets the pivot to center. No manual slicing in the Sprite Editor required. This feature requires the export path to be inside the `Assets/` folder.

### Turntable

Renders a single prefab from multiple angles by orbiting the camera evenly around it, then packs all frames into a spritesheet grid. Useful for generating rotation sequences for 2D games or preview animations.

Select the prefab to render from the Prefabs list before switching to the Turntable tab.

| Setting | Description |
|---|---|
| Frames | Total number of rotation steps. The camera is distributed evenly across 360°. |
| Columns | Number of columns in the output grid |
| Padding (px) | Gap between frames and around the edges |
| File Name | Template for the output file name. Supports tags (see below). |
| Auto-Setup Sprite Import | Configures the exported PNG as a Sprite (Multiple) asset with sequentially numbered slices |

**Tags for turntable file name:**

| Tag | Value |
|---|---|
| `[name]` | Prefab name |
| `[resolution]` | Sheet dimensions, e.g. `2048x1024` |
| `[width]` | Sheet width in pixels |
| `[height]` | Sheet height in pixels |
| `[date]` | Date in `yyyy-MM-dd` format |
| `[timestamp]` | Time in `HHmmss` format |
| `[preset]` | Name of the settings asset |

When **Auto-Setup Sprite Import** is enabled, slices are named `prefabname_00`, `prefabname_01`, and so on. The animation pose set in the Object sub-tab is held fixed across all rotation frames.

---

## Settings Asset

Icon Forge stores all configuration — prefab list, camera settings, export options — in an `IconForgeSettings` ScriptableObject. You can have multiple settings assets for different projects or icon sets. Switch between them by dragging a different asset into the **Settings Asset** field at the top of the window.

Settings assets are fully compatible with version control and can be shared across a team.

---

## Tips

- **Per-prefab camera settings** let you save the perfect angle for each object and re-render it consistently at any time.
- **Center Camera** is the fastest way to set up a new prefab — it fits the camera to the object's renderer bounds automatically.
- **Undo/Redo** (Cmd/Ctrl+Z) works for all camera, transform, and settings changes.
- For UI icons with a separate background, keep the export background transparent (default) and composite in your UI system.
- Use the `[preset]` tag in file names to identify which settings asset produced a given output file.
- For turntable exports, set your preferred animation frame in the Object sub-tab before rendering — the pose is locked in for all rotation steps.

---

## Support

If you run into any issues, have a feature request, or just want to say hello — reach out at **nregoblin@gmail.com**. Quick replies guaranteed.
