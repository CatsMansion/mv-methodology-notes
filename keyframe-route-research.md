# 关键帧路线调查报告：如何做得更真实

> 日期：2026-10-22 · 目标：二次元角色手指舞 MV · 本机硬件：RTX 2080 SUPER 8GB VRAM / 驱动 591.74

## 0. 硬件裁决（先说结论，它否决了一半选项）

| 方案 | 显存需求 | 本机可行性 |
|---|---|---|
| ToonCrafter 本地 | 24-27GB（A100）/ 10-12GB（fp16 优化） | **不可行**，8GB 差一截；可用 Colab/云端 |
| SD/SDXL ControlNet 栈 | 8GB 即可跑 IP-Adapter+FaceID+ControlNet | **可行**（社区公认入门线，刚好踩线） |
| EbSynth | 轻量，无硬性要求 | **随时可用**，免费 |
| DWPose 姿势提取 | onnxruntime CPU 可跑 | **可行**，不吃显存 |

结论：**本地 SD 系控制栈生成关键帧 + EbSynth 传播/ToonCrafter 云端插值**是唯一匹配 8GB 的完整链路。

## 1. 姿势数据源（动作真实度的上限）

| 来源 | 质量 | 结论 |
|---|---|---|
| 手指舞 **VMD 配布动作**（MMD 原生格式） | 二次元骨骼原生，动作即所得 | 最优。源：MMD-PV / NicoNico / BowlRoll，搜"モーション配布" |
| VMD → ControlNet 渲染桥 | Civitai "OpenPose-PMX" 模型 | **有坑**：作者明说手部骨骼还没做（"working on adding hands and feet"）。手指舞恰恰只要手——需先验证，或改用 Blender MMD 工具插件渲染 VMD 后直接过 DWPose |
| OpenMMD（真人视频→VMD） | 基于老 OpenPose + 3D-pose-baseline | **不适合**：身体关节级，无手指。身体舞可以，手指舞不行 |
| **已有 PNUT 参考视频 → DWPose 提取** | 21 个手部关键点，抗遮挡、低分辨率下比 OpenPose 稳 | 立即可用，第一块砖 |

## 2. 手部质量（手指舞的生死线）

2026 标准栈，按投入排序：

1. **DWPose 而非 OpenPose**：同样 21 个手部关键点，遮挡/低分辨率/怪角度下明显更稳，骨架更 anatomically plausible。
2. **OpenPose-hand + Depth 双 ControlNet**（各 0.3-0.5 权重）：Depth 补 2D 骨架给不了的信息——手指前后遮挡关系（哪根压哪根）。手指舞双手交叉频繁，这一条是刚需。
3. **Hand LoRA**：SDXL 用 BadHands -0.3~-0.5（负向加载）/ RealisticHands -0.4；FLUX 用 Flux-Hands 0.4。**别超 0.6**——过矫正出"塑料完美手"，在随意场景里反而假。
4. **ADetailer 二次修复**：hand_yolov8n.pt 检测，inpaint denoise 0.4-0.6（比脸高），配手部专用 prompt。
5. **两遍生成法**：先生成整体构图 → 裁手部区放大到 512² → img2img 局部重绘 → 合成。手的像素预算放大 10 倍，细节率最高。
6. **底模选择**：姿势精确控制选 SDXL/SD1.5-anime + ControlNet 生态；FLUX 手部原生更好但 hand-specific ControlNet 不成熟，只能 generate-then-fix。

**反直觉提醒**：通用教程说"避免手掌正对镜头"（易崩姿势）——手指舞/tutting 恰恰是手掌持续对镜头。所以 ControlNet 约束对这条路线不是可选项，是地基。

## 3. 角色一致性（关键帧路线独有风险：每帧换人）

| 方案 | 投入 | 效果 |
|---|---|---|
| **IP-Adapter + FaceID** | 2-4 张参考图（正/侧/表情），8GB 可跑 | 首选，全序列锁五官服饰 |
| **角色 LoRA** | 15-30 张同角色图、25 epoch、约 2 小时 | 最稳，跨景别无漂移 |
| 固定种子 + 图生图参考首帧（重绘 0.3-0.4） | 零成本 | 锁场景色调光线，配合前两者用 |

EbSynth 传播天然带一致性——它只传播已修好的像素，不重新"想象"角色。

## 4. 插值与时间一致性

**EbSynth**（Secret Weapons，PatchMatch 纹理合成，非生成式）：
- 画 1 个关键帧 → 光流传播到全片；专业项目验证过（《Apollo 10½》、Netflix《Wednesday》）
- 免费 720p / Pro $20 4K；**2-3 个关键帧均匀分布**显著抗长片段漂移
- 2026 年独立评测仍认为其风格传播的时间一致性优于新生成式工具

**ToonCrafter**（CUHK + CityU + Tencent，SIGGRAPH Asia 2024，Apache-2.0）：
- 生成式卡通插帧：中间帧当"条件生成"问题，不靠光流——卡通大色块运动/遮挡下光流会糊，它不会
- 草图引导模式：控制运动弧线 + **压背景闪烁**，手速快的片段收益明显
- 硬上限：512x320 / 16 帧；本机 8GB 跑不动（需 10-12GB+），走 Colab
- 与 EbSynth 互补：ToonCrafter 语义级推理中间帧 → EbSynth 像素级锁定传播

## 5. 分辨率链路

512x320 → 竖屏 1080x1920 需 3x+ 超分：Real-ESRGAN / Ultimate SD Upscale（8GB 可行）。
**铁律：超分修不了结构错误**——六指超分还是六指。手部必须在关键帧阶段解决，超分只负责锐化定义。

## 6. 推荐流水线（匹配本机 8GB）

```
PNUT 参考视频 ──DWPose──▶ 手部骨架序列（onnxruntime，CPU 可跑）
                              │
                              ▼
              SD anime 底模 + IP-Adapter 锁角色
              + OpenPose-hand & Depth 双 ControlNet
              + Hand LoRA + ADetailer 二修
                              │
                              ▼
              关键帧逐张人工验手（抽帧目检，同参考视频验证法）
                              │
                              ▼
              EbSynth 传播（本机）/ ToonCrafter Colab（高质量段）
                              │
                              ▼
              超分 1080x1920 ──▶ beatra/remotion 拼 MV
```

## 7. 待调查/待决策

1. [ ] DWPose 本机可跑性验证（onnxruntime + 模型约 200MB，拿 PNUT 视频试提骨架）——整条链路第一块砖，不依赖显存
2. [ ] 手指舞 VMD 配布源注册调查（BowlRoll / MMD-PV）
3. [ ] Civitai OpenPose-PMX 手部骨骼是否已补（当前版本没有，直接关系到 VMD 路线可行性）
4. [ ] 是否安装 ComfyUI（8GB 是门槛，需确认后再动）
