# Remore container templates with VScode
Templates for some docker configurations with vscode remote containers.

Our typical use case is to launch a container running on a remote host while using vscode locally to edit the source code in the host machine.
The templates are configured to mount a remote local folder in the remote host machine.

<img src="./doc/overview.png" width="480">

A list of the templates: 
* `./docker_base`
  * A basic remote container template
  * Base image: python:3.9.7-slim
* `./docker_glfw`
  * A graphics library support (OpenGL + GLFW)
  * Base image: python:3.9.7-slim
  * Display variable is set to host machine's display
* `./docker_pytorch`)
  * A GPU-accelerated pytorch environment for a host mahine with CUDA-nabled NVIDIA GPUs.
  * Base image: nvcr.io/nvidia/pytorch:21.10-py3 from [NVIDIA GPU Cloud](https://ngc.nvidia.com/catalog/containers/nvidia:pytorch)
* `./docker_ximea`
  * A Ximea camera-supported image.
  * Base image: python:3.9.7-slim
  * This image downloads [Ximea Linux Software Package](https://www.ximea.com/support/wiki/apis/ximea_linux_software_package) at the build time.
  * The container currently mounts the entire `/dev/bus/usb` of the host machine

## Requirement

* Visual Studio Code
* Visual Studio Code extensions:
  * Remote - Containers
* Docker Desktop 2.0+
  * (For Windows, WSL2 backend) 

## How to use

### Step 1: Choose a template project
* Choose one of the template projects and open the folder in the vscode editor.

### Step 2: Configure a remote host
Create a setting file, `.vscode/settings.json`,  in the template project and specify your remote machine:

Example:
```json
{
    "docker.host": "ssh://user@192.168.1.x:22"
}
```

### Step 3: Configure .env file's variables
We make a remote host's folder accessible from the container.

Open `.devcontainer/.env`, and configure variables to be compatible with your remote machine's variables:

Example:
```bash
CONTAINER_USER="username"
CONTAINER_WORKSPACE="/workspace"
HOST_WORKSPACE="/your/host/machine/workspace:"
```

With this setting, the pipeline will perform the following:
* creates a local user `username` in the image
* mounts `/your/host/machine/workspace` in the remote host machine to `/workspace` in the container when launching.

*NOTE*: The current hard-code requirement in `.env` is due to the limitation in the vscode remote container extension where `docker-compose` is called in the local machine environment instead of inside the remote host machine.


### Step 4: Build and Open the container

In the vscode editor, press `F1` and choose `Remote-Containers: Rebuild and Reopen in Container`
