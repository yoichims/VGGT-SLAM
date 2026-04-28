# VGGT-SLAM: Installation and Usage Guide

This guide provides step-by-step instructions for setting up and running [VGGT-SLAM](https://github.com/MIT-SPARK/VGGT-SLAM) in a Conda environment.

### Prerequisites
* **Anaconda / Miniconda** installed.
* **FFmpeg** installed (for extracting image frames from videos).
* **Git** installed.

---

### Step 1: Clone the Repository
First, clone the official VGGT-SLAM repository and navigate into the folder:

`git clone git@github.com:yoichims/VGGT-SLAM.git`
`cd VGGT-SLAM`

### Step 2: Create the Environment
Create and activate a new Conda environment specifically for this project using Python 3.11:

`conda create -n vggt-slam python=3.11`
`conda activate vggt-slam`

### Step 3: Install Dependencies
The repository includes a setup script that installs the core requirements. You need to make it executable and run it:

`chmod +x setup.sh`
`./setup.sh`

**Hardware-Specific PyTorch Installation:**
If you are running on modern high-end GPUs (like the RTX 6000 Blackwell), you need specific versions of PyTorch and xformers to avoid CUDA errors. Run the following commands in order:

1. Install xformers and PyTorch 2.3.1 for CUDA 12.1:
`pip install torch==2.3.1 torchvision==0.18.1 xformers==0.0.27 --index-url https://download.pytorch.org/whl/cu121`

2. Upgrade to the PyTorch Nightly build for CUDA 12.8 support (Required for Blackwell GPUs):
`pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/nightly/cu128`

---

### Step 4: Prepare Your Dataset (Video to Frames)
VGGT-SLAM operates on image sequences. If you recorded a video (e.g., IMG_1506.MOV), use ffmpeg to extract the frames at 10 frames-per-second (FPS) and save them to a dataset folder:

`mkdir -p datasets/exp06`
`ffmpeg -i IMG_1506.MOV -vf "fps=10" datasets/exp06/frame_%04d.jpg`

---

### Step 5: Running VGGT-SLAM
You can run the main SLAM script with various parameters depending on your goal. Ensure the `--log_path` points to a `.txt` file so the outputs format correctly.

**1. Standard Mapping & Visualization**
Runs the SLAM pipeline, builds the map in the Viser browser visualizer, and logs the poses:
`python3 main.py --log_results --max_loops 25 --vis_map --vis_voxel_size 0.005 --lc_thres 1.0 --image_folder datasets/exp05 --log_path output_exp_06/poses.txt`

**2. Open-Set Semantic Search (--run_os)**
Enables the SAM 3 and CLIP models to allow text-to-3D search queries on the mapped environment:
`python3 main.py --log_results --max_loops 25 --vis_map --vis_voxel_size 0.005 --lc_thres 1.0 --image_folder datasets/exp07 --log_path output_exp_06/poses.txt --run_os`
*Note: Using --run_os requires downloading gated models from Hugging Face. You must run `huggingface-cli login` and provide a token with "Gated Repo" read access before running this command.*

**3. Optical Flow Visualization (--vis_flow)**
Visualizes the optical flow from RAFT, which is used for keyframe selection:
`python3 main.py --log_results --max_loops 25 --vis_map --vis_voxel_size 0.005 --lc_thres 1.0 --image_folder datasets/exp06 --log_path output_exp_06/poses.txt --vis_flow`

---

### Step 6: Monitoring Resources
Since VGGT-SLAM is heavily reliant on GPU resources (especially with maximum map density or SAM 3 loaded), you should monitor your GPU memory usage. Open a separate terminal and run:

`nvidia-smi -l 2`
*(This refreshes the GPU status every 2 seconds).*

---
### Accessing the Visualizer
Once main.py is running, it will host a web visualizer.
* Open your browser and navigate to: `http://localhost:8081` (or whichever port is printed in the terminal).
* If you are running this on a remote SSH server, ensure you have port forwarded 8081 to your local machine: `ssh -L 8081:localhost:8081 user@remote-ip`
