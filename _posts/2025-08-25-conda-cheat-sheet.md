---
layout: post
title:  "Conda Cheat Sheet"
date:   2025-08-25 11:00:00
permalink: conda-cheat-sheet
categories: python conda miniconda anaconda ml devtools
---

## Setup

- Install Miniconda [from the command line](https://www.anaconda.com/docs/getting-started/miniconda/install#macos-2)
- Uninstall it using [this script](https://www.anaconda.com/docs/getting-started/miniconda/uninstall#macos%2Flinux)

## Maintenance

- List available commands: `conda commands`
- Remove unused environments and caches: `conda clean --all -y`
- Check health: `conda doctor`

## Configuration

- Display info about current install: `conda info`
- Describe all available settings: `conda config --describe`
  - Describe `some_setting` only: `conda config --describe some_setting`
- Display current settings: `conda config --show`
- Set `foo` to `bar` globally: `conda config --set foo bar`
- Set `foo` to `bar` in the active environment: `conda config --set foo bar --env`

## Environments

- List all environments: `conda env list`
- Create environment `myenv`: `conda create -n myenv`
  - With Python version `version`: `conda create  -n myenv python=version`
  - Run `conda search python` to see available Python versions
- Activate environment `myenv`: `conda activate myenv`
- Deactivate current environment: `conda deactivate`
- Remove environment `myenv`: `conda env remove -n myenv -y`

## Packages

- List all packages in the active environment: `conda list`
- Install package `foo` into the active environment: `conda install foo`
- Update all packages in the active environment: `conda update --all`
- Update package `foo` in the active environment: `conda update foo`
- Uninstall package `foo` from the active environment: `conda uninstall foo`
- You can also explicitly specify the environment, without activating it:
  - List packages in `myenv`: `conda list -n myenv`
  - Install package `foo` into `myenv`: `conda install -n myenv foo`
  - Update all packages in `myenv`: `conda update -n myenv --all`
  - Update package `foo` in `myenv`: `conda update -n myenv foo`
  - Uninstall package `foo` from `myenv`: `conda uninstall -n myenv foo`
