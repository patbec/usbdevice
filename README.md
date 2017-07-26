# Schöner Booten mit GRUB4DOS

![Screenshot GRUB4DOS](https://raw.githubusercontent.com/patbec/usbdevice/master/screenshot-grub4dos.png)

## Getting Started

Ziel: Einen USB-Stick der ISO-Dateien booten kann, mit NTFS als Dateisystem und einer eleganten Oberfläche.

### Vorbereitung

Ein Speichermedium, am besten ein USB3.0 Stick mit Lese- und Schreibraten von ca. 120 MB/s.

Anschließend brauchen wir noch einen Bootloader in diesem Fall nehmen wir Grub4Dos. Das Projekt wird zurzeit von dem Benutzer [chenall](https://github.com/chenall/grub4dos) weitergeführt.

### Die Ausführung

Zuerst erstellen wir einen Ordner mit dem wir arbeiten werden

```
# Der Ordner sollte sich im Home-Verzeichnis befinden
mkdir usbfiles
```

Das Speichermedium löschen und ein NTFS Dateisystem erstellen, als Partitionstabelle nehmen wir **msdos**. Für diesen Schritt bietet sich das Programm [GParted](https://wiki.ubuntuusers.de/GParted/) an.

exFAT bietet die größtmögliche Kompatibilität zu anderen Betriebssystemen, es werden aber auch andere Dateisysteme wie **FAT12 \ FAT16 \ FAT32**, **NTFS** oder **EXT2 \ EXT3 \ EXT4** unterstützt.

Nun die Grub4Dos Dateien herunterladen und entpacken, alternativ können die benötigten Dateien auch selbst kompiliert werden.

```
# Arbeitsverzeichnis zu 'usbfiles' wechseln
cd usbfiles

# Grub4Dos herunterladen
wget http://dl.grub4dos.chenall.net/grub4dos-0.4.6a-2017-06-11.7z

# Archiv entpacken
7z x grub4dos-0.4.6a-2017-06-11.7z
```

Falls ein eigenens kompilieren gewünscht ist:
```
git clone https://github.com/chenall/grub4dos
cd grub4dos
make
```

Anschließend den Bootloader in das Stammverzeichnis des Speichermediums kopieren

```
sudo cp grub4dos-0.4.6a/grldr  /media/<Pfad zur Partition>
```

Es werden noch diese zusätzlichen Dateien benötigten

```
git clone https://github.com/patbec/usbdevice
sudo cp -r usbdevice/files/* /media/<Pfad zur Partition>

# Ordner für eigene Dokumente erstellen (optional)
sudo mkdir /media/<Pfad zur Partition>/Dokumente
```

Zum Schluss muss noch der MBR geschrieben werden

```
sudo grub4dos-0.4.6a/bootlace.com /dev/<Laufwerk>
```

ISO-Abbilder müssen in den Ordner **images** kopiert werden, die dazugehörigen Einträge werden in der **menu.lst** hinterlegt.

Der USB Stick sollte nun in etwa so aussehen:

```
- USB Root
	- images
		- mein-beispiel-abbild.iso
	- Dokumente
	- menu.lst
	- utilities.lst
	- message
	- grldr
```

Glückwunsch!


Zum testen kann das ganze mit einer VHD in einer VM durchgeführt werden. Es wird empfohlen eine Sicherung zu erstellen, um diese bei Bedarf auf einem anderen Gerät wiederherstellen zu können.

Diese Anleitung wurde mit Debain 8.0 und Ubuntu 16 LTS getestet.

## Beispiele

Hier eine Beispiel Auflistung von ISO-Abbildern mit den dazugehörigen **menu.lst** einträgen:


[Parted Magic](https://partedmagic.com)
```
title Parted Magic 2017 (64-Bit)
set iso=/images/pmagic_2017_06_12.iso
map %iso% (0xff)
map --hook
root (0xff)
kernel /pmagic/bzImage64 iso-scan/filename=%iso% quiet splash --
initrd /pmagic/initrd.img /pmagic/fu.img /pmagic/m64.img
```

[Ubuntu](https://www.ubuntu.com/download/desktop)
```
title Ubuntu 16.04.2 LTS
set iso=/images/ubuntu-16.04.2-desktop-amd64.iso
map %iso% (0xff)
map --hook
root (0xff)
kernel /casper/vmlinuz.efi file=/cdrom/preseed/ubuntu.seed boot=casper iso-scan/filename=%iso% quiet splash --
initrd /casper/initrd.lz
```

[BackBox](https://backbox.org/download)
```
title BackBox Linux 4.7
set iso=/images/backbox-4.7-amd64.iso
map %iso% (0xff)
map --hook
root (0xff)
kernel /casper/vmlinuz file=/cdrom/preseed/backbox.seed iso-scan/filename=%iso% quiet splash --
initrd /casper/initrd.gz
```

[Microsoft Windows PE 10.0](https://msdn.microsoft.com/de-de/library/windows/hardware/dn898560(v=vs.85).aspx)
```
title Microsoft Windows PE 10.0
map --mem /images/WinPE_v10.0_amd64.iso (0xff)
map --hook
chainloader (0xff)
rootnoverify (0xff)
```

[Memtest86+](http://www.memtest.org/#downiso)
```
title Memtest86+ v5.01
map --mem /images/memtest86+_v5.01.iso (0xff)
map --hook
chainloader (0xff)
rootnoverify (0xff)
```

## Fehlerbehandlung

```Missing Grub```

Grub wurde nicht oder unvollständig in den Speicher geladen.
Mit dem nächsten Schritt kann überprüft werden ob 'bootlace.com' den MBR vollständig geschrieben hat.

```
# MBR mit Partitionstabelle sichern (1MB Dump)
sudo dd if=/dev/<Laufwerk> of=vm_disk.mbr bs=512 count=2048
```
Die Datei 'vm_disk.mbr' kann mithilfe von 'cmp' oder einem Hex-Editor mit 'grldr.mbr' verglichen werden.

```
# MBR manuell auf das angegebene Laufwerk schreiben
sudo dd if=grub4dos-0.4.6a/grldr.mbr  of=/dev/<Laufwerk> bs=446 count=1
```

Bei der Benutzung von VirtualBox sollte zusätzlich die Erweiterung [VBox+DavidB's VMUB utility](http://www.rmprepusb.com/tutorials/how-to-boot-from-usb-under-vmware-server) verwendet werden, da es sonst zu Problemen beim schreiben auf USB-Geräten kommt.

## Autor

* **Patrick Becker** - [GitHub](https://github.com/patbec)

E-Mail: [github@bec-wolke.de](mailto:github@bec-wolke.de)

## Lizenz

Dieses Projekt ist unter der MIT-Lizenz lizenziert - Weitere Informationen finden Sie in der Datei [LICENSE](LICENSE)

## tl;dr
```
# Arbeitsverzeichnis erstellen
mkdir usbfiles

# Arbeitsverzeichnis zu 'usbfiles' wechseln
cd usbfiles

# Grub4Dos herunterladen
wget http://dl.grub4dos.chenall.net/grub4dos-0.4.6a-2017-06-11.7z

# Archiv entpacken
7z x grub4dos-0.4.6a-2017-06-11.7z

# Bootloader in das Stammverzeichnis kopieren
sudo cp grub4dos-0.4.6a/grldr  /media/<Pfad zur Partition>

# Zusätzlichen Dateien kopieren
git clone https://github.com/patbec/usbdevice
sudo cp -r usbdevice/files/* /media/<Pfad zur Partition>

# Ordner für Dokumente erstellen (optional)
sudo mkdir /media/<Pfad zur Partition>/Dokumente

# MBR schreiben
sudo grub4dos-0.4.6a/bootlace.com /dev/<Laufwerk>
```
