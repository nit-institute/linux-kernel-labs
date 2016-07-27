ifneq ($(KERNELRELEASE),)
obj-m := hello_version.o
else
KDIR := ../../../../src/linux
all:
	$(MAKE) -C $(KDIR) M=$$PWD
endif
