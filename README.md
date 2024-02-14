# Required Hardware:
- Drone with Ardupilot Firmware
- Raspberry Pi4 Model B 8GB RAM with debian bullseye 32 bit OS
- Intel Realsense T265 

# Note: 
This procedure is given for Realsense camera interface with Raspbian 32 bit OS.

## Pre-install Requirements

- Start with updating, upgrading, and installing dependencies and tools:

```
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get install automake libtool vim cmake libusb-1.0-0-dev libx11-dev xorg-dev libglu1-mesa-dev

```

- Expand the filesystem by selecting the `Advanced Options` menu entry, and select yes to rebooting:

```
sudo raspi-config

```

- Increase swap to 2GB by changing the file below to `CONF_SWAPSIZE=2048`:

```
sudo vi /etc/dphys-swapfile

```

- Apply the change:

`sudo /etc/init.d/dphys-swapfile restart swapon -s`

- Create a new `udev` rule:

`cd ~
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense`

 `git checkout v2.48.0  (note: for a specific version)
sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/`

- Apply the change (needs to be run by root):

```
sudo su
udevadm control --reload-rules && udevadm trigger
exit

```

- Modify the path by adding the following line to the `.bashrc` file:

```
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

```

- Apply the change:

`source ~/.bashrc`

## Installation

- Install `protobuf` — Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data:

```
cd ~
git clone --depth=1 -b v3.10.0 https://github.com/google/protobuf.git
cd protobuf
./autogen.sh
./configure
make -j1
sudo make install
cd python
export LD_LIBRARY_PATH=../src/.libs
python3 setup.py build --cpp_implementation
python3 setup.py test --cpp_implementation
sudo python3 setup.py install --cpp_implementation
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION_VERSION=3
sudo ldconfig
protoc --version

```

- Install `libtbb-dev` parallelism library for C++:

`cd ~
wget https://github.com/PINTO0309/TBBonARMv7/raw/master/libtbb-dev_2018U2_armhf.deb
sudo dpkg -i ~/libtbb-dev_2018U2_armhf.deb
sudo ldconfig
rm libtbb-dev_2018U2_armhf.deb`

- Install RealSense SDK `librealsense`:

`cd ~/librealsense
mkdir  build  && cd build` 

`sudo apt install libssl-dev
cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true`  

**before this step ensure that you link these libraries below “include the following lines” in CMakeList.txt file in librealsense:** 

>> target_link_libraries(${LRS_TARGET} PRIVATE GLU)
>> 
>>target_compile_options(${LRS_TARGET} PRIVATE -std=c++11)
>> 
>>target_link_libraries(${LRS_TARGET} PRIVATE -latomic)
>> 

`make -j1
sudo make install`

- Install RealSense SDK `pyrealsense2` Python bindings for `librealsense`:

```
cd ~/librealsense/build
cmake .. -DBUILD_PYTHON_BINDINGS=bool:true -DPYTHON_EXECUTABLE=$(which python3)
make -j1
sudo make install

```

- Modify the path by adding the following line to the `.bashrc` file:

`export PYTHONPATH=$PYTHONPATH:/usr/local/lib`  

• Apply the change:

```
source ~/.bashrc

```

- Install `OpenGL`:

```
sudo apt-get install python-opengl
sudo -H pip3 install pyopengl
sudo -H pip3 install pyopengl_accelerate==3.1.3rc1

```

- Change pi settings (enable `OpenGL`):

`sudo raspi-config
"7. Advanced Options" – "A8 GL Driver" – "G2 GL (Fake KMS)"`

The location of pyrealsense2 library is:

**/home/pi/librealsense/build/wrappers/python**

hence all programs should be in this folder

**Paste the main file named t265_to_mavlink.py in this folder and run.**
