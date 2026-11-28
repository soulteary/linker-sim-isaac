The files of the policy gallery entry `ycb-grasp`, version 0.3.0: grasp YCB objects with the left hand of the LW-v0 workstation. Requires linker-sim-isaac 2.0 (`>=2.0.0,<3`).

You don't need to download these by hand. `uv run linker-sim-isaac gallery run ycb-grasp` downloads them into `~/.cache/linker-sim-isaac/gallery/ycb-grasp/`, checks each sha256 and opens the scene. If you set `XDG_CACHE_HOME`, use that directory in place of `~/.cache`. On a machine without internet access, copy both files into that folder; see [docs/gallery.md](https://github.com/linker-bot/linker-sim-isaac/blob/main/docs/gallery.md).

| File | Contents | sha256 |
|---|---|---|
| `policy.npz` | PointNet actor weights | `f189597121e44d1c2526364417f45e2325303d61933b81d0b7dbc965e42f4c67` |
| `objects.npz` | 31 YCB objects and their resting poses | `9e39a4a3c63e7d4a7699584af9aba48a67439fdf249a9fc86ee6dd6f69dc30a1` |

**Licences.** Policy weights: research use only (non-commercial); trained on DexGraspNet objects (CC BY-NC 4.0). Objects: YCB Object and Model Set, CC BY 4.0 (Calli et al., "The YCB Object and Model Set", IEEE Robotics & Automation Magazine, 2015).
