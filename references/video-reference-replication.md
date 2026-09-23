# Video Reference Replication — Pixel Analysis Workflow

## When vision API is unavailable

When replicating a reference video's visual style and the current model cannot see images:

### 1. Extract Metadata
```bash
ffprobe -v quiet -print_format json -show_format -show_streams video.mp4
# → resolution, fps, duration, aspect ratio
```

### 2. Extract Key Frames
```bash
mkdir frames && ffmpeg -y -i video.mp4 -vf "fps=1,scale=400:-1" -frames:v 8 frames/frame_%02d.jpg
```

### 3. Luminosity Distribution
For each frame, count pixels at brightness thresholds:
- lum > 10: total visible area
- lum > 100: mid-bright percentage
- lum > 200: peak brightness percentage

This reveals whether the visualization is uniform fill, ring pattern, or spot pattern.

### 4. Color at Each Luminosity Band
Group pixels by brightness and compute average RGB per band:
```python
for lo, hi in [(10,30),(30,50),(50,80),(80,120),(120,180),(180,255)]:
    mask = (lum >= lo) & (lum < hi)
    avg = arr[mask].mean(axis=0)
    rg_ratio = avg[0] / (avg[1] + 0.01)
```

**Critical signal:** R/G ratio reveals warm vs cool palette.
- R/G < 0.5: blue/green-dominant (cool)
- R/G 0.5-0.9: green-dominant (cool olive)
- R/G 0.9-1.0: balanced (warm olive)
- R/G > 1.0: red-dominant (warm golden)

### 5. Channel Dominance
```python
r_dom = (r > g) & (r > b)  # red-dominant pixels
g_dom = (g > r) & (g > b)  # green-dominant pixels
b_dom = (b > r) & (b > g)  # blue-dominant pixels
```
If b_dom = 0%, the palette has ZERO blue. This is a hard constraint.

### 6. Radial Brightness Profile
Check if the field is center-bright, ring-bright, or edge-bright:
```python
for r_pct in range(0, 101, 10):
    ring = arr[dist_mask]
    avg = ring[lum > 20].mean(axis=0)
```

### 7. Hotspot Analysis
Find the brightest spots — these are often Wilson loops or strong field regions:
```python
hot = arr[lum > 150]
hot_rgb = hot.mean(axis=0)  # peak glow color
```

### 8. Build Palette from Data
Use the exact RGB values from analysis, NOT from text descriptions.
Build a stepped color ramp matching the R/G ratio at each tier.

## Pitfalls

- **Text descriptions lie.** "Glowing golden surface" may actually be cool green-olive.
- **Memory of a video is unreliable.** Always re-analyze the actual frames.
- **Single frame is not enough.** Sample 5-8 frames at different timestamps — colors shift over time.
- **Low-resolution analysis misses fine structure.** Use at least 400×400 for radial analysis.
