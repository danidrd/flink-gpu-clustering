FROM nvidia/cuda:12.2.0-devel-ubuntu22.04 AS cuda-stage

# Installare le dipendenze necessarie
RUN apt-get update && apt-get install -y \
    git \
    && rm -rf /var/lib/apt/lists/*




# Definire la versione di JCuda da utilizzare
ENV RELEASE="version-10.0.0-RC01"

# Creare la directory per JCuda e spostarsi al suo interno
RUN mkdir -p /opt/jcuda
WORKDIR /opt/jcuda

# Clonare i repository JCuda e checkout della release
RUN git clone https://github.com/jcuda/jcuda-main.git && \
    git clone https://github.com/jcuda/jcuda-parent.git && \
    git clone https://github.com/jcuda/jcuda-common.git && \
    git clone https://github.com/jcuda/jcuda.git && \
    git clone https://github.com/jcuda/jcublas.git && \
    git clone https://github.com/jcuda/jcufft.git && \
    git clone https://github.com/jcuda/jcusparse.git && \
    git clone https://github.com/jcuda/jcurand.git && \
    git clone https://github.com/jcuda/jcusolver.git && \
    git clone https://github.com/jcuda/jcudnn.git 

FROM apache/flink:2.0-java17

RUN apt-get update && apt-get install -y \
    git \
    maven \
    gcc \
    g++ \
    cmake \
    mesa-common-dev \
    libglu1-mesa-dev \
    wget \
    unzip \
    && rm -rf /var/lib/apt/lists/*


COPY --from=cuda-stage /usr/local/cuda /usr/local/cuda
COPY --from=cuda-stage /opt/jcuda /opt/jcuda

ENV JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda
ENV PATH="usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    java -version && \
    readlink -f $(which java) && \
    ls $JAVA_HOME/ 
    # ls ${JAVA_HOME}/lib 
    # ls $JAVA_HOME/include/linux

RUN wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb  &&\
    dpkg -i cuda-keyring_1.1-1_all.deb  && \
    apt-get update  && \
    apt-get -y install cudnn-cuda-12

WORKDIR /opt/jcuda/jcuda-main
# Compilare JCuda con CMake
# Compilare JCuda con CMake
RUN cmake -DJAVA_AWT_LIBRARY=$JAVA_HOME/lib/libjawt.so \
          -DCMAKE_LIBRARY_PATH=$CUDA_TOOLKIT_ROOT_DIR/lib64/stubs \
          -DCUDA_nvgraph_LIBRARY=IGNORE \
          -DCUDA_cuDNN_LIBRARY=/usr/lib/x86_64-linux-gnu/libcudnn.so \
          . && \
    make all
# Spostarsi nella directory principale di JCuda
WORKDIR /opt/jcuda/jcuda-parent

RUN mvn clean install -DskipTests

WORKDIR /opt/jcuda/jcuda-main

RUN mvn clean install -DskipTests


# Pulire i file temporanei per ridurre la dimensione dell'immagine
RUN rm -rf /root/.m2 /opt/jcuda/jcuda-main/target



# Configurare le variabili d'ambiente per JCuda
ENV JCuda_HOME=/opt/jcuda/jcuda-main
ENV PATH="${JCuda_HOME}/bin:${PATH}"
ENV LD_LIBRARY_PATH="${JCuda_HOME}/lib:${LD_LIBRARY_PATH}"

# Configurare la directory di lavoro finale
WORKDIR /opt/flink

# Impostare il comando di default
CMD ["/bin/bash"]
