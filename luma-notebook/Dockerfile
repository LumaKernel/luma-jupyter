FROM lumakernel/haskell-notebook:latest AS haskell
FROM fukamachi/roswell:latest AS roswell

FROM lumakernel/minimal-notebook

LABEL maintainer="Luma <world@luma.email>"

USER root

ENV JULIA_DEPOT_PATH=/opt/julia \
    JULIA_PKGDIR=/opt/julia \
    JULIA_VERSION="1.7.2"

RUN mkdir -p /home/luma/examples

COPY --from=haskell /opt/ghc /opt/ghc
COPY --from=haskell /opt/hvega /opt/hvega
COPY --from=haskell /opt/IHaskell /opt/IHaskell
COPY --from=haskell /opt/stack /opt/stack
COPY --from=haskell /home/luma/ihaskell_examples /home/luma/examples/haskell

ENV PATH "${HOME}/.local/bin:${HOME}/miniconda/bin/:${PATH}"

# ffmpeg for matplotlib anim & dvipng+cm-super for latex labels
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
      ffmpeg \
      dvipng \
      cm-super \
      fonts-dejavu \
      unixodbc \
      unixodbc-dev \
      r-cran-rodbc \
      gfortran \
      gcc \
      git \
      libtinfo-dev \
      libzmq3-dev \
      libcairo2-dev \
      libpango1.0-dev \
      libmagic-dev \
      libblas-dev \
      liblapack-dev \
      libffi-dev \
      libgmp-dev \
      gnupg \
      netbase \
      # for ihaskell-graphviz
      graphviz \
      # for ihaskell-gnuplot
      gnuplot-nox \
      # for Stack download
      curl \
      # Stack Debian/Ubuntu manual install dependencies
      # https://docs.haskellstack.org/en/stable/install_and_upgrade/#linux-generic
      g++ \
      gcc \
      libc6-dev \
      libffi-dev \
      libgmp-dev \
      make \
      xz-utils \
      zlib1g-dev \
      gnupg \
      netbase \
      # ocaml
      opam \
      git \
      rsync \
      bubblewrap \
      darcs \
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
      # CommonLisp
      gpg \
      gettext \
      sbcl \
      ecl \
      libzmq3-dev \
      lsb-release \
      # Maintenance Tools
      git \
      less \
      wget \
      curl \
      jq \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Stack Linux (generic) Manual download
# https://docs.haskellstack.org/en/stable/install_and_upgrade/#linux-generic
#
# So that we can control Stack version, we do manual install instead of
# automatic install:
#
#    curl -sSL https://get.haskellstack.org/ | sh
#
ARG STACK_VERSION="2.7.3"
ARG STACK_BINDIST="stack-${STACK_VERSION}-linux-x86_64"
RUN    cd /tmp \
    && curl -sSL --output ${STACK_BINDIST}.tar.gz https://github.com/commercialhaskell/stack/releases/download/v${STACK_VERSION}/${STACK_BINDIST}.tar.gz \
    && tar zxf ${STACK_BINDIST}.tar.gz \
    && cp ${STACK_BINDIST}/stack /usr/bin/stack \
    && rm -rf ${STACK_BINDIST}.tar.gz ${STACK_BINDIST} \
    && stack --version

# Stack global non-project-specific config stack.config.yaml
# https://docs.haskellstack.org/en/stable/yaml_configuration/#non-project-specific-config
RUN mkdir -p /etc/stack
COPY stack.config.yaml /etc/stack/config.yaml
RUN fix-permissions /etc/stack

ENV STACK_ROOT=/opt/stack

# fix-permissions for /usr/local/share/jupyter so that we can install
# the IHaskell kernel there. Seems like the best place to install it, see
#      jupyter --paths
#      jupyter kernelspec list
RUN    mkdir -p /usr/local/share/jupyter \
    && fix-permissions /usr/local/share/jupyter \
    && mkdir -p /usr/local/share/jupyter/kernels \
    && fix-permissions /usr/local/share/jupyter/kernels

# Now make a bin directory for installing the ihaskell executable on
# the PATH. This /opt/bin is referenced by the stack non-project-specific
# config.
RUN    mkdir -p /opt/bin \
    && fix-permissions /opt/bin
ENV PATH ${PATH}:/opt/bin

# Specify a git branch for IHaskell (can be branch or tag).
# The resolver for all stack builds will be chosen from
# the IHaskell/stack.yaml in this commit.
# https://github.com/gibiansky/IHaskell/commits/master
# IHaskell 2021-12-25
ARG IHASKELL_COMMIT=db8bd784344649be5c5bd072f23cdccb037d93a9

# Specify a git branch for hvega
# https://github.com/DougBurke/hvega/commits/main
# hvega 2021-12-30
# hvega-0.12.0.0
# ihaskell-hvega-0.4.0.0
ARG HVEGA_COMMIT=10704fe4f43ef21d7daceaadfd8a25997840d7b8

ARG EXAMPLES_PATH=/home/luma/examples

