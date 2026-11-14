# film-director

**English** | [简体中文](README.zh-CN.md)

> Distilled from the full production of NASCENT, a 123-second short: 2,952 frames, 9 acts, rendered on one RTX 5060.

An agent skill for making a scripted, multi-act cinematic short film end to end: one lead agent acts as the director, sub-agents build the modules, and the film is rendered with Blender (photoreal shots) + Three.js/Canvas (abstract layers, HUD, subtitles) + Python/ffmpeg (compositing and audio).

It is not a single-scene demo generator. It covers what a narrative film needs on top of that: a locked brief, a music-driven timeline, a module contract between agents, and review gates before any expensive render.

## Pipeline

1. **Grill the brief** – question the idea until story, length, platform, easter eggs and fallback plans are fixed in one `BRIEF.md`.
2. **Generate the targets** – reference images first (characters, key frames), approved before anything is modelled.
3. **Music first** – beat detection decides the cut points; every module reads time from one `timeline.json`.
4. **Split the work** – sub-agents own orthogonal directories and only write code; the director runs every render and review.
5. **Three gates** – asset stills → low-res animatic → final render. A gate that fails does not move on.
6. **Blind judge** – a context-free reviewer scores each key frame against its reference and lists concrete fixes.
7. **Composite and deliver** – grade, grain, mix, platform re-encode test.

## Layout

```
skills/film-director/
  SKILL.md            the skill (bilingual)
  templates/          BRIEF, module contract, judge prompt
docs/                 notes and case studies
```

## Install

Copy the `skills/film-director` folder into `~/.claude/skills/` (Claude Code) or your agent's skill directory, then tell the agent "use film-director to make a short film".

Requires Blender, Node.js, Python (numpy, scipy, Pillow) and ffmpeg on the machine.

## License

MIT
