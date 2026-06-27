# Three.js Galaxy Visualization

A QoderWork Skill for building cosmic/space-themed 3D knowledge graph visualizations using Three.js. Data nodes render as stars in a galaxy, with multi-layered sprites, procedural textures, spiral arm particle systems, and smooth interaction feedback.

## Features

- **Multi-layer node rendering** — each data point composed of core sphere + inner halo + outer glow + diffraction spike
- **Procedural canvas textures** — runtime-generated glow, halo, and spike textures (no external assets needed)
- **Spiral galaxy background** — 8,000-particle system with power-law radial distribution and 4 spiral arms
- **Interaction feedback** — focus/dim system, smooth lerp transitions, twinkle effects, link highlighting
- **Performance optimized** — toggleable bloom, idle-aware rendering (60→30→15 fps), dirty-check hover, GC pressure control
- **3D text labels** — canvas-rendered text sprites with bloom-resistant styling and object pooling

## Installation

This is a [QoderWork](https://qoder.com/qoderwork) Skill. To install:

1. Copy the `SKILL.md` file to `~/.qoderworkcn/skills/threejs-galaxy-visualization/`
2. QoderWork will automatically discover and use the skill when relevant tasks are detected

Or use the SkillHub CLI if available:
```bash
skillhub install threejs-galaxy-visualization
```

## Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Complete skill documentation with code snippets, parameter tables, and verification checklist |

## Skill Sections

1. **Scene Architecture** — layer ordering, deep space environment, camera & controls setup
2. **Procedural Canvas Textures** — glowTexture, starGlowTexture, spikeTexture generation
3. **Multi-Layer Node Rendering** — TYPE_STYLE mapping, four sprite/mesh layers
4. **Background Systems** — spiral galaxy particles, background stars, nebula sprites
5. **Interaction Visual Feedback** — focus/dim system, smooth transitions, link highlighting, 3D labels
6. **Performance Optimization** — bloom toggle, material compensation, idle rendering, dirty checks
7. **Common Pitfalls** — 8 documented pitfalls with solutions

## Requirements

- Three.js ≥ 0.150 (tested with 0.160)
- WebGL 2.0 capable browser
- Vue 3 (for the original implementation, but the skill is framework-agnostic)

## Tech Stack

- Three.js (WebGL rendering)
- Canvas 2D (procedural texture generation)
- Custom GLSL shaders (particle systems)
- Line2/LineGeometry/LineMaterial (fat lines)
- OrbitControls (camera interaction)
- UnrealBloomPass (optional post-processing)

## License

MIT
