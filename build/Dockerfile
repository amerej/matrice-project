FROM multiarch/debian-debootstrap:armhf-stretch

# Informative labels.
LABEL description="Docker container for Tensorflow on a Raspberry Pi."
LABEL maintainer="audran.ditsch@outlook.fr"

ARG TENSORFLOW_VERSION=1.12.0
ARG PROTOBUF_VERSION=3.6.1
ARG PROTOBUF_DIR=/usr/local/src/protobuf
ARG OPENCV_VERSION=4.0.1
ARG OPENCV_DIR=/usr/local/src/opencv
ARG TF_MODELS_VERSION=1.11
ARG TF_MODELS_DIR=/usr/local/lib/python3.5/dist-packages/tensorflow/models

RUN apt-get update && apt-get install -y --no-install-recommends \
	ca-certificates \
	build-essential \
	python3 \
	python3-dev \
	libhdf5-dev \
	libzmq3-dev \
	python3-h5py \
	libatlas-base-dev \
	# git \
	wget \
	unzip \
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

# Install tensorflow wheel for rpi 3
ADD tensorflow-${TENSORFLOW_VERSION}-cp35-none-linux_armv7l.whl /tmp/
RUN pip3 install /tmp/tensorflow-${TENSORFLOW_VERSION}-cp35-none-linux_armv7l.whl \
	&& rm /tmp/tensorflow-${TENSORFLOW_VERSION}-cp35-none-linux_armv7l.whl

# Install tensorflow models object detection
# RUN GIT_SSL_NO_VERIFY=true git clone -q https://github.com/tensorflow/models /usr/local/lib/python3.5/dist-packages/tensorflow/models
RUN wget -q -P ${TF_MODELS_DIR}/ https://github.com/tensorflow/models/archive/v${TF_MODELS_VERSION}.zip
RUN cd ${TF_MODELS_DIR}/ \
	&& unzip models-${TF_MODELS_VERSION}.zip \
	&& rm models-${TF_MODELS_VERSION}.zip

# Download && build protobuf-python
RUN wget -q -P /usr/local/src/ https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOBUF_VERSION}/protobuf-python-${PROTOBUF_VERSION}.zip
RUN cd /usr/local/src/ \
	&& unzip protobuf-python-${PROTOBUF_VERSION}.zip \
	&& rm protobuf-python-${PROTOBUF_VERSION}.zip \
	&& cd ${PROTOBUF_DIR}-${PROTOBUF_VERSION}/ \
	&& ./configure \
	&& make \
	&& make install \
	&& ldconfig \
	&& rm -rf ${PROTOBUF_DIR}-${PROTOBUF_VERSION}/

# Download && build OpenCV
RUN wget -q -P /usr/local/src/ https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip
RUN cd /usr/local/src/ \
	&& unzip opencv-${OPENCV_VERSION}.zip \
	&& rm opencv-${OPENCV_VERSION}.zip \
	&& cd ${OPENCV_DIR}-${OPENCV_VERSION}/ \
	&& mkdir build \
	&& cd ${OPENCV_DIR}-${OPENCV_VERSION}/build \
	&& cmake -D CMAKE_INSTALL_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local/ .. \
	&& make -j4 \
	&& make install \
	&& rm -rf ${OPENCV_DIR}-${OPENCV_VERSION}

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

# Set TF object detection available
ENV PYTHONPATH "$PYTHONPATH:${TF_MODELS_DIR}/research:${TF_MODELS_DIR}/research/slim"
RUN cd ${TF_MODELS_DIR}/research \
	&& protoc object_detection/protos/*.proto --python_out=.

# Minimize image size
RUN apt-get autoremove -y \
	&& apt-get autoclean -y

CMD ["/run_jupyter.sh", "--allow-root"]
