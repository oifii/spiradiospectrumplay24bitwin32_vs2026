# Visualization Modes and Graphics

This section explains how the application renders real-time audio visualizations using the `specmode` parameter. The `specmode` integer (0–18) selects different spectrum or waveform styles. You can set it via the command line or cycle modes at runtime with mouse clicks.

## specmode Overview

`specmode` determines the visualization style:

- **0**: Linear FFT
- **1**: Logarithmic FFT
- **3–18**: Waveform variants (classic, mirrored, filled)

You can configure `specmode` on launch or change it dynamically to suit your preferences.

## Changing specmode at Runtime 🎛️

You can cycle through modes without restarting:

- **Left click**: increment `specmode` (wraps at 18)
- **Right click**: decrement `specmode` (wraps at 0)
- **Clears display buffer** on each change

```c
case WM_LBUTTONUP:
    specmode = (specmode + 1) % 19; // next mode
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    return 0;

case WM_RBUTTONUP:
    specmode = (specmode - 1 + 19) % 19; // previous mode
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    return 0;
```

## specmode Values Summary 📊

| specmode | Visualization | Category | Description |
| --- | --- | --- | --- |
| ---------: | --------------------- | ---------------------- | ------------------------------------------------------------------------- |
| 0 | Linear Spectrum | FFT | Linear FFT bars with smoothing and color gradients |
| 1 | Logarithmic Spectrum | FFT | Log-frequency bins averaged into `global_bands` bands |
| 3, 7, 11, 15 | Classic Waveform | Waveform | Lines connect successive samples; left/right channels colored separately |
| 4, 8, 12, 16 | Mirrored Fill | Waveform Fill | Waveform filled toward the opposite side of center |
| 5, 9, 13, 17 | Center Fill | Waveform Fill | Waveform filled toward the center line |
| 6, 10, 14, 18 | Bottom Fill | Waveform Fill | Waveform filled downward to the bottom of the bitmap |


## FFT-based Modes

### specmode 0: Linear Spectrum

Renders a classic bar spectrum:

- Retrieves FFT data via BASS
- Scales magnitudes (`sqrt` or linear) to pixel height
- Interpolates with previous values for smoothness
- Draws vertical bars using a color palette (encoded in the 8-bit DIB)

```c
if (!specmode) { // specmode == 0
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    for (int x = 0; x < SPECWIDTH/2; x++) {
        y = sqrt(fft[x+1]) * 3 * SPECHEIGHT - 4;
        if (y > SPECHEIGHT) y = SPECHEIGHT;
        if (x && (y1 = (y + y1) / 2))
            while (--y1 >= 0)
                specbuf[y1*SPECWIDTH + x*2 - 1] = (127*y1/SPECHEIGHT) + 1;
        y1 = y;
        while (--y >= 0)
            specbuf[y*SPECWIDTH + x*2] = (127*y/SPECHEIGHT) + 1;
    }
}
```

### specmode 1: Logarithmic Spectrum

Approximates a log-frequency axis:

- Accumulates FFT bins into `global_bands` (power-of-two spacing)
- Averages peak in each band
- Draws each band as a vertical bar scaled to display height

```c
else if (specmode == 1) { // logarithmic
    int b0 = 0;
    memset(specbuf, 0, SPECWIDTH * SPECHEIGHT);
    for (int i = 0; i < global_bands; i++) {
        float peak = 0.0f;
        int b1 = pow(2, i * 10.0 / (global_bands - 1));
        b1 = min(max(b1, b0+1), 1023);
        for (; b0 < b1; b0++)
            peak = max(peak, fftbuf[1 + b0]);
        int height = sqrt(peak) * 3 * SPECHEIGHT - 4;
        height = min(height, SPECHEIGHT);
        while (--height >= 0)
            memset(specbuf + height*SPECWIDTH + i*(SPECWIDTH/global_bands),
                   (127*height/SPECHEIGHT) + 1,
                   (SPECWIDTH/global_bands) - 2);
    }
}
```

