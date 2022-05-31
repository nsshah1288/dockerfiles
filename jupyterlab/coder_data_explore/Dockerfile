# inherit Coder's base image
FROM codercom/enterprise-base:ubuntu

# arg, env
ENV USER_HOME="/home/coder"

# this next system shell PATH may NOT be needed
# ENV PATH="/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/var/tmp/coder/coder-cli"
ARG APP_PATH="${USER_HOME}"
ENV PYTHONPATH $APP_PATH

USER root
RUN apt-get update -y -qq && apt-get install -y -qq fonts-powerline git keychain openssh-server sudo vim zsh rsync

# hide the Jupyter notebook icon - but use jupyter-launcher to launch lab
# RUN mv /usr/local/bin/jupyter /usr/local/bin/jupyter-launcher

# I chose /opt/miniconda3 at random and it seemed to work
# USER coder
ARG CONDA_DIR=/opt/miniconda3
ARG CONDA_VER="py38_4.10.3"

# install conda (channel_priority 'False' is the same as 'strict')
ENV PATH="/opt/miniconda3/bin:${PATH}"
ARG PATH="/opt/miniconda3/bin:${PATH}"
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VER}-Linux-x86_64.sh -O miniconda.sh && \
    /bin/bash miniconda.sh -b -p ${CONDA_DIR} && \
    rm miniconda.sh && \
    ${CONDA_DIR}/bin/conda update -n base conda && \
    ${CONDA_DIR}/bin/conda clean --all -yq && \
    find ${CONDA_DIR} -type f -name '*.pyc' -exec rm -f {} \; && \
    find ${CONDA_DIR} -type f -name '*.so*' -exec strip --strip-all {} \; && \
    find ${CONDA_DIR} -type f -name '*.a*' -exec strip --strip-all {} \; && \
    find ${CONDA_DIR} -type f -name '*.js.map*' -exec strip --strip-all {} \; && \
    rm -rf ${CONDA_DIR}/pkgs && \
    rm -rf ${CONDA_DIR}/lib/*_mc.so && \
    rm -rf ${CONDA_DIR}/lib/*_mc2.so && \
    rm -rf ${CONDA_DIR}/lib/*_mc3.so && \
    rm -rf ${CONDA_DIR}/lib/*_avx512* && \
    rm -rf ${CONDA_DIR}/lib/*_avx.* && \
    ${CONDA_DIR}/bin/conda config --add channels conda-forge && \
    ${CONDA_DIR}/bin/conda config --set channel_priority False && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> ~/.bashrc
    # && \
    # echo "conda activate ${CONDA_ENV}" >> ~/.bashrc

# install necessary conda packages
RUN conda install -y -c conda-forge mamba
COPY mamba_reqs.txt mamba_reqs.txt
RUN conda update -n base -c defaults conda && \
    # conda install -y -c conda-forge mamba && \
    mamba install -y -c conda-forge --file mamba_reqs.txt && \
    conda update -c conda-forge --all && \
    conda clean --all -yq && \
    rm mamba_reqs.txt

# pip install python packages
# note: installing googlemaps via pip, conda version of googlemaps hasn't been updated in years
#
COPY pip_reqs.txt pip_reqs.txt
RUN --mount=type=ssh sudo bash -c "mkdir -p -m 0600 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts"
USER root
RUN --mount=type=ssh pip3 install --ignore-installed --no-cache-dir -r pip_reqs.txt && \
   rm pip_reqs.txt

# RUN --mount=type=secret,id=PYPI_TOKEN --mount=type=ssh pip3 install --index-url https://gitlab_token:$(cat /run/secrets/PYPI_TOKEN)@gitlab.com/api/v4/projects/26581179/packages/pypi/simple --ignore-installed --no-cache-dir -r pip_reqs.txt && \
#     rm pip_reqs.txt

RUN --mount=type=ssh pip3 install --ignore-installed --no-deps git+ssh://git@gitlab.com/jupiterintel/jupiter_csg_utils.git@0.14.9#egg=jupiter-csg-utils[all]
USER coder

# RUN --mount=type=ssh pip install --ignore-installed --no-cache-dir -r pip_reqs.txt && \
#    rm pip_reqs.txt



# USER coder
# RUN mkdir climatescoreglobal ; mkdir climatescoreglobal/query

# RUN --mount=type=ssh pip install --ignore-installed --no-cache-dir -r pip_reqs.txt && \
#    rm pip_reqs.txt

# RUN --mount=type=ssh pip install --ignore-installed --no-deps git+ssh://git@gitlab.com/jupiterintel/jupiter_csg_utils.git@0.14.9#egg=jupiter-csg-utils[all]

# # Install Python packages
# RUN pip3 install pandas && \
#     pip3 install numpy

# # inject the lab icon
# COPY jupyterlab.svg /usr/local/bin/jupyterlab.svg

# configure script to create a dev URL and launch JupyterLab without token or password
COPY ["configure", "/coder/configure"]
RUN chmod +x /coder/configure

# install supervisord
RUN apt-get update && apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR "${USER_HOME}"

# copy example-notebooks folder into the /home directory.
# The 'configure' script will do a lifecycle hook copy.
# We are doing the lifecycle hook because any files copied into /home/coder will be overwritten
# by what is mounted to an EBS volume as a persistent volume.
# More info here: https://coder.com/docs/coder/latest/images/writing
COPY --chown=coder:coder example-notebooks/ /home/example-notebooks

# change back to the coder user
USER coder