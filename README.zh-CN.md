# Web Video Presentation Skill

**把文章、口播稿、PDF 做成点击驱动的 16:9 网页演示，并通过录屏产出有电影感视频的 Agent Skill。**

[English](./README.md)

---

## ⚠️ Fork 与致谢

本仓库是 **`web-video-presentation`** 技能的 fork / 衍生作品，原版由
**[@ConardLi](https://github.com/ConardLi)** 维护，源仓库为
[`garden-skills`](https://github.com/ConardLi/garden-skills/)。

> **原版的所有设计、方法论、脚手架、主题、文档都归功于 ConardLi。**
> 本 fork 仅仅在原版之上**新增了 1 项功能**：PDF 输入桥接（见下方"我改了什么"）。

|              | 上游原版                                                               | 本 fork                            |
| ------------ | ---------------------------------------------------------------------- | ---------------------------------- |
| **仓库**     | [`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/) | `Fattydogs/web-video-presentation` |
| **维护者**   | [@ConardLi](https://github.com/ConardLi)                               | `Fattydogs`                        |
| **技能版本** | v1.2.1                                                                 | v1.2.1 + PDF 桥接                  |

如果你不需要 PDF 输入，请直接用上游原版 —— 本 fork 唯一的价值就是支持 PDF 源料。

---

## 我改了什么

相对上游 v1.2.1，**只新增 1 项**内容：

### ➕ PDF 输入桥接

上游版只能接受 `article.md`（markdown 文本）作为内容源。本 fork 增加
了**明确的桥接流程**，让你可以直接喂 **PDF 文件**（论文 / 技术报告
/ 白皮书 / eBook）作为输入。桥接以**文档 + 工作流指引**形式实现 —
— 没有动上游的脚手架、主题、音频管线的任何一行代码。

**本 fork 新增 / 修改的文件：**

| 文件                            | 改动               | 原因                                                                                               |
| ------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------- |
| `references/PDF-INPUT.md`       | **新增**（176 行） | 4 步 PDF → `article.md` 桥接 + 5 类边界（图、公式、扫描件、加密、超长）+ 自检清单                  |
| `SKILL.md`                      | 改 3 处            | (1) Phase 1.1 "用户给的东西"表新增 PDF 行；(2) 读取指南表加 PDF 桥接提示；(3) "相关资源"表加新条目 |
| `README.md` / `README.zh-CN.md` | **重写**           | 本文件。新增 fork 致谢 + PDF 输入桥接章节                                                          |
| 其它所有文件                    | **未改动**         | 脚手架、主题、音频管线、TTS provider、录屏工作流 —— 全部原样保留                                   |

> **没有删任何代码，没有改任何主题，没有动脚手架。** 唯一的行为变化
> 是：遵循本 skill 的 agent 现在会识别 PDF 输入，并通过
> `PDF-INPUT.md` 路由后，再走原本的 Phase 1.1 → 1.2 → ... 流程。

---

## 这是什么？

`web-video-presentation` 帮 Agent 构建一种 Vite + React + TypeScript 演示：它看起来不是传统幻灯片，而更像为录屏设计的视频舞台。每次点击推进一个口播节拍，每一步独占 1920×1080 舞台，进度 UI 平时隐藏，只有悬浮时出现，方便录出干净画面。

它适合：

- 把文章改写成 B 站 / YouTube / 视频号风格口播稿
- **把 PDF 论文 / 报告做成有旁白的网页视频**（本 fork 新增）
- 把已有口播稿做成有节奏的网页演示
- 做产品演示、教程、keynote 式讲解、视觉 talk
- 做"动态 PPT，但不要像 PPT"的演示体验
- 在视觉 outline 对齐后，可选合成口播音频

这个 Skill 的核心是**方法论 + 协作流程**。脚手架提供 token、舞台原语、主题和示例，但每个项目仍然应该根据主题重新选择视觉语言。

---

## 核心理念

- **固定 16:9 舞台**：内容写在稳定的 1920×1080 坐标系里，再按视口缩放。
- **一个全局 step 游标**：点击或键盘推进 `(chapter, step)`，游标本地持久化。
- **一步一个想法**：每个节拍独占整屏，不堆叠项目符号。
- **口播节拍驱动结构**：讲述节奏直接映射为视觉 step。
- **隐藏 chrome**：进度控制悬浮才出现，录屏画面保持干净。
- **动效优先**：每一步都需要一个移动的视觉锚点，静态正文是坏味道。
- **主题 token**：视觉属性通过语义 token 驱动，换主题不只是换颜色。
- **可插拔 TTS**：provider-agnostic 音频 runner，**内置 2 个 provider**（MiniMax `mmx-cli` + OpenAI TTS via curl）；往 `tts-providers/` 丢一个 `.sh` 就能换成 ElevenLabs / edge-tts / Azure / Google Cloud / macOS `say` / 任何自部署 TTS。
- **硬 checkpoint**：稿子/主题、outline、音频合成前都必须停下来与用户确认。

---

## 工作流

```text
Phase 1.1  识别用户输入
   ├── article.md (markdown) ─────────────────┐
   ├── PDF（本 fork）──► PDF-INPUT.md 桥接     ┤
   └── 已有口播稿 ──────────────────────────────┤
                                               ▼
Phase 1.2  文章 -> 口播稿
   |
Checkpoint A1  稿子、主题、粗略素材计划
   |
Phase 1.3  口播稿 + 原文 -> outline.md
   |
Checkpoint A2  outline 确认 + 开发模式选择
   |
Phase 2    构建 Vite / React / TS 演示
   |
Checkpoint B   询问是否合成音频
   |
Phase 3    可选音频合成
Phase 4    录屏与后期
```

这些 checkpoint 是 Skill 契约的一部分：Agent 不应该从原文一路闷头做到成品。主题选择会影响动效气质，outline 确认能避免章节节奏跑偏。

---

## PDF 输入桥接（本 fork 新增）

如果源料是 PDF，按
[`references/PDF-INPUT.md`](./references/PDF-INPUT.md) 走桥接流程。速览：

### 4 步桥接

```text
PDF 文件
  ↓ 1. Read 工具读取（自动按页转文本）
  ↓ 2. 清洗：去页眉页脚 / 修复断行 / 还原表格 / 公式保留为 $...$ LaTeX
  ↓ 3. 写入 article.md（顶部加元信息头）
  ↓ 4. 走原本的 Phase 1.1 → ... 流程
```

### 速用示例（agent 视角）

```python
# 第 1 步：读 PDF（≤ 20 页一次性读，> 20 页必须分批）
Read(path="paper.pdf")                       # ≤ 20 页
Read(path="paper.pdf", pages="1-20")         # > 20 页，分批
Read(path="paper.pdf", pages="21-45")

# 第 2+3 步：清洗后写入 article.md
Write(path="article.md", content="""\
# <论文标题>

> 来源：paper.pdf（<N> 页 / <作者> / <会议或期刊>）
> 提取方式：Read 工具逐页转文本 + 人工清洗
> 提取日期：<YYYY-MM-DD>

<正文...>
""")

# 第 4 步：skill 原本的 Phase 1.2 看到 article.md 就和其它来源无差异。
```

### 边界情况 —— agent 必须主动告知

| 情况                       | 处理                                                                                                                              |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **图像 / 截图 / 流程图**   | Read 读不了图。用户需手动把图导出为 `assets/figures/fig-N.png`；agent 在 `article.md` 插 `![描述](assets/figures/fig-N.png)` 占位 |
| **公式 / 数学符号**        | 行内 `$...$` / 行间 `$$...$$` 保留；识别错的要回原 PDF 校对                                                                       |
| **扫描件 PDF（无文本层）** | Read 返回空。让用户去找 HTML 版（如 arXiv）或走外部 OCR                                                                           |
| **加密 / DRM**             | 不能绕过。让用户先在合法授权下解密                                                                                                |
| **超长 PDF（> 50 页）**    | 不要全读。让用户标出本次视频覆盖的章节 / 页码范围，或建议拆成多集                                                                 |

完整自检清单见
[`references/PDF-INPUT.md`](./references/PDF-INPUT.md#自检清单-pdf-桥接完成时过一遍)。

---

## 内含内容

```text
skills/web-video-presentation/
├── SKILL.md
├── README.md / README.zh-CN.md
├── references/
│   ├── PDF-INPUT.md            ← 本 fork 新增
│   ├── CHAPTER-CRAFT.md
│   ├── OUTLINE-FORMAT.md
│   ├── SCRIPT-STYLE.md
│   ├── THEMES.md
│   ├── AUDIO.md
│   └── RECORDING.md
├── scripts/
│   └── scaffold.sh
├── templates/
│   ├── index.html
│   ├── vite.config.ts
│   ├── scripts/
│   │   ├── extract-narrations.ts
│   │   ├── synthesize-audio.sh       # provider-agnostic runner
│   │   └── tts-providers/            # 一个文件 = 一个 TTS 后端
│   │       ├── README.md             # 三函数契约 + ElevenLabs / edge-tts / Azure / Google / say 的现成片段
│   │       ├── minimax.sh            # 默认 provider（mmx-cli）
│   │       └── openai.sh             # 内置：OpenAI TTS（curl + OPENAI_API_KEY）
│   └── src/
└── themes/                    # 23 套主题，每套独立设计签名
    ├── midnight-press/
    ├── warm-keynote/
    ├── newsroom/
    ├── bauhaus-bold/
    └── ...                     # 完整列表见 references/THEMES.md
```

---

## 快速上手

把这个 Skill 复制到你的 Agent 会扫描的目录，然后让 Agent 把一篇文章、口播稿、**或 PDF** 做成网页视频演示。

如果要手动脚手架：

```bash
bash skills/web-video-presentation/scripts/scaffold.sh ./presentation --theme=paper-press
```

查看可用主题：

```bash
bash skills/web-video-presentation/scripts/scaffold.sh --list-themes
```

生成的 `presentation/` 是普通 Vite + React + TypeScript 项目。启动后用录屏工具录制 16:9 舞台即可。

---

## 内置主题方向

Skill 内置 **23 套**主题，每套都有独立的设计 DNA —— 不是简单的换色版。下面按底色分两组速览，挑一套接近你目标气质的，或者作为派生新主题的起点。

### 深色（8 套）

- `midnight-press` 暗色印刷 —— 电影感编辑、暖暗底 + 火热橙
- `chalk-garden` 粉笔花园 —— 深石板黑板 + 手写体 + 粉笔黄
- `terminal-green` 终端绿 —— 80 年代磷光终端 + CRT 扫描线
- `blueprint` 工程蓝图 —— 深海军 + 制图青 + 60px 网格
- `dark-botanical` 暗夜植物 —— 暖陶 / 玫粉 / 鎏金叠层，时尚刊物封面
- `neon-cyber` 霓虹赛博 —— 电光青 + 玫红双霓虹，未来派
- `bold-signal` 焦点信号 —— 大橙色焦点色卡 + Archivo Black，pitch deck
- `creative-voltage` 电压创意 —— 饱和电光蓝 + 霓黄 + halftone

### 浅色（15 套）

- `paper-press` 亮色印刷 —— 暖奶油纸 + 火热橙
- `warm-keynote` 暖色 Keynote —— 大圆角 glass slab + 青绿 + 40px 网格
- `newsroom` 报社 —— NYT 大报、奶油 + 墨黑 + 旗红
- `bauhaus-bold` 包豪斯 —— 0 圆角 + 4px 厚边 + 偏移实色阴影
- `sunset-zine` 日落 Zine —— 暖桃 + 玫红 + Fraunces + 虚线剪贴
- `monochrome-print` 黑白印刷 —— 安静精炼，Monocle / Wallpaper 气质
- `vintage-editorial` 复古编辑 —— 俏皮 Fraunces + 几何叠层（圆 / 线 / 点）
- `pastel-dream` 柔光梦 —— 柔粉蓝灰 + 鼠尾草绿 + 右侧 pill 色条
- `split-canvas` 双拼画布 —— 蜜桃 + 薰衣草 50/50 双底色
- `electric-studio` 电光企业 —— 净白 + 电光蓝 + 贴底 4px 蓝条
- `indigo-porcelain` 靛蓝瓷 —— 靛蓝当墨（不是 accent，是字色本身）+ 瓷白
- `forest-ink` 森林墨 —— 森林绿当墨 + 象牙，旧版国家地理感
- `kraft-paper` 牛皮纸 —— 深棕当墨 + 牛皮米 + 紫铜 accent
- `dune` 沙丘 —— 炭褐 + 沙底，几乎无 accent，建筑画廊感
- `swiss-ikb` 瑞士克莱因蓝 —— 极细 200 weight + IKB + 1px 发丝网格

完整 token 契约、每套的设计签名、以及怎么基于现有主题派生新主题（包括 Swiss 黄 / 绿 / 橙变体），见 [THEMES.md](./references/THEMES.md)。

---

## Reference Map

- **[PDF-INPUT.md](./references/PDF-INPUT.md)** —— **本 fork 新增** · PDF → `article.md` 桥接流程 + 边界情况
- [CHAPTER-CRAFT.md](./references/CHAPTER-CRAFT.md) —— 章节实现规则与视觉 checklist
- [OUTLINE-FORMAT.md](./references/OUTLINE-FORMAT.md) —— outline 必须遵循的结构
- [SCRIPT-STYLE.md](./references/SCRIPT-STYLE.md) —— 文章转口播稿规则
- [THEMES.md](./references/THEMES.md) —— 主题 token 契约 + 派生新主题
- [AUDIO.md](./references/AUDIO.md) —— 可选口播音频合成流程（provider-agnostic）
- [tts-providers/README.md](./templates/scripts/tts-providers/README.md) —— TTS provider 三函数契约 + 内置 2 个 (minimax / openai) + ElevenLabs / edge-tts / Azure / Google / macOS say 的现成代码片段
- [RECORDING.md](./references/RECORDING.md) —— 录屏与后期注意事项

---

## 致谢

### 原版技能

- **作者**：[Conard Li (@ConardLi)](https://github.com/ConardLi)
- **源仓库**：[`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/)
- **原 skill 路径**：[`skills/web-video-presentation`](https://github.com/ConardLi/garden-skills/tree/main/skills/web-video-presentation)
- **上游版本**：v1.2.1
- **许可**：继承自上游（见上游仓库 LICENSE 文件）

> 所有方法论、脚手架、主题、音频管线、TTS provider 契约、原文档
> 都是 ConardLi 的工作。本 fork 是衍生作品，新增了 PDF 输入桥接
> 功能（文档在 `references/PDF-INPUT.md` + `SKILL.md`）。

### 本 fork

- **维护者**：`Fattydogs`
- **仓库**：`Fattydogs/web-video-presentation`
- **fork 版本**：v1.2.1 + PDF 桥接
- **唯一新增内容**：`references/PDF-INPUT.md`（新增文件）+ `SKILL.md` 中 3 处小修改（挂上 PDF 桥接指引）
- **其它所有代码**：与上游完全一致

### 如何同步上游更新

```bash
git remote add upstream https://github.com/ConardLi/garden-skills.git
git fetch upstream
git merge upstream/main --no-ff -m "merge upstream vX.Y.Z"
# 若 SKILL.md 出现冲突，优先采用上游版本，再从本 fork 的 commit 历史
# 中重新挂上 3 处 PDF-INPUT.md 引用。
```

---

## 许可

继承自上游 `garden-skills` 仓库。重新分发前请查阅
[`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/)
的 LICENSE 文件。
