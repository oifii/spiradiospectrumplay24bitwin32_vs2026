## Visualization Modes and Graphics – Color Configuration 🎨

This section explains how the **background** and **audio channel** colors are defined, initialized, and applied in the real-time visualization. The application exposes three `RGBQUAD` globals—one for the background and one for each audio channel—configurable via command-line parameters.

### Color Variables Declaration

The three color configurations are declared as global `RGBQUAD` structures in **spiradiospectrumplaywin32.cpp**:

```cpp
RGBQUAD global_c1RGBQUAD;   // Audio channel 1 color  
RGBQUAD global_c2RGBQUAD;   // Audio channel 2 color  
RGBQUAD global_bgRGBQUAD;   // Solid background color  
```

These variables hold 24-bit color components (red, green, blue) for rendering .

### Default Values and Initialization 🖌️

On startup, before parsing overrides, the application sets sensible defaults:

```cpp
// Solid background color (purple)
global_bgRGBQUAD.rgbRed   = 127;
global_bgRGBQUAD.rgbGreen =   0;
global_bgRGBQUAD.rgbBlue  = 127;

// Audio channel 1 color (red)
global_c1RGBQUAD.rgbRed   = 255;
global_c1RGBQUAD.rgbGreen =   0;
global_c1RGBQUAD.rgbBlue  =   0;

// Audio channel 2 color (blue)
global_c2RGBQUAD.rgbRed   =   0;
global_c2RGBQUAD.rgbGreen =   0;
global_c2RGBQUAD.rgbBlue  = 255;
```

These defaults ensure a purple background with red/blue waveforms .

### Command-Line Overrides

Users can override each color component via command-line arguments, indexed as follows:

| Color | Variable | Component | Arg Index | Description |
| --- | --- | --- | --- | --- |
| Background color | `global_bgRGBQUAD` | Red | 19 | Solid background red component |
| Green | 20 | Background green component |  |  |
| Blue | 21 | Background blue component |  |  |
| Channel 1 waveform | `global_c1RGBQUAD` | Red | 22 | Channel 1 red component |
| Green | 23 | Channel 1 green component |  |  |
| Blue | 24 | Channel 1 blue component |  |  |
| Channel 2 waveform | `global_c2RGBQUAD` | Red | 25 | Channel 2 red component |
| Green | 26 | Channel 2 green component |  |  |
| Blue | 27 | Channel 2 blue component |  |  |


Argument parsing snippet:

```cpp
if(nArgs > 19) global_bgRGBQUAD.rgbRed   = atoi(szArgList[19]);
if(nArgs > 20) global_bgRGBQUAD.rgbGreen = atoi(szArgList[20]);
if(nArgs > 21) global_bgRGBQUAD.rgbBlue  = atoi(szArgList[21]);

if(nArgs > 22) global_c1RGBQUAD.rgbRed   = atoi(szArgList[22]);
if(nArgs > 23) global_c1RGBQUAD.rgbGreen = atoi(szArgList[23]);
if(nArgs > 24) global_c1RGBQUAD.rgbBlue  = atoi(szArgList[24]);

if(nArgs > 25) global_c2RGBQUAD.rgbRed   = atoi(szArgList[25]);
if(nArgs > 26) global_c2RGBQUAD.rgbGreen = atoi(szArgList[26]);
if(nArgs > 27) global_c2RGBQUAD.rgbBlue  = atoi(szArgList[27]);
```

This mapping lets users fine-tune each RGB channel on the fly .

### Applying Colors in Rendering

1. **Background Clearing**

Before drawing waveforms or spectrum, the buffer is cleared using the background color:

```cpp
   for(int i = 0; i < SPECWIDTH; i++) {
     for(int j = 0; j < SPECHEIGHT; j++) {
       specbuf[j*SPECWIDTH*3 + i*3]     = global_bgRGBQUAD.rgbBlue;
       specbuf[j*SPECWIDTH*3 + i*3 + 1] = global_bgRGBQUAD.rgbGreen;
       specbuf[j*SPECWIDTH*3 + i*3 + 2] = global_bgRGBQUAD.rgbRed;
     }
   }
```

This fills each pixel’s BGR channels with the background color .

1. **Waveform Drawing**

In waveform modes (e.g., `specmode == 3,4,8,12,16`), each channel is drawn in its own color:

```cpp
   // Inside per-pixel loop for channels
   specbuf[offset]     = (c & 1)
                         ? global_c2RGBQUAD.rgbBlue
                         : global_c1RGBQUAD.rgbBlue;
   specbuf[offset + 1] = (c & 1)
                         ? global_c2RGBQUAD.rgbGreen
                         : global_c1RGBQUAD.rgbGreen;
   specbuf[offset + 2] = (c & 1)
                         ? global_c2RGBQUAD.rgbRed
                         : global_c1RGBQUAD.rgbRed;
```

Bit-masking (`c & 1`) selects channel 2 (right) or channel 1 (left) color for clear visual separation .

### When Colors Matter

- **Solid Background Modes** (`specmode` 3–6): background color fully replaces the buffer each frame.
- **Noisy Background Modes** (`specmode` 7–10): noise overlay ignores the background color.
- **Background-Shifting Modes** (`specmode ≥ 11`): background color shifts across frames, blending with previous content.
- **Waveform & Filled Modes** (`specmode` 3–18): channel colors define overlay at each sample point for both simple and filled waveforms.

---

This configuration system decouples color selection from rendering logic, enabling flexible customization of the visualization via simple command-line switches.