# iccluster-setup

MLO group setup, documentation, and job system information for configuring IC Cluster machines.

## Overview

This repository provides automation scripts and instructions to set up Ubuntu systems for the MLO group. It includes:

- System configuration and package installation scripts
- LDAP/SSSD and PAM mount setup
- Scratch and shared storage configuration
- CUDA installation on first boot
- Development tools (Docker, Bazel, Anaconda, Torch)
- Optional GPU monitoring and Ceph-based container scratch setup
- A Python environment guide and requirements for a PyTorch-oriented workflow

All setup scripts must be executed as root and are tailored for Ubuntu.

## Contents

- mlo-setup-script.sh — Main setup script for Ubuntu hosts in the MLO group
- mlo-haas.sh — Alternative/older Ubuntu setup script (HaaS variant)
- python-env/README.md — Instructions for managing a conda-based Python environment
- python-env/requirement.txt — Python packages for the conda environment
- README.md — This document

## Prerequisites

- Must be run as root (scripts enforce this).
- Target OS: Ubuntu (scripts detect OS; non-Ubuntu paths are not implemented).
- Internet access to fetch packages and configuration from referenced URLs.

## Usage

Run one of the setup scripts as root on a fresh Ubuntu installation. Ensure you understand what each script installs and configures.

- Main MLO setup:
```bash
sudo bash mlo-setup-script.sh
```

- HaaS variant:
```bash
sudo bash mlo-haas.sh
```

Both scripts will remove themselves at the end of execution.

## What the scripts do

### mlo-setup-script.sh (Ubuntu)

- System updates and essentials:
  - apt-get update
  - Installs tools: emacs, tmux, htop, mc, git, subversion, vim, iotop, dos2unix, wget, screen, zsh, software-properties-common, pkg-config, zip, g++, zlib1g-dev, unzip, strace, vim-scripts
  - Compilers/build tools: gdb, cmake, cmake-curses-gui, autoconf, gcc, gcc-multilib, g++-multilib

- Authentication and mounts:
  - Installs and configures SSSD (libpam-sss, libnss-sss, sssd-tools, nfs-common, tcsh)
  - Fetches CA and SSSD config; allows IC-IT-unit, MLO-unit, mlologins groups
  - Enables automatic home directory creation via PAM
  - Installs and configures PAM mount (libpam-mount, cifs-utils, ldap-utils)
  - Disables automount upstart job (manual), runs pam-auth-update
  - Creates /scratch with group permissions (root:MLO-unit)
  - Configures and mounts NFS share /mlodata1 from ic1files.epfl.ch

- CUDA (first boot via rc.local):
  - Schedules CUDA 9.2.88 installation on first boot
  - Ensures /scratch ownership fix on first boot

- Python and Anaconda:
  - Installs python, python3, pip, dev headers, scientific Python packages
  - Installs IPython with all extras via pip
  - Installs Anaconda3 (5.0.1) to /opt/anaconda3 and sets PATH in /etc/environment
  - Installs docker.io and docker-compose (via pip)

- Bazel and Java:
  - Adds Java PPA and installs oracle-java8-installer
  - Adds Bazel apt repo and installs Bazel

- Torch and ML frameworks:
  - Installs Torch (Lua) and its dependencies in /opt/torch
  - Installs PyTorch and torchvision via conda channels
  - Installs TensorFlow (GPU) via conda
  - Installs additional Python packages (tensorboardX, fastdtw, nltk, tpdm, ipdb)

- Permissions and groups:
  - Grants sudo to mlologins group via /etc/sudoers.d/mlologins
  - Adds mlologins to docker group via lab2group.sh
  - Configures mlo-gpu-monitor sudo permissions and runs GPU monitor install script

- NVIDIA Docker and Ceph:
  - Installs specific versions of nvidia-docker2 and nvidia-container-runtime
  - Installs bc and ceph-common
  - Sets up /mlo-container-scratch and mounts Ceph-backed scratch
  - Adjusts dpkg configuration for noninteractive upgrades

### mlo-haas.sh (Ubuntu)

- System updates and essentials:
  - apt-get update
  - Installs tools: emacs, tmux, htop, mc, git, subversion, vim, iotop, dos2unix, wget, screen, zsh, software-properties-common, pkg-config, zip, g++, zlib1g-dev, unzip
  - Compilers/build tools: gdb, cmake, cmake-curses-gui, autoconf, gcc, gcc-multilib, g++-multilib

- Authentication and mounts:
  - Runs ldapAutoMount.sh installer, configures PAM access rules
  - Disables autofs and enables home dir creation via PAM
  - Creates /scratch with group permissions (root:MLO-unit)

- CUDA (first boot via rc.local):
  - Schedules CUDA 8.0.27 installation on first boot

- Docker:
  - Installs Docker via get.docker.com

- Python:
  - Installs python, python3, pip, dev headers, and scientific packages
  - Upgrades IPython via pip
  - Installs NLTK via pip
  - Sets python3 and pip3 as system defaults via update-alternatives

- Bazel and Java:
  - Adds Java PPA and installs oracle-java8-installer
  - Installs Bazel 0.4.1 via the official installer script

## Python environment

See python-env/README.md for full details. Summary:

- Add Anaconda3 to PATH (e.g., in .bashrc):
```bash
export PATH="$PATH:/opt/anaconda3/bin"
```

- Create and use the environment (as root):
```bash
sudo su -
export PATH="$PATH:/opt/anaconda3/bin"
conda create --name pytorch-env
source activate pytorch-env
pip install -r requirement.txt
```

- Optional: Install SentEval in /opt/SentEval:
```bash
cd /opt
git clone https://github.com/facebookresearch/SentEval
cd SentEval/data
bash get_transfer_data_moses.bash
# bash get_transfer_data_ptb.bash
cd ..
python setup.py install
```

- Optional: Build PyTorch from source (note provided comment states it does not work right now):
```bash
cd /opt
export CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"
conda install -c soumith magma-cuda80 --name pytorch-env
git clone --recursive https://github.com/pytorch/pytorch
cd pytorch
python setup.py install
```

- Deactivate and exit:
```bash
source deactivate
exit
```

Required Python packages are listed in python-env/requirement.txt:
- cffi
- cmake
- jupyterlab
- lmdb
- mkl
- numpy
- pyyaml
- setuptools
- tensorboardX
- tensorflow
- tensorflow-tensorboard

## Project structure

- .gitignore
- mlo-haas.sh
- mlo-setup-script.sh
- README.md
- python-env/
  - README.md
  - requirement.txt

## Notes

- Both setup scripts target Ubuntu systems and must be run as root.
- CUDA installation is deferred to first boot via /etc/rc.local in both scripts.
- The scripts fetch configuration and installers from external URLs referenced inline. Ensure network access and verify the sources as needed.
