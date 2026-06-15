# SC-OrthoGS

Official implementation of the paper **"Orthophoto Generation from Sparse Views via Structurally Consistent Prior-Guided 3D Gaussian Splatting"**.

SC-OrthoGS is a structure-consistent 3D Gaussian Splatting framework designed to address geometric distortion, texture artifacts, and low reconstruction efficiency in orthophoto generation from sparse UAV views. By incorporating matching priors to enforce 3D structural consistency, our method reduces surface discontinuities and voids. We also introduce a custom differentiable orthographic projection mechanism to generate true digital orthophotos (DOMs) directly from sparse inputs.

> **Orthophoto Generation from Sparse Views via Structurally Consistent Prior-Guided 3D Gaussian Splatting**
> Zhenglong Cai, He Huang, Tong Ye, Wenyao Bian, Tianjiao Wang, Junxing Yang*
> Journal of Applied Remote Sensing, 2025 ([DOI: 10.1117/1.JRS.19.044506](https://doi.org/10.1117/1.JRS.19.044506))

📍 If there are any bugs in our code, please feel free to raise your issues.

---

## ⚙ Setup

#### 1. Recommended Environment

We recommend using Conda to set up the environment.

```bash
git clone https://github.com/yourusername/SC-OrthoGS.git
cd SC-OrthoGS

conda env create --file environment.yml
conda activate scorthogs

# Install custom submodules for differentiable rasterization and KNN
git clone https://github.com/ashawkey/diff-gaussian-rasterization --recursive submodules/diff-gaussian-rasterization
git clone https://gitlab.inria.fr/bkerbl/simple-knn.git submodules/simple-knn

pip install submodules/diff-gaussian-rasterization submodules/simple-knn

```


#### 2. Dataset Download

**UAV Benchmark Datasets (Coming Soon)**

Our framework is tested over large-scale aerial datasets (e.g., 10 km² areas). The pre-processed UAV datasets with sparse views are scheduled to be publicly released on Hugging Face by the end of December 2026.

**Customized Datasets**

If you want to train the model on your own data, our pipeline directly supports the standard data formats used in **3D Gaussian Splatting** and **nerfstudio**. You will need to process your raw images with COLMAP first to obtain the camera poses and sparse point clouds.

For detailed instructions on preparing your custom data, please refer to the official guides:

* [3DGS: Processing your own scenes](https://www.google.com/search?q=https://github.com/graphdeco-inria/gaussian-splatting%23processing-your-own-scenes)
* [nerfstudio: Custom Dataset Documentation](https://docs.nerf.studio/quickstart/custom_dataset.html)
*Note: For customized UAV datasets, you first need to run COLMAP to obtain the camera poses and sparse point clouds.*

#### 3. Match Prior Extraction

To enforce 3D structural consistency, we extract matching priors before training. You can utilize models like GIM or DUSt3R for feature matching.

We provide a script `get_match_info.py` in the `data_preprocess` folder. Specify your dataset path and run:

```bash
python data_preprocess/get_match_info.py --data_path <path_to_colmap_workspace>

```

Place the generated `match_data.npy` into your corresponding scene folder:

```text
Dataset_Workspace
  ├── scene_name                  
      ├── images
      ├── sparse
      └── match_data.npy

```

---


## 🗺️ True Digital Orthophoto (TDOM) Generation

To bridge the gap between 3D scene representation and 2D mapping, our framework directly projects the optimized, structurally-consistent Gaussians into high-fidelity orthophotos. 

To accommodate different hardware environments and precision requirements, we designed a dual-pipeline orthorectification system:

### 🔹 Pipeline A: Lightweight Virtual Camera (Default)
This plug-and-play approach leverages geometric transformations via a virtual infinite-focal camera. 
* **Mechanism:** Native scene transformation without altering the core CUDA kernel.
* **Advantage:** 100% compatible with the vanilla 3DGS ecosystem. No extra compilation is required.
* **Best for:** Rapid rendering and standard urban/flat mapping tasks.

### 🔹 Pipeline B: Jacobian-Guided Rasterization (High-Precision)
For complex topographies, we introduce a customized rasterizer that enforces strict orthogonal projection at the lowest level.
* **Mechanism:** Direct modification of the CUDA rasterization kernel utilizing Jacobian matrix adaptation.
* **Advantage:** Dynamically adapts to extreme terrain variations, minimizing geometric distortions in sparse-view reconstructions.
* **Requirement:** Needs local compilation of our custom submodule.

---

### ⚙️ How to Render

**1. Using the Default Pipeline (No installation required)**  
For immediate results, simply run the virtual camera renderer:
```bash
python render_dom.py -m <path_to_model> -s <path_to_data> --mode virtual

```

**2. Using the High-Precision Pipeline (Advanced)**

If you opt for the Jacobian-based rendering, please compile the custom backend first:

```bash
# Compile the modified orthographic rasterizer
cd submodules/ortho-rasterization
pip install .

# Rendering script for this pipeline will be released in an upcoming update.

```


---

## ⏳ Customized Dataset

Our method is developed on the standard 3DGS architecture. It can seamlessly read and process your customized UAV datasets formatted in the standard [COLMAP procedure](https://github.com/graphdeco-inria/gaussian-splatting). Ensure your image overlaps are sufficient to extract the necessary matching priors, even under sparse capture conditions.

---

## ⚖ Citation

If you find our work or this code useful in your research, please consider citing our paper:

```bibtex
@article{cai2025orthophoto,
  title={Orthophoto generation from sparse views via structurally consistent prior-guided 3D Gaussian splatting},
  author={Cai, Zhenglong and Huang, He and Ye, Tong and Bian, Wenyao and Wang, Tianjiao and Yang, Junxing},
  journal={Journal of Applied Remote Sensing},
  volume={19},
  number={4},
  pages={044506},
  year={2025},
  publisher={SPIE},
  doi={10.1117/1.JRS.19.044506}
}

```

---

## 👩‍💻 Acknowledgements

This code is built upon the foundational work of [gaussian_splatting](https://github.com/graphdeco-inria/gaussian-splatting). We utilize modified rasterization modules based on [diff-gaussian-rasterization](https://github.com/ashawkey/diff-gaussian-rasterization) and draw inspiration from the structural prior extraction processes in [GIM](https://github.com/xuelunshen/gim).
