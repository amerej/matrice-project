FROM multiarch/debian-debootstrap:armhf-stretch

# Informative labels.
LABEL description="Docker container for Tensorflow on a Raspberry Pi."
LABEL maintainer="audran.ditsch@outlook.fr"

ARG TENSORFLOW_VERSION=1.12.0

RUN apt-get update && apt-get install -y --no-install-recommends \
	ca-certificates \
	build-essential \
	python3 \
	python3-dev \
	libhdf5-dev \
	libzmq3-dev \
	python3-h5py \
	libatlas-base-dev \
	wget \
	pkg-config \
	libfreetype6-dev \
	libpng-dev \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# Install core packages
RUN wget -q -O /tmp/get-pip.py https://bootstrap.pypa.io/get-pip.py \
	&& python3 /tmp/get-pip.py \
	&& rm /tmp/get-pip.py

RUN pip3 --no-cache-dir install -U \
	pip \
	Cython \
	keras \
	numpy \
	ipykernel \
  jupyter \
  matplotlib && \
  python3 -m ipykernel.kernelspec

# Install tensorflow package for Pi 3
ADD tensorflow-${TENSORFLOW_VERSION}-cp35-none-linux_armv7l.whl .
RUN pip3 install tensorflow-${TENSORFLOW_VERSION}-cp35-none-linux_armv7l.whl

COPY jupyter_notebook_config.py /root/.jupyter/

# Copy sample notebooks.
COPY notebooks /notebooks

# Jupyter has issues with being run directly:
# https://github.com/ipython/ipython/issues/7062
# We just add a little wrapper script.
COPY run_jupyter.sh /

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888

# Setting up Jupyter working directory
WORKDIR /notebooks

# Setting up working directory 
# RUN mkdir /lab
# WORKDIR /lab
# # ADD . /lab/

CMD ["/run_jupyter.sh", "--allow-root"]

# Minimize image size
RUN apt-get autoremove -y \
	&& apt-get autoclean -y
