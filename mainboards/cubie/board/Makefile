VER=5.16.0
default: build

build: uCore uImage 

# this target builds an initramfs with only one program, the cpu server.
# It depends on the kernel setting the IP address.
# You need ip=dhcp OR ip=fixed.ip.address.here in the command line
cpu.cpio:
	GOARCH=arm u-root -o cpu.cpio -build=bb -initcmd=cpud -files ~/.ssh/cpu_rsa.pub:key.pub \
		-defaultsh="" \
		github.com/u-root/cpu/cmds/cpud

uCore: netbooturoot
	mkimage -A arm -n 'u-root for Allwinner A10' -d uroot.cpio -T ramdisk uCore

qemu: uCore flashkernel
	qemu-system-arm -kernel flashkernel -machine cubieboard -serial  stdio -initrd uCore -append  'console=ttyAM0 earlyprintk=ttyAM0,115200,keep' -display none
	echo NO
	exit 0

netbooturoot: Makefile
	GOARCH=arm \
	       GO111MODULE=off \
	       u-root \
	       -o uroot.cpio \
		-files ~/.ssh/cpu_rsa.pub:key.pub \
		-files ~/.ssh/cpu_rsa:id_rsa \
		-files ~/.ssh/authorized_keys:authorized_keys \
		github.com/u-root/cpu/cmds/cpud \
		all

uImage: flashkernel
	mkimage -C none -A arm -n 'kernel for Allwinner A10' -d linux/arch/arm/boot/zImage -T kernel -a 0x40008000 -e 0x40008000 uImage

flashkernel: config-$(VER)
	cp config-$(VER) linux/.config
	(cd linux && ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- make oldconfig && ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- make -j32)
	cp linux/arch/arm/boot/zImage flashkernel

fetch: getkernel geturoot

getkernel:
	rm -rf linux
	git clone git://github.com/linux-sunxi/linux-sunxi.git -b sunxi-next --depth=1 linux
	(cd linux && make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- sunxi_defconfig)

geturoot:
	go get -u github.com/u-root/u-root

packages:
	sudo apt install gcc-arm-linux-gnueabi
	sudo apt install u-boot-tools


#notes
#
# 1995  cd linuxboot/mainboards/cubie/board/linux/
# 1996  make mrproper
# 1997  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sunxi_defconfig
# 1998  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j32
# 1999  ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j4 dtbs
# 2000  history > ../DIT
#fatload mmc 0 0x46000000 zImage
#fatload mmc 0 0x49000000 <board>.dtb
#
#setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p<partition> rootwait panic=10 ${extra}
#
#bootz 0x46000000 - 0x49000000
