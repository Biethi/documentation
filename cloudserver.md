# Install instructions for cloud server

## SMART

It is important to check the health of the disks used for data inside the server. To do so we can monitor the SMART data from the disks. To do so we will use the `smartctl` command.

    sudo smartctl -a /dev/sda

## Install the OS

I choose to use Ubuntu 22.04 LTS Server version to run on my server PC. Get an installation image from the official ubuntu homepage and follow there instructions to create a bootable installation medium.

I used the default server configuration for install, so most of the tools I need are already included in the OS. While installing you can add some more server applications to the installation process. Check **nextcloud** for the data file server daemon.

Follow the instructions on the screen to complete the server installation.

### Automated updates

For this purpose the apt feature unattended updates is used. This should be preinstalled with the common server installation. We can just do some additional configurations. Therefore open the `/etc/apt/apt.conf.d/50unattended-upgrades` file. In here you can add a e-mail address to send reports for updates or problems. Afterwards we need to configure the time period to do the unattended updates. Open `/etc/apt/apt.conf.d/20auto-upgrades` and add the following line.

  APT::Periodic::Unattended-Upgrade "1";

This will trigger the updates every day at a random time slot.

### Disk monitoring

https://help.ubuntu.com/community/Smartmontools

Modern disks come with internal self monitoring tools (SMART). To use these information to monitor your storage disks and inform you about any error you can use the `smartmontools` from the ubuntu repository. Just install the tool and the `smartd` daemon will be started with a default configuration.

  sudo apt install smartmontools

To monitor only your storage devices and adjust the actions taken when encounter an error you need to change some of the default configurations. Therefore open the file `/etc/smartd.conf` and add a line with all needed checks for every device you want to monitor. Below is an example file with some useful options. Consider reading the `smartd.conf` manual page for more detailed information.

  sudo vi /etc/smartd.conf

---
**Hint:**

Sometimes it is needed to read the SMART data from the disks to detect a broken one.

  sudo smartctl --all /dev/sda

---

### Mail server

https://askubuntu.com/questions/1112772/send-system-mail-ubuntu-18-04

https://serverspace.io/support/help/postfix-as-a-send-only-smtp-on-ubuntu/
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-22-04

We will use postfix as mailserver to send error notifications from our services.

## Nextcloud

### Remounting the RAID data storage

For the data storage I currently use three 2 TB harddrives which are used to create a RAID5 block device. This should be recognized automaticly when the system is started. A `/dev/md0` device should therefore be present.

As the device is also LUKS encrypted we first need to modify the `/etc/crypttab` file to encrypt the RAID device on system start. We will encrypt the device with a key file stored on the OS disk (`/root/disk_secret_key`). So first place the key file in the right location. Afterwards edit the `/etc/crypttab` as seen below. (Make sure the UUID is the one from your md0 device.)

    md0-crypt UUID=a4cf23f6-ed79-4be1-bf55-61296a3b8d1b /root/disk_secret_key luks

To mount the data storage to the correct location for use as nextcloud storage add the following line to `/etc/fstab`.

    /dev/mapper/md0-crypt /var/snap/nextcloud/common/nextcloud ext3 defaults 0 0

Restart the system and the storage should be mounted. You can check with `mount | grep md0`.

### Replace RAID5 disk

If a disk is about to fail it needs to be replaced in the RAID. If you can add a new disk without removing any of the existing once this is a fairly easy task. Just insert the new disk to your system and format it with a partition similar to the once already part of the RAID. The partition type should propably be `fd` which is `linux RAID auto`. Then detect the disk to be replaced and run the following commands.

    sudo mdadm /dev/md0 --add /dev/<new disk>
    sudo mdadm /dev/md0 --replace /dev/<failed disk>

The new disk will first be added to the RAID device and then with the replace command the new disk will be build up to work as a replacement. This will take some time as all the data from the old disk needs to be copied or newly calculated.

---
**Hint:**