# Clone IHaskell and install ghc from the IHaskell resolver
RUN ( \
  fix-permissions /opt/IHaskell \
  && fix-permissions $STACK_ROOT \
  && fix-permissions /opt/hvega \
)

# Cleanup
# Don't clean IHaskell/.stack-work, 7GB, this causes issue #5
#   && rm -rf $(find /opt/IHaskell -type d -name .stack-work) \
# Don't clean /opt/hvega
# We can't actually figure out anything to cleanup.

# Bug workaround for https://github.com/jamesdbrock/ihaskell-notebook/issues/9
RUN mkdir -p /home/$NB_USER/.local/share/jupyter/runtime \
  && fix-permissions /home/$NB_USER/.local \
  && fix-permissions /home/$NB_USER/.local/share \
  && fix-permissions /home/$NB_USER/.local/share/jupyter \
  && fix-permissions /home/$NB_USER/.local/share/jupyter/runtime

# Install system-level ghc using the ghc which was installed by stack
# using the IHaskell resolver.
RUN mkdir -p /opt/ghc && ln -s `stack path --compiler-bin` /opt/ghc/bin \
  && fix-permissions /opt/ghc
ENV PATH ${PATH}:/opt/ghc/bin

# hadolint ignore=SC2046
RUN set -x && \
  julia_arch=$(uname -m) && \
  julia_short_arch="${julia_arch}" && \
  if [ "${julia_short_arch}" == "x86_64" ]; then \
    julia_short_arch="x64"; \
  fi; \
  julia_installer="julia-${JULIA_VERSION}-linux-${julia_arch}.tar.gz" && \
  julia_major_minor=$(echo "${JULIA_VERSION}" | cut -d. -f 1,2) && \
  mkdir "/opt/julia-${JULIA_VERSION}" && \
  wget -q "https://julialang-s3.julialang.org/bin/linux/${julia_short_arch}/${julia_major_minor}/${julia_installer}" && \
  tar xzf "${julia_installer}" -C "/opt/julia-${JULIA_VERSION}" --strip-components=1 && \
  rm "${julia_installer}" && \
  ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia

# Show Julia where conda libraries are \
RUN mkdir /etc/julia && \
  echo "push!(Libdl.DL_LOAD_PATH, \"${CONDA_DIR}/lib\")" >> /etc/julia/juliarc.jl && \
  # Create JULIA_PKGDIR \
  mkdir "${JULIA_PKGDIR}" && \
  chown "${NB_USER}" "${JULIA_PKGDIR}" && \
  fix-permissions "${JULIA_PKGDIR}"

USER ${NB_UID}

# Install Python 3 packages
RUN mamba install --quiet --yes \
  # scipy
  'altair' \
  'beautifulsoup4' \
  'bokeh' \
  'bottleneck' \
  'cloudpickle' \
  'conda-forge::blas=*=openblas' \
  'cython' \
  'dask' \
  'dill' \
  'h5py' \
  'ipympl'\
  'ipywidgets' \
  'matplotlib-base' \
  'numba' \
  'numexpr' \
  'pandas' \
  'patsy' \
  'protobuf' \
  'pytables' \
  'scikit-image' \
  'scikit-learn' \
  'scipy' \
  'seaborn' \
  'sqlalchemy' \
  'statsmodels' \
  'sympy' \
  'widgetsnbextension'\
  'xlrd' \
  # R language
  'r-base' \
  'r-caret' \
  'r-crayon' \
  'r-devtools' \
  'r-e1071' \
  'r-forecast' \
  'r-hexbin' \
  'r-htmltools' \
  'r-htmlwidgets' \
  'r-irkernel' \
  'r-nycflights13' \
  'r-randomforest' \
  'r-rcurl' \
  'r-rmarkdown' \
  'r-rodbc' \
  'r-rsqlite' \
  'r-shiny' \
  'r-tidyverse' \
  'rpy2' \
  'unixodbc' \
  && mamba clean --all -f -y \
  && fix-permissions "${CONDA_DIR}" \
  && fix-permissions "/home/${NB_USER}"

# `r-tidymodels` is not easy to install under arm
# hadolint ignore=SC2039
RUN set -x && \
    arch=$(uname -m) && \
    if [ "${arch}" == "x86_64" ]; then \
        mamba install --quiet --yes \
            'r-tidymodels' && \
            mamba clean --all -f -y && \
            fix-permissions "${CONDA_DIR}" && \
            fix-permissions "/home/${NB_USER}"; \
    fi;

# Install facets which does not have a pip or conda package at the moment
WORKDIR /tmp
RUN git clone https://github.com/PAIR-code/facets.git && \
    jupyter nbextension install facets/facets-dist/ --sys-prefix && \
    rm -rf /tmp/facets && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# These packages are not easy to install under arm
RUN set -x && \
    arch=$(uname -m) && \
    if [ "${arch}" == "x86_64" ]; then \
        mamba install --quiet --yes \
            'r-rmarkdown' \
            'r-tidymodels' \
            'r-tidyverse' && \
            mamba clean --all -f -y && \
            fix-permissions "${CONDA_DIR}" && \
            fix-permissions "/home/${NB_USER}"; \
    fi;

