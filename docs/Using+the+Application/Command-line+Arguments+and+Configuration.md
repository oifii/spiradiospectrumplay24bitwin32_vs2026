# Using the Application – Command-line Arguments and Configuration

This section describes how to drive **spiradiospectrumplay24bitwin32** via positional command-line arguments. All parameters are parsed in `WinMain` using `CommandLineToArgvA/W`, with sensible defaults defined in global variables and overridden when supplied.

## Parsing Mechanism

The application entry point, `_tWinMain`, splits the raw command line into an `argv` array and assigns each positional argument (1-based) to a corresponding global variable. Defaults are established at variable definition time and only changed if an argument is present.

```cpp
LPSTR *szArgList;
int   nArgs;
// Split ANSI and Unicode command lines
szArgList  = CommandLineToArgvA(GetCommandLineA(), &nArgs);
LPWSTR *szArgListW = CommandLineToArgvW(GetCommandLineW(), &nArgs);

if (nArgs > 1) global_filename                = szArgList[1];
if (nArgs > 2) global_duration_sec            = atof(szArgList[2]);
if (nArgs > 3) global_sleeptimeperstation_sec = atof(szArgList[3]);
…
// Unicode copies for class name and title
if (nArgs > 13) wcscpy(szWindowClass, szArgListW[13]);
if (nArgs > 14) wcscpy(szTitle,       szArgListW[14]);
// Run-on scripts
if (nArgs > 15) global_begin  = szArgList[15];
if (nArgs > 16) global_end    = szArgList[16];
// Spectrum and color settings
if (nArgs > 17) specmode = atoi(szArgList[17]);
if (nArgs > 19) global_bands = atoi(szArgList[19]);
// Color overrides (RGB triplets)
if (nArgs > 19) global_bgRGBQUAD.rgbRed   = atoi(szArgList[19]);
… etc.
LocalFree(szArgList);
LocalFree(szArgListW);
```

*Snippet from `_tWinMain` showing argument parsing*

## Positional Arguments Reference 🎛️

Below is the complete list of **27** supported positional parameters. Omit any you don’t need; defaults will apply.

| Pos | Name | Variable | Type | Default | Description |
| --- | --- | --- | --- | --- | --- |
| 1 | Audio input path/URL | `global_filename` | `string` | `"radiostations.txt"` | File or stream URL list to read stations from |
| 2 | Playback duration (seconds) | `global_duration_sec` | `float` | `180.0` | Total run time; negative ⇒ infinite |
| 3 | Per-station sleep time (seconds) | `global_sleeptimeperstation_sec` | `float` | `30.0` | Delay between station switches |
| 4 | Window X position | `global_x` | `int` | `100` | Top-left corner X coordinate |
| 5 | Window Y position | `global_y` | `int` | `200` | Top-left corner Y coordinate |
| 6 | Window width | `global_xwidth` | `int` | `400` | Window client area width |
| 7 | Window height | `global_yheight` | `int` | `400` | Window client area height |
| 8 | Window alpha (0–255) | `global_alpha` | `BYTE` | `200` | Transparency level applied via `SetLayeredWindowAttributes` |
| 9 | Title bar display flag | `global_titlebardisplay` | `int` (0 / 1) | `1` | `1` ⇒ standard overlapped, `0` ⇒ borderless pop-up |
| 10 | Menu bar display flag | `global_menubardisplay` | `int` (0 / 1) | `0` | Show menu bar |
| 11 | Accelerator keys active flag | `global_acceleratoractive` | `int` (0 / 1) | `0` | Enable keyboard accelerators |
| 12 | Font height (pixels) | `global_fontheight` | `int` | `24` | Height of `Segoe Script` font |
| 13 | Window class name (Unicode) | `szWindowClass` | `TCHAR[MAX_LOADSTRING]` | `"spiradiospectrumplaywin32class"` | Custom Win32 class identifier |
| 14 | Window title (Unicode) | `szTitle` | `TCHAR[MAX_LOADSTRING]` | `"spiradiospectrumplaywin32title"` | Text in title bar |
| 15 | Startup script/command path | `global_begin` | `string` | `"begin.ahk"` | Executed via `ShellExecuteA` on launch |
| 16 | Exit script/command path | `global_end` | `string` | `"end.ahk"` | Executed via `ShellExecuteA` on exit |
| 17 | Initial spectrum mode (0–18) | `specmode` | `int` | `4` | Controls visualization algorithm |
| 18 | Log-spectrum band count | `global_bands` | `int` | `20` | Number of frequency bands for logarithmic modes |
| 19 | Background color – red component | `global_bgRGBQUAD.rgbRed` | `BYTE` | `127` | Solid background fill |
| 20 | Background color – green component | `global_bgRGBQUAD.rgbGreen` | `BYTE` | `0` |  |
| 21 | Background color – blue component | `global_bgRGBQUAD.rgbBlue` | `BYTE` | `127` | Default purple background |
| 22 | Channel-1 color – red component | `global_c1RGBQUAD.rgbRed` | `BYTE` | `255` |  |
| 23 | Channel-1 color – green component | `global_c1RGBQUAD.rgbGreen` | `BYTE` | `0` | Default red waveform |
| 24 | Channel-1 color – blue component | `global_c1RGBQUAD.rgbBlue` | `BYTE` | `0` |  |
| 25 | Channel-2 color – red component | `global_c2RGBQUAD.rgbRed` | `BYTE` | `0` |  |
| 26 | Channel-2 color – green component | `global_c2RGBQUAD.rgbGreen` | `BYTE` | `0` | Default blue waveform |
| 27 | Channel-2 color – blue component | `global_c2RGBQUAD.rgbBlue` | `BYTE` | `255` |  |


*Defaults defined at declaration time*  and *overrides applied after parsing* .

## Example Usage

```bash
spiradiospectrumplay24bitwin32.exe \
  stations.txt    300    20   50   50   800   600  180  1  1  1  28 \
  MyWinClass     "My Spectrum Player"  init.bat  cleanup.bat \
  5   25   0  64  128   255 0 0  0 255 0  0 0 255
```

- Plays stations from `stations.txt` for 300 s.
- Sleeps 20 s between stations.
- Window at (50, 50), size 800 × 600, alpha 180.
- Title bar, menu bar, and accelerators enabled.
- Font size 28px.
- Custom class and title.
- Runs `init.bat` on start, `cleanup.bat` on exit.
- Uses spectrum mode 5 with 25 bands.
- Overrides background to RGB(0,64,128), channel 1 to RGB(255,0,0), channel 2 to RGB(0,255,0).

## Default Color Palette 🎨

- Background: **Purple** (R=127, G=0, B=127)
- Channel 1: **Red** (255, 0, 0)
- Channel 2: **Blue** (0, 0, 255)

```card
{
    "title": "Color Defaults",
    "content": "Omit RGB args to use the built-in purple background and red/blue channel colors."
}
```

---

By leveraging positional arguments, the application stays flexible and scriptable. Simply adjust or omit parameters to customize playback, window layout, transparency, visual modes, and colors without recompiling.