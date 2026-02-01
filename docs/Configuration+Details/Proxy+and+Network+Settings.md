## Configuration Details – Proxy and Network Settings 🎛️

This section explains how the application configures **network behavior** at startup. It covers:

- **Proxy Server** setup
- **Playlist Handling**
- **Pre-buffering** control

All configurations are applied during `InitInstance` immediately after `BASS_Init` succeeds .

### Overview Table

| Setting | BASS Option | Purpose | Code Example |
| --- | --- | --- | --- |
| **Proxy Server** | BASS_CONFIG_NET_PROXY | Routes HTTP/ICY requests through a corporate or custom proxy | `BASS_SetConfigPtr(BASS_CONFIG_NET_PROXY, global_proxy)` |
| **Playlist Handling** | BASS_CONFIG_NET_PLAYLIST | Automatically resolves and plays playlist files (`.pls`, `.m3u`, etc.) | `BASS_SetConfig(BASS_CONFIG_NET_PLAYLIST, 1)` |
| **Pre-buffering** | BASS_CONFIG_NET_PREBUF | Disables built-in buffering so the app can display its own buffering progress via UI/timers | `BASS_SetConfig(BASS_CONFIG_NET_PREBUF, 0)` |


---

### Proxy Server

The **proxy server** setting allows the application to route all HTTP-based stream requests through an HTTP proxy.

- **Variable**: `global_proxy`
- **Type**: `char[100]` (C-string) initialized to an empty string .
- **Application**: Passed to BASS via `BASS_SetConfigPtr` after device initialization.
- **Behavior**:
- If `global_proxy` is empty (`""`), BASS connects directly.
- If set to `"http://proxy.example.com:8080"`, all network calls will use that proxy.

```cpp
// After BASS_Init(...)
BASS_SetConfigPtr(BASS_CONFIG_NET_PROXY, global_proxy);  // setup proxy server
```

- **When to use**:
- Behind corporate firewalls
- To anonymize requests
- For debugging or logging HTTP traffic

```card
{
    "title": "Proxy Usage",
    "content": "Define global_proxy with your HTTP proxy string to route network requests through a proxy."
}
```

---

### Playlist Handling

By default, BASS will treat `.pls` and `.m3u` URLs as plain files. Enabling playlist processing lets BASS parse and play the contained stream URLs automatically.

- **Option**: `BASS_CONFIG_NET_PLAYLIST`
- **Value**: `1` to **enable**, `0` to disable
- **Effect**:
- **Enabled**: BASS reads playlist files, selects the first valid entry, and streams it.
- **Disabled**: The URL is treated as a raw data file; no playlist parsing occurs.

```cpp
// Enable automatic playlist (.pls/.m3u) processing
BASS_SetConfig(BASS_CONFIG_NET_PLAYLIST, 1);
```

---

### Pre-buffering

BASS normally pre-buffers network streams to improve smooth playback. Disabling this lets the application manage buffering explicitly, updating a progress UI and controlling when playback begins.

- **Option**: `BASS_CONFIG_NET_PREBUF`
- **Value**:
- `0` → **automatic buffering disabled**
- `>0` → buffer size in milliseconds
- **Rationale**:
- Custom UI/timers display precise buffer percentage.
- Playback can start exactly when the app deems the buffer sufficient.

```cpp
// Disable BASS’s automatic network pre-buffering
BASS_SetConfig(BASS_CONFIG_NET_PREBUF, 0);
```

---

### Initialization Sequence

1. **Initialize audio device**

```cpp
   BASS_Init(-1, 44100, 0, hWnd, NULL);
```

1. **Enable playlist handling**

```cpp
   BASS_SetConfig(BASS_CONFIG_NET_PLAYLIST, 1);
```

1. **Disable automatic pre-buffering**

```cpp
   BASS_SetConfig(BASS_CONFIG_NET_PREBUF, 0);
```

1. **Apply HTTP proxy**

```cpp
   BASS_SetConfigPtr(BASS_CONFIG_NET_PROXY, global_proxy);
```

1. **Proceed with stream creation and UI timers** .

These settings empower the application with full control over network streaming behavior, ensuring consistent UI feedback and flexible deployment in varied network environments.