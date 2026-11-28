# linker-sim-isaac 快速入门

linker-sim-isaac 是 Isaac Lab 的扩展，用 NVIDIA GPU 仿真 Linker 工作站 **LW-v0**（两台 AR5 机械臂、两只 O6 灵巧手、桌子和三台相机）。模型包含真机的机器人资产、驱动参数和控制器运动限位。具体任务的表现仍需在真实工作站上验证。

它有三种用法：用 **playground** 手动操作机器人；用 **gallery** 运行预训练策略；用 **任务模板** 编写自己的任务，并用 Isaac Lab 的训练器训练。

仓库、软件包和命令统一使用 `linker-sim-isaac`。Python 中使用 `import linker_sim_isaac`。

本页是中文快速入门，内容与英文文档对应：安装、检查、playground、gallery、任务模板、开发与排查。完整说明见英文的 [getting started](../getting-started.md)、[LW-v0 环境](../environment.md) 和 [API 参考](../api.md)。英文文档是准确的版本；两者不一致时以英文为准。

## 1. 安装

需要：Linux（x86_64 或 aarch64，如 Jetson Thor），glibc 2.35 或更新（Ubuntu 22.04 或更新）；`git`；NVIDIA GPU，驱动 580 或更新；约 10 GB 可用磁盘空间；网络连接。详见 [requirements](../requirements.md)。

```bash
git clone https://github.com/linker-bot/linker-sim-isaac.git
cd linker-sim-isaac
./install.sh
```

`install.sh` 先检查上述要求，在需要时把 uv（Python 包管理器）安装到 `~/.local/bin`，再把 Python 3.12、Isaac Lab 3.0 和物理引擎按 `uv.lock` 中锁定的版本安装到本目录的 `.venv` 中（首次需下载约 4 GB），最后运行 `linker-sim-isaac doctor`。

如果安装程序安装了 uv，当前终端还找不到它（`uv: command not found`）：打开一个新终端，或运行 `source ~/.local/bin/env`。

