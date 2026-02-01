## Getting Started – System Requirements

This section outlines the **prerequisites** for building and running **SpiraRadioSpectrumPlay24bit** on Windows. Ensure your environment meets each requirement before proceeding.

| Requirement | Details |  |  |
| --- | --- | --- | --- |
| 🖥️  **Operating System** | Windows desktop OS (Win32 API target).<br>- Uses `<windows.h>` for core APIs.<br>- Employs **layered windows** for transparency (`WS_EX_LAYERED`, `SetLayeredWindowAttributes`).<br>- Relies on **multimedia timers** (`timeSetEvent`, `timeKillEvent`).<br>- Renders via **GDI** (HDC, `CreateDIBSection`, `StretchDIBits`). |  |  |
| 🛠️  **IDE & Toolset** | Visual Studio with C++ support.<br>Project configured for **PlatformToolset v145** in both Debug and Release . |  |  |
| 🎯  **Build Target** | **32-bit** only (Win32).<br>Project defines two configurations: `Debug | Win32` and `Release | Win32` . |
| 📦  **Third-Party Libraries** | Must be present (headers + libs) under `lib-src/`:<br><br>• **BASS** (audio engine) in `.\lib-src\bass24\c`<br>• **PortAudio** in `.\lib-src\portaudio-2021\portaudio_vs2026\include`<br>• **FreeImage** in `.\lib-src\freeimage_vs2026\Source`<br><br>Linked via `.vcxproj` settings ﹣ see below . |  |  |
| 🔧  **Custom Helper Library** | `**spiwavsetlib_vs2019.lib**` provided in `spiwavsetlib_vs2026u/`:<br>– Debug: `spiwavsetlib_vs2026u\debug\spiwavsetlib_vs2019.lib`<br>– Release: `spiwavsetlib_vs2026u\Release\spiwavsetlib_vs2019.lib` . |  |  |


### Visual Studio Configuration

Ensure your `.vcxproj` contains the following snippet for both configurations:

```xml
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'" Label="Configuration">
  <PlatformToolset>v145</PlatformToolset>
  <CharacterSet>Unicode</CharacterSet>
  <!-- ... -->
</PropertyGroup>
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'" Label="Configuration">
  <PlatformToolset>v145</PlatformToolset>
  <CharacterSet>Unicode</CharacterSet>
  <!-- ... -->
</PropertyGroup>
```

### Include & Linker Paths

The project references all libraries via **AdditionalIncludeDirectories** and **AdditionalDependencies**:

```xml
<ClCompile>
  <AdditionalIncludeDirectories>
    .\lib-src\portaudio-2021\portaudio_vs2026\include;
    .\lib-src\freeimage_vs2026\Source;
    .\spiwavsetlib;
    .\lib-src\bass24\c;
    %(AdditionalIncludeDirectories)
  </AdditionalIncludeDirectories>
</ClCompile>
<Link>
  <AdditionalDependencies>
    .\lib-src\freeimage_vs2026\Dist\x32\FreeImage.lib;
    winmm.lib;
    \spiwavsetlib_vs2026u\debug\spiwavsetlib_vs2019.lib;
    \lib-src\portaudio-2021\portaudio_vs2026\build\msvc\Win32\debug\portaudio_x86.lib;
    \lib-src\bass24\c\bass.lib;
    Shlwapi.lib;
    %(AdditionalDependencies)
  </AdditionalDependencies>
</Link>
```

### Sample Directory Layout

```bash
.
├─ lib-src/
│  ├─ bass24/
│  │  └─ c/
│  │     └─ bass.lib
│  ├─ freeimage_vs2026/
│  │  ├─ Source/
│  │  └─ Dist/
│  │     └─ x32/FreeImage.lib
│  └─ portaudio-2021/
│     └─ portaudio_vs2026/
│        ├─ include/
│        └─ build/
│           └─ msvc/Win32/portaudio_x86.lib
└─ spiwavsetlib_vs2026u/
   ├─ debug/
   │  └─ spiwavsetlib_vs2019.lib
   └─ Release/
      └─ spiwavsetlib_vs2019.lib
```

---

Ensure all paths match your local workspace. Once these **system requirements** are in place, you can successfully compile and run the application.