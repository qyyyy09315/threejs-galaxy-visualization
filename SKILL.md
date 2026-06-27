---
name: threejs-galaxy-visualization
description: Create immersive 3D galaxy/stellar knowledge graph visualizations using Three.js. Covers multi-layered node rendering (core+halo+glow+spike), procedural canvas textures, spiral arm particle systems, nebula sprites, interaction feedback, and performance optimization (bloom toggle, idle-aware rendering). Use when building data visualizations, knowledge graphs, or educational tools with a cosmic/space theme.
version: 1.0.0
---

# Three.js Galaxy Visualization

Build a cosmic/space-themed 3D knowledge graph using Three.js, where data nodes appear as stars in a galaxy. This skill covers the complete rendering pipeline from scene setup to interaction and performance tuning.

## 1. Scene Architecture

### Layer Order (back to front)
```
背景层: 银河粒子 + 远景散星 + 星云 sprites
  ↓
连线层: linkGroup (Line2 fat lines)
  ↓
光晕层: haloGroup (halo + glow + spike sprites)
  ↓
核心层: nodeGroup (MeshBasicMaterial spheres)
  ↓
标签层: labelGroup (Canvas texture sprites, on-demand)
```

Add groups to scene in this order. Three.js renders groups in add order, so background draws first.

### Deep Space Environment
```js
scene.background = new THREE.Color(0x04050a)  // near-black with blue tint
scene.fog = new THREE.FogExp2(0x04050a, 0.00028)  // exponential fog for depth
```

### Camera & Controls
```js
camera = new THREE.PerspectiveCamera(58, w/h, 0.1, 4000)
camera.position.set(0, 130, 360)

controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true; controls.dampingFactor = 0.05
controls.rotateSpeed = 0.5; controls.zoomSpeed = 0.7
controls.minDistance = 30; controls.maxDistance = 900
controls.autoRotate = true; controls.autoRotateSpeed = 0.2
controls.enablePan = false
```

### Renderer
```js
renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' })
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.5))  // cap at 1.5x
```

## 2. Procedural Canvas Textures

Three texture types, all generated at runtime via Canvas 2D. Use white (255,255,255) as the base color so `SpriteMaterial.color` multiplication produces any hue.

### glowTexture (128x128) — outer diffusion glow + galaxy core
```js
// Radial gradient: bright center → strong halo → fade out
const g = ctx.createRadialGradient(64, 64, 0, 64, 64, 64)
g.addColorStop(0.0, 'rgba(255,255,255,1.0)')    // white core
g.addColorStop(0.1, 'rgba(255,255,255,0.8)')    // strong halo
g.addColorStop(0.3, 'rgba(255,250,235,0.35)')   // warm mid
g.addColorStop(0.6, 'rgba(255,240,210,0.12)')   // faint outer
g.addColorStop(1.0, 'rgba(0,0,0,0)')            // transparent edge
```

### starGlowTexture (256x256) — inner colored halo + background particles
```js
// Softer radial gradient for colored halos
const g = ctx.createRadialGradient(128, 128, 0, 128, 128, 128)
g.addColorStop(0,    'rgba(255,255,255,0.85)')  // bright center
g.addColorStop(0.15, 'rgba(255,255,255,0.40)')  // mid glow
g.addColorStop(0.45, 'rgba(255,255,255,0.08)')  // soft edge
g.addColorStop(1,    'rgba(0,0,0,0)')            // transparent
```

### spikeTexture (256x256) — cross diffraction spikes
```js
// Step 1: Center radial glow (r=40)
let g = ctx.createRadialGradient(128,128,0,128,128,40)
g.addColorStop(0, "rgba(255,255,255,0.9)")
g.addColorStop(1, "rgba(255,255,255,0)")

// Step 2: Horizontal light band (1.6px wide, centered at y=128)
let gh = ctx.createLinearGradient(0,128,256,128)
gh.addColorStop(0,   "rgba(255,255,255,0)")
gh.addColorStop(0.5, "rgba(255,255,255,0.85)")
gh.addColorStop(1,   "rgba(255,255,255,0)")
ctx.fillRect(0, 127.2, 256, 1.6)

// Step 3: Vertical light band (same pattern, centered at x=128)
// Identical gradient rotated 90°
ctx.fillRect(127.2, 0, 1.6, 256)
```

### Texture Settings
```js
tex.minFilter = THREE.LinearFilter   // avoid mipmap blur
tex.magFilter = THREE.LinearFilter
```

## 3. Multi-Layer Node Rendering

