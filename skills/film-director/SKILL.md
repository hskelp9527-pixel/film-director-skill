---
name: film-director
description: Direct a scripted, multi-act cinematic short film end to end with Blender + Three.js + ffmpeg, using sub-agents, review gates and a blind judge. Use when the user wants a story-driven short (trailer, brand film, concept film) made entirely from code. 用 Blender + Three.js + ffmpeg 从零导演一支有剧本的电影感短片，含子 Agent 分工、审片节点与盲审。用户想用代码做一支有故事的短片（预告片、品牌片、概念片）时使用。
---

# film-director

You are the director. You own the brief, the timeline, every render and every verdict. Sub-agents only write code.
你是导演。需求、时间线、每一次渲染和每一个结论都归你。子 Agent 只写代码。

Work through the seven stages in order. Do not start a stage until the previous one has produced its file.
按顺序走完七个阶段。上一阶段的产物文件没落地，不进下一阶段。

| Stage 阶段 | Output 产物 |
|---|---|
| 1 Brief 需求 | `BRIEF.md` |
| 2 Targets 目标图 | `refs/` + approved key frames 确认过的关键帧 |
| 3 Music and timeline 配乐与时间线 | `timeline.json` |
| 4 Contract and sub-agents 契约与分工 | `CONTRACT.md` + module code 各模块代码 |
| 5 Review gates 审片节点 | stills → animatic → final frames |
| 6 Blind judge 盲审 | `judge/*.txt` |
| 7 Composite and deliver 合成交付 | `out/<film>.mp4` |

## 1. Brief / 需求定稿

Interview the user until every item below has an answer, one question at a time, each with your recommended answer. Then freeze it in `BRIEF.md` (start from `templates/BRIEF.template.md`). After the freeze, never reopen requirements; a change means the user edits the brief.

逐条追问用户，一次一个问题，每个问题附上你的推荐答案，直到下面每一项都有答案。然后写进 `BRIEF.md`（从 `templates/BRIEF.template.md` 起稿）并冻结。冻结后不再讨论需求，要改就由用户改文档。

- Story in one sentence, and the single image the audience must remember. 一句话故事，以及观众必须记住的那一个画面。
- Length, aspect, frame rate, platform. 时长、画幅、帧率、发布平台。
- Acts with rough timings and the emotional job of each. 分幕、大致时长、每一幕的情绪任务。
- The hero asset (what gets the most render budget). 主角资产（渲染预算最多给谁）。
- Music source and loudness target. 音乐来源与响度目标。
- On-screen text, subtitles, languages, easter eggs. 画面文字、字幕、语言、彩蛋。
- Hardware and the allowed toolchain. 硬件与允许使用的工具。
- Acceptance checklist the user will actually check. 用户真正会去核对的验收清单。
- Fallback for each risky shot. 每个有风险镜头的降级方案。

## 2. Targets / 参考图与底稿

Quality comes from references, not adjectives. Before any modelling:

质感来自参考图，形容词没用。建模之前：

1. Collect or generate 9–12 reference images for look, light and materials, plus one target image per key frame. 为画风、光线、材质收集或生成 9 到 12 张参考图，每个关键帧再配一张目标图。
2. Get the user to approve the targets. They are the bar for the blind judge in stage 6. 让用户确认目标图。它们就是第 6 阶段盲审的标尺。
3. Write the bar as a number in the brief ("key frames must reach 8/10 next to the references"). A number tells the model the job is not done when it first runs. 把标准写成数字（"关键帧放在参考图旁边至少 8/10"）。数字能让模型明白"能跑"不等于"做完"。

If no image model is available, use public-domain images and say so in the brief. Never skip this stage.
没有生图模型时用公有领域图片，并在需求里写明。这一阶段不能跳过。

## 3. Music and timeline / 配乐与时间线

Music decides the cut. Run beat and energy analysis on the track (numpy/scipy is enough), snap every act boundary and every hit to a beat, and write one `timeline.json`:

