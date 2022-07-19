# Installation

This repository is tested with CUDA 11.3. Make sure to install [Conda](https://docs.conda.io/en/latest/miniconda.html#linux-installers) before preceding.

<details>
<summary>Installing Conda</summary>

    This step is fairly self-explanatory, but here are the basic steps. You can also find countless tutorials online.

    ```bash
    cd /path/to/install/miniconda

    mkdir -p miniconda3
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda3/miniconda.sh
    bash miniconda3/miniconda.sh -b -u -p miniconda3
    rm -rf miniconda/miniconda.sh
    ```

</details>

Create the python environment

```bash
conda create --name pyrad python=3.8.13
conda activate pyrad
```

Clone the repo

```bash
git clone git@github.com:plenoptix/pyrad.git
```

Install dependencies

```bash
cd pyrad
pip install -r environment/requirements.txt
```

Install pyrad as a library

```bash
pip install -e .
```

Install library with CUDA development support. Change setup.py to `USE_CUDA = True` and then

```bash
python setup.py develop
```

Install tiny-cuda-nn (tcnn) to use with the graph_instant_ngp.yaml config

```bash
pip install git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch
```

# Downloading data

Download the original [NeRF dataset](https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1) and unfold it in the following format. This is for the blender dataset type. We support the major datasets and allow users to create their own dataset, described in detail [here](data/creating_dataset.md).

```
|─ pyrad/
   ├─ data/
   |  ├─ blender/
   |     ├─ fern/
   |     ├─ lego/
         ...
      |- <dataset_format>/
         |- <scene>
         ...
```

# Training a model

To run with all the defaults, e.g. vanilla nerf method with the blender lego image

Run a vanilla nerf model.

```bash
python scripts/run_train.py
```

Run a faster version with instant ngp using tcnn (without the viewer).

```bash
python scripts/run_train.py --config-name=graph_instant_ngp.yaml
```

Run with the viewer. However, you'll have to start the viewer server first. (See the docs.)

```bash
python scripts/run_train.py --config-name=graph_instant_ngp.yaml viewer.enable=true
```

With support for [Hydra](https://hydra.cc/), you can run with other configurations by changing appropriate configs defined in `configs/` or by setting flags via command-line arguments.

# Visualizing training runs

If you run everything with the default configuration, by default, we use [TensorBoard](https://www.tensorflow.org/tensorboard) to log all training curves, test images, and other stats. Once the job is launched, you will be able to track training by launching the tensorboard in `outputs/blender_lego/vanilla_nerf/<timestamp>/<events.tfevents>`.

```bash
tensorboard --logdir outputs
```

Or the following:

```bash
export TENSORBOARD_PORT=<port>
bash environment/run_tensorboard.sh
```

# Rendering a Trajectory

```bash
python scripts/run_eval.py --method=traj --traj=spiral --output-filename=output.mp4 --config-name=graph_instant_ngp.yaml trainer.resume_train.load_dir=outputs/blender_lego/instant_ngp/2022-07-07_230905/checkpoints
```