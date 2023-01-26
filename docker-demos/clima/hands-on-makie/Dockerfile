FROM jupyter/base-notebook:latest as deps

USER root

ENV JULIA_NAME "julia-1.9.0-beta2-linux-x86_64.tar.gz"
RUN wget https://julialang-s3.julialang.org/bin/linux/x64/1.9/${JULIA_NAME} && \
    tar -xvzf ${JULIA_NAME} && \
    mv julia-1.9.0-beta2 /opt/ && \
    ln -s /opt/julia-1.9.0-beta2/bin/julia /usr/local/bin/julia && \
    rm ${JULIA_NAME}

ENV mainpath ./
RUN mkdir -p ${mainpath}

USER ${NB_USER}

COPY Project.toml ${mainpath}/Project.toml

ENV USER_HOME_DIR /home/${NB_USER}
ENV JULIA_PROJECT ${USER_HOME_DIR}
ENV JULIA_DEPOT_PATH ${USER_HOME_DIR}/.julia

RUN julia --project=${mainpath} -e "import Pkg; Pkg.instantiate(); Pkg.precompile()"
RUN julia -e "import Pkg; Pkg.add(\"IJulia\"); Pkg.precompile()"

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential && \
    apt-get install -y --no-install-recommends git-all && \
    apt-get install -y --no-install-recommends unzip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

USER ${NB_USER}

COPY environment.yml ${mainpath}/environment.yml
COPY setup.py ${mainpath}/setup.py
COPY runpluto.sh ${mainpath}/runpluto.sh
COPY plutoserver ${mainpath}/plutoserver

RUN jupyter labextension install @jupyterlab/server-proxy && \
    jupyter lab build && \
    jupyter lab clean && \
    pip install ${mainpath} --no-cache-dir && \
    rm -rf ~/.cache

COPY hands-on-makie.ipynb ${mainpath}/hands-on-makie.ipynb
COPY clima-plot.jl ${mainpath}/clima-plot.jl
COPY widgets.jl ${mainpath}/widgets.jl
COPY time-series ${mainpath}/time-series

RUN mkdir .dev
RUN mv build plutoserver.egg-info .dev
