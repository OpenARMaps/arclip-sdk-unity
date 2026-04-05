# AR Clip SDK — Unity

Unity SDK for building WebAR experiences that run inside **AR Clip** — an instant AR launcher that works via App Clip or QR code, no app install required.

The SDK bridges your Unity WebGL scene to native ARKit on device, giving you full AR tracking (planes, images, VPS) while your content renders in a secure WebView.

## How it works

```
QR scan / App Clip link
        ↓
AR Clip (native iOS)
  ├── ARKit          → tracking, camera, GPS
  ├── VPS Engine     → 5 cm localization
  └── WebView        → your Unity WebGL scene
              ↑
        AR Clip SDK (this repo)
        bridges native ↔ web
```

## Features

- **Zero install** — launch via QR or App Clip link
- **Full ARKit access** — planes, image tracking, VPS, GPS, compass
- **Three build targets** — AR Clip App, 8th Wall, WebXR (switch at build time)
- **Editor testing** — ARLibTester simulates native calls without a device
- **Transparent background** — native camera feed shows through the WebGL canvas

## Requirements

- Unity 2020 LTS or later
- WebGL build target
- For WebXR: [WebXR Export](https://github.com/De-Panther/unity-webxr-export) package

## Installation

### 1. (WebXR only) Install WebXR Export

**Window → Package Manager → + → Add package from git URL:**

```
https://github.com/De-Panther/unity-webxr-export.git?path=/Packages/webxr
```

### 2. Add AR Clip SDK

**Window → Package Manager → + → Add package from git URL:**

```
https://github.com/OpenARMaps/arclip-sdk-unity.git
```

> Remove any existing `ARLib` folder under `Assets/` before adding the package to avoid duplicate type errors.

### 3. Post-install setup

**WebGLTemplates** — in Package Manager → AR Clip SDK → Samples, import WebGLTemplates. Copy the entire folder into `Assets/`.

**TransparentBackground** — import the TransparentBackground sample, move `TransparentBackground.jslib` to `Assets/Plugins/`.

## Quick Start

```csharp
using ARLib;

public class ARBootstrap : MonoBehaviour
{
    void Awake()
    {
        ARLibController.Initialized         += OnReady;
        ARLibController.VPSPositionUpdated  += OnLocalized;
        ARLibController.SurfaceTrackingUpdated += OnPlanes;
    }

    void Start() => ARLibController.Initialize();

    void OnReady()
    {
        ARLibController.EnableCamera();
        ARLibController.EnableAR();

        // Start VPS — auto-search, no location ID needed
        ARLibController.SetupVPS(new VPSSettings {
            serverUrl    = "https://api.openarmaps.org/api/v1",
            locationsIds = new string[] { }   // empty = auto-search
        });
        ARLibController.StartVPS();

        ARLibController.EnableSurfaceTracking("horizontal");
    }

    void OnLocalized(VPSPoseData pose)
    {
        // Place AR content — pose is in world space
    }

    void OnPlanes(PlaneInfo[] planes) { }
}
```

## Build Targets

Open **ARClip → Build Window** to select a target before building:

| Target | Template | Host |
|---|---|---|
| `ARClip App` | `ARLib` | AR Clip native app |
| `8th Wall` | `8thWallVPS` | Mobile browser + 8th Wall |
| `WebXR` | `WebXRVPS` | Mobile browser + WebXR |

## Testing

1. Build WebGL with the target set to `ARClip App`
2. Zip the build (`index.html` at archive root)
3. Upload to AR Clip CDN uploader
4. Scan the generated QR code with [AR Clip](https://apps.apple.com/app/ar-clip/id6742754238)

For Editor testing without a device: add **ARLibTester** next to `ARLibController` — it simulates native calls so you can verify scene logic.

## API Reference

### Core

```csharp
ARLibController.Initialize();
ARLibController.EnableCamera();
ARLibController.EnableAR();
ARLibController.DisableAR();
```

### VPS

```csharp
ARLibController.SetupVPS(new VPSSettings {
    serverUrl    = "https://api.openarmaps.org/api/v1",
    locationsIds = new string[] { }   // empty = auto-search
});
ARLibController.StartVPS();
ARLibController.StopVPS();
ARLibController.PauseVPS();
ARLibController.ResumeVPS();
```

### Surface Tracking

```csharp
ARLibController.EnableSurfaceTracking("horizontal"); // "vertical" | "both"
```

### Image Tracking

```csharp
ARLibController.AddTrackingImage(new TrackingImageData {
    name           = "poster",
    url            = "https://example.com/poster.jpg",
    physicalWidth  = 0.3f   // meters
});
ARLibController.EnableImageTracking();
```

### Events

| Event | Args | Fires when |
|---|---|---|
| `Initialized` | — | AR library ready |
| `VPSInitialized` | — | VPS subsystem ready |
| `VPSPositionUpdated` | `VPSPoseData` | New localization result |
| `OnVPSErrorHappened` | `string` | VPS error |
| `SurfaceTrackingUpdated` | `PlaneInfo[]` | Plane detected/updated |
| `ImageTrackingUpdated` | `TrackedImageInfo[]` | Tracked image changed |
| `LocationUpdated` | `LocationData` | GPS update |
| `HeadingUpdated` | `HeadingData` | Compass update |

## Contributing

See [CONTRIBUTING.md](https://github.com/OpenARMaps/openarmaps/blob/main/CONTRIBUTING.md).

## License

[MIT License](LICENSE)
