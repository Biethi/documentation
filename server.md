# Einrichten des Host-Server

## Einen Admin auf dem System einrichten

http://danilodellaquila.com/en/blog/create-a-system-administrator-user-on-centos-server

Erst einen neuen Nutzer mit `useradd` anlegen. Wenn der Nutzer kein eigenes `/home`-Verzeichnis braucht, dann die option `-M` anhängen. Danach ein Passwort mit `passwd` vergeben.
Jetzt bekommt der Nutzer sudo-Rechte in dem er zur Gruppe `wheel` hinzugefügt wird. Dazu wird der Befehl `usermod -G wheel <username>` genutzt.

Als nächstes entzieht man allen anderen Nutzern das Recht sich als root anzumelden. Dazu ruft man `vi /etc/pam.d/su` auf und kommentiert die Zeile
`auth	required	pam_wheel.so	use_id` ein.

Zum Schluss werden noch alle Benachrichtigungen für root an den neuen Nutzer weitergeleitet. Dazu wird in `vi /etc/aliases` die Zeile
`root:		username`
eingetragen. Danach noch mit `newaliases` die Änderungen im System übernehmen.
http://haraldwingerter.de/?p=213

Man kann den root user auch ganz deaktivieren, damit sich niemand als root im System anmelden kann. (Das Wechseln von admin in root ist weiter möglich)

```sh
sudo passwd -d root
```

Wenn man dann root doch wieder braucht, dann kann man einfach ein neues root-Passwort vergeben.

```sh
sudo passwd
```

## Firewall einrichten (Firewalld)

So lässt sich die Zone für ein Interface ändern: (Mit der Zone home kann man per ssh auf die VM zugreifen)

```sh
firewall-cmd --permanent --zone=home --change-interface=enp4s0
```

Bis zum reboot deaktivieren mit (Wenn man mit Remote auf eine VM zugreifen will ist das eine schnelle einfache, aber unsichere Lösung)

```sh
service firewalld stop
```

## CronJob für Updates einrichten

https://www.rosehosting.com/blog/automate-system-tasks-using-cron-on-centos-7/

Das Paket `cronie` sollte bei der Minimalinstallation bereits dabei sein. Zum prüfen kann folgender Befehl ausgeführt werden.

```sh
rpm -q cronie
```

Um Systemweite „Jobs“ zu erstellen muss die `/etc/crontab` geändert werden. Für einzelne Nutzer werden die entsprechenden Dateien in `/var/spool/cron/username` angelegt.
Für Beispieleinträge einfach in die Datei crontab reinschauen, oder mit `man cron` und `man crontab` weitere Informationen aufrufen.

Um jeden Sonntag um 21 Uhr ein Update durchzuführen wird folgende Zeile in die conetab hinzugefügt.

```sh
0 21 * * sun root yum -y update > /var/log/update_$(date +%Y%m%d).log 2>&1
```

## Installieren von VirtualBox in CentOS

https://wiki.centos.org/HowTos/Virtualization/VirtualBox

Um die Extra Packages for Enterprise Linux (EPEL) nutzen zu können muss man erst `yum install epel-release` ausführen. Danach kann man einfach mit `yum install dkms` das kernel-modul-tool installieren.

Den gcc installiert man mit dem Befehl `yum install gcc`. Das Tool `make` sollte schon in der Minimalinstallation dabei sein.
Es müssen auch noch die header kernel installiert werden, dazu muss `yum install kernel-devel` ausgeführt werden.
Danach kann man mit `yum install VirtualBox-5.2.x86_64` VirtualBox installieren.
Danach noch alle Nutzer die eine VM starten können sollen in die Gruppe `vboxusers` aufnehmen.

```sh
Usermod -a -G vboxusers username
```

Jetzt muss das extension-pack für VirtualBox installiert werden. Dazu wird die passende Datei heruntergeladen. (Versionsnummern müssen angepasst werden) Die Virtualbox Version lässt sich mit `VBoxManager --version` erfragen.

```sh
wget http://download.virtualbox.org/virtualbox/5.2.6/Oracle_VM_VirtualBox_Extension_Pack-5.2.6-120293.vbox-extpack
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.2.6-120293.vbox-extpack
```

Überprüfen, dass alles geklappt hat kann man das dann mit

