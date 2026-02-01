## Using the Application – Window Appearance and Controls

This section describes how the application window is configured, rendered, and controlled at runtime. It covers window chrome settings, layered transparency, background image rendering, overlay text, and mouse interactions.

### Window Style Configuration

The window style adapts based on the **global_titlebardisplay** flag and applies layered transparency for smooth desktop overlays.

#### Title Bar and Border

- Controlled by `global_titlebardisplay` (0 = off, 1 = on)
- When **on**, the window uses the standard overlapped style:

```cpp
  hWnd = CreateWindow(
    szWindowClass, szTitle,
    WS_OVERLAPPEDWINDOW,
    global_x, global_y,
    global_xwidth, global_yheight,
    NULL, NULL, hInstance, NULL
  );
```

- When **off**, the window is borderless and always visible:

```cpp
  hWnd = CreateWindow(
    szWindowClass, szTitle,
    WS_POPUP | WS_VISIBLE,
    global_x, global_y,
    global_xwidth, global_yheight,
    NULL, NULL, hInstance, NULL
  );
```

#### Layered Transparency

- Enables per-pixel opacity via the **WS_EX_LAYERED** extended style
- Applies the alpha value from `global_alpha` (0–255)
- Invokes `SetLayeredWindowAttributes` immediately after creation:

```cpp
  SetWindowLong(hWnd, GWL_EXSTYLE,
    GetWindowLong(hWnd, GWL_EXSTYLE) | WS_EX_LAYERED
  );
  SetLayeredWindowAttributes(hWnd,
    0, global_alpha, LWA_ALPHA
  );
```

| Parameter | Purpose | Default |
| --- | --- | --- |
| `global_titlebardisplay` | Toggle window chrome | 1 |
| `global_alpha` | Overall window opacity (0–255) | 200 |
| `global_x, global_y` | Initial window position (pixels) | 100, 200 |
| `global_xwidth, global_yheight` | Initial window size (pixels) | 400×400 |


---

### Background Image Rendering 🖼️

On startup, **FreeImage** loads `background.jpg` into `global_dib`. Each `WM_PAINT` event redraws it, scaled to fill the client area.

1. **Loading** (in `InitInstance`):

```cpp
   global_dib = FreeImage_Load(
     FIF_JPEG, "background.jpg", JPEG_DEFAULT
   );
```

1. **Painting** (in `WM_PAINT`):

```cpp
   HDC hdc = BeginPaint(hWnd, &ps);
   SetStretchBltMode(hdc, COLORONCOLOR);
   StretchDIBits(
     hdc, 0, 0,
     global_imagewidth, global_imageheight,
     0, 0,
     FreeImage_GetWidth(global_dib),
     FreeImage_GetHeight(global_dib),
     FreeImage_GetBits(global_dib),
     FreeImage_GetInfo(global_dib),
     DIB_RGB_COLORS, SRCCOPY
   );
   EndPaint(hWnd, &ps);
```

Here, `global_imagewidth` and `global_imageheight` reflect the current client size.

---

### Overlay Text Control 📝

A transparent static control overlays the visualization for text output (e.g., stream title).

1. **Font Creation** (in `InitInstance`):

```cpp
   global_hFont = CreateFontW(
     global_fontheight, 0, 0, 0,
     FW_EXTRABOLD, 0, 0, 0, 0, 0, 0, 2, 0,
     L"Segoe Script"
   );
```

1. **Control Creation** (in `WM_CREATE`):

```cpp
   HWND hStatic = CreateWindowEx(
     WS_EX_TRANSPARENT,
     L"STATIC", L"",
     WS_CHILD | WS_VISIBLE | global_staticalignment,
     0, 100, 100, 100,
     hWnd, (HMENU)IDC_MAIN_STATIC,
     GetModuleHandle(NULL), NULL
   );
   SendMessage(hStatic, WM_SETFONT,
     (WPARAM)global_hFont, MAKELPARAM(FALSE, 0)
   );
```

1. **Resizing & Layout** (in `WM_SIZE`):

```cpp
   RECT rc; GetClientRect(hWnd, &rc);
   global_imagewidth  = rc.right;
   global_imageheight = rc.bottom;
   WavSetLib_Initialize(
     global_hwnd, IDC_MAIN_STATIC,
     global_imagewidth,
     global_imageheight,
     global_fontwidth,
     global_fontheight,
     global_staticalignment
   );
   SetWindowPos(
     GetDlgItem(hWnd, IDC_MAIN_STATIC),
     NULL,
     0, 0,
     global_imagewidth,
     global_imageheight,
     SWP_NOZORDER
   );
```

The helper `WavSetLib_Initialize` recalculates line breaks and metrics.

| Setting | Description |
| --- | --- |
| `global_fontheight` | Font height in pixels |
| `global_fontwidth` | Computed average character width |
| `global_staticalignment` | SS_LEFT / SS_CENTER / SS_RIGHT alignment |


---

### Mouse Interaction Controls 🎛️

Users can cycle through 19 visualization modes via mouse clicks. Each click also **clears** the spectrum buffer for a fresh redraw.

- **Left Click (**`**WM_LBUTTONUP**`**)**

```cpp
  specmode = (specmode + 1) % 19;
  memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
```

- **Right Click (**`**WM_RBUTTONUP**`**)**

```cpp
  specmode = (specmode - 1);
  if (specmode < 0) specmode = 18;
  memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
```

This wrap-around logic ensures seamless cycling.

---

**Note:** All global layout and style parameters can be overridden via command-line arguments at launch, enabling fine-grained control of the window’s look and feel.