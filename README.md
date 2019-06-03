# SetupML
Tipsheet with commands to setup ubuntu 16.04 with Tensorflow for ML. May add pytorch later. Specifically;
* Tensorflow GPU with Jupyter notebook compatibility (with docker image)

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

# Install Docker
[Reference](https://docs.docker.com/install/linux/docker-ce/ubuntu/#prerequisites)
```
sudo apt-get remove docker docker-engine docker.io containerd runc #uninstalls any residual docker installs
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88 # last 8 characters should have 0EBF CD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
Then I had the following error:<i>
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
Reading package lists... Done
Building dependency tree       
Reading state information... Done
You might want to run 'apt-get -f install' to correct these:
The following packages have unmet dependencies:
 nvidia-cuda-toolkit : Depends: nvidia-cuda-dev (= 7.5.18-0ubuntu1) but it is not going to be installed
E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a solution).
elusin@VRBox:~/Projects/cudasamples/NVIDIA_
```

Resuming to attempt fix
```
sudo apt-get -f install
```


```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following packages were automatically installed and are no longer required:
  libxfont2 linux-headers-4.15.0-29 linux-headers-4.15.0-29-generic
  linux-headers-4.15.0-36 linux-headers-4.15.0-36-generic
  linux-headers-4.15.0-38 linux-headers-4.15.0-38-generic
  linux-headers-4.15.0-39 linux-headers-4.15.0-39-generic
  linux-headers-4.15.0-42 linux-headers-4.15.0-42-generic
  linux-headers-4.15.0-43 linux-headers-4.15.0-43-generic
  linux-image-4.15.0-29-generic linux-image-4.15.0-36-generic
  linux-image-4.15.0-38-generic linux-image-4.15.0-39-generic
  linux-image-4.15.0-42-generic linux-image-4.15.0-43-generic
  linux-modules-4.15.0-29-generic linux-modules-4.15.0-36-generic
  linux-modules-4.15.0-38-generic linux-modules-4.15.0-39-generic
  linux-modules-4.15.0-42-generic linux-modules-4.15.0-43-generic
  linux-modules-extra-4.15.0-29-generic linux-modules-extra-4.15.0-36-generic
  linux-modules-extra-4.15.0-38-generic linux-modules-extra-4.15.0-39-generic
  linux-modules-extra-4.15.0-42-generic linux-modules-extra-4.15.0-43-generic
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  nvidia-cuda-dev
Recommended packages:
  libnvcuvid1
The following NEW packages will be installed:
  nvidia-cuda-dev
0 upgraded, 1 newly installed, 0 to remove and 88 not upgraded.
24 not fully installed or removed.
Need to get 0 B/201 MB of archives.
After this operation, 467 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 420371 files and directories currently installed.)
Preparing to unpack .../nvidia-cuda-dev_7.5.18-0ubuntu1_amd64.deb ...
Unpacking nvidia-cuda-dev (7.5.18-0ubuntu1) ...
dpkg: error processing archive /var/cache/apt/archives/nvidia-cuda-dev_7.5.18-0ubuntu1_amd64.deb (--unpack):
 trying to overwrite '/usr/lib/x86_64-linux-gnu/stubs/libcublas.so', which is also in package libcublas-dev 10.2.0.168-1
dpkg-deb: error: subprocess paste was killed by signal (Broken pipe)
Errors were encountered while processing:
 /var/cache/apt/archives/nvidia-cuda-dev_7.5.18-0ubuntu1_amd64.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
```


Reference bug fix:
https://devtalk.nvidia.com/default/topic/1048225/linux/issues-after-installing-cuda-10-/
```
sudo dpkg -i --force-overwrite /var/cache/apt/archives/nvidia-cuda-dev_7.
5.18-0ubuntu1_amd64.deb
sudo apt-get -f install
sudo apt autoremove
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Test that docker works!
```
sudo docker run hello-world
```

Fix docker permissions to permanently add user to docker
```
sudo usermod -a -G docker $USER
sudo docker pull tensorflow/tensorflow:latest-gpu-jupyter
sudo docker run -it --rm tensorflow/tensorflow:latest-gpu-jupyter
```
(REBOOT)

# To run tensorflow docker image
Having an error when i try to import and can't do nvidia-smi...
```
sudo docker run -it -p 8888:8888 --rm tensorflow/tensorflow:latest-gpu-jupyter
or
docker run -it --rm -v $(realpath ~/notebooks):/tf/notebooks -p 8888:8888 tensorflow/tensorflow:latest-gpu-jupyter

```
This didn't work i couldn't get cuda working in the docker image or tensorflow to import, but jupyter notebook did work.

# Trying to use nvidia docker
Reference: https://github.com/NVIDIA/nvidia-docker
```
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```
This worked, it got cuda working in the docker containers. Then I had to figure out the right command in order to run it with the gpu image.

# Code to run gpu docker image 
It required opening the ports to access the jupyter notebook
```
# docker run --runtime=nvidia -it --rm tensorflow/tensorflow:latest-gpu-jupyter
docker run --runtime=nvidia -it -p 8888:8888 --rm tensorflow/tensorflow:latest-gpu-jupyter
```
This did it!

# Code to save files on the host
This is required to save your work! The Docker image is ephemeral meaning anything you add to it will be lost, unless you save it to the host drive. Here's how:
```
$ docker run --runtime=nvidia -it -p 8888:8888 --volume $PWD:/tf/Projects --rm tensorflow/tensorflow:latest-gpu-jupyter
```


# Install ML tools
Reference: https://www.tensorflow.org/install/gpu




