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
- At least one defined VM (`.xml` domain) or use `virt-install` to create one  # Virtualization 

A compact, hands-on training program introducing you to server virtualization concepts, setup, networking, storage, and day-to-day operations. Designed for IT professionals and enthusiasts who want to rapidly become productive with hypervisor platforms.

---


## Course Overview

Over four intensive days, you will:

1. Understand **why** and **how** virtualization works  
2. Install and configure both type-1 and type-2 hypervisors  
3. Design and implement virtual networks and shared storage  
4. Manage VM cloning, snapshots, live migrations, and performance tuning  

## Prerequisites

- A modern PC or server (≥ 8 GB RAM, ≥ 50 GB free disk)  
- Host OS: Linux (Ubuntu/CentOS) or Windows 10/11  
- Downloaded ISOs for your chosen hypervisors or VMs  
- Basic familiarity with the command line (bash or PowerShell)  


## Day 1: Virtualization Fundamentals

**Topics Covered**  
1. **What is Virtualization?**  
   - Software abstraction of compute, storage, and networking  
2. **Benefits & Use Cases**  
   - Consolidation, isolation, testing, development  
3. **Hypervisor Types**  
   - Type-1 (bare-metal) vs. Type-2 (hosted)  
4. **Core Components**  
   - Compute, storage, network layers  


## Day 2: Installing & Configuring Hypervisors

**Topics Covered**  
1. **Overview of Popular Platforms**  
   - KVM, Hyper-V, VMware ESXi, VirtualBox  
2. **Type-2 Hypervisor Installation**  
   - Hands-on: install VirtualBox or VMware Workstation  
3. **Type-1 Hypervisor Quickstart**  

## Day 3: Virtual Networking & Storage

**Topics Covered**  
1. **Virtual Switches & Port Groups**  
   - Create isolated and bridged networks  
2. **VLAN Tagging & NIC Teaming**  
   - Segment traffic and improve redundancy  
3. **Datastore Types**  
   - VMFS, NFS, iSCSI fundamentals  
4. **Configuring Shared Storage**  
   - Lab: export a network share (NFS or SMB) and mount in guest  


## Day 4: VM Lifecycle Management

**Topics Covered**  
1. **Cloning VMs**  
   - Create full clones of existing VMs using virsh clone
   - Use QCOW2 backing files for space-efficient linked clones
2. **Snapshots vs. Backups**  
   - Create and manage VM snapshots with virsh snapshot-create-as, snapshot-list, snapshot-revert
   - Perform full backups by shutting down the VM and copying its disk images and XML configuration
3. **Live Migration & High Availability**  
   - Live-migrate running VMs between hosts with virsh migrate for zero downtime
   - Understand requirements for shared storage and consistent networking for HA
4. **Resource Pools & Performance Tuning**  
   - Organize storage using virsh pool-define and manage VM disks with storage pools
   - Adjust CPU, memory, and other resources for VMs using virsh setvcpus, setmem, and tune performance with monitoring tools like virt-top

## References

- [VMware Docs: vSphere Virtualization](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [KVM Project Documentation](https://www.linux-kvm.org/page/Documentation)
- [Red Hat: Virtualization Overview](https://www.redhat.com/en/topics/virtualization)
- [Microsoft Hyper-V Documentation](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/)
- [Proxmox Virtual Environment Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [Oracle VirtualBox Documentation](https://www.virtualbox.org/manual/UserManual.html)
- [IBM: Virtualization Explained](https://www.ibm.com/topics/virtualization)
- [OpenStack Docs](https://docs.openstack.org/)


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