音乐决定剪辑。对曲子做节拍和能量分析（numpy/scipy 足够），把每一幕的边界和每一个重音都吸附到拍点上，写成唯一的一份 `timeline.json`：

```json
{ "fps": 24, "total_frames": 2952,
  "acts":   [{ "id": "act1", "start": 48, "end": 237 }],
  "events": { "reveal_hit": { "f": 2585 } } }
```

Every module reads times only from this file. No module hard-codes a frame number.
所有模块只从这份文件读时间，任何模块都不许写死帧号。

## 4. Module contract and sub-agents / 模块契约与子 Agent

Write `CONTRACT.md` (from `templates/CONTRACT.template.md`) before spawning anyone. It fixes:

派人之前先写 `CONTRACT.md`（从 `templates/CONTRACT.template.md` 起稿）。它规定：

- Directory ownership: one sub-agent, one directory, no overlap (for example `blender/robot`, `blender/scenes`, `web`, `audio`). 目录归属：一个子 Agent 一个目录，互不重叠。
- Frame numbering and output folders (`render/<shot>/NNNNN.png`). 帧编号与输出目录。
- The interface between 3D and 2D: a tracks file per shot with screen-space points (`render/tracks/<shot>.json`) so overlays can lock onto rendered objects. 三维和二维之间的接口：每个镜头一份屏幕坐标轨迹文件，让叠加层能锁住渲染出的物体。
- Every web layer exposes `window.renderFrame(n)` and is deterministic: the image depends only on `n`. 每个网页层都暴露 `window.renderFrame(n)`，且必须确定性：画面只由 `n` 决定。
- Every scene script takes `--still N`, `--frames A B`, `--scale`, `--samples` and skips frames that already exist, so renders can resume. 每个场景脚本都支持这四个参数，并跳过已存在的帧，渲染才能断点续跑。

Sub-agent rules / 子 Agent 规则：

- They write code and return. They do not render, test or iterate to green. 只写代码就返回，不渲染、不测试、不反复修到通过。
- Each task brief states inputs, outputs, files they may touch, and what "done" means. 每份任务书写清输入、输出、可改的文件、完成标准。
- The director runs everything and decides what passes. 一切由导演运行，过不过由导演判断。
- When a sub-agent cannot reach the bar after two rounds, the director edits the code directly. 两轮还达不到标准，导演直接改代码。

## 5. Review gates / 审片节点

Three gates. A shot that fails a gate does not move on.
三个节点，没过的镜头不往下走。

1. **Asset stills / 资产静帧**: hero asset alone, neutral and shot lighting, half resolution. 主角资产单独渲染，中性光和镜头光各一版，半分辨率。
2. **Animatic / 动态分镜**: the whole film at low resolution and low samples, with music. Check cut timing, camera moves and overlay alignment here, where a mistake costs minutes. 全片低分辨率低采样，带音乐。剪辑点、运镜、叠加对位都在这里查，这时候改错只花几分钟。
3. **Final / 正式渲染**: only after the first two pass. Long renders run in the background, one queue at a time. 前两关过了才渲。长渲染放后台，一次只跑一个队列。

Look at renders as contact sheets (cells ≤ 640 px), never as full-size frames one by one. For continuity, tile 8–24 consecutive or evenly spaced frames of one shot.
看渲染结果一律用接触表（每格不超过 640 像素），不要一张张看原图。查连贯性时，把同一镜头的 8 到 24 帧连续或等距帧拼成一张。

## 6. Blind judge / 盲审

Your own review is biased: you know what the frame is supposed to be. Before the final render, give each key frame to a sub-agent with **no context** except `templates/judge-prompt.txt`, the frame and its reference. It returns a score and concrete fixes.

你自己审片是有偏的，因为你知道这一帧"应该"是什么。正式渲染前，把每张关键帧交给一个**没有任何上下文**的子 Agent，只给它 `templates/judge-prompt.txt`、这一帧和对应的参考图。它返回分数和具体改法。

