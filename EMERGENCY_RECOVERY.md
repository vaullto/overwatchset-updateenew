# Emergency Recovery Guide — Accidentally Deleted Visual Studio Project (Windows)

This guide covers step-by-step recovery options for Windows users who have accidentally deleted local Visual Studio project files.

---

## 1. Immediate Damage Control — Do This First

**Stop all disk activity as fast as possible.**

- **Do not save, download, or install anything** on the drive where the project was stored (usually `C:`).
- **Pause or disable OneDrive / cloud sync** to prevent remote copies from being overwritten:
  - Right-click the OneDrive icon in the system tray → **Pause syncing**.
- **Disconnect from the network** if you suspect automatic sync or backup tools might push deletions to cloud storage.
- **Do not run disk-intensive applications** (games, large downloads, build systems) that could overwrite free sectors containing your deleted files.
- **Do not defragment or compact the drive.**

The reason: when files are deleted on NTFS, the data is not immediately erased — only the directory entry is removed. The sectors remain recoverable until Windows reuses them for new writes.

---

## 2. Recovery Options (Try in This Order)

### Option A — Recycle Bin

The simplest recovery. Deleted files from Explorer or the `del` command (without `/f /s`) usually land here.

1. Double-click **Recycle Bin** on the Desktop.
2. Search or scroll for your project folder or `.sln` / `.csproj` files.
3. Right-click → **Restore**.

The files are returned to their original location.

---

### Option B — OneDrive Recycle Bin (if the project was inside your OneDrive folder)

OneDrive keeps its own cloud-side Recycle Bin for 30 days (93 days for personal plans with extra storage).

1. Open a browser and go to [https://onedrive.live.com](https://onedrive.live.com) and sign in.
2. In the left pane click **Recycle bin**.
3. Find your project folder, right-click → **Restore**.
4. Alternatively, in File Explorer right-click the OneDrive tray icon → **View online** → **Recycle bin**.

---

### Option C — Previous Versions / File History

Windows can restore previous snapshots of folders if **File History** or **System Protection** (Volume Shadow Copy) was enabled.

**Via File Explorer (Previous Versions):**

1. Navigate to the *parent* folder of your deleted project (e.g., `C:\Users\<YourName>\source\repos`).
2. Right-click the parent folder → **Properties** → **Previous Versions** tab.
3. Select the most recent snapshot that predates the deletion → click **Open** to preview or **Restore** to recover.

**Via File History (Settings):**

1. Press **Win + I** → **Update & Security** → **Backup** (Windows 10) or **System** → **Storage** → **Advanced storage settings** → **Backup options** (Windows 11).
2. Click **More options** → **Restore files from a current backup**.
3. Browse to your project folder and click the green **Restore** button.

---

### Option D — Git Remote (GitHub / Azure DevOps / GitLab / Bitbucket)

If your project was tracked by Git and pushed to a remote, this is the fastest and most complete recovery.

```powershell
# Navigate to where you want to restore the project
cd C:\Users\<YourName>\source\repos

# Clone the repository back from the remote
git clone https://github.com/<owner>/<repo>.git

# If you only need the latest commit
git clone --depth 1 https://github.com/<owner>/<repo>.git
```

If you had local commits that were *not* pushed, those commits are gone unless recovered via one of the other methods above. After restoring files from disk, you can re-attach them to the remote:

```powershell
cd C:\Users\<YourName>\source\repos\<ProjectFolder>
git init
git remote add origin https://github.com/<owner>/<repo>.git
git fetch origin
git checkout main   # or the branch you were on
```

---

### Option E — Windows File Recovery (winfr)

`winfr` is Microsoft's free command-line recovery tool for deleted files. It works best the sooner you run it after deletion.

**Install Windows File Recovery** (if not already installed):

- Open the Microsoft Store and search for **Windows File Recovery**, or run:
  ```powershell
  winget install "Windows File Recovery"
  ```

**Basic syntax:**

```
winfr <source-drive>: <destination-drive-or-folder> [/mode] [/n <filter>]
```

> ⚠️ Always recover to a **different drive** than the source to avoid overwriting the data you are trying to recover.

---

#### Example Commands for Visual Studio Projects Under `C:\Users\`

**Recover all files from a specific project folder (Regular mode — recently deleted, NTFS):**

```powershell
winfr C: D:\Recovery /regular /n \Users\<YourName>\source\repos\<ProjectName>\
```

**Recover all `.sln` and `.csproj` files anywhere under your user profile:**

```powershell
winfr C: D:\Recovery /regular /n \Users\<YourName>\*.sln
winfr C: D:\Recovery /regular /n \Users\<YourName>\*.csproj
```

**Recover all source files (`.cs`) from a project (Regular mode):**

```powershell
winfr C: D:\Recovery /regular /n \Users\<YourName>\source\repos\<ProjectName>\*.cs
```

**Extensive mode** (use when Regular mode finds nothing — scans raw clusters, slower):

```powershell
winfr C: D:\Recovery /extensive /n \Users\<YourName>\source\repos\<ProjectName>\
```

**Recover all Visual Studio-related file types at once (Extensive mode):**

```powershell
winfr C: D:\Recovery /extensive /n \Users\<YourName>\*.sln /n \Users\<YourName>\*.csproj /n \Users\<YourName>\*.cs /n \Users\<YourName>\*.vb /n \Users\<YourName>\*.xaml /n \Users\<YourName>\*.config
```

After recovery, your files will be in a folder under `D:\Recovery\` named after the source drive and timestamp. Open Visual Studio and re-add or re-open the recovered `.sln` file.

---

## 3. Additional Tips

| Situation | Recommended action |
|---|---|
| Project was on an SSD | Act immediately — SSDs may TRIM free sectors quickly; use `winfr` in `/extensive` mode. |
| Project was on an HDD | Recovery window is longer; still minimize disk writes. |
| Visual Studio auto-save / crash recovery | Check `%LOCALAPPDATA%\Temp\` and `%APPDATA%\Microsoft\VisualStudio\` for `.suo`, `.user`, and temporary backup files. |
| NuGet packages missing after recovery | Run `dotnet restore` or **Build → Restore NuGet Packages** in Visual Studio; packages are re-downloaded from nuget.org. |
| Git index / `.git` folder corrupted | Run `git fsck` inside the recovered project folder to detect and repair object-level corruption. |

---

## 4. Prevention

- **Enable File History**: Connect an external drive and set it up in **Settings → Backup**.
- **Push to a remote regularly**: Even private free repos on GitHub are unlimited.
- **Enable OneDrive backup** for the `Documents` and `source\repos` folders.
- **Create a System Restore point** before major changes: `sysdm.cpl` → **System Protection** → **Create**.
