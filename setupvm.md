# Setup VM

## VirtualBox config

As the development pc I use VirtualBox with an Ubuntu 20.04 LTS. In this document you will find all specified parameters for my virtual machine.

First of all get the latest stable build from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads). If you are using VirtualBox for your private use only you should download the extention pack so you can use some additional features like USB 2.0 and remote desktop.

After everything is installed create a new virtual machine. Follow the steps of the creation wizzard. Especially choose the used OS and create a sufficient large virtual drive. Afterward check all the settings to fit your needs and your host pc.

You can modify the VM to provide some useful features in the settings too.

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

To use KVM in the guest OS we need to enable the virtualization features from the cpu to be passed through to the VM. For intel host this option is greyed out in the UI so we need to execute the following command in a command promd.

```
VBoxManage modifyvm yourvmname --nested-hw-virt on
```

This will enable and check the coresponding option in the UI.

When you are done with the configuration of the VM. Download the ISO image of a OS of your choise and insert it to the optical drive. Now you can start the VM the first time and install the OS. After the installation the VM is ready to go.

## Ubuntu 20.04

As development OS I use Ubuntu Desktop 20.04 LTS. After installing the OS we set up some fundamental things to clone the sources from GitHub.

### SSH Key

First of all we will generate a ssh key pair and configure GitHub to accept the key as authorization. To generate the key execute the following command. Afterwards copy the public key to GitHub. Now we can connect to GitHub with this key. To add the ssh figerprint to your keyring execute the second command and confirm the import with 'yes'. (see https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)

```sh
ssh-keygen
ssh -T git@github.com
```