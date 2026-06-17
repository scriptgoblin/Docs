# SaveForge — User Guide

**Version 1.0**
For questions or issues: nregoblin@gmail.com

---

## Contents

1. Overview
2. Installation
3. Quick Start
4. Reading and Writing Values
5. Default Values and First-Run Initialization
6. Complex Types and Collections
7. Multiple Save Slots
8. Backends
9. Encryption
10. Compression
11. Schema Versioning and Migration
12. Async Save
13. Auto-Save Component
14. Editor Inspector
15. Cloud Sync
16. Platform Notes
17. Demo Scene
18. Support

---

## 1. Overview

SaveForge is a flexible, PlayerPrefs-style save system for Unity. It stores any JSON-serializable value under a string key, persists data to disk automatically, and gives you full control over when and how saves happen.

The static `SaveForge` class is the main entry point. It works like `PlayerPrefs` — call it from anywhere without managing instances — but supports structs, classes, lists, dictionaries, and every other type that Newtonsoft.Json can handle.

Optional features include AES-256 encryption, GZip compression, schema migration, async writes, an auto-save component, and a cloud sync interface.

---

## 2. Installation

Import the package from the Unity Asset Store. SaveForge requires **Newtonsoft.Json for Unity** (`com.unity.nuget.newtonsoft-json`), which is included with Unity 2022.3 and later. If you are on an older version, install it from the Package Manager before importing SaveForge.

After import, the assembly `ScriptGoblin.SaveForge` is available to all scripts in your project automatically.

---

## 3. Quick Start

```csharp
using ScriptGoblin.SaveForge;

// Write
SaveForge.Set("score", 9999);
SaveForge.Set("username", "Hero");

// Read
int score = SaveForge.Get<int>("score");
string name = SaveForge.Get<string>("username");
```

Save files are written to `Application.persistentDataPath/Saves/SaveData.sav`. The `Saves/` folder is created automatically on first write.

---

## 4. Reading and Writing Values

### Writing

```csharp
SaveForge.Set("coins", 500);
SaveForge.Set("volume", 0.8f);
SaveForge.Set("tutorial_done", true);
```

### Reading

```csharp
int coins   = SaveForge.Get<int>("coins");
float vol   = SaveForge.Get<float>("volume");
bool done   = SaveForge.Get<bool>("tutorial_done");
```

If the key does not exist, `Get<T>` returns `default(T)` — `0` for numbers, `false` for bools, `null` for strings and classes.

### Checking and removing

```csharp
bool exists = SaveForge.HasKey("coins");
SaveForge.Remove("coins");
SaveForge.RemoveAll();                    // wipes the entire save file
```

### Enumerating keys

```csharp
foreach (var key in SaveForge.GetAllKeys())
    Debug.Log(key);
```

### Forcing a flush

By default, each `Set` call writes to disk immediately. If you use throttling (see Section 8), call `Save()` to force a flush:

```csharp
SaveForge.Save();
SaveForge.Save(onComplete: () => Debug.Log("Written to disk."));
```

---

## 5. Default Values and First-Run Initialization

### Reading with a fallback

```csharp
int coins = SaveForge.Get("coins", defaultValue: 100);
```

Returns the stored value if the key exists, otherwise returns `100` without writing anything.

### Initializing on first run

```csharp
int difficulty = SaveForge.GetOrSet("difficulty", 1);
```

If the key exists, returns the stored value. If it does not exist, saves `1` and returns `1`. Subsequent calls return whatever value was last saved. Useful for setting starting values once when the game launches for the first time.

---

## 6. Complex Types and Collections

Any JSON-serializable type works out of the box.

### Structs and classes

```csharp
[System.Serializable]
public struct PlayerStats
{
    public string name;
    public int    level;
    public float  health;
}

// Save
var stats = new PlayerStats { name = "Hero", level = 5, health = 100f };
SaveForge.Set("stats", stats);

// Load
var loaded = SaveForge.Get<PlayerStats>("stats");
Debug.Log(loaded.name); // Hero
```

### Lists and dictionaries

```csharp
var inventory = new List<string> { "sword", "shield", "potion" };
SaveForge.Set("inventory", inventory);

var loaded = SaveForge.Get<List<string>>("inventory");
```

### Checking if a collection contains an item

```csharp
bool hasSword = SaveForge.CollectionContains("inventory", "sword");
```

---

## 7. Multiple Save Slots

Create a separate `SaveSystemJson` instance per slot. Each instance writes to its own file.

