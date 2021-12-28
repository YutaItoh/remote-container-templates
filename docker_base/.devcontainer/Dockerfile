FROM python:3.9.7-slim as base

FROM base as development

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
    ENV HOME="/home/${USERNAME}"
    WORKDIR $HOME
