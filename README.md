# AR Clip SDK — Unity

Unity SDK for building WebAR experiences that run inside **AR Clip** — an instant AR launcher via App Clip or QR code scan, no app installation required.

The SDK bridges your Unity WebGL scene to native ARKit on device: camera, surface tracking, image tracking, VPS localization, GPS, and compass are all handled natively while your content renders in a secure WebView.

## How it works

```
QR scan / App Clip link
        ↓
AR Clip (native iOS)
  ├── ARKit          → camera, planes, image tracking
  ├── VPS Engine     → 5 cm localization
  └── WebView        → your Unity WebGL scene
              ↑
        AR Clip SDK (this repo)
        bridges native ↔ web
```

Perfect for:
- Geospatial AR and outdoor installations
- Guided AR tours and educational overlays
- Marketing activations and branded experiences
- Indoor navigation with VPS

## Features

- **Zero install** — launch via QR code or App Clip link
- **Full ARKit access** — camera, planes, image tracking, VPS, GPS, compass
- **Three build targets** — AR Clip App, 8th Wall, WebXR (switch at build time)
- **Editor testing** — ARLibTester simulates native calls without a device
- **Transparent background** — native camera feed shows through WebGL canvas
- **Multi-target camera** — share one scene across all three runtimes

## Requirements