```csharp
var slot1 = new SaveSystemJson("Slot1");
var slot2 = new SaveSystemJson("Slot2");

slot1.Set("hero", "Warrior");
slot2.Set("hero", "Mage");

// Files written to:
// Application.persistentDataPath/Saves/Slot1.sav
// Application.persistentDataPath/Saves/Slot2.sav
```

The instances are independent — writing to `slot1` does not affect `slot2` or `SaveForge.Current`.

---

## 8. Backends

`SaveForge.Current` holds the active backend. It defaults to a `SaveSystemJson` instance on first access. You can replace it at any time without changing any `SaveForge.Set` / `SaveForge.Get` call sites.

```csharp
// Swap to a configured JSON backend
SaveForge.Current = new SaveSystemJson(new SaveSystemJsonOptions
{
    FileName  = "MySave",
    Throttle  = true,
    ThrottleInterval = 2.0f
});
```

### Throttling

By default every `Set` triggers an immediate disk write. For high-frequency updates (e.g. position autosave every frame) enable throttling so writes are rate-limited:

```csharp
var options = new SaveSystemJsonOptions
{
    Throttle         = true,
    ThrottleInterval = 1.0f   // at most one write per second
};
SaveForge.Current = new SaveSystemJson(options);
```

`SaveForge.Save()` always bypasses the throttle and writes immediately.

---

## 9. Encryption

Pass an `AesEncryption` instance to protect the save file from tampering. The entire file is encrypted with AES-256-CBC. A random IV is generated per write, so the ciphertext differs each time even when the data has not changed.

```csharp
var options = new SaveSystemJsonOptions
{
    Encryption = new AesEncryption("my-secret-password")
};
SaveForge.Current = new SaveSystemJson(options);

SaveForge.Set("coins", 5000);
// SaveData.sav on disk is now ciphertext — not human-readable
```

The password is compiled into your game. For stronger security, derive it at runtime from a device identifier or a server-issued token rather than embedding a plain string.

### Custom salt

The default salt is `ScriptGoblin.SaveForge`. Change it per-project so save files from different games are not interchangeable:

```csharp
new AesEncryption("my-password", salt: "MyGame.v1")
```

### Implementing your own encryption

Implement `IDataEncryption` and pass it to `SaveSystemJsonOptions.Encryption`:

```csharp
public class MyEncryption : IDataEncryption
{
    public string Encrypt(string plainText) { /* ... */ }
    public string Decrypt(string cipherText) { /* ... */ }
}
```

---

## 10. Compression

Enable GZip compression to reduce file size. Useful for large save files on mobile where storage is limited. Compression is applied before encryption when both are enabled.

```csharp
var options = new SaveSystemJsonOptions
{
    Compressed = true
};
SaveForge.Current = new SaveSystemJson(options);
```

Compression and encryption can be combined:

```csharp
var options = new SaveSystemJsonOptions
{
    Compressed = true,
    Encryption = new AesEncryption("my-password")
};
```

---

## 11. Schema Versioning and Migration

When your game updates and the save data structure changes, migrations let you automatically upgrade old save files without losing player progress.

### Setting up migrations

```csharp
public class RenameCoinsToGold : ISaveDataMigration
{
    public int FromVersion => 1;
    public int ToVersion   => 2;

    public void Migrate(Dictionary<string, string> data)
    {
        if (data.TryGetValue("coins", out var value))
        {
            data["gold"] = value;
            data.Remove("coins");
        }
    }
}
```

### Registering migrations

```csharp
var options = new SaveSystemJsonOptions
{
    CurrentVersion = 2,
    Migrations = new List<ISaveDataMigration>
    {
        new RenameCoinsToGold()
    }
};
SaveForge.Current = new SaveSystemJson(options);
```

On load, SaveForge reads the version stored in the save file and runs only the migration steps needed to bring it up to `CurrentVersion`. A player on v0 going to v3 runs v0→1, v1→2, v2→3 in sequence. A player already on v3 runs nothing.

The version is stored automatically in the save file — you do not need to manage it manually.

---

## 12. Async Save

`SaveAsync` serializes data on the main thread, then writes to disk on a background thread. This removes the file I/O stutter from the main thread, which is noticeable on mobile with large save files.

```csharp
// Fire and forget
_ = SaveForge.SaveAsync();

// With callback
await SaveForge.SaveAsync(onComplete: () => ShowSavedToast());

// In a coroutine-style context
await SaveForge.SaveAsync();
Debug.Log("Save complete");
```

