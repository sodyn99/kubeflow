FROM public.ecr.aws/j1r0q0g6/notebooks/notebook-servers/jupyter-tensorflow-cuda-full:v1.5.0

USER root

RUN rm /etc/apt/sources.list.d/cuda.list \
  && rm /etc/apt/sources.list.d/nvidia-ml.list

RUN apt-get update && apt-get install -y --no-install-recommends \
  sudo \
  apt-utils \
  && usermod -aG sudo ${NB_USER} \
  && echo ${NB_USER}:${NB_USER} | chpasswd \
  && echo "${NB_USER} ALL=(root) NOPASSWD:SETENV: /init" >> /etc/sudoers

USER jovyan