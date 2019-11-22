# Deep Learning Setup Guide

This is a small guide wrapping up all we have installed to set up our virtual machine.
To be able to write this notebook check the markdown instructions available [here](https://markdown-it.github.io/).

##  Step 1: Install operative system (release: Ubuntu 18.04.1 LTS)
We have installed the latest ubuntu desktop long term support (LTS)  release available.  In our case it was ubuntu 18.04.1 LTS .
To do download the ISO image on [this page](https://www.ubuntu.com/download/desktop).
Once the image is dowloaded you can pass the latter to the virtual machine or create a live usb to install ubuntu on a real pc.

##  Step 2: Install nvidia-drivers (release: 415)
The reference link that we have used for the installation is  [this one](http://www.linuxandubuntu.com/home/how-to-install-latest-nvidia-drivers-in-linux). But looking at [this website](https://www.nvidia.com/download/find.aspx) we have chosen to install the nvidia-drivers 415 because we are using a GTX NVIDIA 1080 Ti on a linux 64 bit operative system. The installation steps are summarized as following:

```
sudo apt-get purge nvidia*
sudo add-apt-repository ppa:graphics-drivers 
sudo apt-get update 
sudo apt install nvidia-driver-415
lsmod | grep nvidia  (check if the installation is fine)
sudo reboot
```

##  Step 3: Install cuda (release: 10)
We are going to install the cuda version associated to ubuntu 18.04.1 LTS. This is going to be cuda 10. To do this we have used this reference link.


1. Go to [this-website](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal) select Linux --> x86:64 --> Ubuntu --> 18.04 --> runfile (local) --> Download
2. Go to the Dowloads folder, open a terminal and type:  
sudo sh cuda_10.0.130_410.48_linux.run
3. While you are installing cuda it will ask you to isntall nvidia-drivers 410.28, type no becasue we have already install 415. For the rest you can accept default configuration.
4. open the bashrc (cd && vim .bashrc) and add the following lines at the end  
   
```
export PATH=/usr/local/cuda-10.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

To check if cuda is installed correctly and it is running do the following ([reference](https://medium.com/@cjanze/how-to-install-tensorflow-with-gpu-support-on-ubuntu-18-04-lts-with-cuda-10-nvidia-gpu-312a693744b5)):

```
cd ~/NVIDIA_CUDA-10.0_Samples
make
cd ~/NVIDIA_CUDA-10.0_Samples/bin/x86_64/linux/release
./deviceQuery (you should see Result = PASS)
```

##  Step 4: Install cuDNN (release: 7.4.2 )
To do this you have to:
1. Go to [this website](https://developer.nvidia.com/cudnn), then select Dowload cuDNN.
2. You need to have an accout to download the library. You can make one it's free.
3. Cross  I Agree To the Terms of the cuDNN Software License Agreement, then Download cuDNN v7.4.2 (Dec 14, 2018), for CUDA 10.0 and finally cuDNN Library for Linux . Your dowload will start.
4.  Follow the installation instructions related to Installing from a Tar File available [here](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html). For the sake of simplicty we wrap up them here.

```
sudo tar -xzvf cudnn-10.0-linux-x64-v7.4.2.24.tgz.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

##  Step 5: Set up python3 virtualenv
We are going to use virtual environment in dealing with python to better organize our pc.
We wan to be able to use our pc with both python 2 and 3 and with differnt deep learning framework.
Here are the instructions:

```
sudo apt install python3-pip (install pip3)
sudo -H pip3 install virtualenv (install virtualenv)
which python3 (location python3 installation)
sudo -H pip3 install virtualenvwrapper (install virtual environmnet wrapper)
cd && vim .bashrc --> open bashrc  

add the following lines at the end:
VIRTUALENVWRAPPER_PYTHON='/usr/bin/python3' # Note this has to be the output of which python3

source /usr/local/bin/virtualenvwrapper.sh
export WORKON_HOME=$HOME/.virtualenvs
cd && mkdir ~/.virtualenvs && source ~/.bashrc

```
Now let's create a python3 and python2 virtual environments:
The following command will create and automatically activate the environments.

```
cd && mkvirtualenv python3-virtualenv
sudo apt install python-minimal (will install python 2.7)
cd && mkvirtualenv --python=python2 python2-virtualenv
lsvirtualenv or $ ls $WORKON_HOME(list all the virtual environments) 
workon python3-env (will activate the environment)
deactivate (deactivate the running environment)
rmvirtualenv python3-virtualenv (remove virtual environment)
cpvirtualenv ENVNAME [TARGETENVNAME] (copy the environment)

```

More information are available [here](https://gist.github.com/IamAdiSri/a379c36b70044725a85a1216e7ee9a46).
All the virtual environments are located inside  ~/.virtualenvs more information on the commands line related to virtualenv and virtualwrapper can be found respectively [here-virtualenv](https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html) and [here-virtualwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/).

Let's focus on python3 virtual environment previously created and let's install all the required library to start dealign with computer vision and deep learning.
```
lsvirtualenv (list all created environments)
workon python3-virtualenv
pip3 install numpy scipy pandas matplotlib scikit-learn  opencv-python tensorflow-gpu==1.13.0-rc1
pip3 list (list all installed library)
python -c "import keras; import tensorflow as tf; print(tf.GIT_VERSION, tf.VERSION)"  # check if tensorflow is working
```

Note that you will have installed opencv-python cpu version. If you need the gpu you need to compile it

##  Extra 1:  Install  OpenCV GPU from source 
Useful reference [link](https://www.learnopencv.com/install-opencv-4-on-ubuntu-18-04/).

```
sudo apt -y remove x264 libx264-dev
sudo apt -y install build-essential checkinstall cmake pkg-config yasm
sudo apt -y install git gfortran
sudo apt -y install libjpeg8-dev libpng-dev
sudo apt -y install software-properties-common
sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
sudo apt -y update
sudo apt -y install libjasper1
sudo apt -y install libtiff-dev
sudo apt -y install libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev
sudo apt -y install libxine2-dev libv4l-dev
cd /usr/include/linux
sudo ln -s -f ../libv4l1-videodev.h videodev.h
sudo apt -y install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt -y install libgtk2.0-dev libtbb-dev qt5-default
sudo apt -y install libatlas-base-dev
sudo apt -y install libfaac-dev libmp3lame-dev libtheora-dev
sudo apt -y install libvorbis-dev libxvidcore-dev
sudo apt -y install libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt -y install libavresample-dev
sudo apt -y install x264 v4l-utils
sudo apt -y install libprotobuf-dev protobuf-compiler
sudo apt -y install libgoogle-glog-dev libgflags-dev
sudo apt -y install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen
sudo apt install libcanberra-gtk-module libcanberra-gtk3-module
$ git clone https://github.com/opencv/opencv.git
$ mkdir build && cd build
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_PNG=OFF -DBUILD_TIFF=OFF -DBUILD_TBB=OFF -DBUILD_JPEG=OFF -DBUILD_JASPER=OFF -DBUILD_ZLIB=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_java=OFF -DBUILD_opencv_python2=ON -DBUILD_opencv_python3=ON -DWITH_V4L=ON -DWITH_OPENGL=ON -DWITH_OPENCL=OFF -DWITH_OPENMP=OFF -DWITH_FFMPEG=ON -DWITH_GSTREAMER=OFF -DWITH_GSTREAMER_0_10=OFF -DWITH_CUDA=ON -DWITH_NVCUVID=OFF -DWITH_GTK=ON -DWITH_VTK=OFF -DWITH_TBB=ON -DWITH_1394=OFF -DWITH_OPENEXR=OFF -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-10.0 -DCUDA_ARCH_BIN='3.0 3.5 5.0 6.0 6.2' -DCUDA_ARCH_PTX="" -DINSTALL_C_EXAMPLES=OFF -DINSTALL_TESTS=OFF -DINSTALL_PYTHON_EXAMPLES=OFF -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules ..sudo 
$ make -j4
$ sudo make install
```

Install  OpenCV-GPU (version: 4.0.1-dev)

```
cp $HOME/lib/opencv/build/lib/python3/cv2.cpython-36m-x86_64-linux-gnu.so $HOME/.virtualenvs/my-python3-env/lib/python3.6/site-packages/cv2.so
python -c "import cv2; print(cv2.__version__); print(cv2.getBuildInformation())"
```

##  Extra 2: Install Dlib GPU from source 
The reference link is [this](https://stackoverflow.com/questions/51697468/how-to-check-if-dlib-is-using-gpu-or-not). The official dlib repository is [this one](https://github.com/davisking/dlib)

```
We are running gcc (Ubuntu 7.3.0-27ubuntu1~18.04) 7.3.0 (gcc --version)
sudo apt-get install libx11-dev libblas-dev
git clone https://github.com/davisking/dlib.git
cd dlib
mkdir build
cd build
cmake ..
cmake --build . --config Release

To be able to use dlib gpu inside a virtual environment you have to do this:
cd ~/lib/dlib
workon python3-virtualenv 
python3 setup.py install --yes USE_AVX_INSTRUCTIONS --yes DLIB_USE_CUDA
```

