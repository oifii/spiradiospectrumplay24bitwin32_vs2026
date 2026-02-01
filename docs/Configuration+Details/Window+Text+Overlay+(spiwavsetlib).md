# Configuration Details - Window Text Overlay (spiwavsetlib)

This section describes how the custom **spiwavsetlib** library integrates with the Win32 message loop to render overlay text in a transparent `STATIC` control above the waveform/spectrum background. It covers creation, sizing, font metrics, color handling, and shutdown.

## Purpose 🎯

spiwavsetlib manages text layout and rendering within a Win32 `STATIC` control.

It ensures overlay text

- adapts to window resizing,
- uses correct font metrics,
- respects alignment and color settings.

## Global Configuration Variables

These globals define alignment, color, and sizing parameters:

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| global_staticalignment | `int` | `SS_LEFT` | Text alignment: `SS_LEFT`, `SS_CENTER`, or `SS_RIGHT` |
| global_statictextcolor | `COLORREF` | `RGB(0,0,255)` (blue) | Overlay text color |
| global_fontwidth | `int` | `-1` | Average character width (computed at paint time) |
| global_fontheight | `int` | `24` | Font height (from `CreateFontW`) |
| global_staticwidth | `int` | `-1` | Width of static control (set on resize) |
| global_staticheight | `int` | `-1` | Height of static control (set on resize) |


## Initialization on WM_CREATE

When the main window receives `WM_CREATE`, a transparent `STATIC` control is created:

```cpp
case WM_CREATE: {
    HWND hStatic = CreateWindowEx(
        WS_EX_TRANSPARENT,
        L"STATIC",
        L"",
        WS_CHILD | WS_VISIBLE | global_staticalignment,
        0, 100, 100, 100,
        hWnd,
        (HMENU)IDC_MAIN_STATIC,
        GetModuleHandle(NULL),
        NULL
    );
    if (hStatic == NULL)
        MessageBox(hWnd, L"Could not create static text.", L"Error", MB_OK | MB_ICONERROR);
    SendMessage(hStatic, WM_SETFONT, (WPARAM)global_hFont, MAKELPARAM(FALSE, 0));
    // ... spectrum initialization ...
} break;
```

- **WS_EX_TRANSPARENT** makes background image visible through the control.
- **global_staticalignment** supplies left/center/right (`SS_LEFT`/`SS_CENTER`/`SS_RIGHT`).

## Resizing on WM_SIZE

Upon `WM_SIZE`, the static control is resized to cover the client area, and the text layout subsystem is configured:

```cpp
case WM_SIZE: {
    RECT rc;
    GetClientRect(hWnd, &rc);
    HWND hStatic = GetDlgItem(hWnd, IDC_MAIN_STATIC);

    global_staticwidth  = rc.right;
    global_staticheight = rc.bottom;

    WavSetLib_Initialize(
        global_hwnd,
        IDC_MAIN_STATIC,
        global_staticwidth,
        global_staticheight,
        global_fontwidth,
        global_fontheight,
        global_staticalignment
    );

    SetWindowPos(
        hStatic, NULL,
        0, 0,
        global_staticwidth,
        global_staticheight,
        SWP_NOZORDER
    );
    // ... spectrum bitmap recreation ...
} break;
```

- **WavSetLib_Initialize** sets up text wrapping, scrolling, and alignment.
- Control is positioned at `(0,0)` and sized to client dimensions.

## Font Metrics on WM_PAINT 🖋️

During `WM_PAINT`, the application calculates and stores font metrics for precise layout:

```cpp
case WM_PAINT: {
    HDC hdc = BeginPaint(hWnd, &ps);
    // draw background image...
    HFONT hOld = (HFONT)SelectObject(hdc, global_hFont);
    TEXTMETRIC tm;
    GetTextMetrics(hdc, &tm);
    global_fontwidth  = tm.tmAveCharWidth;
    global_fontheight = tm.tmHeight;
    SelectObject(hdc, hOld);
    EndPaint(hWnd, &ps);
} break;
```

- **GetTextMetrics** retrieves average character width and line height.
- These metrics feed into `spiwavsetlib` to ensure text fits and wraps correctly.

## Text Color & Background Handling

The `WM_CTLCOLORSTATIC` handler ensures transparent background and correct text color:

```cpp
case WM_CTLCOLORSTATIC: {
    SetBkMode((HDC)wParam, TRANSPARENT);
    SetTextColor((HDC)wParam, global_statictextcolor);
    return (INT_PTR)::GetStockObject(NULL_PEN);
} break;
```

- **TRANSPARENT** background mode prevents control from erasing the JPEG background.
- **global_statictextcolor** (default blue) paints overlay text.

## Shutdown on WM_DESTROY 🛑

When the window is destroyed, the text layout subsystem is cleanly terminated:

```cpp
case WM_DESTROY: {
    // ... timer and DC cleanup ...
    WavSetLib_Terminate();
    // ... other shutdown tasks ...
    PostQuitMessage(0);
} break;
```

- **WavSetLib_Terminate** releases any resources or threads allocated by the library.

## API Reference

| Function | Description |
| --- | --- |
| WavSetLib_Initialize(hwnd, id, w, h, fw, fh, align) | Configure text control `id` in `hwnd` for width `w`, height `h`, font metrics `fw×fh`, and alignment `align`. |
| WavSetLib_Terminate() | Shutdown text subsystem and free resources. |


🛠️ Both functions reside in **spiwavsetlib** and must be called during `WM_SIZE` and `WM_DESTROY`, respectively.

---

By following this configuration, the application ensures overlay text remains crisp, correctly aligned, and integrated seamlessly over the dynamic visualization background.