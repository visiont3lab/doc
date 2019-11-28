# Cross Compile Using Docker
In this tutorial we are going to use [dockcross](https://github.com/dockcross/dockcross), a set of prebuild docker images, to cross compile a project for rasperry pi 3. Our goals are:

* being able to latest opencv library on the rasberry
* being able to use also the Eigen library
* being able to run the program in c++
* beign able to runt the program using python3

To do this we need to divide our work into two steps:

1. Cross Compile Opencv using docker: We need to compile opencv for rasberry. To do this we are going to use the dockcross/linux-armv7 image.
2. Link the cross compiled Opencv library and run a sample project: We are going to install on the rasberry pi the eigen library, link the previous build opencv library for both python and c++ and run the project. In dealing with c++ we are going to use cmake.

## Step 1) Cross Compile Opencv using docker

Set your working directory. You can export it or add it to the $HOME/.bashrc
```
# Create working directory
cd $HOME && mkdir -p cross_compile_dir

# Export the variable, this will work only on this terminal
export CROSS_COMPILE_DIR=$HOME/cross_compile_dir

# Or Add the variable to the $HOME/.bashrc
echo "export CROSS_COMPILE_DIR=$HOME/cross_compile_dir" >> $HOME/.bashrc && source $HOME/.bashrc
```

Clone the dockcross repository
```
cd $CROSS_COMPILE_DIR && 
git https://github.com/dockcross/dockcross.git 
```

Pull the rasberrypi3 image (dockcross/linux-armv7) and create a docker container. We are going to pull this image from docker-hub for simplicity, but it is also possible to build it using the  docker build command. We are going to also add to our container a volume that we will use to share data between the created docker container and the host computer.

```
cd $CROSS_COMPILE_DIR && mkdir -p sharing_data &&
docker run --name rasberry-pi3 -it -v $CROSS_COMPILE_DIR/sharing_data:/work/sharing_data dockcross/linux-armv7:latest /bin/bash
```

The upper command will take a couple of minute but at the end it will open a terminal inside the created container. The path is going to be /work. We should see our sharing_data folder if we type the command "ls" .

Now we are ready to start building opencv. Inside the host machine sharing_data folder create a file "install_opencv.sh" and copy the following content inside.

```
# Create the file using vim
vim install_opencv.sh

# Create the file using gedit
gedit install_opencv.sh
```

Installation script opencv:
```
#!/usr/bin/env bash

set -e
apt-get update && \
    apt-get install -y \
        build-essential cmake git  wget \
        unzip \
        yasm \
        pkg-config \
        libswscale-dev \
        libtbb2 \
        libtbb-dev \
        libjpeg-dev \
        libpng-dev \
        libtiff-dev \
        libavformat-dev \
        libpq-dev \
        libsm6 \
        libxext6 \
        libxrender-dev \
        libgtk2.0-dev \
        qt5-default \
        libimage-exiftool-perl \
    && rm -rf /var/lib/apt/lists/*

mkdir -p /work/sharing_data/lib && cd /work/sharing_data/lib 
OPENCV_VERSION="4.1.1"
wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip && unzip ${OPENCV_VERSION}.zip && rm ${OPENCV_VERSION}.zip  
wget https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip  &&  unzip ${OPENCV_VERSION}.zip  && rm ${OPENCV_VERSION}.zip
mkdir $HOME/lib/opencv-${OPENCV_VERSION}/opencv_build  && cd $HOME/lib/opencv-${OPENCV_VERSION}/opencv_build
cmake  -D OPENCV_EXTRA_MODULES_PATH=$HOME/lib/opencv_contrib-${OPENCV_VERSION}./modules  \
    -DBUILD_TIFF=ON \
    -DBUILD_PNG=ON \
    -DBUILD_JPEG=ON \
    -DBUILD_JASPER=OFF \
    -DWITH_FFMPEG=ON \
    -DWITH_GSTREAMER=OFF \
    -DBUILD_ZLIB=OFF \
    -DOPENCV_GENERATE_PKGCONFIG=ON \
    -DBUILD_opencv_java=OFF \
    -DBUILD_opencv_python2=OFF \
    -DBUILD_opencv_python3=ON \
    -DWITH_CUDA=OFF \
    -DWITH_OPENGL=ON \
    -DWITH_OPENCL=ON \
    -DWITH_IPP=OFF \
    -DWITH_TBB=ON \
    -DWITH_EIGEN=ON \
    -DWITH_V4L=ON \
    -DWITH_GTK=ON  \
    -DWITH_VTK=OFF \
    -DBUILD_TESTS=OFF \
    -DINSTALL_PYTHON_EXAMPLES=OFF \
    -DBUILD_PERF_TESTS=OFF \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DCMAKE_SHARED_LIBS=OFF \
    -D CMAKE_CXX_FLAGS=-latomic \
    -D OPENCV_EXTRA_EXE_LINKER_FLAGS=-latomic \
    -DCMAKE_INSTALL_PREFIX=/usr/local   ..  && \
make -j4 
#make install  && ldconfig 
```

Add executable permission to the file
```
chmod a+x install_opencv.sh
```

Now we go back to our docker container and we can start building opencv. The compiler will be automatically set.
```
# Inside docker contaner
cd /work/sharing_data && ./install_opencv.sh
```
Feel Free to modify the installation script from either the host machine or the docker container as you like. We are using docker volume that are like shared folder and can be modified from both the machine in real time.

It will take around 15 minutes to build. Grub a cup of coffee meanwhile. 
After the building is finished we are going to avoid docker perssion problem by typing the following command from inside the container.
```
cd /work/sharing_data &&
chmod -R 777 lib/
```

## Step 2) Link the cross compile Opencv library and run a sample project
