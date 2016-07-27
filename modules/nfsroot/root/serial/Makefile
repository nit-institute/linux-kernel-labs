ifneq ($(KERNELRELEASE),)
obj-m := feserial.o
else
KDIR := ../../../../src/linux
all:
	$(MAKE) -C $(KDIR) M=$$PWD
endif
