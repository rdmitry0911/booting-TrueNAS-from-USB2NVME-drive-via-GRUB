# booting TrueNAS from USB2NVME or USB2SSD drive via GRUB

For those, like me, who would like to install TrueNAS to USB3.1 Solid Flash Drive like this one: https://www.aliexpress.com/item/4000534378012.html and have problems to boot TrueNAS from it as BIOS/UEFI does not see this drive. 

To boot from it we either have to make a separate bootable usb drive, that is visible by BIOS/UEFI with GRUB on it or make a pxe booting of GRUB. In either case we have to add this menu enry to GRUB:


menuentry "TrueNAS" --class freebsd --class bsd --class os {
    
    insmod part_gpt
    insmod zfs
    insmod bsd
    nativedisk

    search --no-floppy --set -l boot-pool
    kfreebsd /ROOT/default/@/boot/kernel/kernel

    kfreebsd_module_elf /ROOT/default/@/boot/kernel/if_qlxgbe.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/ipmi.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/smbus.ko
    kfreebsd_module_elf /ROOT/default/@/boot/modules/openzfs.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/smartpqi.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/ocs_fc.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/if_bnxt.ko
    kfreebsd_module_elf /ROOT/default/@/boot/kernel/ispfw.ko

    set kFreeBSD.vfs.root.mountfrom=zfs:boot-pool/ROOT/default

}

To prepare GRUB booting via pxe in ubuntu/debian we have to create a bootable image of GRUB with this command:

grub-mkimage -d /usr/lib/grub/i386-pc/ -O i386-pc-pxe -o ./grub-pxe386 -p '/grub' pxe tftp part_gpt zfs bsd nativedisk biosdisk

and then copy grub folder from /boot to tftp_root. Put grub-pxe386 also to tftp_root/grub then
edit /etc/dnsmasq.conf of your dhcp server and put there these lines:

dhcp-mac=set:truenas,XX:XX:XX:XX:XX:XX

dhcp-boot=tag:truenas,/grub/grub-pxe386,,172.16.66.99

where XX:XX:XX:XX:XX:XX is MAC address of your TrueNAS and 172.16.66.100 is ip address of your tftp server
