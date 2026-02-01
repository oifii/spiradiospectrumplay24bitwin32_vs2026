# Troubleshooting and FAQs – Changing Colors and Spectrum Modes

## Changing Visualization Colors 🎨

Colors are defined via **command-line arguments**. You must supply all three RGB components for each element (background, channel 1, channel 2). Missing a component leaves that channel at its **default**.

| Element | Arg Index | Component | Default Value |
| --- | --- | --- | --- |
| ------------------------- | ----------: | ------------- | --------------: |
| Background Red | 19 | `rgbRed` | 127 |
| Background Green | 20 | `rgbGreen` | 0 |
| Background Blue | 21 | `rgbBlue` | 127 |
| Channel 1 Red | 22 | `rgbRed` | 255 |
| Channel 1 Green | 23 | `rgbGreen` | 0 |
| Channel 1 Blue | 24 | `rgbBlue` | 0 |
| Channel 2 Red | 25 | `rgbRed` | 0 |
| Channel 2 Green | 26 | `rgbGreen` | 0 |
| Channel 2 Blue | 27 | `rgbBlue` | 255 |


```cpp
// parse specmode
if(nArgs > 17) {
    specmode = atoi((LPCSTR)(szArgList[17]));
}
// defaults
global_bgRGBQUAD.rgbRed   = 127;
global_bgRGBQUAD.rgbGreen =   0;
global_bgRGBQUAD.rgbBlue  = 127;
// override from args
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

Excerpt from command-line handling  .

## Adjusting Spectrum Modes 🔀

The `**specmode**` parameter (0–18) selects the visualization style. You can set it at launch or change it at runtime:

- **At launch:** pass your desired mode as argument 17.
- **Runtime:**
- Left-click → **next** mode
- Right-click → **previous** mode
- The display buffer is cleared after each change.

```cpp
case WM_LBUTTONUP:
    specmode = (specmode + 1) % 19;      // advance
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    return 0;

case WM_RBUTTONUP:
    specmode--;
    if(specmode < 0) specmode = 19 - 1;  // wrap around
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    return 0;
```

Mouse handling in the main window .

## Tips and Common Issues

- **Incomplete RGB**

Ensure you provide  components. Omitting any argument leaves it at default.

- **Monochrome or Over-saturation**
- Use moderate RGB values (e.g. 50–200) rather than extremes.
- Try different `specmode` values to find a clearer style.
- **Finding Your Style**

Cycle modes with mouse clicks until you discover the best fit.

**Key Takeaway:** Always pass complete RGB triplets and experiment with both color values and `specmode` to tailor the visualization to your display.