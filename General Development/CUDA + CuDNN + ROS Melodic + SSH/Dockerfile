# This is an auto generated Dockerfile for ros:ros-core
# generated from docker_images/create_ros_core_image.Dockerfile.em
FROM nvidia/cuda:10.2-cudnn7-devel
#:10.2-cudnn7-devel
#:9.0-devel

RUN apt update
#lsb release not found...
RUN apt install lsb-release -y


# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && apt-get install -q -y tzdata && rm -rf /var/lib/apt/lists/*



# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8


RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654



RUN apt update

RUN apt install ros-melodic-desktop-full -y


ENV ROS_DISTRO melodic
# bootstrap rosdep
RUN rosdep init && \
  rosdep update && rosdep fix-permissions




############Creating Super Sam###################
RUN useradd -ms /bin/bash sam
RUN echo "sam:mm123456" | chpasswd 
RUN adduser sam sudo


####ADDED to execute in normal user mode
RUN useradd -ms /bin/bash normal
USER normal
WORKDIR /home/normal

RUN rosdep update
RUN echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc

USER sam
RUN echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc

############################SOURCE ROS base######################


USER normal
WORKDIR /home/normal
RUN /bin/bash -c "source /opt/ros/melodic/setup.bash"
####Now revert to ROOT
USER root

RUN apt install python-rosinstall python-rosinstall-generator python-wstool build-essential -y

RUN apt install python3-pip python-pip -y

RUN /bin/bash -c "pip3 install tensorflow-gpu==1.14 --default-timeout=100" 
RUN /bin/bash -c "pip3 install scipy==1.1"
RUN /bin/bash -c "pip3 install imutils scikit-image keras matplotlib"

#SSH
RUN apt-get install -y openssh-server

##################################Entrypoint Script####################
#ROS setup ros environment

RUN echo '#!/bin/bash' >> /ros_entrypoint.sh
RUN echo 'set -e'      >> /ros_entrypoint.sh
RUN echo 'source "/opt/ros/$ROS_DISTRO/setup.bash"' >> /ros_entrypoint.sh

#SSH
RUN echo 'service ssh restart'  >> /ros_entrypoint.sh

RUN echo 'exec "$@"'   >> /ros_entrypoint.sh

RUN chmod +x /ros_entrypoint.sh

ENTRYPOINT ["/ros_entrypoint.sh"]




############bashrc for remote access normal account##########################


#nvcc for remote ssh.... somehow local nvcc is ok but remote cannot find
RUN ln -sf /usr/local/cuda-10.2 /usr/local/cuda
RUN echo 'export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}' >> /home/normal/.bashrc

RUN echo 'export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}' >> /home/sam/.bashrc


#####################SSH Settings for remote access####################
RUN mkdir /var/run/sshd
RUN echo 'root:mm123456' | chpasswd
RUN echo 'normal:mm123456' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN sed -i 's/X11Forwarding no/X11Forwarding yes/g' /etc/ssh/sshd_config
RUN sed -i 's/#X11UseLocalhost yes/X11UseLocalhost no/g' /etc/ssh/sshd_config
RUN sed -i 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN echo "AllowUsers root sam" >> /etc/ssh/sshd_config


ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
#######################################################################


CMD ["bash"]


