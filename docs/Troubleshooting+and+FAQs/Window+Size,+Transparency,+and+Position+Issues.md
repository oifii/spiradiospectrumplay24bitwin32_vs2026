## Troubleshooting and FAQs – Window Size, Transparency, and Position Issues

When the application’s window appears off-screen, too small, or entirely transparent, these steps will help you pinpoint and resolve geometry and alpha problems.

### Common Symptoms

- Window opens at `(0,0)` or outside visible desktop area
- Content is cropped or too small
- Overlay appears full-screen but fully transparent or invisible
- Title bar unexpectedly present or missing

### 1. Verify Integer Parameters

The window’s **X**, **Y**, **Width**, **Height**, and **Alpha** are driven by command-line arguments (arguments 4–8). Passing non-integer or out-of-range values leads to odd behavior.

- Always supply integers for:
- `global_x` (horizontal offset)
- `global_y` (vertical offset)
- `global_xwidth` (window width)
- `global_yheight` (window height)
- `global_alpha` (transparency 0–255)

```bash
spiradiospectrumplaywin32.exe <stream> <specmode> <bands>  X   Y   Width  Height  Alpha  TitleBarDisplay
                                       └─┬─┘   └─┬─┘  └─┬─┘   └─┬─┘ └──┬──┘ └─────────────┘
                                        4        5        6        7        8
```

| Arg Index | Variable | Description |
| --- | --- | --- |
| 4 | global_x | X-coordinate (pixels) |
| 5 | global_y | Y-coordinate (pixels) |
| 6 | global_xwidth | Window width (pixels) |
| 7 | global_yheight | Window height (pixels) |
| 8 | global_alpha | Opacity (0 = transparent, 255 = opaque) |


### 2. Adjust Off-Screen or Tiny Windows

> **Tip:** Default values are set in code if arguments are omitted .

If your window is out of view or too small:

- Increase or decrease `global_x` / `global_y` to bring it onto the desktop.
- Modify `global_xwidth` / `global_yheight` to match your intended window size.

```c
int global_x       = 100;   // default X (pixels)
int global_y       = 200;   // default Y (pixels)
int global_xwidth  = 400;   // default width
int global_yheight = 400;   // default height
```

⚙️ **Example**: On a 1920×1080 display, to center a 600×300 window:

```bash
... 660 390 600 300 200 1
```

### 3. Toggling Border and Title Bar

The `global_titlebardisplay` flag controls window decoration:

- `**1**` → Standard resizable window with title bar
- `**0**` → Borderless overlay (`WS_POPUP`)

```c
if (global_titlebardisplay) {
  hWnd = CreateWindow(szClass, szTitle,
    WS_OVERLAPPEDWINDOW,
    global_x, global_y,
    global_xwidth, global_yheight,
    ...);
} else {
  hWnd = CreateWindow(szClass, szTitle,
    WS_POPUP | WS_VISIBLE,
    global_x, global_y,
    global_xwidth, global_yheight,
    ...);
}
SetLayeredWindowAttributes(hWnd, 0, global_alpha, LWA_ALPHA);
```

This logic is in `InitInstance` .

### 4. FAQs

**Q: My window is completely invisible.**

A: Check your **alpha** value. Values near 0 render the window fully transparent. Use a mid-range alpha (e.g., 128) to verify visibility.

**Q: Why did my border reappear?**

A: Ensure you passed `global_titlebardisplay=0` for a borderless overlay. Any non-zero value shows the caption bar.

**Q: The visualization is cropped on resize.**

A: The image and spectrum buffers recalculate on `WM_SIZE`. If you see black bars, adjust `global_xwidth` and `global_yheight` to dimensions divisible by 4 (for 24-bit DIB alignment).

**Q: Arguments 4–8 aren’t having any effect.**

A: Confirm your AHK or launcher script forwards **all** parameters. Missing arguments fall back to defaults in code .

---

💡 Keep these guidelines handy when tweaking window geometry and transparency. Proper integer values and the correct `global_titlebardisplay` flag will ensure your overlay sits precisely where and how you want it.