Each data node renders as 3-4 stacked sprites/meshes with additive blending. This creates depth without expensive post-processing.

### TYPE_STYLE — data category to star mapping
```js
const TYPE_STYLE = {
  book:    { color: 0x9ec5ff, size: 4.0, glow: 3.0, spike: true  },  // blue-white giant
  unit:    { color: 0xffd98a, size: 3.0, glow: 2.4, spike: true  },  // warm subgiant
  lesson:  { color: 0xfff0b0, size: 2.0, glow: 2.2, spike: true  },  // yellow-white main sequence
  author:  { color: 0x8fd6ff, size: 1.5, glow: 1.9, spike: true  },  // cyan hot star
  dynasty: { color: 0xc6a0ff, size: 1.7, glow: 2.0, spike: true  },  // violet
  genre:   { color: 0xd8dee8, size: 1.6, glow: 1.8, spike: false },  // grey-white dwarf
  task:    { color: 0xff9aae, size: 1.4, glow: 1.7, spike: false }   // rose
}
```

### Four Layers

**Layer 1 — Core Sphere** (bright center point)
```js
const core = new THREE.Mesh(
  sharedSphereGeom,                           // SphereGeometry(1, 12, 12) shared across all nodes
  new THREE.MeshBasicMaterial({ color: 0xffffff })
)
core.scale.setScalar(style.size * 0.55)
```

**Layer 2 — Inner Halo** (colored glow, main visual body)
```js
const halo = new THREE.Sprite(new THREE.SpriteMaterial({
  map: starGlowTex, color: style.color,
  transparent: true, opacity: 0.85,
  depthWrite: false, blending: THREE.AdditiveBlending
}))
halo.scale.setScalar(style.size * style.glow)
```

**Layer 3 — Outer Glow** (soft diffusion aura)
```js
const glow = new THREE.Sprite(new THREE.SpriteMaterial({
  map: glowTex, color: style.color,
  transparent: true, opacity: 0.40,
  depthWrite: false, blending: THREE.AdditiveBlending
}))
glow.scale.setScalar(style.size * style.glow * 2.0)
```

**Layer 4 — Diffraction Spike** (cross-shaped, optional for prominent nodes)
```js
const spike = new THREE.Sprite(new THREE.SpriteMaterial({
  map: spikeTex, color: style.color,
  transparent: true, opacity: 0.85,
  depthWrite: false, blending: THREE.AdditiveBlending
}))
spike.scale.setScalar(style.size * style.glow * 3.2)
spike.material.rotation = time * 0.05  // slow rotation in animate loop
```

### Universal Material Settings
All sprite layers share: `transparent: true`, `depthWrite: false`, `blending: THREE.AdditiveBlending`. This ensures correct alpha compositing without z-fighting.

## 4. Background Systems

### Spiral Galaxy (8000 particles)

**Distribution**: 4 spiral arms, power-law radial density.
```js
const arms = 4, radius = 360, spin = 1.6, randomness = 0.35

for (let i = 0; i < count; i++) {
  const r = Math.pow(Math.random(), 1.7) * radius    // power-law: denser at center
  const arm = i % arms
  const baseAngle = (arm / arms) * Math.PI * 2
  const angle = baseAngle + (r / radius) * spin * Math.PI * 2  // spiral twist
  const rand = (Math.random() - 0.5) * randomness * r * 0.06
  const randY = (Math.random() - 0.5) * (1 - r / radius) * 60  // thinner at edges
}
```

**Color**: 3-stop radial gradient mapped to particle distance from center.
```js
const inner = new THREE.Color(0xfff2c8)   // warm gold at core
const mid   = new THREE.Color(0x6a86d6)   // cool blue at mid-radius
const outer = new THREE.Color(0x2a3a78)   // deep blue at edge
// Lerp between stops based on normalized radius t = r/radius
// Add random HSL offset for variety: col.offsetHSL(0, 0, (Math.random()-0.5)*0.15)
```

**Shader**: Custom ShaderMaterial with per-vertex color and size.
```glsl
// Vertex: pass color to fragment, compute point size with distance attenuation
gl_PointSize = aSize * (300.0 / -mv.z) * uPixelRatio;

// Fragment: multiply vertex color by texture alpha
gl_FragColor = vec4(vColor, 1.0) * texture2D(uTex, gl_PointCoord).a;
```

**Orientation**: Tilt the galaxy disc to lie roughly horizontal.
```js
bgGalaxy.rotation.x = Math.PI * 0.18
bgGalaxy.position.y = -40
```