# Add Julia packages.
# Install IJulia as $NB_USER and then move the kernelspec out
# to the system share location. Avoids problems with runtime UID change not
# taking effect properly on the .local folder in the $NB_USER home dir.
RUN julia -e 'import Pkg; Pkg.update()' && \
    julia -e 'import Pkg; Pkg.add("HDF5")' && \
    julia -e 'using Pkg; pkg"add IJulia"; pkg"precompile"' && \
    # move kernelspec out of home \
    mv "${HOME}/.local/share/jupyter/kernels/julia"* "${CONDA_DIR}/share/jupyter/kernels/" && \
    chmod -R go+rx "${CONDA_DIR}/share/jupyter" && \
    rm -rf "${HOME}/.local" && \
    fix-permissions "${JULIA_PKGDIR}" "${CONDA_DIR}/share/jupyter"

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME="/home/${NB_USER}/.cache/"

RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions "/home/${NB_USER}"

RUN \
# Install the IHaskell kernel at /usr/local/share/jupyter/kernels, which is
# in `jupyter --paths` data:
       stack exec ihaskell -- install --stack --prefix=/usr/local \
    # Install the ihaskell_labextension for JupyterLab syntax highlighting
    && npm install -g typescript \
    && cd /opt/IHaskell/jupyterlab-ihaskell \
    && npm install \
    && npm run build \
    && jupyter labextension install . \
    # Cleanup
    && npm cache clean --force \
    && rm -rf /home/$NB_USER/.cache/yarn \
    # Clean jupyterlab-ihaskell/node_nodemodules, 86MB
    && rm -rf /opt/IHaskell/jupyterlab-ihaskell/node_modules

RUN conda install --quiet --yes \
    # ihaskell-widgets needs ipywidgets
    'ipywidgets' && \
    # ihaskell-hvega doesn't need an extension. https://github.com/jupyterlab/jupyter-renderers
#    'jupyterlab-vega3' && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

RUN opam init \
  --auto-setup \
  --bare \
  --enable-shell-hook \
  --disable-sandboxing

RUN opam switch create -y 4.13.1
RUN opam install -y ocamlfind camlp4 jupyter jupyter-archimedes
RUN opam exec ocaml-jupyter-opam-genspec

# CommonLisp
RUN jupyter serverextension enable --user --py jupyterlab
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
      jupyterlab-edit-magic cytoscape-clj kekule-clj resizable-box-clj \
      ngl-clj jupyterlab-debugger-restarts
RUN jupyter nbextension enable --user --py widgetsnbextension
RUN jupyter lab build
RUN curl -kLO https://beta.quicklisp.org/quicklisp.lisp
RUN sbcl --non-interactive --load quicklisp.lisp \
      --eval "(quicklisp-quickstart:install)" \
      --eval "(ql-util:without-prompting (ql:add-to-init-file))"
RUN ecl --load quicklisp/setup.lisp \
      --eval "(ql-util:without-prompting (ql:add-to-init-file))" \
      --eval "(uiop:quit)"

USER root
RUN mv "$(opam var share)/jupyter"* "${CONDA_DIR}/share/jupyter/kernels/ocaml-jupyter" && \
    fix-permissions "${CONDA_DIR}/share/jupyter"

USER ${NB_UID}

# CommonLisp
RUN git clone https://github.com/yitzchak/shasht.git $HOME/quicklisp/local-projects/shasht; \
    git clone https://github.com/yitzchak/cytoscape-clj.git $HOME/quicklisp/local-projects/cytoscape-clj; \
    mv $HOME/quicklisp/local-projects/cytoscape-clj/examples $HOME/examples/cytoscape-clj; \
    git clone https://github.com/yitzchak/kekule-clj.git $HOME/quicklisp/local-projects/kekule-clj; \
    mv $HOME/quicklisp/local-projects/kekule-clj/examples $HOME/kekule-clj; \
    git clone https://github.com/yitzchak/resizable-box-clj.git $HOME/quicklisp/local-projects/resizable-box-clj; \
    git clone https://github.com/yitzchak/ngl-clj.git $HOME/quicklisp/local-projects/ngl-clj; \
    mv $HOME/quicklisp/local-projects/ngl-clj/examples $HOME/examples/ngl-clj; \
    sbcl --non-interactive \
      --eval "(ql:quickload '(:common-lisp-jupyter :cytoscape-clj :kekule-clj :resizable-box-clj :ngl-clj))" \
      --eval "(clj:install :use-implementation t)"; \
    ecl \
      --eval "(ql:quickload '(:common-lisp-jupyter :cytoscape-clj :kekule-clj :resizable-box-clj :ngl-clj))" \
      --eval "(clj:install :use-implementation t)" \
      --eval "(uiop:quit)";

USER root
COPY ./examples/common-lisp $EXAMPLES_PATH
RUN fix-permissions $EXAMPLES_PATH

USER ${NB_UID}

WORKDIR "${HOME}"
