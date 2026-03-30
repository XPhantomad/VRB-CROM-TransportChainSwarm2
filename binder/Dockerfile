# Dockerfile of the base image: https://github.com/IntEL4CoRo/jupyter-ros2/blob/main/Dockerfile
FROM intel4coro/jupyter-ros2:jazzy-py3.12

# Install VNC server and XFCE desktop environment
USER root
RUN apt-get -y -qq update \
 && apt-get -y -qq install \
        dbus-x11 \
        xfce4 \
        xfce4-panel \
        xfce4-session \
        xfce4-settings \
        xorg \
        xubuntu-icon-theme \
        fonts-dejavu \
        software-properties-common \
    # Disable the automatic screenlock since the account password is unknown
 && apt-get -y -qq remove xfce4-screensaver \
    # chown $HOME to workaround that the xorg installation creates a
    # /home/jovyan/.cache directory owned by root
    # Create /opt/install to ensure it's writable by pip
 && mkdir -p /opt/install \
 && chown -R $NB_UID:$NB_GID $HOME /opt/install \
 && rm -rf /var/lib/apt/lists/*


# Install a VNC server, either TigerVNC (default) or TurboVNC
ENV PATH=/opt/TurboVNC/bin:$PATH
RUN echo "Installing TurboVNC"; \
    # Install instructions from https://turbovnc.org/Downloads/YUM
    wget -q -O- https://packagecloud.io/dcommander/turbovnc/gpgkey | \
    gpg --dearmor >/etc/apt/trusted.gpg.d/TurboVNC.gpg; \
    wget -O /etc/apt/sources.list.d/TurboVNC.list https://raw.githubusercontent.com/TurboVNC/repo/main/TurboVNC.list; \
    apt-get -y -qq update; \
    apt-get -y -qq install \
        turbovnc \
    ; \
    rm -rf /var/lib/apt/lists/*;
USER $NB_USER
RUN conda install -y websockify
RUN pip install jupyter-remote-desktop-proxy
ENV DISPLAY=:1



# Install python packages
USER root
RUN pip install --upgrade
COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt


# Copy content to the image
ENV REPO_DIR=/home/repo
RUN mkdir -p ${REPO_DIR}
COPY . ${REPO_DIR}/
RUN git config --global --add safe.directory ${REPO_DIR}

# Set JupyterLab default directory
WORKDIR ${REPO_DIR}

# Set VSCODE default directory
ENV CODE_WORKING_DIRECTORY=${REPO_DIR}

# The entrypoint of the docker image
COPY binder/entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]


RUN wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && \
    apt-get install -y ./google-chrome-stable_current_amd64.deb && \
    rm google-chrome-stable_current_amd64.deb 
#===========================================================#

EXPOSE 3004

# Change the default shell to Bash
SHELL [ "/bin/bash" , "-c" ]

EXPOSE 3004

USER root
# Install Requirements
RUN apt-get update && apt-get install -y git colcon cmake libfreeimage-dev libfreeimageplus-dev freeglut3-dev libxi-dev libxmu-dev liblua5.3-dev lua5.3 doxygen graphviz libgraphviz-dev asciidoc

USER root
RUN cd /usr/lib/x86_64-linux-gnu \
    && ln -s libglut.so.3.12.0 libglut.so.3

RUN git clone -b master https://github.com/XPhantomad/Context-Role-Oriented-Transport-Chain-Swarm.git

USER root
# Install ARGoS3
RUN cd Context-Role-Oriented-Transport-Chain-Swarm \
    # && git pull
    && cd Dependencies \
    && apt-get install -y ./argos3_simulator-3.0.0-x86_64-beta59.deb

# Install GSL for ARGoS3-ROS2-Bridge
RUN apt-get install -y libgsl-dev

RUN cd Context-Role-Oriented-Transport-Chain-Swarm \
    && cd Dependencies \
    && tar -zxf gsl-latest.tar.gz \
    && cd gsl-2.8 \
    && mkdir /root/gsl \
    && ./configure --prefix=/root/gsl \
    && make \
    && make check \
    && make install 

ENV LD_LIBRARY_PATH="/root/gsl/lib:$LD_LIBRARY_PATH"

# Build ARGoS3-ROS2-Bridge
RUN ln -s Context-Role-Oriented-Transport-Chain-Swarm/rosWorkspace ros_ws 

RUN cd ros_ws \
    && chmod +x build.sh \
    && ./build.sh 

RUN sudo apt install -y python3.12 python3.12-venv ros-dev-tools 

RUN wget https://julialang-s3.julialang.org/bin/linux/x64/1.10/julia-1.10.10-linux-x86_64.tar.gz \
    && tar xzf julia-1.10.10-linux-x86_64.tar.gz

ENV PATH="julia-1.10.10/bin:$PATH"

RUN julia -e 'using Pkg; Pkg.add.(["Sockets", "JSON", "Parameters"])'

RUN cd Context-Role-Oriented-Transport-Chain-Swarm/runtimemodel \
    && python3 -m venv . \
    && source bin/activate \
    && source /opt/ros/jazzy/setup.bash \
    && source ../rosWorkspace/install/setup.bash \
    && pip install pyyaml pyecore numpy 

RUN cd Context-Role-Oriented-Transport-Chain-Swarm/webapp \
    && python3 -m venv . \
    && source bin/activate \
    && source /opt/ros/jazzy/setup.bash \
    && source ../rosWorkspace/install/setup.bash \
    && pip install -r requirements.txt 
    
RUN cd Context-Role-Oriented-Transport-Chain-Swarm/startup \
    && python3 -m venv . \
    && source bin/activate \
    && source /opt/ros/jazzy/setup.bash \
    && source ../rosWorkspace/install/setup.bash \
    && pip install -r requirements.txt 

#============= Install extra software packages =============#
# RUN apt update && apt install -y curl