**Galaxy Core Sprite**: A large bright sprite at the center adds the "nuclear bulge".
```js
const core = new THREE.Sprite(new THREE.SpriteMaterial({
  map: glowTex, color: 0xfff0c0,
  transparent: true, opacity: 0.65,
  depthWrite: false, blending: THREE.AdditiveBlending
}))
core.scale.set(180, 180, 1)
```

### Background Stars (1500 particles)

Uniform distribution on a spherical shell (r = 450 to 1550).
```js
const r = 450 + Math.random() * 1100
const theta = Math.random() * Math.PI * 2
const phi = Math.acos(2 * Math.random() - 1)  // uniform sphere sampling
```

**Tint distribution**: 70% cool blue (0.85, 0.9, 1.0), 20% warm gold (1.0, 0.85, 0.6), 10% blue-violet (0.7, 0.8, 1.0).

**Size**: 3.5 to 8.5 (larger than galaxy particles to remain visible at distance). Min point size in shader: `gl_PointSize = max(ps, 2.5)`.

### Nebula Sprites (9 sprites)

Large, faint colored sprites in a ring around the galaxy.
```js
const colors = [0x4a5abf, 0x7a4abf, 0x3a7abf, 0x6a4a9f]  // blue-violet palette
for (let i = 0; i < 9; i++) {
  const r = 200 + Math.random() * 320
  const theta = Math.random() * Math.PI * 2
  sp.position.set(Math.cos(theta)*r, (Math.random()-0.5)*180, Math.sin(theta)*r)
  const scale = 320 + Math.random() * 420
  sp.scale.set(scale, scale, 1)
  // opacity: 0.45, AdditiveBlending, use nebulaTex (soft radial gradient)
}
```

### Rotation Strategy

Only rotate background layers when `controls.autoRotate` is true (user not interacting). Speeds are very low to avoid distracting motion:
```js
if (controls.autoRotate) {
  bgGalaxy.rotation.y += dt * 0.008
  bgStars.rotation.y -= dt * 0.003
  nebulaGroup.rotation.y += dt * 0.001
}
```

## 5. Interaction Visual Feedback

### Focus/Dim System

When a node is selected ("focused"), all other nodes dim to 25% opacity. The focused node and its neighbors brighten:

| Role | Scale Boost | Opacity Base | Lift | Twinkle |
|------|------------|-------------|------|---------|
| Focus (selected) | 2.0x | 0.85 (halo) | 6.0 | yes |
| Neighbor | 1.35x | 0.65 (halo) | 2.0 | yes |
| Highlight set | 1.25x | 0.60 (halo) | 3.0 | yes |
| Default (dimmed) | 1.0x | 0.45 × 0.25 | 0 | no |
| Default (no focus) | 1.0x | 0.45 × 1.0 | 0 | no |

### Smooth Transitions
Use `lerp` for all visual property changes. Lift animation uses a 0.12 easing factor for natural spring feel:
```js
const targetLift = isFocus ? 6.0 : (isNb ? 2.0 : (inHl ? 3.0 : 0.0))
d.lift += (targetLift - d.lift) * 0.12

// Scale and opacity use lerp(0.25) for snappier response
d.core.scale.lerp(targetScale, 0.25)
d.halo.scale.lerp(targetScale, 0.25)
```

### Twinkle Effect
Focused/neighbor nodes get a subtle brightness oscillation:
```js
const twinkle = 0.92 + Math.sin(time * d.twinkleSpeed + d.phase) * 0.08
// twinkleSpeed is per-node (0.8 to 2.4), phase is hash-based for variety
```

### Link Highlighting (Line2)
```js
// Hover links: warm yellow, opacity 0.85, linewidth 2.0
// Highlight links: purple, opacity 0.75, linewidth 1.8
// Dimmed (focus active): original color, opacity 0.02, linewidth 0.6
// Default: original color, base opacity, linewidth 0.8
```

### 3D Text Labels
Render text to Canvas, create CanvasTexture + SpriteMaterial. Use an object pool to avoid GC:
```js
// Pool: _labelPool array, reuse existing sprites
// Background: dark rounded rect (rgba(6,8,18,0.88))
// Text: white fill with dark stroke (3px) for bloom resistance
// Font: 600 26px "PingFang SC","Microsoft YaHei",sans-serif
// depthTest: false to always render on top
```

## 6. Performance Optimization

