---
categories: ["Virtualization"]
comments: true
date: 2015-07-19T13:31:03Z
title: On Migration of KVM
url: /2015/07/19/on-migration-of-kvm/
---

### Migration
First create the qemu based vm:     

```
$ pwd
/media/arch/home/juju/img/migration
$ qemu-img create -f qcow2 ubuntu1504.qcow2 100G
$ sudo qemu-system-x86_64 -enable-kvm -m 512 -smp 4 -name ubuntu1504 -monitor stdio -boot c -drive file=/media/arch/home/juju/img/migration/ubuntu1504.qcow2,if=none,id=drive-virtio-disk0,boot=on -device virtio-blk-pci,bus=pci.0,addr=0x4,drive=drive-virtio-disk0,id=virtio-disk0 -drive file=/media/arch/home/dash/iso/ubuntu-15.04-server-amd64.iso,if=none,media=cdrom,id=drive-ide0-1-0 -device ide-drive,bus=ide.1,unit=0,drive=drive-ide0-1-0,id=ide0-1-0 -device virtio-net-pci,vlan=0,id=net0,mac=52:54:00:13:08:96 -net tap -vnc 127.0.0.1:3
```
After Installation, startup the vm via(didn't attach the file):    

```
$ sudo qemu-system-x86_64 -enable-kvm -m 512 -smp 4 -name ubuntu1504 -monitor stdio -boot c -drive file=/media/arch/home/juju/img/migration/ubuntu1504.qcow2,if=none,id=drive-virtio-disk0,boot=on -device virtio-blk-pci,bus=pci.0,addr=0x4,drive=drive-virtio-disk0,id=virtio-disk0 -drive if=none,media=cdrom,id=drive-ide0-1-0 -device ide-drive,bus=ide.1,unit=0,drive=drive-ide0-1-0,id=ide0-1-0 -device virtio-net-pci,vlan=0,id=net0,mac=52:54:00:13:08:96 -net tap -vnc 127.0.0.1:3
```
Use `top -d 1` for every second refreshed.   

The same environment is set as the src machine.   

```
$ qemu-img create -f qcow2 dest.img 20G
$ qemu-system-x86_64 -enable-kvm -m 512 -smp 4 -name ubuntu1504 -monitor stdio -boot c -drive file=/root/Code/dest.img,if=none,id=drive-virtio-disk0,boot=on -device virtio-blk-pci,bus=pci.0,addr=0x4,drive=drive-virtio-disk0,id=virtio-disk0 -drive if=none,media=cdrom,id=drive-ide0-1-0 -device ide-drive,bus=ide.1,unit=0,drive=drive-ide0-1-0,id=ide0-1-0 -device virtio-net-pci,vlan=0,id=net0,mac=52:54:00:13:08:96 -net tap -vnc 127.0.0.1:8
(qemu) info status
VM status: paused (inmigrate)
```
Start migration in the src side via:    

```
(qemu) migrate -d -b tcp:192.168.1.18:8888
(qemu) info migrate
```
In destination machine, you can see the status of the migration.    

After migration, the machine stays its top output to the terminal.  

### Trouble Shooting On Newly Installed Arch
#### Bug1 virtual network start fail
libvirt via virt-manager virtual network start failed.     
Change:    

```
--- libvirt-1.2.16.orig/src/util/virfirewall.c  2015-05-23 08:56:12.000000000 -0400
+++ libvirt-1.2.16.new/src/util/virfirewall.c   2015-06-18 10:01:51.954157612 -0400
@@ -932,14 +932,14 @@
 
     virMutexLock(&ruleLock);
 
-    if (currentBackend == VIR_FIREWALL_BACKEND_AUTOMATIC) {
+//    if (currentBackend == VIR_FIREWALL_BACKEND_AUTOMATIC) {
         /* a specific backend should have been set when the firewall
          * object was created. If not, it means none was found.
          */
-        virReportError(VIR_ERR_INTERNAL_ERROR, "%s",
-                       _("Failed to initialize a valid firewall backend"));
-        goto cleanup;
-    }
+//        virReportError(VIR_ERR_INTERNAL_ERROR, "%s",
+//                       _("Failed to initialize a valid firewall backend"));
+//        goto cleanup;
+//    }
     if (!firewall || firewall->err == ENOMEM) {
         virReportOOMError();
         goto cleanup;
```
For building the libvirt, do following:    

```
# pacman -S abs base-devel
# abs 
# cp /var/abs/community/libvirt ~/Code/
# cd ~/Code/libvirt
# makepkg
```
After makepkg, change the code as in above, tar it again, makepkg with following command `makepkg --skipchecksums`, this time it will generate a new tar.xz file. Install it via:    

```
# pacman -U libvirt-1.2.17-1-x86_64.pkg.tar.xz
```

#### starting network out of memory
Solve it via:    

```
# pacman -S ebtables vde2
# reboot
```
Start again and this time is OK.   

#### Bridge Configuration

```
[root@Arch network]# pwd
/etc/systemd/network
[root@Arch network]# cat MyBridge.netdev 
[NetDev]
Name=br0
MACAddress=52:54:00:91:e8:11
Kind=bridge
[root@Arch network]# cat MyBridge.network 
[Match]
Name=br0

[Network]
DNS=180.76.76.76,114.114.114.114

[Address]
Address=192.168.1.18/24

[Route]
Gateway=192.168.1.1
[root@Arch network]# cat MyEth.network 
[Match]
Name=eth0

[Network]
Bridge=br0
```
Now enable and start the systemd's networkd service via:    

```
# systemctl enable systemd-networkd.service
# reboot
```
By this you could enable systemd on ArchLinux.   
