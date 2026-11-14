# film-director

[English](README.md) | **简体中文**

> 流程提炼自一支 123 秒短片《NASCENT》的完整制作：2952 帧，9 幕，一张 RTX 5060 渲完。

一个让 Agent 从零做出有剧本、分幕的电影感短片的 skill：主 Agent 当导演，子 Agent 各做一个模块，画面用 Blender（写实镜头）加 Three.js/Canvas（抽象层、HUD、字幕），再用 Python 和 ffmpeg 合成与混音。

它不是单场景演示的生成器。它管的是一支叙事短片额外需要的东西：定稿的需求文档、由音乐决定的时间线、Agent 之间的模块契约，以及任何昂贵渲染之前的审片节点。

## 流程

1. **拷问需求**：把想法逐条问到底，故事、时长、平台、彩蛋、降级预案都写进一份 `BRIEF.md`。
2. **先生成目标**：先出参考图（角色、关键帧），确认后才开始建模。
3. **先定音乐**：节拍检测决定剪辑点，所有模块只从一份 `timeline.json` 读时间。
4. **拆分工作**：子 Agent 按正交目录分工，只写代码；渲染和审片全部由导演执行。
5. **三个节点**：资产静帧 → 低清动态分镜 → 正式渲染。前一关不过，不进下一关。
6. **盲审**：没有上下文的评审对照参考图给每张关键帧打分，并列出具体改法。
7. **合成交付**：调色、颗粒、混音，按发布平台的码率重新编码测试。

## 目录

```
skills/film-director/
  SKILL.md            skill 本体（中英双语）
  templates/          需求文档、模块契约、评审提示词模板
docs/                 笔记与案例
```

## 安装

把 `skills/film-director` 整个目录复制到 `~/.claude/skills/`（Claude Code）或你所用 Agent 的 skill 目录，然后对 Agent 说"用 film-director 做一支短片"。

需要本机装好 Blender、Node.js、Python（numpy、scipy、Pillow）和 ffmpeg。

## 许可

MIT
