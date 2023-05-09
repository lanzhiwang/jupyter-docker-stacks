# jupyter

```bash
(base) jovyan@2a49e2001778:~$ jupyter -h
usage: jupyter [-h] [--version] [--config-dir] [--data-dir] [--runtime-dir] [--paths] [--json] [--debug] [subcommand]

Jupyter: Interactive Computing

positional arguments:
  subcommand     the subcommand to launch

options:
  -h, --help     show this help message and exit
  --version      show the versions of core jupyter packages and exit
  --config-dir   show Jupyter config dir
  --data-dir     show Jupyter data dir
  --runtime-dir  show Jupyter runtime dir
  --paths        show all Jupyter paths. Add --json for machine-readable format.
  --json         output paths as machine-readable json
  --debug        output debug information about paths

Available subcommands: bundlerextension dejavu events execute fileid kernel kernelspec lab labextension labhub migrate nbclassic nbconvert nbextension notebook run server serverextension troubleshoot trust
```

--------------------------------------------------------------------------------------

# extensions

### Jupyter bundler extensions

```bash
(base) jovyan@2a49e2001778:~$ jupyter bundlerextension -h
Work with Jupyter bundler extensions
```

### JupyterLab extensions

```bash
(base) jovyan@2a49e2001778:~$ jupyter labextension -h
Work with JupyterLab extensions
```

### notebook extensions

```bash
(base) jovyan@2a49e2001778:~$ jupyter nbextension -h
Work with Jupyter notebook extensions
```

### server extensions

```bash
(base) jovyan@2a49e2001778:~$ jupyter serverextension -h
Work with Jupyter server extensions
```

--------------------------------------------------------------------------------------

# kernel

### kernel code

```bash
(base) jovyan@2a49e2001778:~$ jupyter run -h
Run Jupyter kernel code.
```

### kernel locally

```bash
(base) jovyan@2a49e2001778:~$ jupyter kernel -h
Run a kernel locally in a subprocess
```

### Manage Jupyter kernel

```bash
(base) jovyan@2a49e2001778:~$ jupyter kernelspec -h
Manage Jupyter kernel specifications.
```

--------------------------------------------------------------------------------------

# Server

### JupyterLab

```bash
(base) jovyan@2a49e2001778:~$ jupyter lab -h
JupyterLab - An extensible computational environment for Jupyter.
This launches a Tornado based HTML Server that serves up an HTML5/Javascript JupyterLab client.
```

### Jupyter HTML Notebook

```bash
(base) jovyan@2a49e2001778:~$ jupyter nbclassic -h
The Jupyter HTML Notebook.
This launches a Tornado based HTML Notebook Server that serves up an HTML5/Javascript Notebook client.
```

```bash
(base) jovyan@2a49e2001778:~$ jupyter notebook -h
The Jupyter HTML Notebook.
This launches a Tornado based HTML Notebook Server that serves up an HTML5/Javascript Notebook client.
```

### Jupyter Server

```bash
(base) jovyan@2a49e2001778:~$ jupyter server -h
The Jupyter Server.
This launches a Tornado-based Jupyter Server.
```

--------------------------------------------------------------------------------------

# JupyterHub

```bash
(base) jovyan@2a49e2001778:~$ jupyter labhub -h
Single-user server for JupyterHub. Extends the Jupyter Notebook server.
Meant to be invoked by JupyterHub Spawners, not directly.
```

--------------------------------------------------------------------------------------

# convert notebook files to various other formats

```bash
(base) jovyan@2a49e2001778:~$ jupyter dejavu -h
This application is used to convert notebook files (*.ipynb) to various other formats.
```

```bash
(base) jovyan@2a49e2001778:~$ jupyter nbconvert -h
This application is used to convert notebook files (*.ipynb) to various other formats.
```

--------------------------------------------------------------------------------------

# execute notebook files

```bash
(base) jovyan@2a49e2001778:~$ jupyter execute -h
An application used to execute notebook files (*.ipynb)
```

--------------------------------------------------------------------------------------

# validate JSON schemas

```bash
(base) jovyan@2a49e2001778:~$ jupyter events --help
A simple CLI tool to quickly validate JSON schemas against Jupyter Event's custom validator.
```

--------------------------------------------------------------------------------------

# jupyter fileid

```bash
(base) jovyan@2a49e2001778:~$ jupyter fileid --help
Jupyter File ID server extension CLI.
```

--------------------------------------------------------------------------------------

# Migrate

```bash
(base) jovyan@2a49e2001778:~$ jupyter migrate -h
Migrate configuration and data from .ipython prior to 4.0 to Jupyter locations.
```

--------------------------------------------------------------------------------------

# troubleshoot

```bash
(base) jovyan@2a49e2001778:~$ jupyter troubleshoot -h
```

--------------------------------------------------------------------------------------

# trust

```bash
(base) jovyan@2a49e2001778:~$ jupyter trust -h
Sign one or more Jupyter notebooks with your key, to trust their dynamic (HTML, Javascript) output.
Otherwise, you will have to re-execute the notebook to see output.
```

