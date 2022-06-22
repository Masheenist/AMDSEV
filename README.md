# AMDSEV
How to enable SEV hardware security on an AMD machine
Condensed from https://docs.ovh.com/us/en/dedicated/enable-and-use-amd-sme-sev/
Make sure that you are using at least Ubuntu 20.04! SEV is not enabled in 18.04 or below!

1. sudo nano /etc/default/grub
2. GRUB_CMDLINE_LINUX_DEFAULT="modprobe.blacklist=btrfs mem_encrypt=on kvm_amd.sev=1"
3. sudo update-grub
4. sudo reboot
5. sudo apt update
6. sudo apt install libvirt-daemon-system virtinst qemu-utils cloud-image-utils -y
7. wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
8. sudo qemu-img convert focal-server-cloudimg-amd64.img /var/lib/libvirt/images/sev-guest.img
9. cat >cloud-config <<EOF
#cloud-config

password: CHANGEME.aiZ4aetiesig
chpasswd: { expire: False }
ssh_pwauth: False
EOF

10. sudo cloud-localds /var/lib/libvirt/images/sev-guest-cloud-config.iso cloud-config
11. sudo virt-install \
              --name sev-guest \
              --memory 4096 \
              --memtune hard_limit=4563402 \
              --boot uefi \
              --disk /var/lib/libvirt/images/sev-guest.img,device=disk,bus=scsi \
              --disk /var/lib/libvirt/images/sev-guest-cloud-config.iso,device=cdrom \
              --os-type linux \
              --os-variant ubuntu20.04 \
              --import \
              --controller type=scsi,model=virtio-scsi,driver.iommu=on \
              --controller type=virtio-serial,driver.iommu=on \
              --network network=default,model=virtio,driver.iommu=on \
              --memballoon driver.iommu=on \
              --graphics none \
              --launchSecurity sev
