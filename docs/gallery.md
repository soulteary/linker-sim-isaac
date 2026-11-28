# The policy gallery

The gallery is for trying pretrained policies on LW-v0, interactively, in the viewer.
Measuring a policy belongs to the repository it was trained in; the gallery shows it.

```bash
uv run linker-lab gallery list               # the entries
uv run linker-lab gallery info ycb-grasp     # its description, licences and files
uv run linker-lab gallery run ycb-grasp      # open it
```

`run` checks that the entry is made for this linker-lab version, downloads its files into
`~/.cache/linker-lab/gallery/<entry>/` (or under `XDG_CACHE_HOME`), checks each against its sha256 and opens the
entry's scene. It returns when you close the window.

## The files

An entry's weights and data (for `ycb-grasp`: `policy.npz` and `objects.npz`) are not in the
repository or the package. Each entry version is a release of this repository named
`<entry>-<version>` (for example `ycb-grasp-0.3.0`) holding those files; the manifest lists
their URLs and sha256. A download goes to a temporary file and is moved into the cache only
when its sha256 matches, so an interrupted or wrong download never replaces a good file.

Without internet access on the machine, download the files elsewhere and copy them into
`~/.cache/linker-lab/gallery/<entry>/` (with `XDG_CACHE_HOME` set, under that directory
instead of `~/.cache`); `run` uses them if their sha256 matches.

## `ycb-grasp`: grasp YCB objects with the left hand

One workstation and a panel:

- **Object**: one of 31 objects of the YCB Object and Model Set (cans, boxes, bottles, fruit,
  ten stacking cups, balls, a mug, a foam brick, a padlock, a Rubik's cube). None of them was
  among the policy's training objects. The [entry manifest](../policy-gallery/ycb-grasp/manifest.yaml)
  reports how the objects and resting poses were selected: each met a 95% grasp-success
  threshold in the training project's evaluation over the drop region below.
- **Drop**: puts the object at a random spot on the left half of the table (the region the
  policy was trained on: x 0.2-0.62 m, y 0-0.56 m in the environment frame), in one of its
  resting poses at random, turned at random, just above the table. If it does not come to
  rest it is placed again.
- **Grasp** (once the object is at rest): the policy reads 512 surface points stored in `objects.npz`,
  the simulated object pose and velocity, the robot state, and the demonstration grasp pose.
  From this single observation it adjusts the recorded palm path and finger closure. The grasp then runs open loop through
  the arm's path mode within the real controller's limits: approach, close, lift 15 cm, hold
  2 s. The status line says whether the object was held by the left hand alone, 10 cm up, and
  if not, why.
- **Reset**: clears the table and sends the arm home.

Thin and flat objects (cutlery, markers, dice, screwdrivers) and the bowl are not in the
list because grasps are unreliable. The [GPU regression test](../tests/gpu/test_gallery_gpu.py)
checks that all 31 objects settle and that the tomato soup can and apple can each be grasped.

The scene runs at about 0.5-0.8× real time: fastest with the object
at rest, slower while it settles or is grasped (contacts cost time).

**Licences.** The objects are CC BY 4.0 (Calli et al., "The YCB Object and Model Set", 2015).
The policy weights are for research use only (non-commercial). `gallery info` shows the
licences of the weights and the objects.

## Writing an entry

Develop an entry in the `policy-gallery/` folder of a clone of this repository. Editable installs
read entries from that folder, so changes are visible without rebuilding. Wheels include
the gallery inside the `linker_lab` package; ordinary installs read those packaged entries.
Rebuild and reinstall the package to distribute a new or changed entry:

```
policy-gallery/<entry>/
  manifest.yaml   what it is and which files it needs
  play.py         run(artifacts): opens the entry's scene
```

`manifest.yaml` has these fields, all required:

| Field | Content |
| --- | --- |
| `name` | the folder's name |
| `version` | the entry's own version |
| `title`, `description` | what someone trying it sees |
| `license` | the licences of the weights and of any data (objects, scenes) |
| `linker_lab` | the linker-lab versions it runs on, a version range such as `">=2.0,<3"` |
| `artifacts` | file name → `url` and `sha256` of each file it needs (weights, object data) |

`play.py` defines `run(artifacts)`, called with each artifact's local path; it owns
everything else: its scene (usually an `LWv0EnvCfg` with its own objects and actions), the
policy's observations and actions, and the episode. The runner interprets nothing else.
An entry that imports Isaac Lab prepares the engine first, with `linker_lab.bootstrap()` or
by importing `linker_lab.workstation`.

Keep the files out of git: publish them where `url` points, for example as release assets.
Check that each URL serves the matching file before distributing the manifest.

To try an entry before its files are published, put each file where the runner keeps it
(`linker_lab.gallery.artifact_path(entry, file)`, i.e.
`~/.cache/linker-lab/gallery/<entry>/<file>`): a file there is used if its sha256 matches.
