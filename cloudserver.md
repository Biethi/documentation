# Install instructions for cloud server

## Install the OS

I choose to use Ubuntu 22.04 LTS Server version to run on my server PC. Get an installation image from the official ubuntu homepage and follow there instructions to create a bootable installation medium.

I used the default server configuration for install, so most of the tools I need are already included in the OS. While installing you can add some more server applications to the installation process. Check **nextcloud** for the data file server daemon.

Follow the instructions on the screen to complete the server installation.

### Automated updates

For this purpose the apt feature unattended updates is used. This should be preinstalled with the common server installation. We can just do some additional configurations. Therefore open the `/etc/apt/apt.conf.d/50unattended-upgrades` file. In here you can add a e-mail address to send reports for updates or problems. Afterwards we need to configure the time period to do the unattended updates. Open `/etc/apt/apt.conf.d/20auto-upgrades` and add the following line.

  APT::Periodic::Unattended-Upgrade "1";
  
This will trigger the updates every day at a random time slot.

## Nextcloud

### Remounting the RAID data storage

For the data storage I currently use three 2 TB harddrives which are used to create a RAID5 block device. This should be recognized automaticly when the system is started. A `/dev/md0` device should therefore be present.

As the device is also LUKS encrypted we first need to modify the `/etc/crypttab` file to encrypt the RAID device on system start. We will encrypt the device with a key file stored on the OS disk (`/root/disk_secret_key`). So first place the key file in the right location. Afterwards edit the `/etc/crypttab` as seen below. (Make sure the UUID is the one from your md0 device.)

  md0-crypt UUID=a4cf23f6-ed79-4be1-bf55-61296a3b8d1b /root/disk_secret_key luks

To mount the data storage to the correct location for use as nextcloud storage add the following line to `/etc/fstab`.

  /dev/mapper/md0-crypt /var/snap/nextcloud/common/nextcloud ext3 defaults 0 0

Restart the system and the storage should be mounted. You can check with `mount | grep md0`.

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

### Enable SSL

https://askubuntu.com/questions/1223656/nextcloud-with-ssl

For a Strato domain you get a SSL certifacte to connect it to your domain. But this included SSL certificate cannot be exported to another server, so we need to use some different solution. We will use the LetsEncrypt service to secure the connection to our server. All that is needed is a valid e-mail address and the domain(s) to be reached. Important is that port forwarding for both, the port 80 and 443, are configured in the home router, as these port will be used to check the connection and receive the certificate.

  sudo nextcloud.enable-https lets-encrypt

Just follow the on screen instructions and enter all required information. After all is set up the Apache server will be restarted. If this worked without errors a secure connection to Nextcloud should be possible.

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
