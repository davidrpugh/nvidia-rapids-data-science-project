FROM rapidsai/rapidsai:cuda10.1-base-ubuntu16.04

LABEL maintainer="pughdr <david.pugh@kaust.edu.sa>"

SHELL [ "/bin/bash", "--login", "-c" ]

RUN apt-get update --fix-missing && \
    apt-get install -y wget bzip2 curl git gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
ARG username=al-khawarizmi
ARG uid=1000
ARG gid=100
ENV USER ${username}
ENV UID ${uid}
ENV GID ${gid}
ENV HOME /home/$USER

RUN adduser --disabled-password \
    --gecos "Non-root user" \
    --uid $UID \
    --gid $GID \
    --home $HOME \
    $USER

# Dockerhub not yet supporting COPY --chown $UID:$GID syntax
COPY environment.yml requirements.txt /tmp/
RUN chown $UID:$GID /tmp/environment.yml /tmp/requirements.txt

COPY postBuild docker/entrypoint.sh /usr/local/bin/
RUN chown $UID:$GID /usr/local/bin/postBuild /usr/local/bin/entrypoint.sh && \
    chmod +x /usr/local/bin/postBuild /usr/local/bin/entrypoint.sh

# install Miniconda as non-root user
USER $USER

ENV MINICONDA_VERSION 4.7.12
ENV CONDA_DIR $HOME/miniconda3
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -O ~/miniconda.sh && \
    chmod +x ~/miniconda.sh && \
    ~/miniconda.sh -b -p $CONDA_DIR && \
    rm ~/miniconda.sh

# make non-activate conda commands available
ENV PATH=$CONDA_DIR/bin:$PATH

# make conda activate commands available from /bin/bash --login shells
RUN echo ". $CONDA_DIR/etc/profile.d/conda.sh" >> ~/.profile

# make conda activate command available from /bin/bash --interactive shells
RUN conda init bash

RUN mkdir $HOME/app
WORKDIR $HOME/app

RUN conda update --name base --channel defaults conda && \
    conda env create --prefix ./env --file /tmp/environment.yml && \
    conda clean --all --yes
RUN conda activate ./env && \
    . /usr/local/bin/postBuild && \
    conda deactivate

# use an entrypoint script to insure conda environment is properly activated at runtime
ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]

# default command will be to launch JupyterLab server for development
CMD [ "jupyter", "lab", "--no-browser", "--ip", "0.0.0.0" ]
