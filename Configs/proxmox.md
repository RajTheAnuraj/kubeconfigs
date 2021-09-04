to use a disk inside a vm
Run following from the shell of proxmox root

```
qm set 101 -scsi2 /dev/disk/by-id/ata-ST4000VN008-2DR166_ZGY75F6G
```

qm - command 
set - param
101 - vm Id. This is set when you create VM
-scsi2 - If you have multiple disks keep incrementing the number. If you go to the hardware tab of the VM node you want this to attach to, you can see the current scsi devices
finally the path in which disk is mounted on host OS

qm command lets you assign upto 30 devices