- Unity 2020 LTS or newer (tested on 2021+)
- WebGL build target
- For **8th Wall**: self-hosted 8th Wall engine files
- For **WebXR**: [WebXR Export](https://github.com/De-Panther/unity-webxr-export) package (`com.de-panther.webxr`)

## Installation

### 1. (WebXR only) Install WebXR Export first

**Window → Package Manager → + → Add package from git URL:**

```
https://github.com/De-Panther/unity-webxr-export.git?path=/Packages/webxr
```

### 2. Add AR Clip SDK

**Window → Package Manager → + → Add package from git URL:**

```
https://github.com/OpenARMaps/arclip-sdk-unity.git
```

> ⚠️ Remove any existing `Assets/ARLib` folder before installing to avoid duplicate type errors:
> ```
> error CS0433: The type 'ARLibTester' exists in both 'ARLib, Version=0.0.0.0' and 'Assembly-CSharp'
> ```

### 3. Post-install setup

**WebGLTemplates** — in Package Manager → AR Clip SDK → Samples, import WebGLTemplates. Copy the entire folder to `Assets/` root (Unity only reads templates placed there).

Available templates:
- `ARLib` — for AR Clip App
- `8thWallVPS` — for 8th Wall
- `WebXRVPS` — for WebXR

**TransparentBackground** — import the TransparentBackground sample, move `TransparentBackground.jslib` to `Assets/Plugins/`. This enables transparent WebGL canvas so the native camera shows through.

**Build Window** — open `ARClip → Build Window` to select the runtime target before each build. This is the primary way to switch targets — it applies the correct WebGL template and bakes the runtime config.

## Quick Start

### Camera setup

1. Create a camera GameObject, **disable its Camera component**
2. Set **Clear Flags = Solid Color**, background **RGBA(0,0,0,0)**
3. Add `ARLibController` to any GameObject and assign that camera to `renderCamera`
4. Optional: add `ARLibTester` next to `ARLibController` to simulate native calls in Editor

For multi-target scenes (AR Clip App + 8th Wall + WebXR from the same scene):
- Use `ARClipCameraPlaceholder` on the camera anchor
- Use `ARClipCameraBootstrap` on the object owning `ARLibController`

### Bootstrap script

```csharp
using UnityEngine;
using ARLib;

public class ARBootstrap : MonoBehaviour
{
    void Awake()
    {
        ARLibController.Initialized            += OnARReady;
        ARLibController.SurfaceTrackingUpdated += OnPlanes;
        ARLibController.VPSPositionUpdated     += OnVPSPose;
    }

    void Start()
    {
        ARLibController.Initialize();
    }

    void OnARReady()
    {
        ARLibController.EnableCamera();
        ARLibController.EnableAR();

        // Surface tracking
        ARLibController.EnableSurfaceTracking("horizontal");

        // VPS — empty locationIds = auto-search entire OpenARMaps network
        ARLibController.SetupVPS(new VPSSettings {
            serverUrl    = "https://api.openarmaps.org/api/v1",
            locationsIds = new string[] { }
        });
        ARLibController.StartVPS();
    }

    void OnPlanes(PlaneInfo[] planes)
    {
        Debug.Log($"Detected {planes.Length} planes");
    }

    void OnVPSPose(VPSPoseData pose)
    {
        Debug.Log($"VPS localized: {pose.Position}");
        // Place AR content — pose is in world space
    }

    void OnDisable()
    {
        ARLibController.Initialized            -= OnARReady;
        ARLibController.SurfaceTrackingUpdated -= OnPlanes;
        ARLibController.VPSPositionUpdated     -= OnVPSPose;
    }
}
```

## Build Targets

Open **ARClip → Build Window** to select a target before building:

| Target | WebGL Template | Host | Notes |
|---|---|---|---|
| `ARClip App` | `ARLib` | AR Clip native app / WebView | Standard setup |
| `8th Wall` | `8thWallVPS` | Mobile browser + 8th Wall | Requires engine files in `Assets/WebGLTemplates/8thWallVPS/vendor/8thwall/` |
| `WebXR` | `WebXRVPS` | Mobile browser + WebXR | Requires `com.de-panther.webxr` |

Runtime selection is baked at build time — the SDK does not detect the host environment dynamically. `ARClipCameraBootstrap` reads the baked config and selects the correct camera.

### WebXR notes

- `WebXRVPS` includes an AR Clip–compatible JS bridge: `window.ARLib`, `window.ARLibNative`, VPS callbacks back into Unity (`OnVPSReady`, `OnVPSLocalized`, `OnVPSError`)
- WebXR owns camera pose; bridge provides API compatibility but does not push camera pose updates manually
- `SetupVPS(settings)` is the main VPS config source: `serverUrl`, `locationsIds`, `maxFailsCount`
- If `XRRig` shows missing components: install `com.de-panther.webxr`, then recreate via **XR → Convert to XR Rig**, reassign to `ARClipCameraPlaceholder.webXrRigRoot`
- When switching away from WebXR, the Build Window disables the WebXR Export loader for WebGL to avoid the project staying in XR mode

### 8th Wall notes

- `8thWallVPS` includes the AR Clip JS bridge and supports common, camera, and VPS APIs
- `ARClip App` and `8th Wall` share the same scene camera setup
- Surface tracking and image tracking not yet implemented in the 8th Wall bridge

## Testing Workflow

1. Import WebGLTemplates sample → copy to `Assets/`
2. Open **ARClip → Build Window**, select target
3. If 8th Wall: ensure engine files are in `Assets/WebGLTemplates/8thWallVPS/vendor/8thwall/`
4. If WebXR: let the Build Window install/validate `com.de-panther.webxr` and `WebXR Export` for WebGL
5. Build WebGL player — the selected target determines the template, baked config, and which camera `ARClipCameraBootstrap` uses
6. **AR Clip App**:
   - Zip build with `index.html` at archive root
   - Upload to AR Clip CDN uploader
   - Scan generated QR with [AR Clip](https://apps.apple.com/app/ar-clip/id6742754238)
7. **8th Wall / WebXR**:
   - Serve over HTTPS (use ngrok for local testing if needed)
   - Open in mobile browser

> You can verify the active template in **Project Settings → Player → WebGL → Resolution and Presentation → WebGL Template**, but the Build Window should be the source of truth.

## Events

All events are `static` — subscribe from anywhere, unsubscribe in `OnDisable`/`OnDestroy`.

| Event | Args | When |
|---|---|---|
| `Initialized` | — | AR library initialized |
| `SurfaceTrackingUpdated` | `PlaneInfo[]` | Planes detected/updated |
| `ImageTrackingUpdated` | `TrackedImageInfo[]` | Tracked images changed |
| `TrackedImagesArrayUpdate` | `ImagesArrayData` | Active tracking image list changed |
| `VPSInitialized` | — | VPS subsystem ready |
| `VPSPositionUpdated` | `VPSPoseData` | New VPS localization |
| `OnVPSErrorHappened` | `string` | VPS error |
| `OnVPSSessionIdUpdated` | `string` | VPS session ID updated |
| `LocationUpdated` | `LocationData` | GPS update |
| `HeadingUpdated` | `HeadingData` | Compass update |

## API Reference

### Initialization & Core

```csharp
ARLibController.Initialize();
ARLibController.EnableCamera();
ARLibController.DisableCamera();
ARLibController.EnableAR();
ARLibController.DisableAR();
ARLibController.DisableTracking();   // stops all tracking subsystems
```

### Surface (Plane) Tracking

```csharp
ARLibController.EnableSurfaceTracking("horizontal"); // "vertical" | "both"
// Results via SurfaceTrackingUpdated → PlaneInfo[]
```

### Image Tracking

```csharp
// 1. Register all images before enabling tracking
ARLibController.AddTrackingImage(new TrackingImageData {
    name          = "poster",
    url           = "https://example.com/poster.jpg",
    physicalWidth = 0.3f   // real-world width in meters
});

// 2. Wait for TrackedImagesArrayUpdate — confirms all images loaded
// 3. Then enable
ARLibController.EnableImageTracking();

ARLibController.RemoveTrackingImage("poster");
ARLibController.RemoveAllTrackingImages();
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
ARLibController.ResetTracking();
ARLibController.SetLocationIds();
```

**Timing helpers:**

```csharp
ARLibController.SetAnimationTime(float value);
ARLibController.SetSendPhotoDelay(float value);
ARLibController.SetSendFastPhotoDelay(float value);
ARLibController.SetFirstRequestDelay(float value);
ARLibController.SetDistanceForInterp(float value);
ARLibController.SetAngleForInterp(float value);
ARLibController.SetGpsAccuracyBarrier(float value);
ARLibController.SetTimeOutDuration(float value);
```

### Location & Heading

```csharp
ARLibController.GetCurrentPosition();   // one-off → LocationUpdated once
ARLibController.WatchPosition();        // continuous updates
ARLibController.ClearWatch();           // stop continuous updates

ARLibController.StartHeadingUpdates();
ARLibController.StopHeadingUpdates();
// Results via HeadingUpdated → HeadingData
```

## Editor vs Build Behavior

Almost every native call is guarded by `Application.isEditor`, so live AR, VPS, and GPS won't work in play mode. Use **ARLibTester** (included in Samples) to simulate native callbacks for editor-time development.

## Rendering & Transparent Background

- The GameObject holding `renderCamera` must have its **Camera component disabled** — `ARLibController` calls `renderCamera.Render()` manually each frame
- **Clear Flags = Solid Color**, background **RGBA(0,0,0,0)**
- `TransparentBackground.jslib` must be in `Assets/Plugins`

## QA / FAQ

**No camera image in WebGL background?**
1. `TransparentBackground.jslib` is in `Assets/Plugins`
2. `renderCamera` uses Solid Color clear flags with RGBA(0,0,0,0)
3. Camera component on the GameObject is **disabled**

**No AR events in Editor?**
Expected. Use `ARLibTester` or build to WebGL and scan the QR.

**Duplicate symbol errors on import?**
Remove the legacy `Assets/ARLib` folder before adding the package.

**XRRig shows missing components?**
Install `com.de-panther.webxr`, then **XR → Convert to XR Rig**, reassign to `ARClipCameraPlaceholder.webXrRigRoot`.

## Get an API Key

1. Sign up at [space.web-ar.studio](https://space.web-ar.studio)
2. Create a project → copy your `oam_live_...` key
3. Free tier: 1,000 localizations/month, unlimited public maps

## Contributing

See [CONTRIBUTING.md](https://github.com/OpenARMaps/openarmaps/blob/main/CONTRIBUTING.md).

## License

[MIT License](LICENSE)