```sh
VBoxManage list extpacks
```


### Eine virtuelle Maschine (VM) erstellen

http://xmodulo.com/how-to-create-and-start-virtualbox-vm-without-gui.html

Eine virtuelle Maschine erstellen und gleichzeitig registrieren. (das ist wichtig sonst muss man vor den weiteren Schritten erst VBoxManage registervm für die virtuelle Maschine aufgerufen werden)

```sh
VBoxManage createvm --name <vmname> --register
```

Hier wird die VM dann so modifiziert wie man es für den Anwendungsfall benötigt. Für alle möglichen Einstellungen die Doku zu VirtualBox lesen.

```sh
VBoxManage modifyvm <uuid|vmname> --memory <size> --vram <size> --boot1 dvd --nic1 bridged --bridgeadapter1 enp4s0 --ostype Ubuntu
```

Weil das nicht standardmäßig aktiviert ist und man ohne die Option nicht auf die VM remote zugreifen kann ist es wichtig den Remote Deskop Service zu aktivieren. Die Portnummer sollte gleich mit geändert werden, weil auf einem Windows der Standartport (3389) oft schon durch den eigenen Remote Desktop Service belegt ist. (Ports zwischen 5000 und 5050 oft nicht genutzt)

```sh
VBoxManage modifyvm <uuid|vmname> --vrde <on|off> --vrdeport <port>
```

Hier legt man die virtuelle Festplatte an.

```sh
VBoxManage createvdi --filename ~/VirtualBox\ VMs/Ubuntu_16.04_LTS/Ubuntu_16.04_LTS-disk01.vdi --size 100000
```

Hier wird ein SATA-Controler hinzugefügt.

```sh
VBoxManage storagectl <uuid/vmname> --name <name> --add sata
```

### Erstelle Festplatte als IDE-Gerät hinzufügen.

```sh
VBoxManage storageattach "Ubuntu_16.04_LTS" --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium ~/VirtualBox\ VMs/Ubuntu_16.04_LTS/Ubuntu_16.04_LTS-disk01.vdi
```

Und danach noch für die Installation ein passendes iso herunter laden und mit dem virtuellen DVD-Laufwerk der VM hinzufügen.

```sh
wget http://releases.ubuntu.com/16.04.3/ubuntu-16.04.3-desktop-amd64.iso
wget http://mirror.netcologne.de/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso
VBoxManage storageattach "Ubuntu_16.04_LTS" --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium ubuntu-16.04.3-desktop-amd64.iso
```

Jetzt kann man die virtuelle Maschine starten, entweder mit diesem Befehl,

```sh
VBoxManage startvm ubuntu-server --type headless &
```

oder besser noch direkt mit

```sh
VBoxHeadless --startvm VM-Name &
```

Mit der Option `-e "TCP/Ports=4444"` lässt sich ein anderer Port für den Remote Desktop festlegen. Danach kann sich per Remote an der Maschine angemeldet werden. Wichtig ist dabei die IP-Adresse des Host muss verwendet werden und nicht die der VM.
Mit der option `--medium emptydrive` kann das Image wieder ausgeworfen werden.

### Die VM beim starten des Host automatisch starten lassen

Als erstes muss die Datei `/etc/default/virtualbox` angelegt werden. Darin setzt man dann die folgenden Variablen.

```sh
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/autostart.cfg
```

Die Datei `autostart.cfg` muss dann auch noch angelegt werden und um allen Nutzern das automatische Starten von VMs zu erlauben mit folgendem Inhalt gefüllt werden.

```sh
# Default policy is to deny starting a VM, the other option is "allow".
default_policy = allow
```

Danach schreiben Zugriff für alle Nutzer der Gruppe vboxusers einrichten und das sticky bit setzen.

```sh
chgrp vboxusers /etc/vbox
chmod 1775 /etc/vbox
```

Für den Nutzer der die VM automatisch starten lassen will führt man jetzt noch folgendes Kommando aus.

```sh
vboxmanage setproperty autostartdbpath /etc/vbox
```

und danach der entsprechenden VM sagen, dass Sie automatisch gestartet werden soll.

```sh
vboxmanage modifyvm fileServer --autostart-enabled on
```

