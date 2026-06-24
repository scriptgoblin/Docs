# Scene Forge — Documentation

**Version 1.0**
Publisher: Script Goblin

---

## Overview

Scene Forge is a scene management tool for Unity that replaces the default workflow of hunting through the Project window. It gives you a unified place to browse, organize, and load scenes — both in the Editor and at runtime during development.

---

## Installation

1. Import the Scene Forge package from the Unity Asset Store
2. Open **Window → Script Goblin → Scene Forge**

No additional setup is required. Scene Forge automatically discovers all scenes in your project.

---

## Editor Window

Open via **Window → Script Goblin → Scene Forge**.

### Scenes Tab

Displays all scenes in your project, grouped into **Favorites** and **All Scenes**.

| Icon | Meaning |
|------|---------|
| ★ (yellow) | Scene is a favorite |
| ▶ (accent) | Scene is the Play Mode start scene |
| ● (green) | Scene is currently loaded |
| #N | Build index |
| — | Not in Build Settings |

**Dropdown actions (▾):**
- **Open (Single)** — close all open scenes and load this one
- **Open Additive** — load alongside currently open scenes
- **Close Scene** — unload this scene (prompts to save if dirty)
- **Set as Play Mode Start Scene** — Unity will always start from this scene in Play Mode
- **Clear Play Mode Start Scene** — revert to normal Play Mode behavior
- **Add / Remove from Build Settings**
- **Move Up / Down in Build** — reorder Build Settings entries
- **Add / Remove from Favorites**
- **Move Favorite Up / Down**
- **Show in Project** — ping the scene asset

**Search bar** — filters scenes by name in real time.

**↺ button** — manually refresh the scene list.

---

### Workspaces Tab

Workspaces save your current set of open scenes and restore them in one click. Useful for switching between gameplay, UI, and level editing contexts.

**Saving a workspace:**
1. Open the scenes you want to save
2. Type a name (and optional description) in the input field
3. Click **Save**

**Restoring a workspace:**
- Click **▾** next to a workspace → **Restore**
- If you have unsaved changes, Unity will prompt you to save first

**Workspace dropdown actions:**
- **Restore** — restore the saved scene layout
- **Rename** — edit the workspace name inline
- **Move Up / Down** — reorder workspaces
- **Delete** — permanently remove the workspace

---

### Settings Tab

#### Runtime Overlay

Toggle **Auto-show overlay (SCENE_FORGE_OVERLAY)** to enable automatic spawning of the runtime scene switcher in Development Builds.

When disabled, call from code instead:

```csharp
// Show the overlay
SceneForgeOverlay.Show();

// Hide the overlay
SceneForgeOverlay.Hide();
```

---

## Runtime Overlay

The runtime overlay is a draggable in-game panel for switching scenes without leaving Play Mode. It only appears in **Development Builds** (never in release builds).

**Features:**
- Lists all scenes from Build Settings
- Load Additive toggle
- Drag to reposition — position is saved across sessions
- Close button minimizes to a small **SF** pip in the corner; click to restore

**Auto-init:** Enable **SCENE_FORGE_OVERLAY** in the Settings tab to spawn automatically on play. Disable it to call `SceneForgeOverlay.Show()` manually.

**Platforms:** PC, Mac, Linux, iOS, Android. Touch targets are automatically enlarged on mobile platforms.

---

## Support

Email: nregoblin@gmail.com
