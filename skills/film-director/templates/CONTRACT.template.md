# Module contract / 模块契约

<!-- WIP template. Shared by all sub-agents, maintained by the director. -->

- Time: absolute frame numbers; act boundaries come only from `timeline.json`.
- Ownership: each sub-agent writes only inside its own directory.
- Sub-agents do not render, build or test; the director runs everything.
- Render outputs: `render/<shot>/%05d.png`, `render/web/<layer>/%05d.png`.
- Tracking data from 3D to 2D layers: `render/tracks/<shot>.json`.
- Asset interfaces: function signatures exported by each module.
