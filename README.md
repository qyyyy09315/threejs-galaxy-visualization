<p align="center">
  <img src="https://img.shields.io/badge/Three.js-000000?style=flat-square&logo=threedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/WebGL-990000?style=flat-square&logo=webgl&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/AI_Skill-Universal-8B5CF6?style=flat-square" />
  <a href="https://skillhub.cn/skills/threejs-galaxy-visualization"><img src="https://img.shields.io/badge/SkillHub-安装-10B981?style=flat-square" /></a>
</p>

<br />

<h1 align="center">Three.js Galaxy Visualization</h1>

<p align="center">
  <b>用 Three.js 构建沉浸式 3D 星系知识图谱</b><br />
  <sub>多层精灵渲染 · 程序化纹理 · 螺旋臂粒子系统 · 交互反馈 · 性能优化</sub>
</p>

<p align="center">
  <a href="#-快速开始">快速开始</a> •
  <a href="#-技能文档">技能文档</a> •
  <a href="#-技术栈">技术栈</a> •
  <a href="#-安装方式">安装</a>
</p>

<br />

---

## ✨ 特性

<table>
<tr>
<td width="50%">

### 🎨 多层节点渲染

每个数据节点由 **核心球体 + 内光晕 + 外辉光 + 衍射十字星** 四层叠加构成，无需昂贵的后处理即可呈现丰富的视觉层次。

</td>
<td width="50%">

### 🖼️ 程序化纹理

所有纹理（辉光、星光、十字芒）均通过 **Canvas 2D** 运行时生成，零外部资源依赖，纯白底图配合 `SpriteMaterial.color` 实现任意色彩。

</td>
</tr>
<tr>
<td>

### 🌀 螺旋臂粒子系统

8000 颗背景粒子以幂律径向分布排列成 4 条螺旋臂，配合三色径向渐变（暖金 → 冷蓝 → 深蓝）营造纵深感。

</td>
<td>

### ⚡ 性能优化

可切换 Bloom 后处理、空闲感知帧率调节（60→30→15 fps）、脏检查跳过无变化帧、GC 压力控制。

</td>
</tr>
<tr>
<td>

### 🖱️ 交互反馈

聚焦/暗淡系统 — 选中节点高亮，非关联节点淡出至 25% 透明度。支持平滑 lerp 过渡、微光闪烁、连线高亮。

</td>
<td>

### 📝 3D 文字标签

Canvas 渲染的文字精灵，配备对象池避免 GC 抖动，深色底板 + 描边文字确保 Bloom 下依然清晰可读。

</td>
</tr>
</table>

---

## 🚀 快速开始

### 场景架构

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

### 节点样式映射

| 类型 | 色彩 | 大小 | 辉光 | 十字星 | 视觉角色 |
|:----:|:----:|:----:|:----:|:------:|:--------:|
| `book` | 蓝白 | 4.0 | 3.0 | ✅ | 蓝白巨星 |
| `unit` | 暖金 | 3.0 | 2.4 | ✅ | 暖色亚巨星 |
| `lesson` | 黄白 | 2.0 | 2.2 | ✅ | 主序星 |
| `author` | 青蓝 | 1.5 | 1.9 | ✅ | 高温恒星 |
| `dynasty` | 紫罗兰 | 1.7 | 2.0 | ✅ | 紫色星 |
| `genre` | 灰白 | 1.6 | 1.8 | — | 白矮星 |
| `task` | 玫瑰 | 1.4 | 1.7 | — | 红色矮星 |

### 性能参数

> [!TIP]
> Bloom 默认关闭以节省约 60% GPU 开销，用户可通过快捷键 **B** 或界面按钮切换。

| 优化项 | 策略 | 效果 |
|:------:|:----:|:----:|
| Bloom 分辨率 | 屏幕 1/3 尺寸 | GPU 负载降低约 75% |
| 空闲帧率 | 30 帧无操作后跳帧 | 60→15 fps |
| 粒子数量 | 银河 8000 + 散星 1500 | 总计 ≤ 10000 |
| 材质补偿 | 提升透明度与缩放 | 关闭 Bloom 仍保持视觉丰富度 |
| 脏检查 | 复合键对比跳过遍历 | 消除 O(n) 冗余计算 |

---

## 📖 技能文档

[SKILL.md](./SKILL.md) 包含完整的实现指南，分为 7 个章节：

<details>
<summary><b>1. 场景架构</b> — 图层顺序、深空环境、相机与控制</summary>

场景背景设为近黑色 `0x04050a`，配合指数雾 `FogExp2` 制造深度感。使用 `OrbitControls` 控制相机，启用阻尼与自动旋转。

