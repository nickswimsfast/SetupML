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

# Install ML tools
```
$ pip install tensorflow==2.0.0-alpha0
```
