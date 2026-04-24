# Invisible Phone

Hold your phone in front of any scene and see through it — the rear-camera feed is displayed on screen, scaled and panned to match the background behind the glass. A face tracker on the front camera measures your eye position and corrects for parallax so the illusion holds as your head moves.

---

## Quick start

1. **Open on a mobile device** (Chrome or Safari) and grant camera permissions for both cameras when prompted.
2. **Pinch to zoom** and **drag to pan** until the rear-camera image lines up with the physical background visible around the phone edges.
3. **Move your head slowly left and right.** Adjust the **Parallax** slider until the background stays locked in place during lateral motion.
   Tip: find a straight edge — door frame, shelf, window sill — that crosses the screen border and tune until the line looks continuous across the boundary.
4. **Double-tap** to reset pan and zoom if you get lost.

---

## Geometry

### Cameras and view frustum

The phone sits between you and the background. Three optical elements matter:

```
                        BACKGROUND
                  ┌─────────────────────┐
                  │    ┌───────────┐    │  ← rear cam FOV
                  │    │  visible  │    │    (captures this region)
                  │    │  region   │    │
                  │    └───────────┘    │
                  └─────────────────────┘
                           ╱  ╲
                          ╱    ╲   view frustum
                         ╱      ╲  (defined by eye + screen)
                         │ ┌──┐ │
                         │ │sc│ │  ← PHONE
                         │ │r.│ │    screen faces user
                         │ └──┘ │    rear cam on far side
                         ╲ [f] ╱
                          ╲  ╱     front cam FOV
                           ╲╱      (tracks eye via face landmarks)
                            ◉
                        USER'S EYE
```

- **View frustum** (╱╲ above the phone): the cone from your eye, through the screen rectangle, onto the background. This is the region you *expect* to see.
- **Rear camera FOV** (outer box in background): wider than the view frustum. Pan and zoom crop it to the sub-region matching the frustum.
- **Front camera FOV** (╲╱ below the phone): captures your face so MediaPipe can extract the 3-D eye offset.

### Parallax correction

When your eye moves sideways, the view frustum shifts — you'd naturally see a different slice of the background through the screen. The front camera measures that offset and pans the image to compensate:

```
   Eye centred                    Eye shifted left

        ┌──────────┐                   ┌──────────┐
        │ ████████ │                   │   ██████ │
    ◉──►│ ████████ │            ◉      │   ██████ │
        │ ████████ │             ╲────►│   ██████ │
        └──────────┘                   └──────────┘

  image shift  =  eye_offset  ×  gain  ×  screen_width
```

The **Parallax** slider sets `gain`. When calibrated, a straight edge crossing the screen border looks continuous as your head moves.

### Depth zoom

If you lean closer or farther from the phone, the face tracker's depth estimate (`tz`) automatically scales the image, keeping the scene locked as your distance changes.

---

## Controls

| Gesture / Control | Action |
|---|---|
| Drag | Pan the rear-camera image |
| Pinch | Zoom the rear-camera image |
| Double-tap | Reset pan and zoom |
| Parallax slider | Tune the eye-parallax correction gain |

---

## Stack

- **WebGL** — rear-camera frames uploaded as GPU textures; a GLSL fragment shader applies pan, zoom, and parallax with no CPU readback
- **MediaPipe FaceLandmarker** — GPU-delegate face tracking; facial transformation matrix gives 3-D eye position
- **`requestVideoFrameCallback`** — face detection fires on every delivered front-camera frame, not on a fixed timer
- Single `index.html`, no build step, no framework
