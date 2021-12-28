# FROM python:3.9.7-slim as python-base
FROM nvcr.io/nvidia/pytorch:21.10-py3 as python-base

FROM python-base as development-base

    ENV \
        # Turns off buffering for easier container logging
        PYTHONUNBUFFERED=1 \
        # Keeps Python from generating .pyc files in the container
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
            git

    RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python


    WORKDIR $PYSETUP_PATH
    ENV POETRY_NO_INTERACTION=1
    #COPY poetry.lock pyproject.toml ./
    COPY ./pyproject.toml ./
    RUN poetry config experimental.new-installer false
    RUN poetry run pip install -U pip setuptools
    RUN poetry install

FROM development-base as development

    # [Option] Install zsh
    ARG INSTALL_ZSH="true"
    # [Option] Upgrade OS packages to their latest versions
    ARG UPGRADE_PACKAGES="false"
    # Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
    ARG USERNAME
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

# FROM development-base as builder-base
# RUN poetry install --no-dev

# FROM python-base as production
# COPY --from=builder-base /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages
# COPY ./src /app/
# WORKDIR /app