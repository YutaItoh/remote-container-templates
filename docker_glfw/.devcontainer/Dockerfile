FROM python:3.9.7-slim as base
#FROM nvidia/opengl:1.2-glvnd-devel-ubuntu20.04 as python-base


FROM base as python-base

    ENV PYTHONUNBUFFERED=1 \
        PYTHONDONTWRITEBYTECODE=1 \
        \
        PIP_NO_CACHE_DIR=off \
        PIP_DISABLE_PIP_VERSION_CHECK=on \
        PIP_DEFAULT_TIMEOUT=100 \
        \
        POETRY_VERSION=1.1.5 \
        POETRY_HOME="/opt/poetry" \
        POETRY_VIRTUALENVS_CREATE=false \
        \
        PYSETUP_PATH="/opt/pysetup"


    ENV PATH="$POETRY_HOME/bin:$PATH"

    # For OpenCV installation
    ENV DEBIAN_FRONTEND=noninteractive

    RUN apt update \
        && apt install --no-install-recommends -y \
            curl \
            build-essential \
            git \
            libopencv-dev \
            python3-opencv

    RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

    WORKDIR $PYSETUP_PATH
    ENV POETRY_NO_INTERACTION=1
    #COPY poetry.lock pyproject.toml ./
    COPY ./pyproject.toml ./
    RUN poetry config experimental.new-installer false
    RUN poetry run pip install -U pip setuptools
    RUN poetry install


# Setup GLFW/OpenGL/GLSL
# Partially using:
# https://gitlab.com/nvidia/container-images/opengl/-/tree/ubuntu20.04/
FROM python-base as gl-base


    # Base image layer
    RUN dpkg --add-architecture i386 && \
    apt-get update && apt-get install -y --no-install-recommends \
        libxau6 libxau6:i386 \
        libxdmcp6 libxdmcp6:i386 \
        libxcb1 libxcb1:i386 \
        libxext6 libxext6:i386 \
        libx11-6 libx11-6:i386 && \
    rm -rf /var/lib/apt/lists/*

    # nvidia-container-runtime
    ENV NVIDIA_VISIBLE_DEVICES \
            ${NVIDIA_VISIBLE_DEVICES:-all}
    ENV NVIDIA_DRIVER_CAPABILITIES \
            ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics,compat32,utility

    RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
        echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

    # Required for non-glvnd setups.
    ENV LD_LIBRARY_PATH /usr/lib/x86_64-linux-gnu:/usr/lib/i386-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}:/usr/local/nvidia/lib:/usr/local/nvidia/lib64

    # Runtime image layer 
    RUN apt-get update && apt-get install -y --no-install-recommends \
        libglvnd0 libglvnd0:i386 \
        libgl1 libgl1:i386 \
        libglx0 libglx0:i386 \
        libegl1 libegl1:i386 \
        libgles2 libgles2:i386 && \
    rm -rf /var/lib/apt/lists/*
    COPY ./10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

    # Dev image layer
    RUN apt update && apt install -y --no-install-recommends \
        pkg-config \
        libglvnd-dev libglvnd-dev:i386 \
        libgl1-mesa-dev libgl1-mesa-dev:i386 \
        libegl1-mesa-dev libegl1-mesa-dev:i386 \
        libgles2-mesa-dev libgles2-mesa-dev:i386 \
        libglew-dev libglew-dev:i386 \
        libglfw3-dev libglfw3-dev:i386 && \
    rm -rf /var/lib/apt/lists/*

    RUN apt-get update && apt install -y cmake cmake-gui
    RUN apt clean\
        && rm -rf /var/lib/apt/lists/*

FROM gl-base as development
    # # Setup a local user based on the host system
    # RUN apt update && apt -y install gosu
    # COPY entrypoint.sh /usr/local/bin/entrypoint.sh
    # RUN chmod +x /usr/local/bin/entrypoint.sh
    # ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]


    # [Option] Install zsh
    ARG INSTALL_ZSH="true"
    # [Option] Upgrade OS packages to their latest versions
    ARG UPGRADE_PACKAGES="false"
    # Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
    ARG USERNAME
    # USERNAME must be defined in docker-compose.yml
    # e.g. 
    # app:
    #   - USERNAME=${CONTAINER_USER} 
    # here ${CONTAINER_USER}  is supposed to be defined in .env
    ARG USER_UID=1000
    ARG USER_GID=$USER_UID

    # Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
    COPY library-scripts/common-debian.sh /tmp/library-scripts/
    RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
        && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
        && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
    
    #    WORKDIR /app
    ENV HOME="/home/$USERNAME"
    WORKDIR $HOME
    # ARG USERNAME=user-name-goes-here
    # ARG USER_UID=1000
    # ARG USER_GID=$USER_UID

    # # Create the user
    # RUN groupadd --gid $USER_GID $USERNAME \
    #     && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #     #
    #     # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    #     && apt-get update \
    #     && apt-get install -y sudo \
    #     && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    #     && chmod 0440 /etc/sudoers.d/$USERNAME

# FROM development-base as builder-base
#     WORKDIR $PYSETUP_PATH
#     RUN poetry install --no-dev
    
#     # [Optional] Set the default user. Omit if you want to keep the default as root.
#     USER $USERNAME

# FROM python-base as production
#     COPY --from=builder-base /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages
#     COPY ./src /app/
#     WORKDIR /app
#     # [Optional] Set the default user. Omit if you want to keep the default as root.
#     USER $USERNAME