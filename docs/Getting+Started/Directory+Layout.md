# Getting Started – Directory Layout

The **directory layout** outlines how the project organizes source code, third-party libraries, custom modules, and resources. This structure simplifies builds in Visual Studio, manages dependencies, and keeps the codebase maintainable.

## 📁 Project Root

At the top level you’ll find the core application files and the Visual Studio solution. These define the entry point, configuration, and global headers.

| File / Folder | Purpose |
| --- | --- |
| **spiradiospectrumplaywin32.vcxproj** | Visual C++ project file; build configurations for Debug / Release (Win32) |
| **spiradiospectrumplaywin32.cpp** | Main source implementing `_tWinMain`, audio playback, and rendering loops |
| **stdafx.h**, **stdafx.cpp** | Precompiled header setup for faster builds |
| **targetver.h** | Defines the minimum required Windows platform |
| **spiradiospectrumplaywin32.h** | Application-wide declarations and resource identifiers |
| **spiradiospectrumplaywin32.rc** | Resource script (dialogs, menus, icons) |
| **small.ico**, **spiradiospectrumplaywin32.ico** | Application icons used by the executable |
| **lib-src/** | Third-party library sources (see below) |
| **spiwavsetlib/** | WAV/text helper library sources |
| **spiwavsetlib_vs2026u/** | Prebuilt WAV helper binaries |


```bash
oifii/
└── spiradiospectrumplaywin32_vs2026/
    ├── spiradiospectrumplaywin32.vcxproj
    ├── spiradiospectrumplaywin32.cpp
    ├── stdafx.h
    ├── stdafx.cpp
    ├── targetver.h
    ├── spiradiospectrumplaywin32.h
    ├── spiradiospectrumplaywin32.rc
    ├── small.ico
    ├── spiradiospectrumplaywin32.ico
    ├── lib-src/
    ├── spiwavsetlib/
    └── spiwavsetlib_vs2026u/
```

## 📁 lib-src

Third-party dependencies live under **lib-src**. Each subfolder contains the source and build outputs for the corresponding library.

| Subfolder | Contents |
| --- | --- |
| **bass24/c/** | BASS audio library C headers and source files |
| **portaudio-2021/portaudio_vs2026/** | PortAudio source, headers, and VS2026 project |
| **freeimage_vs2026/** | FreeImage source (Source/) and distribution binaries (Dist/) |


- **Include paths** in Debug/Release point to:
- `.\lib-src\portaudio-2021\portaudio_vs2026\include`
- `.\lib-src\freeimage_vs2026\Source`
- `.\lib-src\bass24\c`
- **Linker inputs** reference:
- `.\lib-src\freeimage_vs2026\Dist\x32\FreeImage.lib`
- `.\lib-src\portaudio-2021\portaudio_vs2026\build\msvc\Win32\debug\portaudio_x86.lib`
- `.\lib-src\bass24\c\bass.lib`

## 📁 spiwavsetlib & spiwavsetlib_vs2026u

This custom module handles WAV file parsing and text rendering. It is consumed by the main application via include and linker settings.

- **spiwavsetlib/**

Source code for the WAV helper library; included in the VC++ project’s Additional Include Directories .

- **spiwavsetlib_vs2026u/**

Prebuilt binaries for both configurations:

- `debug/spiwavsetlib_vs2019.lib`
- `Release/spiwavsetlib_vs2019.lib`

Linker entries reference these libs for Debug and Release builds .

## 🎨 Resources

All static assets and resources are co-located with the root:

- **Icons**
- `spiradiospectrumplaywin32.ico` (application icon)
- `small.ico` (small icon variant)

- **Resource Script**
- `spiradiospectrumplaywin32.rc` defines dialogs, menus, icon resources

> **Tip:** Keep all graphical assets in the root to simplify the `LoadImage` calls in `MyRegisterClass` and `InitInstance`.