To list all your RAID devices or list the current disks part of the RAID just use one of the below commands.

    sudo mdadm -D --all
    sudo mdadm -D /dev/md0

---

### Install Nextcloud snap

Install the Nextcloud snap package and you should be good to go.

    sudo snap install nextcloud

### Initialize the nextcloud users

To initialize the Nextcloud users open the server IP in a web browser. The Nextcloud interface should be displayed and prompt you to add an administrator user. Just enter the initial user name and a password. This will create the user and all needed files and folders in the system.

All additional users now can be added using the just created admin account.

---
**Hint:**

If an error is displayed saying the domain is untrusted you need to add the domain to the Nextcloud trusted domain list.

    sudo nextcloud.occ config:system:set trusted_domains 2 --value=192.168.1.27

---

### Add trusted domains

All domains which should be used to access the Nextclound server need to be added to the trusted domains property. To do so use the `nextcould.occ` command.

    sudo nextcloud.occ config:system:set trusted_domains 3 --value=biethmann.de

### Port forwarding

To access your Nextcloud server through the internet you need to tell your router to send all requests for the corresponding port to the local server IP. To do so activate the port forwarding feature of your router and add a rule for your Nextcloud port(s). By default these port are 80 (HTTP) and 443 (HTTPS).

Currently we have no global IPv4 address provided by O2. So we need to use the IPv6 address to access the server. Sadly the FritzBox is not capable of port forwarding the ports 80 and 443. Therefore we need to change the ports used by nextcloud (see also https://github.com/nextcloud-snap/nextcloud-snap/wiki/Port-configuration).

    sudo snap set nextcloud ports.http=2780 ports.https=27443

Now we can do the port forwarding for these ports and the server should be reachable.

### Enable SSL

https://askubuntu.com/questions/1223656/nextcloud-with-ssl

For a Strato domain you get a SSL certifacte to connect it to your domain. But this included SSL certificate cannot be exported to another server, so we need to use some different solution. We will use the LetsEncrypt service to secure the connection to our server. All that is needed is a valid e-mail address and the domain(s) to be reached. Important is that port forwarding for both, the port 80 and 443, are configured in the home router, as these port will be used to check the connection and receive the certificate.

    sudo nextcloud.enable-https lets-encrypt

Just follow the on screen instructions and enter all required information. After all is set up the Apache server will be restarted. If this worked without errors a secure connection to Nextcloud should be possible.
  
Just in case you get a email for renewing the certificate, this will be done automaticly by nextcloud. There should be a service `snap.nextcloud.renew-certs.service` which will take care of renewing your certificates automaticaly.

As we moved the ports to some others than the standard 80 and 443, we also need to use self-signed SSL certificates as the lets-encrypt solution will not work without the standard ports. Solution is inspired from https://help.nextcloud.com/t/self-signed-cert-for-snap/47260

    sudo nextcloud.enable-https self-signed

## The dynDNS daemon

See the link for detailed information (https://help.ubuntu.com/community/DynamicDNS). Also check the Strato faq for details on the needed information to configure the dynDNS client (https://www.strato.de/faq/hosting/so-einfach-richten-sie-dyndns-fuer-ihre-domains-ein/).

In general just install the `ddclient` and follow the instructions while installing. The needed information are a valid e-mail address, the server to send the update to (dyndns.strato.com/nic/update), the login (for stato this is the domain) and the password. After installation check if the configuration is working. This will also trigger IP address to be updated. The command should result with *SUCCESS*. Check also your dynDNS provider site if the IP address was updated properly.

    sudo apt install ddclient
    sudo ddclient -daemon=0 -debug -verbose -noquiet

If the IP was updated you can now access your home router through the internet if this is allowed. As this could be a potential thread to your home network you should deactivate this feature if this is possible and is not done by default.

## Firewall

## Port config and forwarding

## DNS service

## NTP service

Route the domain inside the local network?

## Onlyoffice document server

Install the snap package with the next command. Documentation can be found on github (https://github.com/ONLYOFFICE/snap-documentserver).
