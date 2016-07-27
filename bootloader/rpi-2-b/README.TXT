=========================================================================
Instalacija toolchain-a, tftp-a, nfs servera i drugo na host racunar
=========================================================================
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install tftpd-hpa
sudo apt-get install nfs-kernel-server
(potrebno za svaki terminal pokrenuti ponovo komande koje slede...)
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
priprema usb2serial na host PC (u VM):
	podešavanje USB settings u VM tako da se usb2serial prijavi i na VM
	sudo apt-get install picocom
	picocom -b 115200 ttyUSB0

=========================================================================
Przenje image-a root fs-a i kernela na sd karticu
=========================================================================
sdcard.img - image sd kartice koji je dovoljno naprziti na karticu:
	1) (nije potrebno ako se przi image) formatiranje sd kartice:
		a. Windows
			i. Preuzeti alat ya formatiranje sa https://www.sdcard.org/downloads/formatter_4/eula_windows/
			ii. Instalirati i pokrenuti alat na racunaru
			iii. Postaviti "FORMAT SIZE ADJUSTMENT" opciju na "ON" u "Options" meniju
			iv. Proveriti da ubacena kartica odgovara onoj odabranoj u alatu
			v. Kliknite "Format" dugme
		b. Linux
			i. Predlazemo koriscenje alata gparted (ili command line verziju parted)
			ii. Formatirajte ceo disk kao FAT
	2) przenje sd kartice
		a. Windows
			i. Preuzeti Win32DiskImager sa http://downloads.sourceforge.net/project/win32diskimager/Archive/Win32DiskImager-0.9.5-install.exe?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fwin32diskimager%2F&ts=1469553342&use_mirror=netix
			ii. Instalirati i pokrenuti alat na racunaru
			iii. Odaberite image datoteku koju zelite da naprzite na karticu
			iv. Proveriti da ubacena kartica odgovara onoj odabranoj u alatu
			v. Kliknite na dugme "Write"
		b. Linux
			i. df -h
			ii. umount /dev/sdd1, ako je sdd1 prikljucena sd kartica
			iii. dd bs=4M status=progress if=sdcard.img of=/dev/sdd1, ako je sdd1 prikljucena sd kartica, a sdcard.img image koji treba naprziti
			iv. sudo sync

=========================================================================
Podesavanja za boot preko NFS
=========================================================================
u u-boot folderu se nalazi:
	u-boot.bin / image u-boot-a, da bi se pokrenuo treba ga kopirati na sd karticu rpi-a na boot particiju i u config.txt fajlu dodati liniju "kernel=u-boot.bin"
	Kada se upali rpi, treba prekinuti timer koji pocne da odbrojava pritiskom na enter i ulazi se u u-boot.

u kernelAndDtb folderu se nalaze:
	zImage - image kernela
	bcm2709-rpi-2-b.dtb - device tree blob koji se koristi uz kernel

	Ove fajlove je potrebno podeliti preko tftp servera kako bi mogli da se prebace na rpi. Koristimo tftpd-hpa server.
	Instalira se komandom
		"sudo apt-get install tftpd-hpa".
	Ukoliko se nista ne menja folder koji se deli na mrezi je
		/var/lib/tftpboot.
	U taj folder treba prebaciti fajlove (image i device tree blob)
	Restartovati server komandom
		"sudo service tftpd-hpa restart".
	Fajlovi se u pokrenuti u-boot prebaciju komandama:
		setenv ipaddr <ip adresa> - ip koji dodeljujemo ploci, bilo koji slobodan ip na mrezi
		setenv serverip <ip adresa> - ip adresa racunara na kom se nalazi tftp server
		setenv bootargs "root=/dev/nfs rw ip=<ip adresa rpi-a> console=ttyAMA0 nfsroot=<ip adresa racunara na kom je nfs server>:<putanja do rfs-a na racunaru>"
		saveenv
		tftp 0x1000000 zImage - download kernela
		tftp 0x2000000 bcm2709-rpi-2-b.dtb - download dtb-a
		bootz 0x1000000 - 0x2000000

	boot se kasnije moze automatizovati	
		setenv bootcmd "tftp 0x1000000 zImage; tftp 0x2000000 bcm2709-rpi-2-b.dtb; bootz 0x1000000 - 0x2000000"

u folderu rootfs se nalaze: rootfs.ext2, rootfs.tar i rootfs.tar.bz2
	opcija 1: rootfs.tar.bz2 - kompresovana verzija rfs-a, potrebno je raspakovati i dobijeni folder exportovati preko nfs servera pre pokretanja
		"sudo tar xvjf rootfs.tar.bz2 [-C <putanja do direktorijuma gde ce se raspakovati>]"
	opcija 2: rootfs.ext2 - image koji se može pržiti i na karticu, pre pokretanja ga treba mountovati na neki folder koji se exportuje nfs serverom
		"mount rootfs.ext2 <putanja do direktorijuma gde ce se mountovati>"

	Koristimo nfs-kernel-server. Instalira se komandom
		"sudo apt-get install nfs-kernel-server".
	Posle toga dodati lokacije koje se exportuju u fajl /etc/exports u formatu
		"<putanja koja se exportuje> <ip adresa rpi-a>(rw,no_root_squash,no_subtree_check)"
	Restartovati nfs server komandom
		"sudo /etc/init.d/nfs-kernel-server restart"

=========================================================================
Przenje na sd karticu
=========================================================================
<TBD>
1) u-boot
2) kernel i dtb
3) rootfs
4) podesavanja
	a.
	b.
	c.


