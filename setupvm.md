# Setup VM

As the development pc I use VirtualBox with an Ubuntu 20.04 LTS. In this document you will find all specified parameters for my virtual machine.

First of all get the latest stable build from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads). If you are using VirtualBox for your private use only you should download the extention pack so you can use some additional features like USB 2.0 and remote desktop.

After everything is installed create a new virtual machine. Follow the steps of the creation wizzard. Especially choose the used OS and create a sufficient large virtual drive. Afterward check all the settings to fit your needs and your host pc.

You can modify the VM to provide some usefull features in the settings too.

1. General
    - Advanced
        - Shared Clipboard: Bedirectional
        - Drag'n'Drop: Bedirectional
2. Network
    - Adapter 1
        - Attached to: NAT
3. USB
    - USB 2.0
4. Shared Folder
    - choose a folder as shared data folder between host and guest

When you are done with the configuration of the VM. Download the ISO image of a OS of your choise and insert it to the optical drive. Now you can start the VM the first time and install the OS. After the installation the VM is ready to go.