`SaveAsync` requires a C# `async` method or `UniTask` on the calling side. Do not call it from a background thread — `Time.realtimeSinceStartup`, used internally for throttle tracking, is main-thread only.

---

## 13. Auto-Save Component

`SaveForgeAutoSave` is a MonoBehaviour that periodically flushes the current backend to disk. Attach it to any persistent GameObject.

```
GameObject (DontDestroyOnLoad recommended)
└── SaveForgeAutoSave
      Interval: 60          ← seconds between saves, 0 to disable
      Save On Scene Load: ☐
      Save On Scene Unload: ☑
```

The component saves to whatever `SaveForge.Current` is at the time. If you replace `SaveForge.Current` at runtime, make sure to do so before the component's `Start()` runs, or assign it in `Awake()`.

---

## 14. Editor Inspector

Open the inspector via **Window → Script Goblin → Save Forge**.

The inspector reads the save file directly from disk and displays all stored keys and values. It works independently of Play Mode — you can inspect saves after a build has run on the same machine.

### Features

- **File picker** — dropdown lists every file in the `Saves/` folder; click ⟳ to rescan after a new file appears
- **Browse** — search keys, see value previews with type badges
- **Edit** — click Edit on any row to modify the raw JSON value inline
- **Delete** — remove individual keys or wipe the entire file
- **File size** — shown next to the key count

### Encrypted or compressed files

The inspector reads plain JSON only. If the file is encrypted or compressed, it will show a warning. To inspect encrypted saves, temporarily disable encryption, run the game to write a plain file, then inspect it.

---

## 15. Cloud Sync

SaveForge includes an `ICloudSync` interface for uploading and downloading save files. SaveForge does not implement a specific cloud provider — the right choice depends on your platform (Unity Gaming Services, Steam Cloud, Game Center, a custom REST backend, etc.).

### Implementing a provider

```csharp
public class MyCloudSync : ICloudSync
{
    public async Task UploadAsync(string localPath, Action<bool> onComplete = null)
    {
        var bytes = File.ReadAllBytes(localPath);
        var success = await MyBackend.Upload("savefile", bytes);
        onComplete?.Invoke(success);
    }

    public async Task DownloadAsync(string localPath, Action<bool> onComplete = null)
    {
        var (bytes, success) = await MyBackend.Download("savefile");
        if (success) File.WriteAllBytes(localPath, bytes);
        onComplete?.Invoke(success);
    }

    public async Task<bool> ExistsAsync() =>
        await MyBackend.Exists("savefile");
}
```

### Registering and using

```csharp
var options = new SaveSystemJsonOptions
{
    CloudSync = new MyCloudSync()
};
var json = new SaveSystemJson(options);
SaveForge.Current = json;

// Upload after saving
await json.UploadAsync(onComplete: success => Debug.Log($"Upload: {success}"));

// Download on launch (overwrites local file)
await json.DownloadAsync(onComplete: success =>
{
    if (success) Debug.Log("Cloud save loaded.");
});
```

---

## 16. Platform Notes

| Platform | Status | Notes |
|---|---|---|
| Windows | ✅ Full support | |
| macOS | ✅ Full support | |
| Linux | ✅ Full support | |
| iOS | ✅ Full support | `Saves/` folder is automatically marked `NSURLIsExcludedFromBackupKey` — see below |
| Android | ✅ Full support | Internal storage, no permissions required |

### iOS backup behaviour

By default, SaveForge marks the `Saves/` folder with Apple's no-backup flag. This means save files are **not** backed up to iCloud, which is correct for reconstructible data (scores, game state) and required by Apple's data storage guidelines for cache-like content.

If your save data should be backed up (e.g. paid unlocks, account data), remove the `#if UNITY_IOS` block from `SaveSystemJson.cs` or call `Device.SetNoBackupFlag` selectively on individual files instead.

---

## 17. Demo Scene

Open `Assets/ScriptGoblin/SaveForge/Demo/SaveForgeDemo.unity` and press Play. All output appears in the Console window. The scene demonstrates:

- Basic type read/write
- Default values and `GetOrSet` first-run initialization
- Structs and collections
- Multiple save slots
- Async save with callback
- AES-256 encryption
- GZip compression
- Schema migration
- Key removal

---

## 18. Support

For questions or issues, contact **nregoblin@gmail.com** — quick replies guaranteed.