=========================================================================
Detaljni opis pojedinacnih fajlova:
=========================================================================

sdcard.img.tar.bz2 - kompresovana verzija image-a sd kartice koji je dovoljno naprziti na karticu po opisanom postupku

rpi-firmware/ - direktorijum sa datotekama neophodnim za boot, treba da se nalaze na boot particiji SD kartice

u-boot.bin - image u-boot-a, da bi se pokrenuo treba ga kopirati na sd karticu rpi-a na boot particiju i u config.txt fajlu dodati liniju "kernel=u-boot.bin"
	Kada se upali rpi, treba prekinuti timer koji pocne da odbrojava pritiskom na enter i ulazi se u u-boot.
uboot.env - bin fajl sa opisom env varijabli u-boot-a, treba da se nalazi na sd kartici, pored u-boot.bin i u tom slucaju ce ga u-boot koristiti za pocetnu konfiguraciju, a ako ne postoji koristice default.

zImage - image kernela
bcm2709-rpi-2-b.dtb - device tree blob koji se koristi uz kernel
	Ove fajlove je potrebno podeliti preko tftp servera kako bi mogli da se prebace na rpi. Koristimo tftpd-hpa server. Instalira se sa "sudo apt-get install tftpd-hpa". Ukoliko se nista ne menja folder koji se deli na mrezi je /var/lib/tftpboot. U taj folder treba prebaciti fajlove i restartovati server sa komandom "sudo service tftpd-hpa restart".
	Fajlovi se u pokrenuti u-boot prebaciju komandama.
	setenv ipaddr <ip adresa> - ip koji dodeljujemo ploci, bilo koji slobodan ip na mrezi
	setenv serverip <ip adresa> - ip adresa racunara na kom se nalazi tftp server
	setenv bootargs "root=/dev/nfs rw ip=<ip adresa rpi-a> console=ttyAMA0 nfsroot=<ip adresa racunara na kom je nfs server>:<putanja do rfs-a na racunaru>"
	saveenv
	tftp 0x1000000 zImage - download kernela
	tftp 0x2000000 bcm2709-rpi-2-b.dtb - download dtb-a
	bootz 0x1000000 - 0x2000000

	boot se kasnije moze automatizovati	
	setenv bootcmd "tftp 0x1000000 zImage; tftp 0x2000000 bcm2709-rpi-2-b.dtb; bootz 0x1000000 - 0x2000000"

rootfs.ext2.tar.bz2 - kompresovana verzija image-a root fs, pre pokretanja ga treba mountovati na neki folder koji se exportuje sa nfs serverom
rootfs.tar.bz2 - kompresovana verzija rfs-a, potrebno je raspakovati i taj folder exportovati preko nfs servera pre pokretanja
	raspakovan root fs se vec nalazi na putanji ../../modules/nfsroot

	Koristimo nfs-kernel-server. Instalira se sa "sudo apt-get install nfs-kernel-server". Posle toga dodati lokacije koje se exportuju u fajl /etc/exports u formatu
"<putanja koja se exportuje> <ip adresa rpi-a>(rw,no_root_squash,no_subtree_check)"
	Restartovati nfs server sa komandom "sudo /etc/init.d/nfs-kernel-server restart"

src/rootfs/ - minimalni rootfs (busybox)

=========================================================================
Bildovanje kernela
=========================================================================
git clone --depth=1 https://github.com/raspberrypi/linux
cd linux
make bcm2709_defconfig
make

=========================================================================
Bildovanje minimalnog rootfs-a
=========================================================================
wget http://www.busybox.net/downloads/busybox-1.21.1.tar.bz2
tar xvjf busybox-1.21.1.tar.bz2 <direktorijum gde ce biti raspakovano>
make defconfig
make menuconfig (build with static libraries, install directory nfsroot)
make
make install

U nfsroot folderu napraviti foldere proc/ sys/ etc/init.d
U etc/ folderu napraviti fajl inittab i popuniti ga sa 
::sysinit:/etc/init.d/rcS
::askfirst:/bin/sh

U etc/init.d/ folderu napraviti fajl rcS i popuniti ga sa
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys

=========================================================================
Bildovanje u-boot-a
=========================================================================
git clone git://git.denx.de/u-boot.git
make rpi_2_defconfig
make all

=========================================================================
Bildovanje rootfs (buildroot)
=========================================================================
<TBD>

=========================================================================
Bildovanje toolchain-a (buildroot)
=========================================================================
<TBD>

=========================================================================
Bildovanje kernel-a, rootfs-a i toolchain-a (buildroot)
=========================================================================
preuzimanje najnovije verzije buildroot-a
	git clone git://git.buildroot.net/buildroot 
	cd buildroot/
<Preuzeti konkretnu verziju, najbolje rešenje ukoliko se u koraku konfiguracija koristi priložena .config datoteka>
	git checkout 2016.08
	
konfiguracija
	opcija 1)
		opcija a) ručno:
			make raspberrypi2_defconfig
			make xconfig
			...
		opcija b) kopirati priloženu datoteku kao .config u buildroot/
			modules/data/buildroot-2016.08-all.config

		<Uključivanje u-boot.bin u sdcard.img i dodatne datoteke>
			post-build.sh
			genimage-raspberrypi2.cfg
	opcija 2)
		koristiti patch bootloader/rpi-2-b/src/0001-RPI2-customization-for-RTRK-EMBEDDED-LINUX-PROGRAMMI.patch
		git apply <patch>

bildovanje
	make


