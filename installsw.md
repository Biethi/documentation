# Installation of Software

## libraries

For development with C++ we want to use some features out of the boost library, so we need to install at least the used parts. As we want to dive deeper into development with C++ and boost we will install all parts beforhand.

For testing our fusepp-lowlevel repository we need the development header from the fuse library so we will install these, too.

```sh
sudo apt install libboost-all-dev libfuse-dev
```

## CMake

As make tool I use cmake so we need an appropriate version to build ower software projects. From the official ubuntu repos we do not get the latest stable version so first we install the official kitkat ubuntu repo. A all steps are taken from https://apt.kitware.com/. To keep the repo up to date we additionaly install the `kitware-archive-keyring`. Afterwards we can install the latest stable cmake version.

```sh
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | sudo tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null
sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ focal main'
sudo apt-get update
sudo apt-get install kitware-archive-keyring
sudo rm /etc/apt/trusted.gpg.d/kitware.gpg
sudo apt-get install cmake
```

## SmartGit https://www.syntevo.com/smartgit/download/

For keeping the GitHub repos up to date an stay in sync with our locale branches we use the tool "SmartGit". For private use its free to use. As the official ubuntu repo does not provide this tool we will get it from the SmartGit homepage. For linux we can get a .deb package so the installation is quite easy. Just download the package and install it with package manager of your choise.

To allow *force push* and amending to commits we need to enable *Allow modifing push commits* in the preferences of *SmartGit*.

## VS Code

My personal editor of choise is VS Code from *Microsoft&trade;*. It can be installed through the **Ubuntu Software** app. After first starting the editor I install some useful plugins. As we use CMake to automate the make process I install the cmake plugin from *Microsoft&trade;*. Moreover I install these useful plugins: C/C++ and Python. The Python plugin in used only for code highlighting. For coding in python I use PyCharm as I experienced some performance issues with the plugin for VS Code in large projects.

## PyCharm

The PyCharm Community Edition is an open source project from *jetbrains*. It can be installed with the **Ubuntu Software** app. So installation should be easy. To use a virtual environment for python projects the package *python3.8-venv* is needed as the default preinstalled version on *Ubuntu 20.04* is *python3.8*. When everything is installed no additional steps are needed if you want to use the standard directories and standard python version.

## Doxygen

Installed from the ubuntu repo.

## Android Studio

For development of andoid apps I use *Andoid Studio* as IDE. So install the latest version with the **Ubuntu Software**. I installed the default modules and followed the steps provided to use kwm for virtualization. (see https://developer.android.com/studio/run/emulator-acceleration?utm_source=android-studio#vm-linux and https://help.ubuntu.com/community/KVM/Installation). In specific for Ubuntu 20.04 these commands need to be executed.

```sh
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
sudo adduser `id -un` kvm
```

Confiure project and virtual device need to be done.