</details>

<details>
<summary><b>2. 程序化纹理</b> — 辉光、星光、十字芒纹理生成</summary>

三种纹理均通过 Canvas 2D API 生成，使用白色底图以便 `SpriteMaterial.color` 着色。`minFilter` 设为 `LinearFilter` 避免 mipmap 模糊。

</details>

<details>
<summary><b>3. 多层节点渲染</b> — TYPE_STYLE 映射、四层精灵叠加</summary>

每个节点最多 4 层：核心球体 → 内光晕 → 外辉光 → 十字芒。所有精灵使用 `AdditiveBlending` + `depthWrite: false`。

</details>

<details>
<summary><b>4. 背景系统</b> — 螺旋臂粒子、远景散星、星云精灵</summary>

8000 粒子以 4 臂螺旋分布，1500 散星均匀球壳分布，9 个星云精灵环形散布。

</details>

<details>
<summary><b>5. 交互视觉反馈</b> — 聚焦暗淡、平滑过渡、连线高亮</summary>

选中节点时非关联节点淡出至 25%，聚焦/邻居节点放大并添加微光闪烁。连线根据关系类型改变颜色与粗细。

</details>

<details>
<summary><b>6. 性能优化</b> — Bloom 切换、材质补偿、空闲渲染</summary>

Bloom 默认关闭，通过提升材质参数补偿视觉。空闲时自动降低帧率，脏检查避免冗余遍历。

</details>

<details>
<summary><b>7. 常见陷阱</b> — 8 个实战中踩过的坑</summary>

包括 Bloom + AdditiveBlending 爆炸、depthWrite 透明冲突、纹理 mipmap 模糊、共享几何体等问题的详细解法。

</details>

---

## 🛠️ 技术栈

| 技术 | 用途 |
|:----:|:----:|
| **Three.js** >= 0.150 | WebGL 3D 渲染核心 |
| **Canvas 2D API** | 程序化纹理生成 |
| **Custom GLSL Shaders** | 粒子系统顶点/片元着色器 |
| **Line2 / LineGeometry** | 屏幕空间等宽粗线 |
| **OrbitControls** | 相机交互控制 |
| **UnrealBloomPass** | 可选泛光后处理 |

---

## 📦 安装方式

本技能已上架 [SkillHub](https://skillhub.cn/skills/threejs-galaxy-visualization)，适用于所有支持 AI 上下文文件的编码工具。

### SkillHub 安装（推荐）

<details>
<summary><b>🚀 一键安装提示词</b> — 复制下方提示词发送给 AI 助手即可自动安装</summary>

```
请先检查是否已安装 SkillHub 商店，若未安装，请根据 https://skillhub.cn/install/skillhub.md 安装SkillHub商店，但是只安装CLI，然后安装threejs-galaxy-visualization技能。若已安装，则直接安装threejs-galaxy-visualization技能。
```

</details>

### 手动安装

```bash
# 克隆仓库
git clone https://github.com/CanCanNeed/threejs-galaxy-visualization.git

# 将 SKILL.md 复制到你的项目中
cp threejs-galaxy-visualization/SKILL.md ./your-project/
```

### QoderWork

```bash
cp SKILL.md ~/.qoderworkcn/skills/threejs-galaxy-visualization/SKILL.md
```

### Claude Code

```bash
# 复制到项目根目录，Claude Code 会自动读取
cp SKILL.md ./your-project/CLAUDE.md
# 或追加到已有的 CLAUDE.md
cat SKILL.md >> ./your-project/CLAUDE.md
```

### Codex / Cursor / Windsurf / Trae

```bash
# 放入项目规则目录，具体路径请参考各工具文档
cp SKILL.md ./your-project/.cursor/rules/threejs-galaxy.md
```

> [!NOTE]
> 本技能文档遵循通用 Markdown 格式，不依赖任何特定 AI 工具的私有语法。任何能读取 Markdown 上下文文件的 AI 编码助手均可直接使用。

---

## ✅ 验证清单

实现完成后，建议逐项确认：

- [ ] 节点在 Bloom 关闭时仍有明显辉光效果（材质补偿生效）
- [ ] 选中节点后，非关联节点淡出至约 25% 透明度
- [ ] 背景银河仅在自动旋转激活时缓慢转动
- [ ] 停止交互后 CPU 使用率明显下降（空闲检测生效）
- [ ] 按 B 键可切换 Bloom 开关，无视觉异常
- [ ] 3D 文字标签出现在聚焦节点及其邻居上方

---

<p align="center">
  <sub>MIT License · 欢迎贡献与分享</sub>
</p>
