# 🎹 Piano Orchestrator

[![GitHub](https://img.shields.io/badge/GitHub-Code-black)](https://github.com/JeffreyZhou798/Piano-Orchestrator)
[![ModelScope](https://img.shields.io/badge/ModelScope-Live_Demo-blue)](https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05)
[![Python](https://img.shields.io/badge/Python-3.10-green)](https://www.python.org/)
[![Gradio](https://img.shields.io/badge/Gradio-6.8.0-orange)](https://gradio.app/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

**Upload a piano score — get a full multi-instrument orchestration. AI-driven, note-for-note melody-locked.**

Piano Orchestrator takes a **two-hand piano grand-staff score** (MusicXML / MXL / MIDI) and re-orchestrates it into a complete multi-instrument arrangement: your melody stays **untouched on top**, while METEOR generates N accompaniment tracks with the instruments you choose.

🔗 **GitHub**: https://github.com/JeffreyZhou798/Piano-Orchestrator
🌐 **Live Demo (ModelScope Space)**: https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05

---

## 🌟 Highlights

- **🎹 Real piano input**: works directly on two-hand grand-staff scores — broken chords, inner voices, everything
- **🔒 Melody lock (hard guarantee)**: the top track of every output equals your melody **note-for-note** — same pitches, same onsets, same durations, strictly monophonic
- **🤖 METEOR Re-orchestration**: built on [METEOR](https://github.com/dinhviettoanle/meteor) (*Melody-aware Texture-controllable Symbolic Orchestral Music Generation via Transformer VAE*, IJCAI-25), used through its **native multi-track re-orchestration** path — pure inference, no fine-tuning
- **🎭 Two melody modes**:
  - **Case A (default)** — melody = right-hand **skyline** (highest note per 16th grid); melody notes are stripped from the accompaniment reference for clean part-writing
  - **Case B (override)** — upload a **separate melody file** to pin the melody exactly (e.g. when it lives in an inner voice or the left hand); the piano score is then kept complete as the accompaniment reference
- **🎼 88 GM instruments** across 8 families, each with musicologically accurate profiles (clef, range, monophonic/polyphonic behavior)
- **🎛️ Per-track texture controls**: Rhythmic Intensity, Polyphonicity, Average Pitch, Pitch Diversity — all auto-inferred from your piano score by default ("keep the original flavor")
- **📄 MuseScore-ready output**: MusicXML / MXL / MIDI with key & time signatures preserved
- **🌐 Bilingual UI**: English / 中文
- **⚡ CPU-fast**: KV-cached sampling makes a 16-bar, 3-track arrangement finish in **under a minute** on a free 2-vCPU cloud instance

---

## 🎬 Live Demo

**Try it online — no installation**: [ModelScope Space](https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05)

### Interface

![Piano Orchestrator UI](docs/UI%20PianoOrchetrator2026-09-09%20114157.png)

---

## 🏗️ How It Works

```
Upload piano score (MusicXML / .mxl / MIDI)  +  optional melody score
        │
        ▼  piano_analyzer — grand-staff detection, right/left-hand split,
        │   skyline melody (Case A) or user melody (Case B),
        │   16th-grid quantization, PPQ=480, explicit key/time/tempo
   Standard METEOR Re-orchestration reference
        │  Track 0 : monophonic melody (locked)
        │  Track 1 : left hand (full)
        │  Track 2 : right hand (Case A: minus melody / Case B: complete)
        │  (MIDI input → one combined accompaniment track)
        ▼  REMI+ tokenization + attribute analysis
       METEOR  (native multi-track Re-orchestration, KV-cached inference)
        ▼
   Final Multi-Track Score (.musicxml / .mxl / .midi)
        top = your melody (unchanged) · N accompaniment tracks in your
        order · bottom track automatically carries the bass role
```

**Zero-obstacle design**: generation is never blocked by validation — every accompaniment track is guaranteed non-empty and non-all-rests. METEOR is the **only** generation model in the pipeline; there is no alternate model and no degradation path.

---

## 🎼 Instrument Support

**88 General MIDI instruments** mapped onto METEOR's 64 MMT instrument space:

| Family | Examples |
|--------|----------|
| **Keyboards** | Acoustic Grand Piano, Harpsichord, Celesta, Marimba, Glockenspiel… |
| **Organs** | Drawbar Organ, Church Organ, Accordion, Harmonica… |
| **Guitars / Basses** | Nylon/Steel Guitar, Electric Guitar, Acoustic Bass, Synth Bass… |
| **Strings** | Violin, Viola, Cello, Contrabass, String Ensemble, Harp… |
| **Woodwinds** | Flute, Oboe, Clarinet, Bassoon, Saxophones, Piccolo, Pan Flute… |
| **Brass** | Trumpet, Trombone, Tuba, French Horn, Brass Section… |
| **Synthesizers** | Synth Lead, Synth Pad, Synth Strings, Synth Brass… |
| **Ethnic / Percussion / Voice** | Sitar, Banjo, Shamisen, Koto, Kalimba, Timpani, Choir… |

Polyphonic instruments (piano, organ, guitar, harp…) stay polyphonic even on the bass track; monophonic instruments (violin, winds, brass…) are kept monophonic. Every generated note is checked against the instrument's real-world range.

---

## 🚀 Run Locally

```bash
# Clone the repository
git clone https://github.com/JeffreyZhou798/Piano-Orchestrator.git
cd Piano-Orchestrator/PianoOrchestrator-ModelScope01

# Install dependencies (Python 3.10)
pip install -r requirements.txt

# Run the app
python app.py
```

**Model weights** (~305 MB) are downloaded automatically from
[ModelScope: JeffreyZhou2026/meteor_checkpoint](https://www.modelscope.cn/models/JeffreyZhou2026/meteor_checkpoint/)
on first start — never during generation.

---

## 📖 Usage

1. **Upload** a two-hand piano score (MusicXML / MXL / MIDI) — and optionally a separate melody file to pin the melody exactly
2. **Configure**: number of accompaniment tracks (1–12), one of 88 instruments per track; Key / Mode / Tempo are auto-detected from the score and overridable
3. **Generate** — watch the real, asynchronous progress bar
4. **Download** the result as MusicXML, MXL, or MIDI (opens directly in MuseScore)

### Advanced per-track texture

Unfold the advanced panel to control per track: **Rhythmic Intensity** (8 levels), **Polyphonicity** (8 levels), **Average Pitch** (13 levels, MIDI 10–130), **Pitch Diversity** (13 levels). Leave them at `0` (default) and each value is inferred from your piano score's own texture.

---

## 🔧 Technical Notes

| | |
|---|---|
| **Model** | METEOR (Transformer VAE, ~67M) — native multi-track Re-orchestration |
| **Runtime** | Gradio 6.8.0, PyTorch 2.3.1 (CPU) |
| **Deployed on** | ModelScope Space — free CPU (2 vCPU + 16 GB) |
| **Sampling** | KV-cached incremental decoder, numerically equivalent to the original full-window forward (max logits Δ ≈ 1e-5), ~70× faster |
| **Melody guarantee** | locked copy, verified note-for-note by automated tests (pitch / onset / duration / attack count) |

---

## 📁 Repository Structure

```
PianoOrchestrator-ModelScope01/
├── app.py                  # Gradio application
├── backend/
│   ├── piano_analyzer.py   # grand-staff analysis, skyline/user melody
│   ├── meteor_engine.py    # METEOR inference orchestration
│   ├── midi_utils.py       # merge / MusicXML / MXL export
│   ├── config.py            # 88-instrument profiles & mappings
│   └── i18n.py              # EN/中文 dictionaries
├── meteor_src/              # METEOR inference source (REMI+, model, sampler)
│   ├── fast_sampler.py      # KV-cached incremental decoding
│   └── generate.py          # generation loop
├── custom_data/             # instrument register / repeatability tables
├── Dockerfile / setup.sh    # ModelScope build (weights pre-downloaded)
└── requirements.txt
```

---

## 🤝 Acknowledgments

- **METEOR**: [dinhviettoanle/meteor](https://github.com/dinhviettoanle/meteor) — *Melody-aware Texture-controllable Symbolic Orchestral Music Generation via Transformer VAE* (IJCAI-25)
- Pre-trained weights hosted on ModelScope

---

## 📄 License

MIT License

---

## 👤 Author

**Jeffrey Zhou**

---
---

# 🎹 Piano Orchestrator（钢琴谱多声部编配器）

[![GitHub](https://img.shields.io/badge/GitHub-代码-black)](https://github.com/JeffreyZhou798/Piano-Orchestrator)
[![ModelScope](https://img.shields.io/badge/ModelScope-在线演示-blue)](https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05)
[![Python](https://img.shields.io/badge/Python-3.10-green)](https://www.python.org/)
[![Gradio](https://img.shields.io/badge/Gradio-6.8.0-orange)](https://gradio.app/)

**上传钢琴谱 —— 生成完整的多乐器管弦乐编配。AI 驱动，旋律逐音锁定。**

上传**双手钢琴大谱表乐谱**（MusicXML / MXL / MIDI），系统自动重新编配为完整的多乐器作品：你的旋律**原封不动**保留在最上方，METEOR 按你选择的乐器生成 N 条伴奏轨。

🔗 **GitHub**：https://github.com/JeffreyZhou798/Piano-Orchestrator
🌐 **在线演示（ModelScope 创空间）**：https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05

---

## 🌟 亮点

- **🎹 真实钢琴输入**：直接处理双手大谱表乐谱——分解和弦、内声部，全部支持
- **🔒 旋律锁定（硬性保证）**：输出最上方声部与你的旋律**逐音一致**——音高、起音、时值完全相同，严格单声部
- **🤖 METEOR 重编配**：基于 [METEOR](https://github.com/dinhviettoanle/meteor)（*Melody-aware Texture-controllable Symbolic Orchestral Music Generation via Transformer VAE*，IJCAI-25），使用其**原生多轨 Re-orchestration** 路径——纯推理，无需微调
- **🎭 双旋律模式**：
  - **情况 A（默认）**——旋律 = 右手 **skyline**（逐 16 分网格最高音）；伴奏参考中剔除旋律音，声部更干净
  - **情况 B（覆盖）**——额外上传**单独旋律谱**精确指定旋律（适用于旋律在内声部或左手的情况）；此时钢琴谱完整保留为伴奏参考
- **🎼 88 种 GM 乐器**，8 大类分组，每件乐器都有符合音乐学常识的特征配置（谱号、音域、单音/复音性质）
- **🎛️ 每轨织体控制**：节奏强度、复音密度、平均音高、音高多样性——默认全部从你的钢琴谱自动推断（"保持原味"）
- **📄 MuseScore 直接打开**：输出 MusicXML / MXL / MIDI，保留调号与拍号
- **🌐 双语界面**：英文 / 中文
- **⚡ CPU 快速**：KV 缓存采样让 16 小节 3 轨编配在**免费 2vCPU 云实例上一分钟内**完成

---

## 🎬 在线演示

**在线体验（无需安装）**：[ModelScope 创空间](https://www.modelscope.cn/studios/JeffreyZhou02/PianoOrchestrator05)

### 界面截图

![Piano Orchestrator 界面](docs/UI%20PianoOrchetrator2026-09-09%20114157.png)

---

## 🏗️ 工作原理

```
上传钢琴谱 (MusicXML / .mxl / MIDI)  +  可选旋律谱
        │
        ▼  piano_analyzer —— 大谱表检测、左右手分离、
        │   skyline 旋律（情况 A）或用户旋律（情况 B）、
        │   16 分网格量化、PPQ=480、显式调号/拍号/速度
   标准 METEOR 重编配参考
        │  Track 0 : 单声部旋律（锁定）
        │  Track 1 : 左手（完整）
        │  Track 2 : 右手（情况 A 剔除旋律 / 情况 B 完整保留）
        │  （MIDI 输入 → 合并为单一伴奏轨）
        ▼  REMI+ 词元化 + 织体属性分析
       METEOR  （原生多轨 Re-orchestration，KV 缓存推理）
        ▼
   最终多轨乐谱 (.musicxml / .mxl / .midi)
        最上方 = 你的旋律（零改动）· N 条伴奏轨按你的顺序 ·
        最下方伴奏轨自动承担低音声部
```

**零障碍设计**：生成绝不因验证而中断——每条伴奏轨保证非空且非全休止。METEOR 是管线中**唯一**的生成模型，没有替代模型、没有降级路径。

---

## 🎼 乐器支持

**88 种 General MIDI 乐器**，映射到 METEOR 的 64 种 MMT 乐器空间：

| 类别 | 示例 |
|------|------|
| **键盘** | 大钢琴、羽管键琴、钢片琴、马林巴、钟琴… |
| **风琴** | 拉杆风琴、教堂风琴、手风琴、口琴… |
| **吉他 / 贝斯** | 尼龙/钢弦吉他、电吉他、原声贝斯、合成贝斯… |
| **弦乐** | 小提琴、中提琴、大提琴、低音提琴、弦乐合奏、竖琴… |
| **木管** | 长笛、双簧管、单簧管、大管、萨克斯、短笛、排箫… |
| **铜管** | 小号、长号、大号、圆号、铜管合奏… |
| **合成器** | 合成主音、合成铺底、合成弦乐、合成铜管… |
| **民族 / 打击 / 人声** | 西塔琴、班卓琴、三味线、古筝、卡林巴、定音鼓、合唱… |

复音乐器（钢琴、风琴、吉他、竖琴…）即使承担低音声部仍保持复音；单音乐器（小提琴、木管、铜管…）始终保持单音。每个生成音符都对照乐器真实音域校验。

---

## 🚀 本地运行

```bash
# 克隆仓库
git clone https://github.com/JeffreyZhou798/Piano-Orchestrator.git
cd Piano-Orchestrator/PianoOrchestrator-ModelScope01

# 安装依赖（Python 3.10）
pip install -r requirements.txt

# 运行应用
python app.py
```

**模型权重**（约 305 MB）在首次启动时自动从
[ModelScope：JeffreyZhou2026/meteor_checkpoint](https://www.modelscope.cn/models/JeffreyZhou2026/meteor_checkpoint/)
下载——绝不会发生在点击生成之后。

---

## 📖 使用说明

1. **上传**双手钢琴谱（MusicXML / MXL / MIDI）——可选额外上传旋律谱以精确指定旋律
2. **配置**：伴奏轨数（1–12）、每轨从 88 种乐器中选择；调性 / 调式 / 速度自动从原谱读取且可手动覆盖
3. **生成**——真实异步进度条全程可见
4. **下载** MusicXML、MXL 或 MIDI 结果（MuseScore 直接打开）

### 每轨高级织体设置

展开高级面板可逐轨控制：**节奏强度**（8 档）、**复音密度**（8 档）、**平均音高**（13 档，MIDI 10–130）、**音高多样性**（13 档）。保持 `0`（默认）则全部从你的钢琴谱织体自动推断。

---

## 🔧 技术细节

| | |
|---|---|
| **模型** | METEOR（Transformer VAE，约 67M）——原生多轨 Re-orchestration |
| **运行时** | Gradio 6.8.0、PyTorch 2.3.1（CPU） |
| **部署于** | ModelScope 创空间——免费 CPU（2 vCPU + 16 GB） |
| **采样** | KV 缓存增量解码器，与原版全窗口前向数值等价（max logits Δ ≈ 1e-5），提速约 70 倍 |
| **旋律保证** | 锁定拷贝，自动化测试逐音验证（音高 / 起音 / 时值 / 击键数） |

---

## 📁 仓库结构

```
PianoOrchestrator-ModelScope01/
├── app.py                  # Gradio 应用主程序
├── backend/
│   ├── piano_analyzer.py   # 大谱表分析、skyline/用户旋律
│   ├── meteor_engine.py    # METEOR 推理编排
│   ├── midi_utils.py       # 合并 / MusicXML / MXL 导出
│   ├── config.py            # 88 乐器特征与映射
│   └── i18n.py              # 中英词典
├── meteor_src/              # METEOR 推理源码（REMI+、模型、采样器）
│   ├── fast_sampler.py      # KV 缓存增量解码
│   └── generate.py          # 生成循环
├── custom_data/             # 乐器音区 / 重复度表
├── Dockerfile / setup.sh    # ModelScope 构建（权重预下载）
└── requirements.txt
```

---

## 🤝 致谢

- **METEOR**：[dinhviettoanle/meteor](https://github.com/dinhviettoanle/meteor) —— *Melody-aware Texture-controllable Symbolic Orchestral Music Generation via Transformer VAE*（IJCAI-25）
- 预训练权重托管于 ModelScope

---

## 📄 许可证

MIT License

---

## 👤 作者

**Jeffrey Zhou**