- Run it on the hero asset first. 先审主角资产。
- Fix what the judge names, re-render the still, judge again. Keep the verdict files. 按评审点名的问题改，重渲静帧，再审。评审结果文件留档。
- Stop when two rounds in a row gain less than one point, and tell the user plainly what is still unsolved. Do not chase the score forever. 连续两轮涨不到一分就停，并把没解决的问题如实告诉用户，不要无限刷分。
- A design decision frozen in the brief outranks the judge. 需求里冻结的设计决定高于评审意见。

## 7. Composite, mix, deliver / 合成、混音、交付

- One compositor script reads the timeline and stacks layers per frame: 3D base → alpha overlays → per-act grade → bloom → letterbox → subtitles → grain. Pipe raw frames straight into ffmpeg. 一个合成脚本读时间线，逐帧叠层，原始帧直接管道送进 ffmpeg。
- The compositor must have a "few frames to PNG" mode. Test every change on 3–4 frames before a full run. 合成脚本必须有"只出几帧 PNG"的模式，任何改动先用 3 到 4 帧验证。
- Cheap fixes belong here: exposure of a dark shot, a slow push-in on a static hold, moving a label. Try the compositor before re-rendering. 便宜的修正放在这里做：提亮偏暗的镜头、给静止镜头加缓推、挪标签。能在合成里改的不要重渲。
- Normalize loudness to the brief's target and measure the final file, not the mix. 响度按需求目标归一，测量对象是成片，不是混音文件。
- Re-encode at the platform's bitrate and inspect the dark scenes with gamma raised. Grain and a slightly lifted black hide banding; too much grain multiplies file size. 按平台码率重新编码，把暗场提亮检查色带。颗粒加一点抬黑能藏色带，颗粒过重会让文件体积成倍增加。
- Keep rendered frames until the user accepts the film. Delete temp files after. 用户验收前保留渲染帧，验收后再清理临时文件。

Report to the user in this order: what is done and verified, what is unsolved, what was not checked.
向用户汇报的顺序：做完且验证过的、没解决的、没检查的。

## 8. Pitfalls / 踩过的坑

All of these happened in production.
以下每一条都在实际制作中发生过。

- **Self-review passes, the blind judge gives 3/10.** Run the judge before the expensive render, not after. 自审通过，盲审只给 3 分。盲审要放在昂贵渲染之前。
- **Resume-safe renders skip stale frames.** After changing a shot, delete its frame folder before re-rendering. 断点续渲会跳过旧帧。改过镜头后先删它的帧目录再渲。
- **Killing the renderer does not kill the queue script.** The shell loop starts the next shot and competes for the GPU. Kill the queue first, then check that zero render processes remain. 杀掉渲染进程杀不掉队列脚本，循环会接着渲下一个镜头抢显卡。先停队列，再确认渲染进程为零。
- **Headless browser screenshots can come back blank.** Composite the page's canvases in-page and read back `toDataURL` instead. 无头浏览器截图可能是空白的。改为在页面内合并 canvas 再读 `toDataURL`。
- **Scripts run by Blender cannot import their neighbours** until the script's folder is added to `sys.path`. Blender 运行的脚本要先把自身目录加进 `sys.path` 才能导入同目录模块。
- **A backlit human reads as a mannequin.** Make silhouettes pure black; a quick procedural coat looked worse than none. 逆光的人会被看出是人体模型。剪影用纯黑材质；临时加的程序化外套比不加更糟。
- **Skin-derived armour looks muscular.** Flatten the source mesh before cutting plates. 从人体表面切出来的装甲会带肌肉感。切板之前先把源网格削平。
- **The money shot rendered too dark.** Check mean luminance of key frames numerically, not only by eye. 最重要的镜头渲得太暗。关键帧的平均亮度要用数值检查，不能只靠眼睛。
- **Heavy grain made a 123 s film 1.19 GB.** Decide the delivery size first, then the grain. 颗粒太重让 123 秒的片子变成 1.19 GB。先定交付体积，再定颗粒量。
- **Sub-agents hit rate limits mid-task.** Resume the same agent with a message rather than spawning a new one. 子 Agent 中途撞到限额时，给原来的 Agent 发消息续上，不要新开一个。
