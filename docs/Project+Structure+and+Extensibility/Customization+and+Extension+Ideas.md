# Project Structure and Extensibility – Customization and Extension Ideas

This section highlights the key extension points in **spiradiospectrumplaywin32**, guiding developers on how to tailor and expand the application. Each idea points to the relevant code locus and suggests concrete steps to implement alternatives or enhancements.

## 🎨 Alternative Spectrum Visualizations

Extend the `UpdateSpectrum` timer callback by adding new `specmode` branches. The `specmode` variable controls which visualization to render into the `specbuf` bitmap buffer .

| specmode | Visualization | Data source |
| --- | --- | --- |
| ---------: | --------------------------------------------- | ---------------- |
| 0 | “Normal” FFT bar graph | FFT bins |
| 1 | Logarithmic FFT (band‐based accumulation) | FFT bins |
| 3 | Waveform (line drawing per sample) | PCM samples |
| 4–6, 7–10, 11–14, 15–18 | Variations of filled/colored waveforms | PCM samples |


**To add a new visualization** (e.g., circular spectrum, peak-hold meter):

1. **Choose** an unused `specmode` value (e.g., `20`).
2. **Insert** an `else if` branch in `UpdateSpectrum`:

```cpp
   } else if (specmode == 20) {
       // Compute circular radii from FFT data
       float fft[2048];
       BASS_ChannelGetData(global_chan, fft, BASS_DATA_FFT2048);
       renderCircularSpectrum(fft, SPECWIDTH, SPECHEIGHT, specbuf);
   }
```

1. **Implement** helper functions (e.g., `renderCircularSpectrum`) that fill `specbuf` based on FFT or waveform arrays.
2. **Update** command-line parsing to accept the new `specmode` .

This pattern lets you **reuse** the existing bitmap creation and palette logic in `CreateBitmapToDrawSpectrum`.

## ⌨️ Keyboard Shortcuts via Accelerator Table

By default, **mouse clicks** (WM_LBUTTONUP/WM_RBUTTONUP) swap `specmode`. To complement this, define **keyboard shortcuts**:

1. **Enable accelerators** when parsing the `global_acceleratoractive` flag in WinMain:

```cpp
   if (global_acceleratoractive) {
       hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_SPIWAVWIN32));
   } else {
       hAccelTable = NULL;
   }
```

1. **Add** entries in `spiradiospectrumplaywin32.rc` under the `ACCELERATORS` section, for example:

```rc
   BEGIN ACCELERATORS
       "3",     ID_SPECTYPE_WAVEFORM,   VIRTKEY, NOINVERT
       "4",     ID_SPECTYPE_CIRCULAR,   VIRTKEY, NOINVERT
   END
```

1. **Handle** accelerator commands in the `WM_COMMAND` switch of `WndProc`:

```cpp
   case ID_SPECTYPE_WAVEFORM:
       specmode = 3;
       break;
   case ID_SPECTYPE_CIRCULAR:
       specmode = 20;
       break;
```

This approach ensures **consistent** handling of keyboard and mouse inputs through `TranslateAccelerator` in the main message loop .

## 📻 Enhanced Station Management

Currently, the app reads a simple `radiostations.txt` list via `StartGlobalProcess`, launched in a background thread . To support **advanced playlists**:

- **Parse** M3U, PLS or XSPF formats using a lightweight parser before calling `OpenURL`.
- **Expose** a simple UI (e.g., a drop-down or dialog) to **select** stations manually.
- **Allow** dynamic station addition/removal at runtime via keyboard shortcuts or context menu.

By modularizing `StartGlobalProcess`, you can plug in any playlist format with minimal changes to the playback loop.

## ⚙️ Configuration Beyond Positional Arguments

The `_tWinMain` function currently maps positional arguments to settings (window position, size, alpha, `specmode`, etc.) :

```cpp
if (nArgs > 17) specmode = atoi(szArgList[17]);
if (nArgs > 18) global_bands = atoi(szArgList[18]);
```

To create a **more user-friendly** configuration:

- **Parse** an INI/JSON/YAML file using a small library (e.g., [inih], [nlohmann/json]).
- **Switch** to a **named options** parser (Boost.Program_options or CLI11) for `--width=400`, `--specmode=3`.
- **Fall back** to command-line when no config file is found.

This decouples UI settings from rigid positional ordering, improving discoverability.

## 📝 Integrating Additional Text Overlays

A **static text control** displays metadata using **spiwavsetlib**. In `WM_CREATE`, the control is created and font applied :

```cpp
HWND hStatic = CreateWindowEx(
    WS_EX_TRANSPARENT, L"STATIC", L"",
    WS_CHILD | WS_VISIBLE | global_staticalignment,
    0, 0, 100, 100, hWnd, (HMENU)IDC_MAIN_STATIC,
    GetModuleHandle(NULL), NULL
);
SendMessage(hStatic, WM_SETFONT, (WPARAM)global_hFont, FALSE);
```

Then in `WM_SIZE` the library is initialized:

```cpp
WavSetLib_Initialize(
    global_hwnd, IDC_MAIN_STATIC,
    global_staticwidth, global_staticheight,
    global_fontwidth, global_fontheight,
    global_staticalignment
);
```

**To display** song title, bitrate or station name:

1. **Use** BASS metadata APIs inside your buffer‐monitor timer (`WM_TIMER`) or sync callback:

```cpp
   const char* tags = BASS_ChannelGetTags(global_chan, BASS_TAG_META);
   if (tags) {
       StatusAddText(tags);  // appends to static control
   }
```

1. **Call** `SendMessage(hStatic, WM_SETTEXT, 0, (LPARAM)textBuf);` to update the overlay.

1. **Leverage** `spiwavsetlib` features (scrolling, color changes) to animate text.

---

Each of these customization points adheres to the project’s modular design—**isolated** callbacks (`UpdateSpectrum`, `StartGlobalProcess`, window messages)—making them straightforward to implement without altering core playback or rendering logic. By following the patterns above, you can enrich visualizations, improve user interaction, and tailor the app to new use cases.