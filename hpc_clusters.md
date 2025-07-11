## HPC Clusters Container Workflow

This guide provides a **general** step-by-step workflow for using Enroot (or similar container runtimes) on HPC clusters. You will learn how to:

1. Import a base container image
2. Create and enter a writable sandbox
3. Mount your project directory
4. Set up a Python virtual environment and install dependencies
5. Export the sandbox as a reusable image
6. Launch the container interactively or in batch mode
7. (Optional) Create a persistent sandbox alias

---

## Prerequisites

- **Enroot** or equivalent container runtime installed and configured on the HPC system.
- **Scheduler** (SLURM, PBS, etc.) for batch jobs, if needed.
- Access credentials for private registries (e.g., NVIDIA NGC) stored in `~/.config/enroot/.credentials` or similar.
- A **project directory** on shared storage (e.g. GPFS, Lustre), referred to below as `$PROJECT_DIR`.

---

## 1. Import a Base Container Image

Choose a container that matches your hardware (GPU drivers, MPI, etc.) and import it:

```bash
# Replace with the desired image URI
enroot import docker://registry.example.com/namespace/image:tag
```

This command pulls and packs the container into a local SquashFS file named like `registry+example+com_namespace_image+tag.sqsh`.

---

## 2. Create a Writable Sandbox

Unpack the image into a sandbox directory for development:

```bash
enroot create --name build-env registry+example+com_namespace_image+tag.sqsh
```

A new directory `build-env` appears under Enroot’s storage (e.g., `/run/enroot/user-*/build-env`).

---

## 3. Launch the Container with Your Project Mounted

Start an interactive shell as root (or your user) and bind-mount your project directory:

```bash
enroot start --root --rw \
  --mount $PROJECT_DIR:/workspace \
  build-env \
  bash
```

- `--root` grants root rights inside the container.
- `--rw` makes the container filesystem writeable for installations.
- `--mount host_path:container_path` binds your code into `/workspace`.

---

## 4. Inside the Container: Set Up Python & Dependencies

Once at the container shell:

```bash
cd /workspace
python3 -m venv venv                # create virtual environment
source venv/bin/activate             # activate it
pip install --upgrade pip            # ensure latest pip
pip install <your-dependencies>      # e.g., numpy, scipy, torch
```

All packages install into `venv/` under your shared project directory, persisting across sessions.

---

## 5. Test Your Environment

Run any quick checks or scripts to verify your setup:

```bash
python -c "import numpy; print(numpy.__version__)"
# Or run your code:
python script.py
```

---

## 6. Exit & Export the Provisioned Image

After confirming everything works, exit the container and export its state to a new image:

```bash
exit
enroot export build-env > my-project-with-deps.sqsh
file my-project-with-deps.sqsh  # verify SquashFS image
```

The file `my-project-with-deps.sqsh` now contains the base image plus your installed packages.

---

## 7. Reuse the Baked Image

### Interactive Launch

```bash
enroot start --root --no-overlay \
  --mount $PROJECT_DIR:/workspace \
  my-project-with-deps.sqsh \
  bash
```

Inside:

```bash
cd /workspace
source venv/bin/activate
python script.py
```

### Batch Submission Example (SLURM)

```bash
#!/bin/bash
#SBATCH --gres=gpu:1 --time=02:00:00 --mem=32G
srun --container-image=file://$PWD/my-project-with-deps.sqsh \
     --container-mounts=$PROJECT_DIR:/workspace \
     bash -lc "cd /workspace && source venv/bin/activate && python script.py"
```

---

## 8. (Optional) Create a Named Sandbox

To avoid specifying the image file, unpack the baked image into a new sandbox alias:

```bash
enroot remove myenv 2>/dev/null
enroot create --name myenv my-project-with-deps.sqsh
```

Then start with:

```bash
enroot start --root --rw \
  --mount $PROJECT_DIR:/workspace \
  myenv \
  bash
```

---

### Notes

- Replace `$PROJECT_DIR` with the path to your actual project.
- Adjust image URIs and dependency lists to fit your technology stack.
- Use `--no-overlay` if your cluster lacks FUSE or kernel overlay support.

This general workflow can be adapted to any HPC container-backed development and batch execution scenario.

