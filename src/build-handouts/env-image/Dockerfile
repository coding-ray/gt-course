ARG UBUNTU_IMAGE_TAG=jammy-20240212

FROM ubuntu:${UBUNTU_IMAGE_TAG}

ARG UID_USER=1000
ARG GID_USER=1000
ARG USERNAME=user
ARG HOME_DIR=/home/"${USERNAME}"

# set timezone and install TeX Live dependencies
# reference: https://stackoverflow.com/a/40235306
RUN ln -fs /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    apt -qqy update && \
    apt -qqy install wget perl tzdata && \
    apt clean

# create a normal user
RUN groupadd -g $GID_USER "${USERNAME}" && \
    useradd -m -g "${USERNAME}" -u $UID_USER -s /bin/bash "${USERNAME}"

# install TeX Live and dependencies with a normal user
USER ${USERNAME}
ENV HOME="$HOME_DIR"
ENV PATH="$HOME/.texlive/bin/x86_64-linux/:$PATH"
ARG INSTALL_TL_URL="https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz"
ARG TL_DOC_PATH=$HOME/.texlive/texmf-dist/doc/
RUN cd /tmp && \
    echo "Downloading TeX Live installer..." && \
    wget -qO- "$INSTALL_TL_URL" --no-check-certificate | tar -xz && \
    cd install-tl-* && \
    echo "Installing TeX Live with basic scheme..." && \
    perl ./install-tl -scheme=scheme-basic --no-interaction -texdir="~/.texlive" -no-doc-install -no-src-install && \
    tlmgr backup -q --all --clean=0 && \
    rm -rf "$TL_DOC_PATH" && \
    find ~/.texlive -name *src* -exec rm {} \; && \
    cd /tmp && \
    rm -rf ./*

# install required packages to build the handouts
ARG PACKAGES=mathtools
RUN echo "Installing TeX Live packages: ${PACKAGES}" && \
    tlmgr install ${PACKAGES} && \
    tlmgr backup -q --all --clean=0 && \
    rm -rf "$TL_DOC_PATH" && \
    find ~/.texlive -name *src* -exec rm {} \;

# patch missing fonts
ARG SCRIPT_PATCH_FONTS=/tmp/patch-missing-fonts.sh
COPY --chown=$USERNAME:$USERNAME ./patch-missing-fonts.sh $SCRIPT_PATCH_FONTS
RUN $SCRIPT_PATCH_FONTS && \
    rm -rf /tmp/*

USER root
WORKDIR /app
COPY entrypoint.sh ./entrypoint.sh
RUN chmod +x ./entrypoint.sh && \
    apt autopurge -qqy wget perl
# after uninstalling perl, you can not use tlmgr command any more

USER ${USERNAME}
ENTRYPOINT [ "./entrypoint.sh" ]