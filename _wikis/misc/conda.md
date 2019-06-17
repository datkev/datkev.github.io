---
layout: article
title: Conda
permalink: /wikis/misc/conda
aside:
    toc: true
sidebar:
    nav: wikis
---


Conda Environments

## Basics

Create emtpy environment
```bash
conda create -n <name> python=x.x
```

Create environment from environment file
```bash
conda env create --file=<env-file>
```

List environments
```bash
conda env list
```

Activate environment
```bash
conda activate <env-name>
```

Update conda
```bash
conda update conda
```