LW-v0 的机器人模型文件随安装一起装好：来自 [linker-sim-assets](https://github.com/linker-bot/linker-sim-assets) 的软件包 `linker-lw-v0`，版本标签 `v1.0.0`。

## 2. 检查

```bash
uv run linker-sim-isaac doctor
```

它报告已安装的版本、GPU 和引擎补丁；检查通过后，再仿真 LW-v0 一秒钟。每项检查显示 `ok` 或 `FAIL` 及原因；全部通过时最后一行是 `All checks passed.`，有失败时退出码为 1。第一次运行还会编译 GPU 内核（约一分钟，之后有缓存）。

所有命令都通过 `uv run` 在本目录的环境中运行；也可以在每个终端中先执行一次 `source .venv/bin/activate`，之后省去 `uv run`。

## 3. Playground：手动操作机器人

```bash
uv run linker-sim-isaac play
```

窗口中出现工作站，桌上有一个方块。你移动一只手臂的 **掌心点**（掌面中部前方 3.5 cm）的目标，机械臂像真机在 xCore SDK 跟踪模式下一样，在真实控制器的限位内跟随。

| 输入 | 作用 |
| --- | --- |
| I / K、J / L、U / O | 把目标向前 / 后、左 / 右、上 / 下移动（相对于视角） |
| Shift + 上述按键 | 改为绕掌心点旋转目标 |
| 鼠标 | 移动视角；右键拖动物体；拖动手柄移动或旋转目标 |
| Tab | 切换机械臂 |
| 1 / 2 / 3 / 4 | 手：张开 / 握紧 / 捏取 / 拇指对掌 |
| R | 复位 |

W/A/S/D、Q/E 和方向键移动视角，空格暂停，H 隐藏面板，Esc 退出。面板显示机械臂的工具（TCP）位姿（与控制器报告的一致）、掌心点离目标多远、接触和仿真速度。机械臂的速度受真实控制器限位约束，掌心点最快约 0.15 m/s。

Playground 和 gallery 的窗口都需要可用的图形显示环境。`uv run linker-sim-isaac play --selftest` 也会打开窗口；加上 `--out tests/outputs/playground` 可保存报告和截图。

## 4. Gallery：预训练策略

策略权重和物体数据不在仓库里：`run` 第一次运行时从本仓库的 release（如 `ycb-grasp-0.3.0`）下载，校验 sha256 后放入缓存。离线机器上手动放置文件的方法见 [gallery](../gallery.md#the-files)。

```bash
uv run linker-sim-isaac gallery list               # 所有条目
uv run linker-sim-isaac gallery info ycb-grasp     # 说明、许可证和文件
uv run linker-sim-isaac gallery run ycb-grasp      # 打开
```

`run` 把条目的文件（约 12 MB）下载到 `~/.cache/linker-sim-isaac/gallery/`，校验后打开工作站和控制面板。从 31 个训练时未见过的 YCB 物体中选择一个，按 **Drop** 将它放到桌面左侧的随机位置，静置姿态和朝向随机。入选物体及其静置姿态在训练项目指定区域的评测中达到了 95% 的抓取成功率门槛。

等物体静止后按 **Grasp**。策略读取物体文件中的 512 个表面点，以及仿真的物体、机器人状态和示教抓取位姿，一次性修正示教轨迹。左手随后沿轨迹尝试抓取、抬升 15 cm 并保持 2 秒。状态行显示结果，**Reset** 清空桌面。策略权重仅供非商业研究使用，许可证见 `gallery info ycb-grasp`。详见 [Gallery 的操作与评测说明](../gallery.md)。

## 5. 任务模板

[templates/task](../../templates/task/README.md) 是一个独立项目形式的任务：复制这个文件夹，改名，修改任务。例子任务是左手抓起方块：场景、动作、复位、观测、奖励和终止条件都已写好，抓取的控制器或策略由你来写（`run.py` 只让机器人保持初始姿态）。任务配置 `MyTaskEnvCfg` 是 linker-sim-isaac 默认环境 `LWv0EnvCfg` 的子类，按 Isaac Lab 的方式逐项修改场景、动作、事件、观测、奖励和终止条件。

```bash
(
  set -e
  checkout=$(pwd -P)
  cp -r templates/task ~/my-task
  cd ~/my-task
  uv add --editable "$checkout"
  uv run python run.py             # 无窗口，机器人保持初始姿态
  uv run python run.py --viewer
)
```

上面的命令让 `~/my-task` 使用当前仓库里的 linker-sim-isaac，修改源码后即可在任务中验证。分享任务前，请确认模板指定的发布标签已存在，把任务副本中 `pyproject.toml` 的 `linker-sim-isaac` 来源改回该标签，运行 `uv sync` 更新 `uv.lock`，再一起提交这两个文件。详见[模板说明](../../templates/task/README.md)。

加入学习器配置后，可以用 Isaac Lab 自带的 `isaaclab train` / `isaaclab play` 训练和回放，见模板的 README。

## 6. 开发与排查

请在第 1 节克隆的 `linker-sim-isaac` 仓库根目录运行以下命令。第 5 节只在子 shell 中切换目录，执行完后当前终端仍在仓库根目录。

```bash
uv sync --locked --group dev
uv run --no-sync ruff check .
uv run --no-sync python -m pytest tests --ignore=tests/gpu --ignore=tests/distribution
```

这些是不需要 GPU 的测试，但仍需要完整安装（会导入 Isaac Lab 并转换 LW-v0 的文件）；不代表项目支持 CPU 仿真。GPU、窗口和 wheel 测试见[开发指南](../development.md)。

在自己的脚本中，先调用 `linker_sim_isaac.bootstrap()` 或先导入 `linker_sim_isaac.workstation`，再导入 Isaac Lab 的仿真模块；仅执行 `import linker_sim_isaac` 不会应用兼容补丁。版本或导入失败后，先同步锁定依赖，再在新的 Python 进程里运行 doctor。缓存默认位于 `~/.cache/linker-sim-isaac/{usd,objects,gallery}/`，也可以通过 `XDG_CACHE_HOME` 指定根目录。详见[排查说明](../getting-started.md#troubleshooting)。
