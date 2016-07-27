ifneq ($(KERNELRELEASE),)
obj-m := nunchuk.o
else
KDIR := ../../../../src/linux
all:
	$(MAKE) -C $(KDIR) M=$$PWD
endif
