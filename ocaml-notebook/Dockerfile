# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG OWNER=lumakernel
ARG BASE_CONTAINER=$OWNER/base-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Luma <world@luma.email>"

USER root

RUN apt-get update --yes
RUN apt-get install --yes --no-install-recommends \
    opam \
    git \
    rsync \
    bubblewrap \
    darcs \ \
    g++-multilib \
    gcc-multilib \
    mercurial \
    musl-tools \
    m4 \
    unzip \
    pkg-config \
    libcairo2-dev \
    libgmp-dev \
    libzmq3-dev \
    zlib1g-dev \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

USER ${NB_UID}

RUN opam init \
  --auto-setup \
  --bare \
  --enable-shell-hook \
  --disable-sandboxing
RUN opam switch create -y 4.13.1
RUN opam install -y ocamlfind camlp4 jupyter jupyter-archimedes
RUN opam exec ocaml-jupyter-opam-genspec

USER root
RUN mv "$(opam var share)/jupyter"* "${CONDA_DIR}/share/jupyter/kernels/ocaml-jupyter" && \
    fix-permissions "${CONDA_DIR}/share/jupyter"

USER ${NB_UID}

# https://github.com/ocaml/setup-ocaml
