## Synthetic image generation for benthic object detection training

This tree adds underwater/AUV features on top of [Infinigen](https://github.com/princeton-vl/infinigen). It is a **known-good snapshot**, not a live follow of upstream.

**Pinned base:** Infinigen **v1.11.2**  
**Commit:** `e254428b40d4b5007fe23293206034885912fb4d` (2024-11-20)  
**Tested on:** Ubuntu 22.04.5, Python 3.11, Blender/`bpy` 4.2.0, CUDA terrain on NVIDIA GPU.

Do **not** `git pull` on `main`. Upstream is hundreds of commits ahead and is a different stack.

---

## Recreate this environment

### 1. Clone the frozen fork branch

This branch is Infinigen v1.11.2 plus the benthic overlay. Do not clone `princeton-vl/infinigen` `main`.

```bash
git clone --recurse-submodules -b benthic-v1.11.2 https://github.com/hdoi5324/infinigen.git
cd infinigen
```

Equivalent pin: base commit `e254428b40d4b5007fe23293206034885912fb4d` (v1.11.2). Submodules at that commit include OcMesher `d3d1441` and infinigen_gpl `10c1d76`.

### 2. System packages (Ubuntu 22.04)

```bash
sudo apt-get update
sudo apt-get install -y wget cmake g++ libgles2-mesa-dev libglew-dev \
  libglfw3-dev libglm-dev zlib1g-dev
```

Need: `g++` 11.x, `cmake` 3.22+. This machine used `g++ 11.4.0` and `cmake 3.22.1`.

### 3. Conda env and install (Python module, full Nature)

```bash
conda create --name infinigen_4_2 python=3.11
conda activate infinigen_4_2
conda install -y conda-forge::gxx=11.4.0 mesalib glew glm menpo::glfw3

export C_INCLUDE_PATH=$CONDA_PREFIX/include
export CPLUS_INCLUDE_PATH=$CONDA_PREFIX/include
export LIBRARY_PATH=$CONDA_PREFIX/lib
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib

pip install -e ".[terrain,vis]"
pip install pycocotools
```

`jinja2` is required (`manage_jobs`). It is already listed in this tree’s `pyproject.toml`.

Optional Blender UI (same 4.2 series):

```bash
bash scripts/install/interactive_blender.sh
```

This checkout has a standalone **Blender 4.2.0** under `./blender/`.

### 4. Versions that worked here

| Piece | Version |
|---|---|
| OS | Ubuntu 22.04.5 |
| Python | 3.11.10 |
| `bpy` / Blender | 4.2.0 |
| numpy | 1.26.4 (`numpy<2` required) |
| landlab | 2.6.0 |
| opencv-python | 4.10.0.84 |
| OpenEXR | 3.3.2 |
| gin-config | 0.5.0 |
| gcc/gxx (conda-forge) | 11.4.0 |
| mesalib | 24.2.7 |
| GPU (optional, CUDA terrain) | NVIDIA, driver 570.x |

Infinigen 1.11.2 install docs: [docs/Installation.md](docs/Installation.md) at the pinned commit.

### 5. Run a benthic scene

```bash
conda activate infinigen_4_2
bash scripts/benthic/generate_images.sh
```

Configs: `infinigen_examples/configs_nature/benthic/`  
Entry point: `infinigen_examples/generate_auv_mission.py`

```bash
python -m infinigen.datagen.manage_jobs -- --output_folder outputs/benthic_demo --num_scenes 1 \
  --configs coral_reef_hd.gin --pipeline_configs local_16GB.gin monocular.gin cuda_terrain.gin hd_coral_reef_datagen.gin
```

---

## What this overlay adds

**Vehicles / cameras**
- Camera-rig spotlights
- Mow-the-lawn camera path (AUV)
- Focal length, sensor size, lens distortion

**Scene**
- Water volume absorption + scattering
- Assets: Black Spiny Urchin, Kina, Pink Handfish, plastic bags, colourboard
- `ComplexSand` material
- Distinct GT colours for instance masks / boxes

---

## Do not

- Fast-forward this branch to current `princeton-vl/infinigen` `main` (500+ commits; Blender/API drift).
- Copy files from `infinigenBenthic` onto this tree (`pyproject.toml` there is Blender 3.6 / Python 3.10).
- Use `infinigenBenthic` for new work; it is an archive. Development is this fork branch.
