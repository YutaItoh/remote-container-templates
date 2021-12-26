# FROM python:3.9.7-slim as python-base
FROM python:3.8-slim-buster as base


ENV DEBIAN_FRONTEND=noninteractive

FROM base as ximea-base

#    ENV HOME="/home/root"
    WORKDIR "/home/$USERNAME"
    # Setup Ximea driver
    RUN apt-get update \
        && apt install -y \
            libtiff5-dev udev usbutils wget  \
            sed gawk lsof pciutils lshw curl \
            libopencv-dev \
        && apt clean\
        && rm -rf /var/lib/apt/lists/*

    # Download and decompress XIMEA SDK
    RUN wget https://www.ximea.com/downloads/recent/XIMEA_Linux_SP.tgz &&\
        tar xzf XIMEA_Linux_SP.tgz
    # Edit an install script to remove sudo headers for root user installation on Docker
    RUN cat ./package/install | sed -e 's/sudo //g' > ./package/install2 &&\
        chmod +x ./package/install2
    # Run the installer of XIMEA SDK
    RUN /lib/systemd/systemd-udevd --daemon&& \
        cd ./package && ./install2
    RUN rm XIMEA_Linux_SP.tgz \
        && rm -rf ./package


FROM ximea-base as python-base
    # Keeps Python from generating .pyc files in the container
    ENV PYTHONDONTWRITEBYTECODE=1
    # Turns off buffering for easier container logging
    ENV PYTHONUNBUFFERED=1

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

    RUN apt-get update \
        && apt-get install --no-install-recommends -y \
            curl \
            build-essential \
            git \
            python3-opencv \
        && rm -rf /var/lib/apt/lists/*

    RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

    WORKDIR $PYSETUP_PATH
    ENV POETRY_NO_INTERACTION=1
    #COPY poetry.lock pyproject.toml ./
    COPY ./pyproject.toml ./
    RUN poetry config experimental.new-installer false
    RUN poetry run pip install -U pip setuptools
    RUN poetry install


FROM python-base as development
    # [Option] Install zsh
    ARG INSTALL_ZSH="true"
    # [Option] Upgrade OS packages to their latest versions
    ARG UPGRADE_PACKAGES="false"
    # Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
    ARG USERNAME=vscode
    ARG USER_UID=1000
    ARG USER_GID=$USER_UID

    # Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
    COPY library-scripts/common-debian.sh /tmp/library-scripts/
    RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
        # Here add a user to plugdev for XIMEA camera
        && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
        && gpasswd -a ${USERNAME} plugdev \
        && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
    
    #    WORKDIR /app
    ENV HOME="/home/$USERNAME"
    WORKDIR $HOME

# FROM development-base as builder-base
#     WORKDIR $PYSETUP_PATH
#     RUN poetry install --no-dev

# FROM python-base as production
#     COPY --from=builder-base /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages
#     COPY ./src /app/
#     WORKDIR /app