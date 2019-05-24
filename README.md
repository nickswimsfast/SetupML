# SetupML
Tipsheet with commands to setup ubuntu 16.04 with tools and commands for ML.

# Computer
Graphics = GTX 1070
OS = Ubuntu 16.04

# Setup VNC
Both these solutions use the [Real VNC client](https://www.realvnc.com/en/connect/download/viewer/)
* For jetson nano, you'll have to use vino. 
* For x86-64bit computers, the built in remote desktop sharing can work, it just has to be activated and unencrypted:
http://ubuntuhandbook.org/index.php/2016/07/remote-access-ubuntu-16-04/

# Install Python3 and start venv
```
$ sudo apt-get install python3-venv
$ python3 -m venv <DIR>
$ source <DIR>/bin/activate
```
# Install GPU Support
[https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation](CUDA GPU Driver install)
## Pre-install verifications
```
lspci | grep -i nvidia #To verify you have an nvidia compatible graphics (kinda pointless cause i know 1070 is supported)
uname -m && cat /etc/*release # To verify linux version
gcc --version # verify you have GCC compiler
uname -r # determine the kernel version
sudo apt-get install linux-headers-$(uname -r) #update kernel headers (didn't do anything)
```
## Install CUDA Toolkit
Find latest [CUDA toolkit installer](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=deblocal)
```
#REFERENCE:cuda-repo-<distro>_<version>_<architecture>.deb
sudo dpkg -i cuda-repo-ubuntu1604-10-1-local-10.1.168-418.67_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-1-local-10.1.168-418.67/7fa2af80.pub #this command derived by the installer
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub #had to deduce this url from file name, that said it didn't change any keys
sudo apt-get update
sudo apt-get install cuda
```

Perform [post-installation setup](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions)
```
By looking in /usr/local/cuda10.1/NsightCompute-.... I found the version number of Cuda Toolkit (2019.3):
export PATH=/usr/local/cuda-10.1/bin:/usr/local/cuda-10.1/NsightCompute-2019.3${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64\${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Check that persistance daemon is working:
```
systemctl status nvidia-persistenced #
```
If inactive, do the following:
```
sudo systemctl enable nvidia-persistenced
```
I decided to skip the udev rule stuff.

## Install optional stuff
```
/usr/bin/nvidia-persistenced --verbose #this one didn't work
```
***(COMPUTER RESTART)***
to verify driver installed:
```
cat /proc/driver/nvidia/version
```
<b>Output is:</b>
NVRM version: NVIDIA UNIX x86_64 Kernel Module  418.67  Sat Apr  6 03:07:24 CDT 2019
GCC version:  gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) 

## Compiling CUDA examples
```
sudo apt install nvidia-cuda-toolkit
nvcc -V #Tells you cuda version
cd ~/Projects/cudasamples/NVIDIA_CUDA-10.1_Samples
make

```

## Verify CUDA compatibility
```
cd ~/Projects/cudasamples/NVIDIA_CUDA-10.1_Samples/bin/x86_64/linux/release
./deviceQuery
```
Output was:
```
 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 1070"
  CUDA Driver Version / Runtime Version          10.1 / 10.1
  CUDA Capability Major/Minor version number:    6.1
  Total amount of global memory:                 8116 MBytes (8510701568 bytes)
  (15) Multiprocessors, (128) CUDA Cores/MP:     1920 CUDA Cores
  GPU Max Clock rate:                            1797 MHz (1.80 GHz)
  Memory Clock rate:                             4004 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.1, CUDA Runtime Version = 10.1, NumDevs = 1
Result = PASS
```


# Install ML tools
Reference: https://www.tensorflow.org/install/gpu


```
$ pip install tensorflow==2.0.0-alpha0
```


