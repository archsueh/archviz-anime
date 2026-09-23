# Yang-Mills Reference Video Analysis

**Source:** `SaveTwitter.Net_cHz2E9_bOU2yATGo_(2160p).mp4`  
**Format:** 2160×2160 (square), 30fps, 30s  
**Content:** Yang-Mills gauge field visualization (THEMATHFLOW style)

## Pixel Analysis Results (8 frames, t=2-29s)

### Aspect & Composition
- Square format (2160×2160)
- Visualization zone: rows 15%–85% (~70% of height)
- Top 15%: dark (12% bright) — header area
- Center: 89-100% bright — field visualization fills width
- Bottom 15%: 35-45% bright — fade to edge

### Color Palette by Luminosity Band

| Band | RGB | R/G Ratio | Tone |
|------|-----|-----------|------|
| 10-30 (darkest) | (3, 12, 9) | 0.22 | Pure dark green, almost zero red |
| 30-50 | (29-35, 37-39, 20-23) | 0.80 | Dark olive |
| 50-80 | (51-58, 60-64, 34-39) | 0.88 | Olive green |
| 80-120 | (80-89, 88-92, 50-54) | 0.95 | Warm olive |
| 120-180 | (118-135, 123-132, 65-78) | 0.96 | Golden olive |
| 180-255 (brightest) | (170-205, 165-188, 82-116) | 1.03 | Bright gold |

### Key Finding: Cool-Green Dominant

R/G ratio ranges from 0.22 → 1.03 across the brightness spectrum.  
**Green dominates at ALL levels except the very brightest peaks.**  
Blue is consistently the lowest channel.  
This contradicts the text description ("glowing golden surface") — the actual palette is cool olive-green with golden peaks only at extremes.

### Radial Brightness
- Center (r=0%): varies from 89 to 231 depending on animation phase
- r=15%: typically the brightest ring (RGB ~106-129)
- r=50%: fading (RGB ~90-110)
- r=90%: significantly darker (RGB ~48-67)
- Not a simple center-bright pattern — field strength varies with animation

### Hotspots
- Peak luminance spots at ~(83%, 6%) position → **persistent UI watermark**, NOT field visualization
- Size: 11-16px, RGB(225,229,229) — bright white/gray
- Exclude from palette analysis

### Quadrant Distribution
- Bottom-left tends darker than other quadrants
- Top-right occasionally warmer
- Generally uniform color tone

## Shader Implementation Notes

### Correct Palette (v4 — based on pixel data)
```glsl
// Dark: R/G≈0.22 — almost pure dark green
col = mix(vec3(.012,.052,.035), vec3(.08,.15,.10), dark_mask);
// Mid-dark: R/G≈0.80 — dark olive  
col = mix(col, vec3(.16,.20,.12), mid_dark_mask);
// Mid: R/G≈0.88 — olive green
col = mix(col, vec3(.28,.32,.18), mid_mask);
// Mid-bright: R/G≈0.95 — warm olive
col = mix(col, vec3(.42,.44,.26), mid_bright_mask);
// Bright: R/G≈1.0 — golden olive
col = mix(col, vec3(.55,.55,.33), bright_mask);
// Peak: R/G≈1.05 — warm gold (only at brightest)
col = mix(col, vec3(.70,.67,.40), peak_mask);
```

### Failed Attempts
- v1: Viridis (blue-teal-yellow) → completely wrong, 0% match
- v2: Warm golden-olive → overcorrected, too warm
- v3: Ring pattern + olive → structure wrong
- v4: Cool-green + correct R/G gradient → closest match