## Waveform-based Modes

The waveform modes read raw time-domain samples and render them directly. For each channel and X position, they:

1. Compute `v = (1 - buf[x*NUM_CHANNELS + c]) * SPECHEIGHT/2`
2. Clamp `v` to `[0, SPECHEIGHT-1]`
3. Either draw a line (classic) or fill vertical spans (filled)
4. Write per-channel colors (`global_c1RGBQUAD`, `global_c2RGBQUAD`) into `specbuf` in BGR order

### specmode 3, 7, 11, 15: Classic Waveform 🎨

Connects successive sample points with lines:

```c
else if (specmode == 3 || specmode == 7 || specmode == 11 || specmode == 15) {
    for (int c = 0; c < NUM_CHANNELS; c++) {
        for (int x = 0; x < SPECWIDTH; x++) {
            int v = (1 - buf[x*NUM_CHANNELS+c]) * SPECHEIGHT/2;
            v = clamp(v, 0, SPECHEIGHT-1);
            if (x == 0) y = v;
            do {
                y += (y < v) ? 1 : (y > v) ? -1 : 0;
                specbuf[y*SPECWIDTH*3 + x*3 + 0] = (c&1 ? global_c2RGBQUAD.rgbBlue  : global_c1RGBQUAD.rgbBlue);
                specbuf[y*SPECWIDTH*3 + x*3 + 1] = (c&1 ? global_c2RGBQUAD.rgbGreen : global_c1RGBQUAD.rgbGreen);
                specbuf[y*SPECWIDTH*3 + x*3 + 2] = (c&1 ? global_c2RGBQUAD.rgbRed   : global_c1RGBQUAD.rgbRed);
            } while (y != v);
        }
    }
}
```

### specmode 4, 8, 12, 16: Mirrored Fill 🌗

Fills the waveform toward the opposite side of the center line:

```c
else if (specmode==4 || specmode==8 || specmode==12 || specmode==16) {
    // ...compute v as above...
    if (y > (SPECHEIGHT/2))
        while (--y >= (SPECHEIGHT/2 - (v - SPECHEIGHT/2)))
            setPixel(y, x, c);
    else if (y < (SPECHEIGHT/2))
        while (++y <= (SPECHEIGHT/2 + ((SPECHEIGHT/2) - v)))
            setPixel(y, x, c);
}
```

### specmode 5, 9, 13, 17: Center Fill 🔵

Fills the waveform toward the center line from either side:

```c
else if (specmode==5 || specmode==9 || specmode==13 || specmode==17) {
    // ...compute v as above...
    if (y > (SPECHEIGHT/2))
        while (--y >= (SPECHEIGHT/2))
            setPixel(y, x, c);
    else if (y < (SPECHEIGHT/2))
        while (++y <= (SPECHEIGHT/2))
            setPixel(y, x, c);
}
```

### specmode 6, 10, 14, 18: Bottom Fill ⬇️

Fills the waveform downward to the bottom of the display:

```c
else if (specmode==6 || specmode==10 || specmode==14 || specmode==18) {
    // ...compute v as above...
    while (--y >= 0)
        setPixel(y, x, c);
}
```

## Implementation Details

- **Buffer setup**: `CreateBitmapToDrawSpectrum` allocates a 24-bit DIB and sets `specbuf` pointer .
- **Color channels**: Uses `global_c1RGBQUAD` for channel 1, `global_c2RGBQUAD` for channel 2.
- **Drawing pipeline**: `UpdateSpectrum` fills `specbuf`; `WM_PAINT` blits it via `BitBlt` or `StretchDIBits` over the JPEG background .
- **Performance**: Runs at ∼40 Hz via `timeSetEvent`, with a skip timer to throttle resizing races.

---

By selecting the appropriate `specmode`, you can switch between frequency-domain and time-domain visualizations, offering flexible real-time feedback for both files and streams.