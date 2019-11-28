# Cross Compile Using Docker
In this tutorial we are going to use [dockcross](https://github.com/dockcross/dockcross), a set of prebuilt docker images, to cross compile a project for raspberry  pi 3. Our goals are:

* being able to latest Opencv library on the raspberry  pi3
* being able to use also the Eigen library
* being able to run the program in c++
* being able to runt the program using python3

To do this we need to divide our work into two steps:

1. Cross Compile Opencv using docker: We need to compile opencv for raspberry . To do this we are going to use the dockcross/linux-armv7 image.
2. Link the cross compiled Opencv library and run a sample project: We are going to install on the raspberry  pi the eigen library, link the previous build Opencv library for both python and c++ and run the project. In dealing with c++ we are going to use cmake.

## Step 1) Cross Compile Opencv using docker

Set your working directory. You can export it or add it to the $HOME/.bashrc
```shell
# Create working directory
cd $HOME && mkdir -p cross_compile_dir

# Export the variable, this will work only on this terminal
export CROSS_COMPILE_DIR=$HOME/cross_compile_dir

# Or Add the variable to the $HOME/.bashrc
echo "export CROSS_COMPILE_DIR=$HOME/cross_compile_dir" >> $HOME/.bashrc && source $HOME/.bashrc
```

Clone the dockcross repository
```shell
cd $CROSS_COMPILE_DIR && \
git https://github.com/dockcross/dockcross.git 
```

Pull the raspberry pi3 image (dockcross/linux-armv7) and create a docker container. We are going to pull this image from docker-hub for simplicity, but it is also possible to build it using the  docker build command. We are going to also add to our container a volume that we will use to share data between the created docker container and the host computer.

```shell
cd $CROSS_COMPILE_DIR && mkdir -p sharing_data && \
docker run --name raspberry -pi3 -it -v $CROSS_COMPILE_DIR/sharing_data:/work/sharing_data dockcross/linux-armv7:latest /bin/bash
```

The upper command will take a couple of minute but at the end it will open a terminal inside the created container. The path is going to be /work. We should see our sharing_data folder if we type the command "ls" .

Now we are ready to start building Opencv. Inside the host machine sharing_data folder create a file "install_opencv.sh" and copy the following content inside.

```shell
# Create the file using vim
vim install_opencv.sh

# Create the file using gedit
gedit install_opencv.sh
```

[Installation script Opencv](https://www.learnopencv.com/install-opencv-4-on-raspberry-pi/):
```bash
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
        python3-dev \
        python3-pip  

mkdir -p /work/sharing_data/lib && cd /work/sharing_data/lib 
OPENCV_VERSION="4.1.1"
wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip && unzip ${OPENCV_VERSION}.zip && rm ${OPENCV_VERSION}.zip  
wget https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip  &&  unzip ${OPENCV_VERSION}.zip  && rm ${OPENCV_VERSION}.zip
mkdir /work/sharing_data/lib/opencv-${OPENCV_VERSION}/opencv_build  && cd /work/sharing_data/lib/opencv-${OPENCV_VERSION}/opencv_build

cmake  -D OPENCV_EXTRA_MODULES_PATH=/work/sharing_data/lib/opencv_contrib-${OPENCV_VERSION}/modules  \
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
    -DCMAKE_INSTALL_PREFIX=/usr/local  ..  && \
make -j4 && make install  && ldconfig 
```

Add executable permission to the file
```shell
chmod a+x install_opencv.sh
```

Now we go back to our docker container and we can start building opencv. The compiler will be automatically set.
```shell
# Inside docker contaner
cd /work/sharing_data && ./install_opencv.sh
```
Feel Free to modify the installation script from either the host machine or the docker container as you like. We are using docker volume that are like shared folder and can be modified from both the machine in real time.

It will take around 15 minutes to build. Grub a cup of coffee meanwhile. 
After the building is finished we are going to avoid docker permission problem by typing the following command from inside the container.
```shell
cd /work/sharing_data && \
chmod -R 777 /work/sharing_data/lib/
```

The compile Opencv library is available inside the folder 

```shell
$CROSS_COMPILE_DIR/sharing_data/lib/opencv-4.1.1/opencv_build
```

## Step 2) Link the cross compile Opencv library and run a sample project
We want to implement a simple Video capture based on this [example](https://www.learnopencv.com/read-write-and-display-a-video-using-opencv-cpp-python/) on the raspberry pi that use our previous build cross compiled libary. For this reason we have created two sample projects:
1. C++ Opencv Sample project using cmake
2. Python3 Opencv sample project
   
First of all inside the raspberry pi we are going to set up a working directory
```
# Inside the raspberry pi
cd $HOME &&
mkdir -p template_project/lib
```

Then, we need to copy the Opencv previously built library  inside the template project.
We are going to use scp to copy the file from the host machine to the raspberry but it is also possible to transfer the file using a simple USB pen.
```
# Inside the host machine
cd $CROSS_COMPILE_DIR/sharing_data/lib && 
scp -r opencv-4.1.1 pi@<raspberry_local_ip>:<rasberry_home_dir>/template_project/lib
```

To use ssh it is required that both host and raspberry are on the same local network ([see ssh guide](https://www.cyberciti.biz/faq/howto-restart-ssh/)). Furthermore it is required to have installed on both host and raspberry  two different packages

```
# Inside both host machine and raspberry pi3
sudo apt update && sudo apt install openssh-server openssh-client
sudo systemctl start sshd
```

### C++ Opencv Sample project using cmake
We are going to build from scratch a c++ small project using cmake.
```shell
cd $HOME/template_project && mkdir -p src/include && mkdir -p src/sources && mkdir -p src/test
# Create files
touch CMakeLists.txt && touch src/include/sample.h &&  touch src/sources/sample.cpp && touch src/test/main_opencv.cpp 
```

Edit the CMakeLists.txt with the following text
```
cmake_minimum_required(VERSION 2.8.3)
project(template_project)

### Use version 2011 of C++ (c++11). By default ROS uses c++98
#see: http://stackoverflow.com/questions/10851247/how-to-activate-c-11-in-cmake
#see: http://stackoverflow.com/questions/10984442/how-to-detect-c11-support-of-a-compiler-with-cmake
add_definitions(-std=c++11)
#add_definitions(-std=c++0x)
#add_definitions(-std=c++03)

set(PROJECT_SOURCE_DIR src/sources)
set(PROJECT_INCLUDE_DIR src/include)

set(PROJECT_HEADER_FILES
        src/include/sample.h
        )
set(PROJECT_SOURCE_FILES
        src/sources/sample.cpp
        )

find_package(OpenCV REQUIRED PATHS ENV{$HOME}/${PROJECT_NAME}/lib/opencv-4.1.1/cmake)

include_directories(${PROJECT_INCLUDE_DIR})
include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(/usr/include/eigen3)

add_library(${PROJECT_NAME} ${PROJECT_SOURCE_FILES} ${PROJECT_HEADER_FILES})
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})
target_link_libraries(${PROJECT_NAME} ${Eigen_LIBS})

## Declare a C++ executable
add_executable(main_opencv src/test/main_opencv.cpp)
target_link_libraries(main_opencv ${PROJECT_NAME})
```

We also add the [Eigen library](http://yang.amp.i.kyoto-u.ac.jp/~yyama/computer/FAQ/eigen/install.html) that is used a lot in working with C++ project. To install the library it is required to run to following command.
```shell
# Inside the raspberry 
sudo apt-get install libeigen3-dev
```

The library is installed below /usr/include/eigen3/. For this reason in the upper CMakeList we have included the path.

We can now edit our sample.h, sample.cpp and main_opencv.cpp files.
Edit "sample.h" file
```
#ifndef _CLASS_MODULE_H_
#define _CLASS_MODULE_H_

#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/core/core.hpp>
#include "opencv2/opencv.hpp"
#include <iostream>
#include <math.h>
#include <Eigen/Dense>

class ClassModule
{
  private:

  public:	
	void Run(cv::Mat frame, cv::Mat &res);

	// Constructor and Distructor
	ClassModule(void);
	~ClassModule(void);
};
#endif 
```

Edit "sample.cpp" file
```
#include "sample.h"

ClassModule::ClassModule(void)
{
}

ClassModule::~ClassModule(void)
{
}

void ClassModule::Run(cv::Mat frame, cv::Mat &res)
{
    if (frame.empty())
        std::cout << "Frame is empty" std::endl;
 
    cv::Mat gray, edge;
    cv::cvtColor(src1, gray, cv::CV_BGR2GRAY);
    cv::Canny( gray, edge, 50, 150, 3);
    edge.convertTo(res, cv::CV_8U);
}

```

Edit "main_opencv.cpp" file
```
#include "sample.h"

int main(int argc, char** argv)
{
    ClassModule MyClassModule;

    cv::VideoCapture cap(0); 

    if(!cap.isOpened()){ 
        str::cout << "Error opening video stream or file" << std::endl;
    }

    while(1){

        cv::Mat res;

        // Capture frame-by-frame 
        cap >> frame;

        // Run Canny Edge detector
        MyClassModule.Run(frame,res);
        
        // Display
        cv::imshow( "Frame", res );
        char c=(char)waitKey(25);
        if(c==27){break;} // Press Esc to stop
    }

    cap.release();
 
    return 0;
}
```

Let's build the project
```shell
$HOME/template_project  && \
mkdir -p build && cd build && cmake .. && make && \
./main_opencv
```

You should see as output an image showing the edges.


### Python3 Opencv sample project
In dealign with python we need firstly to be sure that there is python3 install on the raspberry.
```
sudo apt install python3-dev python3-pip
```
We are going to use pip as packet manager to install extra compile available libraries.

First of all let's add the previously compiled Opencv python library to our python3 default environment.
```
# Simbolic Link between pyhton path and installed opencv path
#ln -s /usr/local/python/cv2/python-3.6/cv2.cpython-37m-x86_64-linux-gnu.so /usr/local/lib/python3.6/site-packages/cv2.so
```