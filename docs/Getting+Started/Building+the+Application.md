# 🛠️ Getting Started – Building the Application

Follow these steps to configure and compile **spiradiospectrumplaywin32** on Windows using Visual Studio.

## 1. Prerequisites

- **Visual Studio 2019** or later with support for **PlatformToolset v145**.

This project uses the v145 toolset for both Debug and Release configurations .

- Ensure you have cloned or unpacked the following directories at the project root:
- `lib-src/portaudio-2021/portaudio_vs2026`
- `lib-src/freeimage_vs2026`
- `lib-src/bass24`
- `spiwavsetlib`
- A working Windows SDK (included with Visual Studio).

```card
{
    "title": "Missing Dependencies",
    "content": "Verify that all third-party libraries under lib-src and the spiwavsetlib folder exist before building."
}
```

## 2. Open the Project

1. Launch **Visual Studio**.
2. Select **File → Open → Project/Solution**.
3. Navigate to the repository root and open:

```text
   spiradiospectrumplaywin32.vcxproj
```

1. Confirm the **Solution Platforms** dropdown shows **Win32** and the **Solution Configurations** dropdown lists **Debug** and **Release**.

## 3. Configure Include Paths

The project file defines the following include directories. Ensure each path exists relative to the project root:

```xml
<AdditionalIncludeDirectories>
  .\lib-src\portaudio-2021\portaudio_vs2026\include;
  .\lib-src\freeimage_vs2026\Source\;
  .\spiwavsetlib;
  .\lib-src\bass24\c
</AdditionalIncludeDirectories>
```

– Debug & Release

- `.\lib-src\portaudio-2021\portaudio_vs2026\include`
- `.\lib-src\freeimage_vs2026\Source`
- `.\spiwavsetlib`
- `.\lib-src\bass24\c`

## 4. Configure Library Dependencies

The linker settings reference these static libraries. Verify each file at the specified path:

| Library | Debug Path | Release Path |
| --- | --- | --- |
| **FreeImage** | `.\lib-src\freeimage_vs2026\Dist\x32\FreeImage.lib` | `.\lib-src\freeimage_vs2026\Dist\x32\FreeImage.lib` |
| **PortAudio** | `.\lib-src\portaudio-2021\portaudio_vs2026\build\msvc\Win32\debug\portaudio_x86.lib` | `.\lib-src\portaudio-2021\portaudio_vs2026\build\msvc\Win32\Release\portaudio_x86.lib` |
| **BASS** | `.\lib-src\bass24\c\bass.lib` | `.\lib-src\bass24\c\bass.lib` |
| **spiwavsetlib** | `.\spiwavsetlib_vs2026u\debug\spiwavsetlib_vs2019.lib` | `.\spiwavsetlib_vs2026u\Release\spiwavsetlib_vs2019.lib` |
| **WinMM** | `winmm.lib` | `winmm.lib` |
| **Shlwapi** | `Shlwapi.lib` | `Shlwapi.lib` |


These entries come from the `<AdditionalDependencies>` sections in Debug and Release respectively  .

## 5. Select Build Configuration

- In the **Standard toolbar**, choose either **Debug** or **Release**.
- Confirm **Platform** is **Win32**.
- The project is pre-configured to use the **Unicode** character set .

## 6. Build the Solution

1. Go to **Build → Build Solution** (or press **F7**).
2. Monitor the **Output** window for compilation and link messages.
3. On success, the executable `spiradiospectrumplaywin32.exe` will be generated in:
4. `.\Debug\` (for Debug build)
5. `.\Release\` (for Release build)

## 7. Verify Linked Dependencies

- Ensure no **LNK1104** errors occur.
- If a library is missing, double-check its path in **Project Properties → Linker → Input → Additional Dependencies**.
- Confirm **Additional Library Directories** (if modified) under **Linker → General**.

---

You are now ready to run **spiradiospectrumplaywin32**, pass command-line arguments, and enjoy real-time 24-bit waveform and spectrum visualizations over a JPEG background. 🚀