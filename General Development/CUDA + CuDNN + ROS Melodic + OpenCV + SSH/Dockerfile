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


########################OPENCV################
#https://supercrazysam.github.io/2000/11/25/OpenCV/

RUN apt-get install -y software-properties-common
#to get add-apt-repository command
RUN add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
RUN apt update
RUN apt-get install libjpeg8-dev libjasper1 libjasper-dev libpng-dev -y  
#libjasper-dev not found in package so remove from install...  in 18.04 need to install libjasper1 then libjasper-dev
#need to do add-repository according to https://blog.csdn.net/weixin_41053564/article/details/81254410
#libpng12-dev become libpng-dev after 16.04

RUN apt-get install build-essential checkinstall cmake pkg-config yasm -y
RUN apt-get install git gfortran -y


RUN apt-get install libtiff5-dev -y
RUN apt-get install libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev -y
RUN apt-get install libxine2-dev libv4l-dev -y

#RUN apt-get install libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev -y   ##outdated
#https://askubuntu.com/questions/1121122/unable-to-locate-package-libgstreamer0-10-dev-on-ubuntu-18-04

RUN apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev -y
RUN apt-get install qt5-default libgtk2.0-dev libtbb-dev -y
RUN apt-get install libatlas-base-dev -y
RUN apt-get install libfaac-dev libmp3lame-dev libtheora-dev -y
RUN apt-get install libvorbis-dev libxvidcore-dev -y
RUN apt-get install libopencore-amrnb-dev libopencore-amrwb-dev -y
RUN apt-get install x264 v4l-utils -y

RUN apt-get install libprotobuf-dev protobuf-compiler -y
RUN apt-get install libgoogle-glog-dev libgflags-dev -y
RUN apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen -y


RUN mkdir /home/sam/OPENCV_INSTALL
WORKDIR /home/sam/OPENCV_INSTALL

RUN git clone https://github.com/opencv/opencv.git
RUN git clone https://github.com/opencv/opencv_contrib.git

RUN mkdir /home/sam/OPENCV_INSTALL/opencv/build
WORKDIR /home/sam/OPENCV_INSTALL/opencv/build

RUN cmake -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules/ -DBUILD_TIFF=ON -DBUILD_opencv_java=OFF -DWITH_CUDA=ON -DWITH_OPENGL=ON -DWITH_OPENCL=ON -DWITH_IPP=ON -DWITH_TBB=ON -DWITH_EIGEN=ON -DWITH_V4L=ON -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DENABLE_FAST_MATH=ON -DCUDA_FAST_MATH=ON -DWITH_CUBLAS=ON -DWITH_GSTREAMER=ON -DWITH_FFMPEG=ON -DCMAKE_BUILD_TYPE=RELEASE -DOPENCV_ENABLE_NONFREE:BOOL=ON -DCUDA_GENERATION=Auto ..

RUN make -j$(nproc)
RUN make install
RUN echo “/usr/local/lib” >> /etc/ld.so.conf.d/opencv.conf
RUN ldconfig

#####################Nautilus remote setting######################
RUN apt install libcanberra-gtk-module libcanberra-gtk3-module -y
RUN apt install nautilus -y
RUN mkdir /home/sam/.config
RUN mkdir /home/sam/.config/nautilus
RUN chmod a+rwx /home/sam/.config/nautilus

RUN apt-get update
RUN apt-get install eog gedit -y 
RUN apt install idle3 -y
#gedit and image_viewer

############################################

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
#######################################################################

CMD ["bash"]


