---
title: How to setup WSL2 for Pytorch deep learning.
description: A liminal guide on installing cuda toolkit.
permalink: /blog/{{ title | slug }}/
date: 2025-01-28
layout: layouts/blog.liquid
timetoread: 5 min

tags:
    - posts
---

## My Specs

Just for sharing, here is my PC built specs. (It is not a beefy rig though😅)

OS: Windows 11 Pro 24H2<br/>
WSL2 distro: Ubuntu 22.04.5 LTS
Specs: 
- CPU: AMD Ryzen 5 5600X 6-Core Processor (3.70 GHz)
- Motherboard: B550M Aorus Elite
- Cooling: Corsair RGB fan cooling
- GPU: RTX 3060
- Memory: PNY XLRB DDR4 Ram (8GB) x 4
- Storage (Total: 3TB): 
	- Samsung SSD 980 Evo 1TB x 3

# Install CUDA driver on Windows

First of all, we are going to install NVIDIA App. From the NVIDIA app, we further install the NVIDIA driver in Windows.
WSL2 will automatically using the CUDA driver that installed in native Windows. For updating the driver, just using GeForce Experience to update.

# Installing Cuda 11.8

## Prerequisites

Checking the WSL2 linux kernel and distro information.
```bash
uname -m && cat /etc/*release
```
<div style="max-width: 100%">
<img src="/static/img/post/setup_cuda01.png" alt="check distro uname" style='height: 100%; width: 100%; object-fit: contain'>
</div>

Make sure the gcc and build-essential were installed.
```bash
sudo apt install gcc && \ 
sudo apt install build-essential
sudo apt-get install libfreeimage3 libfreeimage-dev
```

Gets the cuda 11.8 runtime file through wget
```bash
sudo wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
```
Now run the runtime, this might take a few minutes to pop up the selection interface
```bash
sudo sh cuda_11.8.0_520.61.05_linux.run
```

The `PATH` variable needs to include `export PATH=/usr/local/cuda-11.8/bin${PATH:+:${PATH}}`. Nsight Compute has moved to `/opt/nvidia/nsight-compute/` only in rpm/deb installation method. When using `.run` installer it is still located under `/usr/local/cuda-11.8/`.

To add this path to the `PATH` variable:

```bash
export PATH=/usr/local/cuda-11.8/bin${PATH:+:${PATH}}
```

Also we have to change the environment variables for 64-bit system (only for runtime installation).
```bash
export LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Link to cuda 11.8 runtime: [CUDA Toolkit 11.8 Downloads](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Linux)
Documentation: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html

# Install CuDNN v9.4

```bash
sudo install zlib1g
```

Get the network repository of cuda keyring (for Ubuntu 22.04 LTS) and install the deb file. This allows installation of cudnn 9.

```bash
sudo wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
# install the keyring
sudo dpkg -i cuda-keyring_1.1-1_all.deb
```

After that, we can now install cudnn 9 through package manager.
```bash
sudo apt-get -y install cudnn9-cuda-11
```

Now install `libcudnn9-samples` and verify the cudnn installation.

```bash
sudo apt install -y libcudnn9-samples
```

## Verification of CuDNN

Check the directory that the mnistCUDNN located.

```bash
find /usr -name "mnistCUDNN" -type d
# /usr/src/cudnn_samples_v9/mnistCUDNN
```
Go inside that directory and make it.
```bash
cd /usr/src/cudnn_samples_v9/mnistCUDNN && sudo make
```

If the make build success and not prompting any error message. Then congrats! You can proceed and test the Mnist example

```bash
# run the example
./mnistCUDNN
```

<div style="max-width: 100%">
<img src="/static/img/post/setup_cuda02.png" alt="Test MNIST" style='height: 100%; width: 100%; object-fit: contain'>
</div>

Documentation: [Installing cuDNN on Linux — NVIDIA cuDNN v9.4.0 documentation](https://docs.nvidia.com/deeplearning/cudnn/latest/installation/linux.html)
# Install TensorRT

```bash
sudo wget https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.5.0/local_repo/nv-tensorrt-local-repo-ubuntu2204-10.5.0-cuda-11.8_1.0-1_amd64.deb
# install tensorrt deb file
sudo dpkg -i nv-tensorrt-local-repo-ubuntu2204-10.5.0-cuda-11.8_1.0-1_amd64.deb
```

After installation, copy the keyring.
```bash
sudo cp /var/nv-tensorrt-local-repo-ubuntu2204-10.5.0-cuda-11.8/nv-tensorrt-local-EE22FB8A-keyring.gpg /usr/share/keyrings/
```

Next, install `tensorrt` through package manager.
```bash
sudo apt-get install tensorrt
```
and install the library for python3 as well.
```bash
sudo apt-get install python3-libnvinfer-lean
```

and run `dpkg-query -W tensorrt` to verify the installation for tensorRT.

Links
- Documentation: [Installation Guide :: NVIDIA Deep Learning TensorRT Documentation](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html)
- Download Page: [TensorRT 10.x Download | NVIDIA Developer](https://developer.nvidia.com/tensorrt/download/10x)

# Install Miniconda3

Download the installation shell script from Conda website
```bash
sudo wget https://repo.anaconda.com/miniconda/Miniconda3-py310_24.7.1-0-Linux-x86_64.sh
# run the installation shell script
bash ./Miniconda3-py310_24.7.1-0-Linux-x86_64.sh
```

Set auto activation base to false by default.
```bash
conda config --set auto_activate_base false
```
Here is the command to create a new conda environment with Python 3.10.
```bash
conda create -n <your_env_name> python=3.10
```

Link: [Latest Miniconda installer links by Python version — Anaconda documentation](https://docs.anaconda.com/miniconda/miniconda-other-installer-links/)

# VSCode Setup

My favourite IDE for development is VSCode. After setup WSL2 with cuda toolkit installed, 
further grab VSCode installer from official website and install VSCode. In VSCode you need to install
WSL extension to connect with your local WSL2 environment

<div style="max-width: 100%">
<img src="/static/img/post/setup_cuda03.png" alt="WSL extension" style='height: 100%; width: 100%; object-fit: contain'>
</div>

# Pytorch verification

Now install pytroch and test the following code snippet.
```python
import torch

if __name__ == '__main__':
	if torch.cuda.is_available():
		print('cuda')
	else:
		print('cpu')
```
The code should output `cuda` if everything is setup correctly.

# 🐧Move WSL2 distribution outside from C drive (Optional)

Link: [GitHub - LpCodes/Moving-WSL-Distribution-to-Another-Drive: This guide provides step-by-step instructions on how to move a Linux distribution installed on Windows Subsystem for Linux (WSL) to a different drive.](https://github.com/LpCodes/Moving-WSL-Distribution-to-Another-Drive)

