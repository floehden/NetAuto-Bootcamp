# Virtualization 

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
