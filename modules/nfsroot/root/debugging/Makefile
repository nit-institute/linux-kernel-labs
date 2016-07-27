ifneq ($(KERNELRELEASE),)
obj-m := drvbroken.o
else
KDIR := ../../../../src/linux
all:
	$(MAKE) -C $(KDIR) M=$$PWD
endif
