# mv-methodology-notes

Generalized, reusable methodology notes from MV (music video) production R&D.
**No client assets, no character IP, no project-specific lore are included** — these are
pure technique write-ups meant to be reused across any project.

MV 制作研发中沉淀的**通用、可复用**方法论笔记。
本仓库**不含任何客户素材、角色 IP 或项目专有背景**——全部为纯技术复盘，可跨项目复用。

---

## Contents / 目录

### 1. `keyframe-route-research.md` — 关键帧路线调查报告（8GB 硬件）
How to make believable anime-character finger-dance / dance MVs on consumer hardware.
关键帧路线调研：在消费级硬件上（RTX 2080 SUPER 8GB）做可信的二次元角色舞蹈 MV。

Covers / 涵盖：
- **Hardware verdict** — why local SD-ControlNet stack + EbSynth (local) / ToonCrafter (Colab) is the only 8GB-matching chain.
- **Pose data sources** — VMD distribution motions, DWPose extraction, why OpenMMD is wrong for finger dance.
- **Hand quality** (the life-or-death line for finger dance) — DWPose vs OpenPose, dual ControlNet (OpenPose-hand + Depth), Hand LoRA, ADetailer, two-pass generation, base-model choice.
- **Character consistency** — IP-Adapter + FaceID, character LoRA, fixed-seed img2img.
- **Interpolation & temporal consistency** — EbSynth (patch-match, free) vs ToonCrafter (generative, SIGGRAPH Asia 2024, Apache-2.0).
- **Resolution chain** — upscale cannot fix structural errors (six fingers stay six fingers).
- **Recommended 8GB pipeline** — end-to-end diagram.

### 2. `programmatic-visual-methodology.md` — 程序化视觉方法论反思（通用）
An honest self-critique of a "programmatic visual" MV attempt and the concrete method
to do it right next time.
对自己一版「伪程序化视觉」的诚实复盘，以及下次做对的可用方法。

Covers / 涵盖：
- 7 concrete mistakes — "executing a design with code" vs truly *generating*; lack of scale hierarchy; over-reliance on `sin()`/`smoothstep()`; no feedback (`frame = g(frame_prev, env)`); no math/physics kernel; broken post-processing chain; low event density.
- What real programmatic MVs share (Autechre, Rhizomatiks×Perfume, Max Cooper, Ryoji Ikeda, Nigel Stanford) — a non-visual kernel + strict mapping rules + multi-scale + evolution-not-playback.
- Three runnable numpy kernels: **Curl Noise flow field + particle advection**, **Gray-Scott reaction-diffusion**, **2D wave-field interference**.
- Parameter federation (one audio-spectrum envelope drives everything).
- Density rhythm table; full post-processing chain order.
- Honest capability boundary table (numpy/PIL only, no GPU).

---

## License / 许可

MIT — reuse freely, attribute if convenient.
MIT —— 自由复用，方便时注明出处即可。
