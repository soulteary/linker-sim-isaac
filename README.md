![LinkerBot and NVIDIA Isaac Lab banner](docs/images/banner.png)

# linker-sim-isaac

The repository, distribution and command share the name `linker-sim-isaac`.
In Python, use `import linker_sim_isaac`.

An Isaac Lab extension providing one Linker workstation, **LW-v0**: two AR5 arms with O6
hands, a table and three cameras, behaving like the real robot. The robot's files, its
drives, the real controller's motion limits and one physics setup are built in. Simulation
models these constraints; task performance still needs validation on the real workstation.

Use it to

- **drive the robot** by hand in the playground (`linker-sim-isaac play`);
- **run pretrained policies** from the gallery (`linker-sim-isaac gallery run ycb-grasp`);
- **write your own tasks** and train on them with Isaac Lab's trainers (`templates/task/`).

For the earlier Mirror and Kaleidoscope commands, see the [v1.0.0 documentation](https://github.com/linker-bot/linker-sim-isaac/tree/v1.0.0#readme).

## Quick start

Needs Linux (x86_64 or aarch64) with glibc 2.35 or newer (Ubuntu 22.04 or newer), git, an NVIDIA GPU with
driver 580 or newer and 10 GB of free disk ([requirements](docs/requirements.md)).

```bash
git clone https://github.com/linker-bot/linker-sim-isaac.git
cd linker-sim-isaac
./install.sh                              # installs into .venv and checks a headless simulation
```

If `install.sh` installed uv, open a new terminal (or `source ~/.local/bin/env`) before
the `uv run` commands.

| Available display | First run | Expected result |
| --- | --- | --- |
| Headless NVIDIA server | Run the [first Python script](docs/environment.md#a-first-script) in the project environment | 16 workstations step for 10 simulated seconds and the policy observation has shape `(16, 52)` |
| Desktop with a display | `uv run linker-sim-isaac play` | A window opens for keyboard and mouse control |
| Desktop with a display | `uv run linker-sim-isaac gallery run ycb-grasp` | The policy and object files download on first use, then the grasp viewer opens |

## Documentation

- [Getting started](docs/getting-started.md): install, the playground, the gallery
- [The LW-v0 environment](docs/environment.md): the scene, frames, arm and hand commands,
  objects, cameras, in your own scripts
- [The task template](templates/task/README.md): your own task, as your own project
- [The policy gallery](docs/gallery.md): the entries, and how to write one
- [API reference](docs/api.md): everything linker-sim-isaac provides, and what stays stable
- [Requirements](docs/requirements.md), [troubleshooting](docs/getting-started.md#troubleshooting)
- [Development and validation](docs/development.md), [changelog](CHANGELOG.md)
- [中文快速入门](docs/zh-CN/README.md)

## Develop

```bash
uv sync --locked --group dev
uv run --no-sync ruff check .
uv run --no-sync python -m pytest tests --ignore=tests/gpu --ignore=tests/distribution
```

These are the tests that need no GPU; they still need the full installation (they import
Isaac Lab and convert LW-v0's files). The GPU, viewer and wheel tests are in
[the development guide](docs/development.md).

## API and versions

[docs/api.md](docs/api.md) is the public API: from 2.0.0 it follows semantic versions, and the
[changelog](CHANGELOG.md) lists every change to it. LW-v0's files come from
[linker-sim-assets](https://github.com/linker-bot/linker-sim-assets) at the release tag
`v1.0.0`, which keeps LW-v0 the same robot throughout linker-sim-assets 1.x.

## Earlier releases of this repository

Releases before 2.0.0 (the last one is `v1.0.0`) are a different package, `linkerbot_sim`
with the Mirror and Kaleidoscope entry points. linker-sim-isaac does not include them or a
compatibility layer: scripts written for them have to be rewritten against
`linker_sim_isaac.workstation` and Isaac Lab's task API.

## License

Apache License 2.0 ([LICENSE](LICENSE), [NOTICE](NOTICE)). The gallery's policy weights
and objects have their own licences, shown by `linker-sim-isaac gallery info ycb-grasp`.