Damit der Autostart auch klappt muss der Service aktiviert sein.

```sh
systemctl enable vboxautostart-service 
sudo systemctl start vboxautostart-service
```

Jetzt wird die VM automatisch beim hochfahren des Host mit gestartet. Für weitere Einstellungen muss man die VirtualBox Docu lesen.

http://nathangiesbrecht.com/centos-7-virtualbox-vboxautostart-service-setup

### VMs mit dem Host herunterfahren lassen

### VirtualBox Guest Addition in CentOS installieren

Vorher müssen alle Pakete wie bei CentOS als Host installiert werden (dkms, etc.)
Die passende Guest Addition herunterladen. Danach mounten und dann das Skript starten.

```sh
wget https://www.virtualbox.org/download/testcase/VBoxGuestAdditions_5.2.7-120528.iso
mount VBoxGuestAdditions_5.2.7-120528.iso /mnt/ 
sh ./VBoxLinuxAdditions.run
```

## Wieder einbinden der verschlüsselten RAID5 Festplatten nach einer Neuinstallation

Das Paket `mdadm` muss installiert werden. Dann wird das Software RAID von alleine erkannt und eingehängt. Das `md` Device sollte dann in der `cat /proc/mdstat` eingetragen sein. Falls nicht, können die Festplatten mit dem Befehl `mdadm --assemble /dev/md0 /dev/sdXX /dev/sdYY` zu dem Device `md0` hinzugefügt werden. (siehe auch https://askubuntu.com/questions/11293/rebuild-mdadm-raid5-after-os-hard-drive-died)
Um die das automatisch benannte md-device umzubenennen können die nächsten beiden Kommandos benutzt werden. (https://serverfault.com/questions/267480/how-do-i-rename-an-mdadm-raid-array)

```sh
Mdadm --stop /dev/md127
mdadm --assemble --update=name /dev/md0 /dev/sdb1 /dev/sdc1 /dev/sdd1
```

Für die Verschlüsselung und das richtige wiedereinhängen einfach die entsprechenden Zeilen aus der `/etc/fstab` und `/etc/crypttab` kopieren.
In die `/etc/crypttab` folgende Zeile einfügen. (`/dev/md0` möglicherweise noch durch UUID ersetzen. Hierbei unter `/dev/disk/by-uuid` nachschauen wie die uuid ist)

```sh
md0-crypt /dev/md0 none luks
```

In der `/etc/fstab` das entschlüsselte System an die entsprechende Stelle hängen.

```sh
/dev/mapper/md0-crypt /home ext3 defaults 0 0
```

Für das mounten von verschlüsselten Partitionen ist das Paket `cryptsetup` notwendig. Für Details siehe die manpage. (https://askubuntu.com/questions/63594/mount-encrypted-volumes-from-command-line)

## Entschlüsseln der Festplatten beim boot mit einem USB-Stick

### Die 3 Festplatten an die VM fileServer hängen
https://www.virtualbox.org/manual/ch09.html#rawdisk
Um die Festplatten einzuhängen muss eine `.vmdk` Datei erstellt werden. Dazu ist folgender Befehl notwendig. Danach wird die Datei einfach als Festplatte an die VM gehängt, wie eine virtuelle Festplatte.

```sh
VBoxManage internalcommands createrawvmdk -filename /path/to/file.vmdk -rawdisk /dev/sda
```

Wichtig ist hierbei, dass der user der die VMs startet auch lese und schreibberechtigung auf die `/dev/sdx` Festplatten hat. Das kann man mit `usermod -aG disk <username>` erreichen.

### Entschlüsselung einrichten

#### USB-Stick automatisch an die VM hängen

Um den USB-Kontroller einzurichten muss die VM mit `vboxmanage modifyvm fileServer --usbxhci on` eingestellt werden.
Den Filter für das usb-device legt man wie unten gezeigt an. Alle benötigten Informationen kann man mit `VBoxManage list usbhost` herausfinden.

```sh
vboxmanage usbfilter add 0 --target fileServer --name dongle --action hold --active yes --vendorid 0204 --productid 6025 --revision 0100 --serialnumber 11171500B8063A00 --remote no
```

#### Automatisches entschlüsseln mit USB-Stick

## CronJob für Updates einrichten

