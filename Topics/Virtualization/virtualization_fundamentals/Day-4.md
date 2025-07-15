# VM lifecycle managment

The VM lifecycle refers to the series of stages a virtual machine goes through, from creation and configuration, to use, maintenance (such as cloning, snapshotting, or migration), and finally to retirement or deletion.
Properly managing the VM lifecycle ensures optimal resource usage, system security, compliance, and agility in IT environments. It allows organizations to quickly provision new workloads, maintain backups, recover from failures, migrate services with minimal downtime, and safely decommission obsolete systems.
A well-defined VM lifecycle is essential for efficient operations, cost control, and maintaining service reliability in virtualized data centers.


## Objectives
- Learn what `virsh` is and why it’s useful  
- Perform core VM lifecycle operations via CLI  
- Snapshot, migrate, and automate VMs  
- Manage virtual networks & storage pools

---

## Prerequisites
- A Linux host with **libvirt** & **QEMU/KVM** (or Xen) installed  
- Basic shell familiarity (bash)  
- At least one defined VM (`.xml` domain) or use `virt-install` to create one  

---

## 1. Quick Overview  
`virsh` is the command-line tool for **libvirt**, an abstraction layer over hypervisors. It uses a simple XML description ("domain XML") to define and control VMs, networks, and storage objects.

```bash
# Check libvirt version & connection
virsh --version
virsh uri
```

---

## 2. Inspecting Your Environment  

```bash
# List all VMs (running & stopped)
virsh list --all

# Show details for "myvm"
virsh dominfo myvm

# Get its XML definition
virsh dumpxml myvm > myvm.xml
```

---

## 3. VM Lifecycle Basics  

```bash
# Define/register a VM from XML (one-time)
virsh define /path/to/myvm.xml

# Start and stop
virsh start myvm
virsh shutdown myvm        
virsh destroy myvm          

# Pause & resume
virsh suspend myvm
virsh resume myvm

# Reboot
virsh reboot myvm
```

---

## 4. Snapshots vs. Backups  

```bash
# Create a disk-only, quiesced snapshot
virsh snapshot-create-as myvm pre-update \
  --description "Before patch" --disk-only --atomic --quiesce

# List & revert
virsh snapshot-list myvm
virsh snapshot-revert myvm pre-update

# Manual full backup (outside libvirt)
virsh shutdown myvm
cp /var/lib/libvirt/images/myvm.qcow2 /backups/
virsh start myvm
```

---

## 5. Live Migration & High Availability  

```bash
# Prepare passwordless SSH: ssh-copy-id target-host
# Live-migrate a running VM
virsh migrate --live --persistent --undefinesource \
  myvm qemu+ssh://target-host/system

# Verify on destination
ssh target-host virsh list
```

> For HA in clusters, combine `virsh` with tools like **Pacemaker** or use your distro’s high-availability stack.

---

## 6. Managing Networks & Storage  

```bash
# List virtual networks & pools
virsh net-list --all
virsh pool-list --all

# Start/stop a network
virsh net-start default
virsh net-destroy default

# Create a storage volume
virsh vol-create-as default smallvol 5G --format qcow2
virsh vol-list default
```

---

## Reference

- [libvirt](https://libvirt.org/virshcmdref.html)
- [VM Lifecycle Management Overview – VMware Docs](https://docs.vmware.com/en/vRealize-Suite-Lifecycle-Manager/8.10/vrealize-suite-lifecycle-manager-810/GUID-3A5FADAC-5BC8-4DD9-9E3A-7A62A18C7D03.html)