# Installing & Configuring Hypervisors

In this lab I will be showing how to install an VM using QEMU

1. Install QEMU/KVM  
2. Create a disk image  
3. Download an OS ISO  
4. Boot into the installer  
5. Run your new VM

## 1. Install QEMU (and KVM)

### Ubuntu / Debian
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system virtinst bridge-utils
sudo usermod -aG libvirt,qemu $USER
newgrp libvirt
```
## 2. Create a Disk Image
Choose a filename, format (qcow2 is recommended), and size:
```bash
qemu-img create -f qcow2 ~/vms/myvm.qcow2 20G
```
qemu-img create -f qcow2 ~/vms/myvm.qcow2 20G

## 3. Download an OS Installer ISO
For example, Ubuntu 22.04:

```bash
cd ~/isos
curl -LO https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
```

## 4. Boot the VM and Install
Run QEMU with KVM acceleration (if available), point at your disk and ISO:

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -m 4G \
  -smp 2 \
  -cpu host \
  -drive file=~/vms/myvm.qcow2,format=qcow2 \
  -cdrom ~/isos/ubuntu-22.04.3-live-server-amd64.iso \
  -boot d \
  -nic user,model=virtio \
  -device virtio-balloon \
  -device virtio-rng-pci \
  -device virtio-serial \
  -device virtio-net-pci,netdev=net0 \
  -netdev user,id=net0,hostfwd=tcp::2222-:22
```

## 5. First Boot

Once installation finishes, shut down the VM (inside the guest). Then reboot from disk:
```bash
qemu-system-x86_64 \
  -enable-kvm \
  -m 4G \
  -smp 2 \
  -cpu host \
  -drive file=~/vms/myvm.qcow2,format=qcow2 \
  -nic user,model=virtio,hostfwd=tcp::2222-:22
```
Now you can SSH into it from your host:

```bash
ssh -p 2222 <your-vm-username>@localhost
```

# Challenge

Create a VM in vmware workstation, oracle box, or get fancy and make it in aws cloud