### Bloom: Toggle, Not Always-On
UnrealBloomPass adds ~3x GPU fragment workload. **Default it OFF**, let users toggle with a key (B) or button.
```js
// When bloom is ON: use 1/3 screen resolution (not 1/2) for better perf
bloomPass = new UnrealBloomPass(new THREE.Vector2(w/3, h/3), 0.5, 0.4, 0.35)

// In render loop:
if (bloomEnabled && composer) {
  composer.render()
} else {
  renderer.render(scene, camera)
}
```

### Material Compensation (No-Bloom Mode)
When bloom is OFF, boost material properties to maintain visual richness:

| Property | With Bloom | Without Bloom |
|----------|-----------|--------------|
| Halo opacity | 0.6 | 0.85 |
| Glow opacity | 0.22 | 0.40 |
| Glow scale multiplier | 1.6x | 2.0x |
| Spike opacity | 0.7 | 0.85 |
| Core size multiplier | 0.45 | 0.55 |
| Nebula opacity | 0.34 | 0.45 |
| Galaxy core opacity | 0.55 | 0.65 |

### Idle-Aware Rendering
Track a dirty flag `_needsRender`. Only render when something changed:
```js
let _idleFrames = 0
const IDLE_THRESHOLD = 30

// In animate():
if (frameActive || _needsRender) {
  _idleFrames = 0; _needsRender = false
  // Auto-rotate only: skip every other frame (60→30fps)
  if (onlyAutoRotate) {
    _skipRender = !_skipRender
    if (_skipRender) return
  }
} else {
  _idleFrames++
  // Fully idle: skip every other frame (~15fps)
  if (_idleFrames > IDLE_THRESHOLD) {
    _skipRender = !_skipRender
    if (_skipRender) return
  }
}
```

Set `_needsRender = true` in: pointer events, resize, camera animation, focus change, bloom toggle, visibility change, buildScene.

### setHover Dirty Check
Skip the O(n) node iteration when nothing changed:
```js
const compositeKey = focusId + '|' + hlKey + '|' + anyLifting
if (compositeKey === prevHoverKey && !anyLifting) return
prevHoverKey = compositeKey
```

### GC Pressure Control
Reuse module-level containers instead of creating new ones per frame:
```js
const _reusableNeighborIds = new Set()
const _reusableUpdateSet = new Set()
const _raycasterTargets = []
const _v1 = new THREE.Vector3()  // reusable for lerp targets
```

### Particle Budget
Keep total GPU particles under 10,000:
- Spiral galaxy: ≤8000
- Background stars: ≤1500
- Total: ~9500

## 7. Common Pitfalls

1. **Bloom + AdditiveBlending explosion**: Bloom amplifies every bright fragment. With 21,500 additive-blended particles, the bloom threshold pass processes all of them, causing GPU stalls. Always make bloom toggleable.

2. **depthWrite + transparent conflict**: Every sprite/mesh with `transparent: true` MUST also have `depthWrite: false`. Otherwise, nearer transparent objects occlude farther ones incorrectly.

3. **Canvas texture mipmap blur**: Default Three.js texture filtering uses mipmaps, which blur small canvas textures. Always set `tex.minFilter = THREE.LinearFilter` for canvas-generated textures.

4. **SpriteMaterial color is multiplicative**: `SpriteMaterial({ color: 0xff0000, map: whiteTexture })` produces red. Use white textures as the base — they're the most flexible because any color multiplication works.

5. **OrbitControls damping needs per-frame update**: Even when idle, `controls.update()` must be called every frame for damping inertia to settle correctly. Skipping it causes sudden stops.

6. **FNV-1a hash for deterministic randomness**: Use `hash(string)` for stable per-node values (phase, speed, position jitter). Avoids `Math.random()` which gives different results on each build:
```js
function hash(str) {
  let h = 2166136261
  for (let i = 0; i < str.length; i++) { h ^= str.charCodeAt(i); h = Math.imul(h, 16777619) }
  return (h >>> 0) / 4294967295
}
```

7. **Shared geometry for cores**: All node core meshes share one `SphereGeometry(1, 12, 12)`. Creating per-node geometry wastes GPU memory. Only materials differ.

8. **Line2 over native Line**: Use `Line2`/`LineGeometry`/`LineMaterial` from three/examples for fat lines with screen-space width. Native `THREE.Line` has inconsistent width across platforms.

## Verification

After implementing, verify:
1. Nodes glow visibly without bloom enabled (material compensation working)
2. Selecting a node dims non-neighbors to ~25% opacity
3. Background galaxy rotates slowly only when auto-rotate is active
4. CPU usage drops significantly when user stops interacting (idle detection)
5. Pressing B toggles bloom on/off without visual artifacts
6. 3D text labels appear above focused node and its neighbors
