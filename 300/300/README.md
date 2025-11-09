# 300 - Azure Virtual Machine Reverse Engineering

I’ll create a comprehensive reverse engineering document for Azure Virtual Machine following the same structure and depth as your NSG document. Let me build this for you.​​​​​​​​​​​​​​​​

Done! I’ve created a comprehensive reverse engineering document for Azure Virtual Machine following the exact same structure as your NSG document.

[View your Azure VM document](computer:///mnt/user-data/outputs/Azure_Virtual_Machine_Reverse_Engineering.md)

The document includes:

**8 Layers of Deep Technical Analysis:**

1. **Layer 1**: What you see in the Azure Portal (the simple UI)
1. **Layer 2**: VM components & structure (sizes, storage, networking, extensions)
1. **Layer 3**: How virtual machines work (virtualization fundamentals, Hyper-V, CPU/memory/I/O virtualization)
1. **Layer 4**: Under the hood technology (Azure Fabric, Windows Server + Hyper-V, SDN, storage backend)
1. **Layer 5**: Mapping to compute architecture (from application code down to physical silicon)
1. **Layer 6**: VM lifecycle & operations (creation flow, resize, snapshots, backups)
1. **Layer 7**: Physical implementation (datacenter layout, physical servers, network/storage paths)
1. **Layer 8**: Practical examples (web server, HA database cluster, auto-scaling VMSS, GPU ML training)

**Key topics covered:**

- Complete virtualization explanation (Type 1 vs Type 2 hypervisors)
- Hyper-V architecture and how it works
- CPU, memory, storage, and network virtualization
- VM sizes and families explained
- Physical hardware specifications (Intel Xeon CPUs, RAM, NICs, GPUs)
- Live migration and high availability mechanisms
- Complete packet and I/O journeys from VM to physical hardware
- Real-world scenarios with costs and configurations

The document takes you from clicking “Create VM” in the portal all the way down to transistors switching in the physical Intel/AMD processors in Azure datacenters. Perfect preparation for your Cloud Engineer technical assessment!​​​​​​​​​​​​​​​​

# Reverse Engineering Azure Virtual Machines

## From Azure Portal to Physical Silicon - A Complete Breakdown

A comprehensive deconstruction of how Azure VMs work, from what you click in the portal down to the physical hardware in the datacenter.

-----

## Table of Contents

- [Layer 1: What You See (Azure Portal)](#layer-1-what-you-see-azure-portal)
- [Layer 2: VM Components & Structure](#layer-2-vm-components--structure)
- [Layer 3: How Virtual Machines Work](#layer-3-how-virtual-machines-work)
- [Layer 4: Under the Hood - The Technology](#layer-4-under-the-hood---the-technology)
- [Layer 5: Mapping to Compute Architecture](#layer-5-mapping-to-compute-architecture)
- [Layer 6: VM Lifecycle & Operations](#layer-6-vm-lifecycle--operations)
- [Layer 7: Physical Implementation](#layer-7-physical-implementation)
- [Layer 8: Practical Examples](#layer-8-practical-examples)

-----

## Layer 1: What You See (Azure Portal)

### The User Interface View

When you navigate to create a VM in the Azure Portal, here’s what you see:

```
Azure Portal → Create a resource → Virtual machine
├── Basics
│   ├── Subscription
│   ├── Resource Group
│   ├── VM name
│   ├── Region
│   ├── Availability options
│   ├── Image (OS)
│   ├── Size (CPU/RAM)
│   └── Authentication
├── Disks
│   ├── OS disk type
│   └── Data disks
├── Networking
│   ├── Virtual network
│   ├── Subnet
│   ├── Public IP
│   └── NSG
├── Management
│   ├── Monitoring
│   ├── Auto-shutdown
│   └── Backup
└── Review + create
```

### Creating a VM: The Simple View

**Portal Steps**:

1. Click “Create a resource”
1. Select “Virtual machine”
1. Fill in:
- Name: `WebServer1`
- Region: `West Europe`
- Image: `Ubuntu 22.04 LTS`
- Size: `Standard_D2s_v3` (2 vCPUs, 8 GB RAM)
- Username: `azureuser`
1. Click “Create”

**What Just Happened?**

- Azure reserved physical resources
- Created a JSON ARM template
- Provisioned virtual hardware
- Deployed and configured the OS
- Set up networking and storage
- Started the VM automatically

### The Abstraction

**What Azure Hides From You**:

```
Simple Portal View:
┌────────────────────────┐
│  Virtual Machine       │
│  - Name: WebServer1    │
│  - Size: D2s_v3        │
│  - Status: Running     │
│  - OS: Ubuntu 22.04    │
└────────────────────────┘

What's Actually Running:
┌────────────────────────────────────────┐
│  Complex Virtualization Infrastructure │
│  - Hyper-V hypervisor partition        │
│  - Virtual hardware devices            │
│  - Memory management unit              │
│  - Virtual CPU scheduling              │
│  - Storage I/O virtualization          │
│  - Network I/O virtualization          │
│  - Security isolation boundaries       │
│  - Live migration capability           │
│  - High availability orchestration     │
│  - Distributed across physical hosts   │
└────────────────────────────────────────┘
```

-----

## Layer 2: VM Components & Structure

### Anatomy of an Azure VM

An Azure VM consists of several interconnected components:

#### 1. VM Resource Itself

```json
{
  "name": "WebServer1",
  "location": "westeurope",
  "type": "Microsoft.Compute/virtualMachines",
  "id": "/subscriptions/{sub-id}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/WebServer1",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_D2s_v3"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "0001-com-ubuntu-server-jammy",
        "sku": "22_04-lts-gen2",
        "version": "latest"
      },
      "osDisk": {
        "name": "WebServer1_OsDisk",
        "caching": "ReadWrite",
        "createOption": "FromImage",
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        }
      }
    },
    "osProfile": {
      "computerName": "WebServer1",
      "adminUsername": "azureuser",
      "linuxConfiguration": {
        "disablePasswordAuthentication": true,
        "ssh": {
          "publicKeys": [...]
        }
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/.../networkInterfaces/WebServer1-nic"
        }
      ]
    }
  }
}
```

**Resource Properties**:

- **Name**: VM identifier (hostname)
- **Location**: Azure region (determines physical datacenter)
- **VM Size**: Compute resources (vCPUs, RAM, temp storage)
- **Image**: Operating system template
- **OS Profile**: Configuration settings
- **Network Profile**: Network connectivity
- **Storage Profile**: Disk configuration

#### 2. VM Sizes & Series

Azure offers VM sizes organized by family:

**General Purpose (B, D series)**:

```
Standard_B2s:     2 vCPUs, 4 GB RAM  (Burstable)
Standard_D2s_v3:  2 vCPUs, 8 GB RAM  (Balanced)
Standard_D4s_v3:  4 vCPUs, 16 GB RAM (Balanced)
Standard_D8s_v3:  8 vCPUs, 32 GB RAM (Balanced)
```

**Compute Optimized (F series)**:

```
Standard_F2s_v2:  2 vCPUs, 4 GB RAM  (High CPU:RAM ratio)
Standard_F4s_v2:  4 vCPUs, 8 GB RAM  (High CPU:RAM ratio)
Standard_F8s_v2:  8 vCPUs, 16 GB RAM (High CPU:RAM ratio)
```

**Memory Optimized (E series)**:

```
Standard_E2s_v3:  2 vCPUs, 16 GB RAM (High RAM:CPU ratio)
Standard_E4s_v3:  4 vCPUs, 32 GB RAM (High RAM:CPU ratio)
Standard_E8s_v3:  8 vCPUs, 64 GB RAM (High RAM:CPU ratio)
```

**Storage Optimized (L series)**:

```
Standard_L4s:     4 vCPUs, 32 GB RAM, 678 GB Temp (High I/O)
Standard_L8s:     8 vCPUs, 64 GB RAM, 1388 GB Temp (High I/O)
```

**GPU (N series)**:

```
Standard_NC6:     6 vCPUs, 56 GB RAM, 1x NVIDIA Tesla K80
Standard_NC12:    12 vCPUs, 112 GB RAM, 2x NVIDIA Tesla K80
```

**Size Naming Convention**:

```
Standard_D2s_v3
│       │││ ││
│       │││ │└─ Version (v3 = 3rd generation)
│       │││ └── "s" = Premium SSD support
│       ││└──── Size number (2 = small, 4 = medium, etc.)
│       │└───── Series letter (D = general purpose)
│       └────── Family indicator
└────────────── Pricing tier
```

#### 3. Virtual Hardware Components

Each VM gets virtualized hardware:

```
Virtual Machine:
├── Virtual CPU(s)
│   ├── vCPU cores (Intel or AMD)
│   ├── Hyperthreading capable
│   └── CPU scheduling by hypervisor
├── Virtual Memory
│   ├── RAM allocation
│   ├── Memory management unit
│   └── NUMA awareness
├── Virtual Network Interface(s)
│   ├── Virtual NIC (vNIC)
│   ├── MAC address
│   ├── Accelerated Networking (SR-IOV)
│   └── IP configuration
├── Virtual Disks
│   ├── OS Disk (required)
│   ├── Temporary Disk (ephemeral)
│   └── Data Disks (optional, up to 64)
├── Virtual BIOS/UEFI
│   ├── Generation 1 (BIOS)
│   └── Generation 2 (UEFI, TPM, Secure Boot)
└── Virtual Devices
    ├── Virtual mouse/keyboard
    ├── Virtual display adapter
    └── Virtual serial console
```

#### 4. Storage Architecture

**OS Disk**:

```json
{
  "name": "WebServer1_OsDisk",
  "diskSizeGB": 30,
  "managedDisk": {
    "storageAccountType": "Premium_LRS",
    "id": "/subscriptions/.../disks/WebServer1_OsDisk"
  },
  "caching": "ReadWrite",
  "createOption": "FromImage"
}
```

**Disk Types**:

```
Standard HDD (Standard_LRS):
- Magnetic disk storage
- Up to 500 IOPS
- Up to 60 MB/s throughput
- Lowest cost
- Use case: Dev/test, backup

Standard SSD (StandardSSD_LRS):
- Solid state storage
- Up to 6,000 IOPS
- Up to 750 MB/s throughput
- Moderate cost
- Use case: Web servers, light workloads

Premium SSD (Premium_LRS):
- High-performance SSD
- Up to 20,000 IOPS
- Up to 900 MB/s throughput
- Higher cost
- Use case: Production workloads, databases

Premium SSD v2 (PremiumV2_LRS):
- Next-gen SSD
- Up to 80,000 IOPS
- Up to 1,200 MB/s throughput
- Performance configurable independent of size
- Use case: Mission-critical applications

Ultra Disk (UltraSSD_LRS):
- Ultra-high performance
- Up to 160,000 IOPS
- Up to 4,000 MB/s throughput
- Sub-millisecond latency
- Use case: SAP HANA, top-tier databases
```

**Temporary Disk**:

```
Every VM includes a temporary disk:
- Local to the physical host
- Non-persistent (data lost on VM stop/redeploy)
- High performance (locally attached)
- No cost (included with VM)
- Typically /dev/sdb (Linux) or D: (Windows)
- Use case: Page file, temp data, cache
```

#### 5. Network Interface (NIC)

```json
{
  "name": "WebServer1-nic",
  "type": "Microsoft.Network/networkInterfaces",
  "location": "westeurope",
  "properties": {
    "ipConfigurations": [
      {
        "name": "ipconfig1",
        "properties": {
          "privateIPAddress": "10.0.1.5",
          "privateIPAllocationMethod": "Dynamic",
          "subnet": {
            "id": "/subscriptions/.../subnets/default"
          },
          "publicIPAddress": {
            "id": "/subscriptions/.../publicIPAddresses/WebServer1-ip"
          }
        }
      }
    ],
    "enableAcceleratedNetworking": true,
    "enableIPForwarding": false,
    "networkSecurityGroup": {
      "id": "/subscriptions/.../networkSecurityGroups/WebServer1-nsg"
    }
  }
}
```

**NIC Features**:

- **Private IP**: Internal VNet address (10.0.1.5)
- **Public IP**: Internet-facing address (optional)
- **Accelerated Networking**: SR-IOV for low latency
- **Multiple NICs**: Up to 8 per VM (depends on size)
- **Multiple IPs**: Multiple IPs per NIC

#### 6. VM Extensions

Extensions add functionality post-deployment:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "WebServer1/CustomScript",
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://mystorageaccount.blob.core.windows.net/scripts/setup.sh"
      ]
    },
    "protectedSettings": {
      "commandToExecute": "sh setup.sh"
    }
  }
}
```

**Common Extensions**:

```
CustomScript Extension:
- Run scripts on VM after deployment
- Linux: bash/python/perl
- Windows: PowerShell/CMD

Azure Monitor Agent:
- Collect metrics and logs
- Send to Log Analytics workspace

Azure Backup Extension:
- Enable VM-level backup
- Application-consistent snapshots

Desired State Configuration (DSC):
- Configure Windows VMs
- Enforce configuration state

Dependency Agent:
- Network dependency mapping
- Service Map integration

Antimalware Extension:
- Microsoft Antimalware
- Real-time protection (Windows)
```

#### 7. Availability Options

**Single VM**:

```
No redundancy:
- Single instance
- No SLA (unless using Premium SSD)
- 99.9% SLA with Premium SSD for all disks
```

**Availability Set**:

```
VMs distributed across:
- Fault Domains (FD): Different power/network
- Update Domains (UD): Different maintenance windows

Configuration:
- Up to 3 Fault Domains
- Up to 20 Update Domains
- 99.95% SLA
```

**Availability Zone**:

```
VMs in different physical datacenters:
- Zone 1: Datacenter A
- Zone 2: Datacenter B  
- Zone 3: Datacenter C

Benefits:
- Physical separation
- Independent power, cooling, networking
- 99.99% SLA
```

**Virtual Machine Scale Set (VMSS)**:

```
Auto-scaling group of identical VMs:
- Load balanced
- Auto-scale based on metrics
- Rolling updates
- 99.95% SLA (multiple instances)
```

-----

## Layer 3: How Virtual Machines Work

### Virtualization Fundamentals

#### What is Virtualization?

```
Physical Server WITHOUT Virtualization:
┌────────────────────────────────────┐
│         Single Application         │
├────────────────────────────────────┤
│      Operating System (Windows)    │
├────────────────────────────────────┤
│        Physical Hardware           │
│  (CPU, RAM, Disk, Network)         │
└────────────────────────────────────┘
Problem: Underutilized (typically 5-15% CPU usage)

Physical Server WITH Virtualization:
┌─────────────┬─────────────┬─────────────┐
│    VM 1     │    VM 2     │    VM 3     │
│  Ubuntu     │   Windows   │   Ubuntu    │
│  App A      │   App B     │   App C     │
├─────────────┴─────────────┴─────────────┤
│        Hypervisor (Hyper-V)             │
├─────────────────────────────────────────┤
│        Physical Hardware                │
│  (CPU, RAM, Disk, Network)              │
└─────────────────────────────────────────┘
Solution: Multiple VMs, better utilization (60-80%)
```

#### Type 1 vs Type 2 Hypervisors

**Type 1 (Bare Metal)** - What Azure Uses:

```
┌─────────────┬─────────────┬─────────────┐
│    VM 1     │    VM 2     │    VM 3     │
├─────────────┴─────────────┴─────────────┤
│     Hypervisor (Hyper-V)                │ ← Runs directly on hardware
├─────────────────────────────────────────┤
│     Physical Hardware                   │
└─────────────────────────────────────────┘

Characteristics:
- Direct hardware access
- Better performance
- Lower overhead
- Examples: Hyper-V, VMware ESXi, Xen
```

**Type 2 (Hosted)**:

```
┌─────────────┬─────────────┐
│    VM 1     │    VM 2     │
├─────────────┴─────────────┤
│  Hypervisor (VirtualBox)  │
├───────────────────────────┤
│   Host OS (Windows/Linux) │ ← Extra layer
├───────────────────────────┤
│   Physical Hardware       │
└───────────────────────────┘

Characteristics:
- Runs on top of an OS
- More overhead
- Easier to install
- Examples: VirtualBox, VMware Workstation
```

### Hyper-V Architecture (Azure’s Hypervisor)

#### Hyper-V Components

```
Azure Physical Server:
┌──────────────────────────────────────────────────┐
│  Management Partition (Root/Parent Partition)    │
│  - Hyper-V management                            │
│  - Device drivers                                │
│  - Azure Host Agent                              │
│  - Fabric Controller communication               │
├──────────────────────────────────────────────────┤
│              Hypervisor (Hyper-V)                │
│  - CPU virtualization                            │
│  - Memory virtualization                         │
│  - Interrupt handling                            │
│  - I/O MMU                                       │
├──────────────────────────────────────────────────┤
│  Child Partition 1  │  Child Partition 2  │  ... │
│  (Your Azure VM)    │  (Another customer) │      │
│  - Guest OS         │  - Guest OS         │      │
│  - Applications     │  - Applications     │      │
└──────────────────────────────────────────────────┘
       ↓                       ↓
┌──────────────────────────────────────────────────┐
│           Physical Hardware                      │
│  - Intel Xeon or AMD EPYC CPUs                   │
│  - DDR4/DDR5 RAM (hundreds of GB)                │
│  - NVMe SSDs (local storage)                     │
│  - 25/100 Gbps NICs                              │
└──────────────────────────────────────────────────┘
```

**Partition Types**:

- **Root Partition (Parent)**:
  - Runs Windows Server
  - Has direct hardware access
  - Manages child partitions
  - Runs virtualization stack
- **Child Partitions**:
  - Your Azure VMs run here
  - Isolated from each other
  - No direct hardware access
  - All I/O goes through parent or VMBus

#### CPU Virtualization

**How VMs Share Physical CPUs**:

```
Physical Server: 64 cores (Intel Xeon)

Hypervisor CPU Scheduler:
├── VM 1: 2 vCPUs → Scheduled on cores 0, 1
├── VM 2: 4 vCPUs → Scheduled on cores 2, 3, 4, 5
├── VM 3: 2 vCPUs → Scheduled on cores 6, 7
└── VM 4: 8 vCPUs → Scheduled on cores 8-15

Scheduling Algorithm:
- Fair-share scheduling
- vCPUs time-sliced on physical cores
- Context switching between VMs
- CPU limits and reservations enforced
```

**Intel VT-x / AMD-V Hardware Assistance**:

Modern CPUs have hardware support for virtualization:

```
Without Hardware Virtualization:
Guest OS → Trap → Hypervisor emulates → Slow

With Hardware Virtualization (VT-x/AMD-V):
Guest OS → Direct execution → Hardware handles → Fast

Intel VT-x Features Used:
- VMX (Virtual Machine Extensions)
- EPT (Extended Page Tables)
- VT-d (Directed I/O)
- VPID (Virtual Processor ID)
```

**CPU Oversubscription**:

```
Physical cores: 64
Total vCPUs allocated: 128

Oversubscription ratio: 2:1

Why it works:
- VMs don't use 100% CPU constantly
- Average utilization: 20-30%
- Scheduler balances load
- Performance impact minimal if not under heavy load

Azure limits:
- Controlled oversubscription
- Performance guarantees maintained
- Burst capability during low contention
```

#### Memory Virtualization

**Virtual Memory to Physical Memory Mapping**:

```
VM's View (Guest Physical Memory):
┌────────────────────────────┐
│  VM thinks it has 8 GB     │
│  Address: 0x00000000 to    │
│           0x200000000      │
└────────────────────────────┘
           ↓
    Hypervisor Translation
           ↓
┌────────────────────────────┐
│  Actual Physical RAM       │
│  Location: Random pages    │
│  across server's 256 GB    │
└────────────────────────────┘
```

**Memory Management Techniques**:

1. **Second Level Address Translation (SLAT)**:

```
Intel EPT (Extended Page Tables):
Guest Virtual Addr → Guest Physical Addr → Host Physical Addr
      (OS)                (Hypervisor)          (Hardware)

Benefits:
- Hardware-accelerated translation
- No software overhead
- TLB caching at multiple levels
```

1. **Memory Ballooning**:

```
If host memory pressure is high:

Hypervisor → VM Memory Driver → Inflates "balloon"
                                (allocates memory)
                                  ↓
                         Memory returned to host
                                  ↓
                         Can be given to other VMs

Note: Azure rarely uses this due to strict resource allocation
```

1. **Memory Overcommit**:

```
Azure does NOT overcommit memory:
- Each VM gets reserved physical RAM
- No memory sharing between customer VMs
- Guarantees performance
- Prevents "noisy neighbor"
```

#### I/O Virtualization

**Storage I/O Path**:

```
Traditional (Slow) Path:
VM App → Guest OS → Virtual SCSI → Hypervisor → Storage Driver → Disk
  ↓        ↓            ↓              ↓              ↓           ↓
User     Kernel    Virtualized    Translation    Physical    Physical
Space    Space     Device         Layer          Driver      Storage

Optimized (Fast) Path - Azure Storage:
VM App → Guest OS → Azure Storage Driver (in VM)
                          ↓
                    Direct to Azure Storage API
                          ↓
                    Azure Storage service
                          ↓
                    Distributed storage backend
```

**Accelerated Networking (SR-IOV)**:

```
Without Accelerated Networking:
VM → Virtual NIC → Hypervisor vSwitch → Physical NIC
  ↓        ↓            ↓                     ↓
  App    Kernel     Translation          Physical
         ↑________________↓
         High latency (microseconds)

With Accelerated Networking (SR-IOV):
VM → Virtual Function → Physical NIC
  ↓           ↓              ↓
  App    Direct access    Physical
         ↑____________↓
         Low latency (nanoseconds)

SR-IOV (Single Root I/O Virtualization):
Physical NIC ← Presents multiple virtual functions
├── VF 1 → Directly to VM 1 (your VM)
├── VF 2 → Directly to VM 2
└── VF 3 → Directly to VM 3

Benefits:
- Up to 30 Gbps throughput
- Ultra-low latency
- Reduced CPU usage
- No hypervisor in data path
```

### VM State Machine

A VM goes through various states:

```
State Diagram:
                    
     [Creating] ← Initial provisioning
         ↓
     [Starting]
         ↓
     [Running] ← VM is operational
         ↓ ↑
    [Stopping]
         ↓
     [Stopped] ← VM deallocated, no compute charges
         ↓ ↑
    [Starting]
         ↓
     [Running]
         ↓
    [Deleting]
         ↓
     [Deleted]
```

**State Details**:

```
Running:
- VM fully operational
- OS booted
- Consuming compute resources
- Billing: Compute + Storage

Stopped (Deallocated):
- VM powered off
- Resources released to Azure pool
- IP addresses may change on restart
- Billing: Storage only (no compute)

Stopped (but still allocated):
- VM powered off
- Resources still reserved
- IP addresses retained
- Billing: Compute + Storage (full cost!)
```

-----

## Layer 4: Under the Hood - The Technology

### Azure Fabric Architecture

#### Fabric Terminology

```
Azure Region (e.g., West Europe):
├── Availability Zones (3 per region)
│   ├── Zone 1: Multiple datacenters
│   ├── Zone 2: Multiple datacenters
│   └── Zone 3: Multiple datacenters
├── Datacenters
│   ├── Datacenter 1
│   │   ├── Clusters (groups of racks)
│   │   │   ├── Cluster 1
│   │   │   │   ├── Racks (42U cabinets)
│   │   │   │   │   ├── Rack 1
│   │   │   │   │   │   ├── Physical Servers (1U/2U)
│   │   │   │   │   │   │   ├── Server 1 (Your VM here)
│   │   │   │   │   │   │   ├── Server 2
│   │   │   │   │   │   │   └── ...
│   │   │   │   │   │   └── Top-of-Rack (ToR) Switch
│   │   │   │   │   └── Rack 2
│   │   │   │   └── ...
│   │   │   └── Cluster 2
│   │   └── ...
│   └── Datacenter 2
└── Fabric Controllers (manage all resources)
```

#### Fabric Controller

The brain of Azure:

```
Fabric Controller Cluster:
┌────────────────────────────────────┐
│  Fabric Controller (FC)            │
│  - Multiple redundant instances    │
│  - Consensus-based (Paxos/Raft)    │
│  - Distributed state machine       │
│                                    │
│  Responsibilities:                 │
│  ├── Resource allocation           │
│  ├── VM placement                  │
│  ├── Health monitoring             │
│  ├── Failure detection             │
│  ├── Live migration                │
│  ├── Scale operations              │
│  └── Update orchestration          │
└────────────────────────────────────┘
         ↓
    Communicates with
         ↓
┌────────────────────────────────────┐
│  Host Agents (on every server)     │
│  - Receives FC commands            │
│  - Reports server health           │
│  - Manages local VMs               │
│  - Executes operations             │
└────────────────────────────────────┘
```

**Fabric Controller Operations**:

```
VM Creation Flow:
1. User clicks "Create VM" in portal
2. ARM API receives request
3. Request validated and queued
4. Fabric Controller picks up request
5. FC finds suitable physical server:
   - Enough CPU cores available
   - Enough RAM available
   - In correct region/zone
   - Not over-utilized
   - Meets VM family requirements
6. FC instructs Host Agent on selected server
7. Host Agent provisions VM
8. Host Agent reports back to FC
9. FC updates state: VM = Running
10. User sees VM as "Running" in portal
```

### Windows Server Core + Hyper-V Role

Azure hosts run Windows Server:

```
Physical Server OS Stack:
┌────────────────────────────────────┐
│  Windows Server Core               │
│  - Minimal GUI (no desktop)        │
│  - Optimized for Hyper-V           │
│  - Hardened and secured            │
│                                    │
│  Hyper-V Role:                     │
│  ├── Hypervisor layer              │
│  ├── Virtual switch                │
│  ├── VM management                 │
│  └── Storage/Network stack         │
│                                    │
│  Azure Host Agent:                 │
│  ├── Fabric Controller client      │
│  ├── VM lifecycle management       │
│  └── Monitoring and reporting      │
└────────────────────────────────────┘
```

### Storage Backend

Azure VMs connect to Azure Storage:

```
VM's Disk I/O Journey:
┌────────────────────────────────────┐
│  VM: Running Application           │
│  └── Writes to /dev/sda            │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Guest OS: Linux Kernel            │
│  └── Block layer                   │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Virtual SCSI Controller           │
│  └── Azure VM Guest Agent          │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Hyper-V Host: Storage Stack       │
│  └── Azure Storage Driver          │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Network: Azure Storage API        │
│  └── HTTPS to storage endpoints    │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Azure Storage Service             │
│  ├── Storage stamps (clusters)     │
│  ├── 3x replication                │
│  └── Distributed across nodes      │
└────────────────────────────────────┘
```

**Managed Disks Architecture**:

```
Managed Disk:
┌────────────────────────────────────┐
│  Disk Resource (Azure Resource)    │
│  - Name: WebServer1_OsDisk         │
│  - Size: 30 GB                     │
│  - Type: Premium_LRS               │
└────────────────┬───────────────────┘
                 ↓
         Backed by VHD blob
                 ↓
┌────────────────────────────────────┐
│  Azure Storage Account             │
│  (Microsoft-managed, hidden)       │
│  └── Blob Storage                  │
│      └── .vhd file (page blob)     │
└────────────────────────────────────┘
         Replicated 3x
                 ↓
    ┌────────┬────────┬────────┐
    │ Copy 1 │ Copy 2 │ Copy 3 │
    └────────┴────────┴────────┘
```

**Caching Layers**:

```
Read/Write Path with Caching:
VM Application
    ↓
OS Page Cache (in VM's RAM)
    ↓
Disk Cache (ReadWrite, ReadOnly, or None)
    ↓
Local SSD Cache (on physical host) ← If caching enabled
    ↓
Azure Storage Backend
```

### Networking Backend

**Azure Virtual Network (VNet) Implementation**:

```
Software-Defined Network (SDN):
┌────────────────────────────────────┐
│  VM Network Interface              │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Hyper-V Virtual Switch            │
│  - Virtual ports                   │
│  - NSG enforcement                 │
│  - Routing decisions               │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  VFP (Virtual Filtering Platform)  │
│  - Packet encapsulation            │
│  - VXLAN tunneling                 │
│  - Load balancing                  │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Physical NIC                      │
│  - 25/100 Gbps                     │
│  - Connects to ToR switch          │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Azure Network Fabric              │
│  - VXLAN overlay network           │
│  - Distributed routing             │
│  - Cross-datacenter connectivity   │
└────────────────────────────────────┘
```

**VXLAN Encapsulation**:

```
Your VM sends packet:
┌────────────────────────────────────┐
│  Original Packet                   │
│  Src IP: 10.0.1.5                  │
│  Dst IP: 10.0.2.10                 │
│  Payload: HTTP request             │
└────────────────────────────────────┘
         ↓ Encapsulated by VFP
┌────────────────────────────────────┐
│  Outer IP Header                   │
│  Src: Physical host A IP           │
│  Dst: Physical host B IP           │
│  ├── VXLAN Header                  │
│  │   VNI: 12345 (VNet identifier)  │
│  ├── Original Packet               │
│  │   Src IP: 10.0.1.5              │
│  │   Dst IP: 10.0.2.10             │
│  │   Payload: HTTP request         │
└────────────────────────────────────┘
         ↓ Sent over physical network
    Routed to destination host
         ↓ Decapsulated
┌────────────────────────────────────┐
│  Original Packet                   │
│  Src IP: 10.0.1.5                  │
│  Dst IP: 10.0.2.10                 │
│  Payload: HTTP request             │
└────────────────────────────────────┘
         ↓ Delivered to destination VM
```

### High Availability Mechanisms

#### Live Migration

Azure can move VMs between hosts without downtime:

```
Live Migration Process:
Time T0: VM running on Host A
┌──────────────┐              ┌──────────────┐
│   Host A     │              │   Host B     │
│  ┌────────┐  │              │              │
│  │  VM    │  │              │              │
│  │ Running│  │              │              │
│  └────────┘  │              │              │
└──────────────┘              └──────────────┘

Time T1: Memory copied to Host B
┌──────────────┐              ┌──────────────┐
│   Host A     │     copy     │   Host B     │
│  ┌────────┐  │ ─────────────→│  ┌────────┐  │
│  │  VM    │  │   memory     │  │ VM copy│  │
│  │ Running│  │              │  │(paused)│  │
│  └────────┘  │              │  └────────┘  │
└──────────────┘              └──────────────┘

Time T2: Final sync and cutover
┌──────────────┐              ┌──────────────┐
│   Host A     │    final     │   Host B     │
│  ┌────────┐  │ ─────────────→│  ┌────────┐  │
│  │ VM     │  │    delta     │  │  VM    │  │
│  │Pausing │  │              │  │ Running│  │
│  └────────┘  │              │  └────────┘  │
└──────────────┘              └──────────────┘

Time T3: Migration complete
┌──────────────┐              ┌──────────────┐
│   Host A     │              │   Host B     │
│              │              │  ┌────────┐  │
│   (empty)    │              │  │  VM    │  │
│              │              │  │ Running│  │
│              │              │  └────────┘  │
└──────────────┘              └──────────────┘

Downtime: Typically < 1 second
```

**When Live Migration Occurs**:

- Host maintenance (planned)
- Host failure (automated failover)
- Resource rebalancing
- Performance optimization

#### Healing and Self-Recovery

```
Failure Detection and Response:
┌────────────────────────────────────┐
│  1. Host Agent stops responding    │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  2. Fabric Controller detects      │
│     (via heartbeat timeout)        │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  3. FC determines host is down     │
│     (confirmed by multiple checks) │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  4. FC finds replacement host      │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  5. VMs restarted on new host      │
│     (boot from Azure Storage)      │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  6. VMs back online                │
│     (downtime: 2-5 minutes)        │
└────────────────────────────────────┘
```

-----

## Layer 5: Mapping to Compute Architecture

### From Application to Silicon

Let’s trace a simple program execution:

```
Layer 8: User Application
┌────────────────────────────────────┐
│  $ python app.py                   │
│  print("Hello Azure")              │
└────────────────┬───────────────────┘
                 ↓
Layer 7: Application Runtime
┌────────────────────────────────────┐
│  Python Interpreter                │
│  - Bytecode compilation            │
│  - Memory allocation request       │
└────────────────┬───────────────────┘
                 ↓
Layer 6: Operating System (Guest)
┌────────────────────────────────────┐
│  Linux Kernel (in VM)              │
│  - System call: write()            │
│  - Process scheduling              │
│  - Virtual memory management       │
└────────────────┬───────────────────┘
                 ↓
Layer 5: Virtual Hardware
┌────────────────────────────────────┐
│  Virtual CPU, RAM, Devices         │
│  - VM thinks it has physical HW    │
└────────────────┬───────────────────┘
                 ↓
Layer 4: Hypervisor (Hyper-V)
┌────────────────────────────────────┐
│  - Trap and translate guest ops    │
│  - Schedule vCPU on physical CPU   │
│  - Map guest RAM to host RAM       │
└────────────────┬───────────────────┘
                 ↓
Layer 3: Host Operating System
┌────────────────────────────────────┐
│  Windows Server + Hyper-V Role     │
│  - Hypervisor instructions         │
│  - Physical device drivers         │
└────────────────┬───────────────────┘
                 ↓
Layer 2: Hardware Abstraction
┌────────────────────────────────────┐
│  CPU Instructions                  │
│  - x86-64 instruction set          │
│  - Hardware virtualization (VT-x)  │
└────────────────┬───────────────────┘
                 ↓
Layer 1: Physical Hardware (Silicon)
┌────────────────────────────────────┐
│  Intel Xeon or AMD EPYC CPU        │
│  - Transistors switching           │
│  - Execution units processing      │
│  - Cache hierarchy (L1/L2/L3)      │
└────────────────────────────────────┘
```

### CPU Instruction Flow

**Example: Simple Addition**

```python
# Code in VM
result = 5 + 10
```

**What actually happens**:

```
1. Guest OS (Linux in VM):
   MOV RAX, 5      ; Load 5 into register RAX
   MOV RBX, 10     ; Load 10 into register RBX
   ADD RAX, RBX    ; Add RBX to RAX
   MOV [result], RAX ; Store result

2. Hypervisor:
   - Sees VM trying to execute instructions
   - With VT-x: Allows direct execution
   - CPU runs in "non-root mode" for guest
   - Certain privileged instructions trap to hypervisor

3. Physical CPU:
   - Decodes x86-64 instructions
   - Executes on actual ALU (Arithmetic Logic Unit)
   - Pipeline stages: Fetch → Decode → Execute → Write-back
   - Result written to physical register
   - Cache updated

4. Result:
   - CPU register now contains 15
   - Memory updated with result
   - Guest OS continues execution
```

### Memory Access Flow

**Example: Read from RAM**

```python
# Code in VM
data = memory_array[1000]
```

**Address Translation Journey**:

```
1. Application (in VM):
   Virtual Address: 0x00007FFF12345000
   
2. Guest OS Page Table:
   0x00007FFF12345000 → Guest Physical: 0x40001000
   (OS thinks this is "physical" memory)

3. Hypervisor EPT (Extended Page Tables):
   Guest Physical: 0x40001000 → Host Physical: 0x8000A5000
   (Actual location in physical RAM)

4. Physical RAM:
   Read from DIMM address 0x8000A5000
   Data travels through:
   - Memory controller
   - CPU cache (L3 → L2 → L1)
   - CPU register

5. Return path:
   Value returned to application
```

**Address Translation Diagram**:

```
Virtual Addr    Guest Physical    Host Physical
(Process View)  (VM View)         (Real RAM)
     │               │                 │
     │  Guest        │                 │
     │  Page Table   │                 │
     │      ↓        │                 │
     └──────────────→│                 │
                     │  EPT            │
                     │  Translation    │
                     │      ↓          │
                     └─────────────────→│
                                       │
                                    Physical
                                     DRAM
```

### I/O Operations

**Example: Write to Disk**

```python
# Code in VM
file.write("Hello Azure")
```

**I/O Path**:

```
1. Application (in VM):
   write() system call

2. Guest OS (Linux):
   - VFS layer
   - ext4 filesystem
   - Block layer
   - SCSI subsystem

3. Virtual SCSI Controller:
   - VM sees "/dev/sda"
   - Sends SCSI commands

4. Hyper-V Hypervisor:
   - Intercepts virtual SCSI commands
   - Translates to Azure Storage API calls

5. Azure Storage Driver:
   - Forms HTTPS PUT request
   - Targets VHD blob endpoint

6. Network Path:
   - Through vSwitch
   - Encapsulated in VXLAN
   - Sent to Azure Storage service

7. Azure Storage Service:
   - Receives blob update
   - Writes to storage stamp
   - Replicates 3x
   - Returns acknowledgment

8. Return Path:
   - ACK back to VM
   - SCSI command complete
   - Guest OS updates cache
   - Application continues
```

-----

## Layer 6: VM Lifecycle & Operations

### Complete VM Creation Flow

Let’s trace what happens when you create a VM:

```
Timeline: 0 to ~90 seconds

T+0s: Portal Button Click
User: Clicks "Create" button
Action: Browser sends HTTPS POST to Azure ARM API
Data: JSON ARM template with VM configuration

T+0.1s: ARM API Gateway
Action: Authentication via Azure AD
Check: Does user have permissions?
Validation: Is JSON syntax correct?
Result: Request accepted, assigned correlation ID

T+0.5s: Resource Provider (Microsoft.Compute)
Action: Compute RP validates VM configuration
Checks:
- Is VM size available in region?
- Is image available?
- Are quotas sufficient?
- Is resource group valid?
Result: Validation passed

T+1s: ARM Orchestration
Action: Determines resource dependencies
Creates:
1. Storage account (if needed for boot diagnostics)
2. Network interface
3. Public IP (if requested)
4. Virtual machine resource

T+2s: Fabric Controller Selection
Action: FC finds suitable physical host
Algorithm:
- Check available capacity in region/zone
- Find host with enough CPU cores
- Find host with enough RAM
- Avoid overutilized hosts
- Consider VM family requirements (e.g., F-series needs high CPU)
Result: Host selected: Rack 7, Server 23

T+3s: Host Agent Contacted
Fabric Controller → Host Agent on selected server
Command: "Provision VM with these specs"
Payload: VM configuration JSON

T+5s: VM Provisioning Begins
Host Agent actions:
1. Reserve CPU cores: 2 vCPUs
2. Reserve RAM: 8 GB
3. Create Hyper-V VM partition
4. Configure virtual hardware:
   - Virtual CPU
   - Virtual RAM
   - Virtual NIC (MAC address assigned)
   - Virtual SCSI controller

T+10s: Storage Attachment
Actions:
1. Connect to Azure Storage endpoint
2. Attach OS disk VHD blob
3. Attach data disks (if any)
4. Configure caching policy (ReadWrite for OS disk)

T+15s: Network Configuration
Actions:
1. Create vNIC in VM
2. Assign private IP (10.0.1.5)
3. Configure vSwitch port
4. Program NSG rules into vSwitch
5. Set up VXLAN tunnel for VNet

T+20s: VM Boot - BIOS/UEFI
Action: Hyper-V starts VM
Process:
1. Virtual BIOS/UEFI initializes
2. POST (Power-On Self Test)
3. Detect virtual hardware devices
4. Boot from virtual disk

T+25s: Boot Loader
Action: GRUB (Linux) or Windows Boot Manager loads
Process:
1. Read MBR/GPT
2. Load boot configuration
3. Load kernel

T+30s: Operating System Kernel Loads
Linux kernel initialization:
1. Kernel decompresses itself
2. Hardware detection (virtual hardware)
3. Mount root filesystem
4. Start init system (systemd)

T+45s: OS Services Start
Actions:
1. Network services
2. SSH daemon (sshd)
3. Cloud-init (Azure Linux Agent)
4. System logging

T+60s: Azure Linux Agent (waagent)
Actions:
1. Detects running in Azure
2. Configures networking
3. Enables extensions
4. Reports to Azure Fabric

T+70s: Cloud-init (Custom Data)
If you provided custom data:
1. Runs initialization scripts
2. Installs packages
3. Configures services

T+75s: VM Extensions
If extensions specified:
1. Download extension packages
2. Install and configure
3. Run extension scripts

T+80s: Health Check
Fabric Controller checks:
- Is VM responding?
- Is OS booted?
- Is agent reporting?
Result: VM healthy

T+90s: Portal Update
Azure Portal shows:
Status: Running ✓
Public IP: 20.50.60.70
Private IP: 10.0.1.5
```

### VM Resize Operation

```
User: Changes VM size from D2s_v3 to D4s_v3
(2 vCPUs, 8 GB → 4 vCPUs, 16 GB)

Process:
1. User initiates resize in portal
2. ARM validates new size
3. Check: Can current host support new size?
   
   If YES (hot resize - rare):
   - Resize without migration
   - Downtime: ~2-3 seconds
   
   If NO (cold resize - common):
   - Find new host with capacity
   - Deallocate VM
   - Migrate to new host
   - Allocate new resources
   - Boot VM with new size
   - Downtime: 2-5 minutes

4. VM restarts with new configuration
5. OS sees more CPUs/RAM
6. Applications can now use additional resources
```

### Snapshot Creation

```
User: Creates snapshot of OS disk

Process:
T+0s: User clicks "Create snapshot"

T+1s: Azure Storage receives request
Action: Initiate blob snapshot

T+2s: Storage creates point-in-time copy
Method: Copy-on-write (COW)
- Original blocks marked read-only
- New writes go to new blocks
- Snapshot references original blocks
- Instant snapshot (no full copy needed)

T+3s: Snapshot available
- Appears as separate disk resource
- Can be used to create new VM
- Can be copied to another region
- Billed for actual data size

Time: ~5 seconds
```

### VM Backup

```
Azure Backup Process:

T+0s: Scheduled backup job triggered

T+5s: Backup extension invoked
Extension: Microsoft.Azure.RecoveryServices.VMSnapshot

T+10s: Application-consistent snapshot
Linux:
1. Run pre-freeze script
2. Call filesystem sync
3. Take snapshot
4. Run post-thaw script

Windows:
1. VSS (Volume Shadow Copy) quiesce
2. Application writers notified
3. Take snapshot
4. Resume applications

T+15s: Snapshot created
- All disks snapshotted simultaneously
- Crash-consistent or app-consistent

T+20s: Transfer to vault begins
- Snapshot copied to Recovery Services vault
- Incremental copy (only changed blocks)
- Compression applied
- Encryption applied

T+30min-2hr: Transfer complete
- Backup stored in vault
- Retention policy applied
- Available for restore

Frequency: Daily (typical)
Retention: 7-30 days (configurable)
```

-----

## Layer 7: Physical Implementation

### What’s in an Azure Datacenter

#### Datacenter Building

```
Azure Datacenter Facility:
├── Physical Security
│   ├── Perimeter fence
│   ├── Security checkpoints
│   ├── Biometric access
│   └── 24/7 surveillance
├── Power Systems
│   ├── Multiple utility feeds
│   ├── Diesel generators
│   ├── UPS (Uninterruptible Power Supply)
│   ├── Battery rooms
│   └── N+1 redundancy
├── Cooling Systems
│   ├── CRAC (Computer Room Air Conditioning)
│   ├── Chillers
│   ├── Hot aisle / Cold aisle containment
│   ├── Free cooling (outside air)
│   └── Liquid cooling for high-density racks
├── Network Infrastructure
│   ├── Redundant fiber paths
│   ├── Multiple ISP connections
│   ├── Internal spine-leaf topology
│   └── Dark fiber between DCs
└── Server Rooms
    ├── Raised floors
    ├── Fire suppression (gas, not water)
    ├── Temperature monitoring
    └── Server racks
```

#### The Physical Server Rack

```
42U Server Rack (Standard):
┌─────────────────────────────┐
│ Top-of-Rack (ToR) Switch    │ ← 1U (25/100 Gbps)
├─────────────────────────────┤
│ Patch Panel                 │ ← 1U
├─────────────────────────────┤
│ Server 1 (Hyper-V Host)     │ ← 1U (Your VM might be here)
├─────────────────────────────┤
│ Server 2 (Hyper-V Host)     │ ← 1U
├─────────────────────────────┤
│ Server 3 (Hyper-V Host)     │ ← 1U
├─────────────────────────────┤
│ ... (38 more servers)       │
├─────────────────────────────┤
│ Server 40 (Hyper-V Host)    │ ← 1U
├─────────────────────────────┤
│ PDU (Power Distribution)    │ ← Rear-mounted
└─────────────────────────────┘

Per Rack:
- 40 servers (typical)
- Power: 15-25 kW
- Network: 1-4 Tbps aggregated
```

### Inside a Physical Server

```
1U Server Chassis (e.g., Dell PowerEdge, HP ProLiant):
┌────────────────────────────────────────────────┐
│  Front Panel:                                  │
│  ├── Power button                              │
│  ├── Status LEDs                               │
│  └── iDRAC/iLO port (management)               │
└────────────────────────────────────────────────┘

Internal Components:
┌────────────────────────────────────────────────┐
│  Motherboard                                   │
│  ├── CPU Sockets: 2x Intel Xeon Platinum 8370C│
│  │   ├── Cores: 32 per CPU (64 total)         │
│  │   ├── Threads: 64 per CPU (128 total)      │
│  │   ├── Base Clock: 2.8 GHz                  │
│  │   ├── Turbo: 3.5 GHz                       │
│  │   ├── L3 Cache: 48 MB per CPU              │
│  │   └── TDP: 270W per CPU                    │
│  ├── RAM Slots: 24 DIMM slots                 │
│  │   ├── Type: DDR4 ECC RDIMM                 │
│  │   ├── Speed: 3200 MHz                      │
│  │   ├── Capacity: 16x 32GB = 512 GB          │
│  │   └── Bandwidth: 204.8 GB/s                │
│  ├── Storage Controllers                      │
│  │   ├── RAID controller (optional)           │
│  │   └── NVMe controller (PCIe Gen4)          │
│  ├── Network Interfaces                       │
│  │   ├── Primary NIC: Mellanox ConnectX-6     │
│  │   │   └── 100 Gbps Ethernet (QSFP28)       │
│  │   └── Secondary NIC: 25 Gbps (redundancy)  │
│  ├── PCIe Slots                               │
│  │   ├── Slot 1: Network adapter              │
│  │   ├── Slot 2: GPU (if N-series)            │
│  │   └── Slot 3: NVMe expansion               │
│  ├── Local Storage                            │
│  │   ├── Boot: 2x 480GB SATA SSD (RAID 1)     │
│  │   └── Temp: 2x 1.92TB NVMe SSD             │
│  └── Power Supplies                           │
│      ├── PSU 1: 1600W (redundant)             │
│      └── PSU 2: 1600W (redundant)             │
└────────────────────────────────────────────────┘

Total Server Specs:
- 64 physical cores (128 threads)
- 512 GB RAM
- 100 Gbps network
- ~3.8 TB local NVMe storage
- Power: 800-1200W under load
- Cost: $30,000-50,000 per server
```

### Your VM on the Physical Server

```
Server Capacity Example:
Physical Server: 64 cores, 512 GB RAM

VM Allocation (example mix):
├── VM 1 (Your WebServer1): 2 vCPUs, 8 GB   ← Your VM is here
├── VM 2 (Database):        8 vCPUs, 32 GB
├── VM 3 (App Server):      4 vCPUs, 16 GB
├── VM 4 (Worker):          2 vCPUs, 4 GB
├── VM 5 (Cache):           4 vCPUs, 16 GB
├── VM 6 (API):             2 vCPUs, 8 GB
├── VM 7 (Queue):           2 vCPUs, 8 GB
├── VM 8-15:                Various sizes
└── Reserved:               For hypervisor and overhead

Total Allocated: 48 vCPUs, 384 GB RAM
Remaining: 16 cores, 128 GB RAM

Note: Azure maintains buffer capacity for:
- Host OS and hypervisor
- Live migration operations
- Performance guarantees
```

### Physical Network Path

When your VM communicates:

```
VM Network Packet Journey:
┌────────────────────────────────────┐
│  Your VM (WebServer1)              │
│  IP: 10.0.1.5                      │
│  Sends: HTTP response to client    │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Virtual NIC (in hypervisor)       │
│  MAC: 00:0D:3A:12:34:56            │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Hyper-V Virtual Switch            │
│  - NSG rule check                  │
│  - VXLAN encapsulation             │
│  VNI: 54321 (your VNet)            │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Physical NIC (Mellanox)           │
│  - 100 Gbps Ethernet               │
│  - Packet sent on wire             │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Top-of-Rack (ToR) Switch          │
│  - Layer 2/3 switching             │
│  - Routes to spine network         │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Spine Switch (Aggregation)        │
│  - High-speed backplane            │
│  - Cross-datacenter routing        │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Edge Router                       │
│  - NAT (if public IP)              │
│  - Firewall                        │
│  - DDoS protection                 │
└────────────────┬───────────────────┘
                 ↓
        Internet
```

### Storage Physical Path

```
Disk Write Physical Journey:
┌────────────────────────────────────┐
│  Your VM writes to disk            │
│  write(fd, buffer, size)           │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Virtual SCSI Controller           │
│  - SCSI command created            │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Hyper-V Host Storage Stack        │
│  - Translates to Azure Storage API │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Network (HTTPS REST API)          │
│  PUT /vhds/disk.vhd?blockid=123    │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Azure Storage Service             │
│  - Receives blob update            │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Storage Stamp (Cluster)           │
│  ├── Primary replica                │
│  ├── Secondary replica (same DC)    │
│  └── Tertiary replica (same DC)     │
│  All on separate physical servers  │
└────────────────┬───────────────────┘
                 ↓
┌────────────────────────────────────┐
│  Physical SSD Drives               │
│  - NVMe SSDs in JBOD enclosures    │
│  - Data written to flash memory    │
└────────────────────────────────────┘
```

-----

## Layer 8: Practical Examples

### Example 1: Basic Web Server VM

**Scenario**: Deploy a simple web server

**Requirements**:

- Ubuntu 22.04 LTS
- 2 vCPUs, 8 GB RAM
- Premium SSD
- Public IP for internet access
- Open ports 80, 443, 22

**ARM Template** (simplified):

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminUsername": {
      "type": "string",
      "defaultValue": "azureuser"
    },
    "sshPublicKey": {
      "type": "securestring"
    }
  },
  "variables": {
    "vmName": "WebServer1",
    "vmSize": "Standard_D2s_v3",
    "vnetName": "MyVNet",
    "subnetName": "default",
    "nicName": "WebServer1-nic",
    "publicIPName": "WebServer1-ip",
    "nsgName": "WebServer1-nsg"
  },
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-02-01",
      "name": "[variables('publicIPName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publicIPAllocationMethod": "Static",
        "dnsSettings": {
          "domainNameLabel": "[toLower(variables('vmName'))]"
        }
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-02-01",
      "name": "[variables('nsgName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": [
          {
            "name": "Allow-HTTP",
            "properties": {
              "priority": 100,
              "direction": "Inbound",
              "access": "Allow",
              "protocol": "Tcp",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "80"
            }
          },
          {
            "name": "Allow-HTTPS",
            "properties": {
              "priority": 110,
              "direction": "Inbound",
              "access": "Allow",
              "protocol": "Tcp",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "443"
            }
          },
          {
            "name": "Allow-SSH",
            "properties": {
              "priority": 120,
              "direction": "Inbound",
              "access": "Allow",
              "protocol": "Tcp",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "22"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-02-01",
      "name": "[variables('nicName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
              },
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
              }
            }
          }
        ],
        "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
        },
        "enableAcceleratedNetworking": true
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[variables('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[variables('vmSize')]"
        },
        "osProfile": {
          "computerName": "[variables('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "linuxConfiguration": {
            "disablePasswordAuthentication": true,
            "ssh": {
              "publicKeys": [
                {
                  "path": "[concat('/home/', parameters('adminUsername'), '/.ssh/authorized_keys')]",
                  "keyData": "[parameters('sshPublicKey')]"
                }
              ]
            }
          }
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "0001-com-ubuntu-server-jammy",
            "sku": "22_04-lts-gen2",
            "version": "latest"
          },
          "osDisk": {
            "name": "[concat(variables('vmName'), '_OsDisk')]",
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            },
            "diskSizeGB": 30
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "apiVersion": "2021-03-01",
      "name": "[concat(variables('vmName'), '/CustomScript')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines', variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.1",
        "autoUpgradeMinorVersion": true,
        "settings": {},
        "protectedSettings": {
          "commandToExecute": "apt-get update && apt-get install -y nginx && systemctl start nginx"
        }
      }
    }
  ],
  "outputs": {
    "publicIP": {
      "type": "string",
      "value": "[reference(variables('publicIPName')).ipAddress]"
    },
    "fqdn": {
      "type": "string",
      "value": "[reference(variables('publicIPName')).dnsSettings.fqdn]"
    }
  }
}
```

**What happens when deployed**:

```
1. Public IP created: 20.50.60.70
2. NSG created with rules (HTTP, HTTPS, SSH)
3. NIC created with public IP and NSG
4. Fabric Controller finds suitable host
5. Hyper-V creates VM partition
6. Ubuntu 22.04 image deployed
7. VM boots
8. CustomScript extension runs:
   - Updates packages
   - Installs nginx
   - Starts nginx service
9. VM ready: http://20.50.60.70
```

**Cost Analysis** (West Europe):

```
VM (D2s_v3): $0.112/hour
- 2 vCPUs, 8 GB RAM
- ~$81.50/month (730 hours)

OS Disk (Premium 30 GB): $4.97/month

Public IP (Static): $3.65/month

Egress (outbound data): $0.087/GB
- First 5 GB free
- Example: 100 GB = $8.70/month

Total monthly estimate: ~$99/month
```

### Example 2: High-Availability Database Cluster

**Scenario**: SQL Server Always On with 2 nodes

**Architecture**:

```
Availability Set:
├── VM 1: SQL-Node1
│   ├── Size: E4s_v3 (4 vCPU, 32 GB RAM)
│   ├── OS Disk: Premium SSD 128 GB
│   ├── Data Disk 1: Premium SSD 1 TB (Database files)
│   ├── Data Disk 2: Premium SSD 512 GB (Log files)
│   ├── Data Disk 3: Premium SSD 256 GB (TempDB)
│   └── Fault Domain: 0, Update Domain: 0
└── VM 2: SQL-Node2
    ├── Size: E4s_v3 (4 vCPU, 32 GB RAM)
    ├── OS Disk: Premium SSD 128 GB
    ├── Data Disk 1: Premium SSD 1 TB (Database files)
    ├── Data Disk 2: Premium SSD 512 GB (Log files)
    ├── Data Disk 3: Premium SSD 256 GB (TempDB)
    └── Fault Domain: 1, Update Domain: 1

Internal Load Balancer:
├── Frontend IP: 10.0.2.100
└── Backend Pool: SQL-Node1, SQL-Node2

Witness:
└── File Share Witness (Azure Files)
```

**Physical Distribution**:

```
Datacenter:
├── Rack 15 (Fault Domain 0)
│   └── Physical Server 8
│       └── SQL-Node1 ← Different physical hardware
├── Rack 22 (Fault Domain 1)
│   └── Physical Server 14
│       └── SQL-Node2 ← Different physical hardware

Benefits:
- Different power sources
- Different network switches
- Survives single rack failure
- Survives single server failure
```

**Failure Scenarios**:

```
Scenario 1: SQL-Node1 server hardware fails
Action:
- Fabric Controller detects failure (30 seconds)
- SQL-Node2 becomes primary (10 seconds)
- Load balancer redirects traffic (5 seconds)
- SQL-Node1 restarted on different host (3 minutes)
- Rejoins cluster as secondary
Downtime: ~45 seconds

Scenario 2: Planned maintenance on Rack 15
Action:
- Azure notifies 30 days in advance
- Live migration scheduled
- SQL-Node1 migrated to different host
- Zero downtime
Downtime: 0 seconds

Scenario 3: Entire rack power failure
Action:
- SQL-Node2 (Rack 22) continues serving
- SQL-Node1 (Rack 15) offline
- Load balancer uses only SQL-Node2
- Degraded redundancy until Rack 15 restored
Downtime: 0 seconds (SQL-Node2 serving)
```

### Example 3: Auto-Scaling Web Farm

**Scenario**: Website with variable traffic using VMSS

**Configuration**:

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "WebFarm-VMSS",
  "location": "westeurope",
  "sku": {
    "name": "Standard_D2s_v3",
    "tier": "Standard",
    "capacity": 2
  },
  "properties": {
    "overprovision": true,
    "upgradePolicy": {
      "mode": "Rolling",
      "rollingUpgradePolicy": {
        "maxBatchInstancePercent": 20,
        "maxUnhealthyInstancePercent": 20,
        "maxUnhealthyUpgradedInstancePercent": 20,
        "pauseTimeBetweenBatches": "PT5S"
      }
    },
    "automaticRepairsPolicy": {
      "enabled": true,
      "gracePeriod": "PT30M"
    },
    "virtualMachineProfile": {
      "storageProfile": {
        "imageReference": {
          "publisher": "Canonical",
          "offer": "0001-com-ubuntu-server-jammy",
          "sku": "22_04-lts-gen2",
          "version": "latest"
        }
      },
      "osProfile": {
        "computerNamePrefix": "webvm",
        "adminUsername": "azureuser"
      },
      "networkProfile": {
        "networkInterfaceConfigurations": [
          {
            "name": "nic-config",
            "properties": {
              "primary": true,
              "enableAcceleratedNetworking": true,
              "ipConfigurations": [
                {
                  "name": "ipconfig",
                  "properties": {
                    "subnet": {
                      "id": "/subscriptions/.../subnets/web-subnet"
                    },
                    "loadBalancerBackendAddressPools": [
                      {
                        "id": "/subscriptions/.../backendAddressPools/web-backend"
                      }
                    ]
                  }
                }
              ]
            }
          }
        ]
      },
      "extensionProfile": {
        "extensions": [
          {
            "name": "CustomScript",
            "properties": {
              "publisher": "Microsoft.Azure.Extensions",
              "type": "CustomScript",
              "typeHandlerVersion": "2.1",
              "settings": {},
              "protectedSettings": {
                "commandToExecute": "apt-get update && apt-get install -y nginx && systemctl start nginx"
              }
            }
          }
        ]
      }
    }
  }
}
```

**Auto-Scale Rules**:

```json
{
  "type": "Microsoft.Insights/autoscaleSettings",
  "name": "WebFarm-AutoScale",
  "properties": {
    "profiles": [
      {
        "name": "Default",
        "capacity": {
          "minimum": "2",
          "maximum": "10",
          "default": "2"
        },
        "rules": [
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceId": "/subscriptions/.../virtualMachineScaleSets/WebFarm-VMSS",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT5M",
              "timeAggregation": "Average",
              "operator": "GreaterThan",
              "threshold": 70
            },
            "scaleAction": {
              "direction": "Increase",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          },
          {
            "metricTrigger": {
              "metricName": "Percentage CPU",
              "metricResourceId": "/subscriptions/.../virtualMachineScaleSets/WebFarm-VMSS",
              "timeGrain": "PT1M",
              "statistic": "Average",
              "timeWindow": "PT5M",
              "timeAggregation": "Average",
              "operator": "LessThan",
              "threshold": 30
            },
            "scaleAction": {
              "direction": "Decrease",
              "type": "ChangeCount",
              "value": "1",
              "cooldown": "PT5M"
            }
          }
        ]
      }
    ]
  }
}
```

**Scaling in Action**:

```
Time: 09:00 - Low traffic
Instances: 2 VMs
CPU: 20% average
Action: None (at minimum)

Time: 12:00 - Traffic increases
CPU: 75% average for 5 minutes
Action: Scale out (add 1 VM)
Process:
1. Auto-scale service triggers
2. VMSS requests new VM from Fabric Controller
3. Fabric Controller provisions VM (60 seconds)
4. VM boots and runs CustomScript (30 seconds)
5. Load balancer health probe passes (15 seconds)
6. New VM added to backend pool
7. Traffic distributed across 3 VMs
Total time: ~2 minutes

Instances: 3 VMs
CPU: 50% average

Time: 12:30 - Even more traffic
CPU: 72% average for 5 minutes
Action: Scale out (add 1 VM)
Instances: 4 VMs
CPU: 55% average

Time: 18:00 - Traffic decreases
CPU: 25% average for 5 minutes
Action: Scale in (remove 1 VM)
Process:
1. Auto-scale service triggers
2. VMSS selects VM to remove (newest)
3. Load balancer drains connections (30 seconds)
4. VM deallocated
5. Resources returned to pool
Total time: ~1 minute

Instances: 3 VMs
CPU: 32% average

Time: 20:00 - Very low traffic
CPU: 20% average for 5 minutes
Action: Scale in (remove 1 VM)
Instances: 2 VMs (at minimum)
```

**Physical Distribution Across Hosts**:

```
Before scaling (2 VMs):
├── Rack 5, Server 12: webvm_0
└── Rack 11, Server 8: webvm_1

After scaling to 4 VMs:
├── Rack 5, Server 12: webvm_0
├── Rack 11, Server 8: webvm_1
├── Rack 18, Server 3: webvm_2
└── Rack 24, Server 15: webvm_3

VMSS automatically distributes across:
- Different racks (fault domains)
- Different update domains
- Different physical servers
Result: High availability and fault tolerance
```

### Example 4: GPU-Accelerated ML Training

**Scenario**: Train machine learning models with GPU acceleration

**VM Configuration**:

```
VM Size: Standard_NC6s_v3
├── vCPUs: 6
├── RAM: 112 GB
├── GPU: 1x NVIDIA Tesla V100 (16 GB HBM2)
├── Temp Storage: 736 GB NVMe SSD
├── Network: 24 Gbps
└── Cost: ~$3.06/hour (~$2,234/month)
```

**Physical Reality**:

```
Physical Server for GPU VMs:
┌────────────────────────────────────┐
│  Chassis: 2U (double height)       │
│  ├── CPUs: 2x Intel Xeon Gold      │
│  ├── RAM: 768 GB DDR4              │
│  ├── GPUs: 4x NVIDIA Tesla V100    │
│  │   └── PCIe Gen3 x16 per GPU     │
│  └── Power: 2x 2000W PSUs          │
└────────────────────────────────────┘

Your VM gets:
- 1 GPU passed through to VM (PCIe passthrough)
- Direct access to GPU hardware
- NVIDIA drivers in VM
- CUDA/cuDNN libraries
```

**Training Workload**:

```python
# In the VM
import tensorflow as tf

# TensorFlow detects GPU
print(tf.config.list_physical_devices('GPU'))
# Output: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

# Training uses GPU
model = tf.keras.Sequential([...])
model.compile(...)

# Training data
train_dataset = ...

# This runs on the Tesla V100
model.fit(train_dataset, epochs=100)
```

**What happens physically**:

```
1. Python code in VM calls TensorFlow
2. TensorFlow uses CUDA library
3. CUDA sends commands to GPU driver
4. GPU driver communicates with physical Tesla V100
5. GPU executes matrix operations
   - Tensor cores (mixed precision)
   - 7 TFLOPS (FP64)
   - 14 TFLOPS (FP32)
   - 112 TFLOPS (FP16 with Tensor Cores)
6. Results returned to CPU/RAM
7. Training loop continues
```

-----

## Summary: The Complete Picture

### What You Now Understand About Azure VMs

**Top Layer (Portal)**:

- Simple interface: pick size, image, region
- Hides massive complexity
- One click = orchestrating entire datacenter infrastructure

**Middle Layer (Components)**:

- VMs are collections of resources (compute, storage, network)
- Complex dependency chain
- ARM manages everything

**Bottom Layer (Technology)**:

- Hyper-V Type 1 hypervisor
- Hardware-assisted virtualization (VT-x/AMD-V)
- Distributed fabric architecture
- Physical servers in datacenters

**Physical Reality**:

- Your VM runs on a specific physical server
- In a specific rack
- In a specific datacenter
- Sharing hardware with other customers (but isolated)

### Key Insights

**Azure VM is NOT**:

- ❌ A physical server you own
- ❌ Running in a single location
- ❌ Guaranteed to stay on the same host
- ❌ Visible to other VMs on same host
- ❌ Your only VM on that hardware

**Azure VM IS**:

- ✅ Virtualized compute partition
- ✅ Isolated from other customers
- ✅ Abstraction over physical hardware
- ✅ Highly available (with right configuration)
- ✅ Scalable and flexible
- ✅ Backed by enterprise-grade infrastructure

### From Click to Silicon: Complete Flow

```
1. Portal Click
   ↓
2. ARM API call
   ↓
3. Resource Provider validation
   ↓
4. Fabric Controller orchestration
   ↓
5. Physical host selection
   ↓
6. Hyper-V VM provisioning
   ↓
7. Virtual hardware configuration
   ↓
8. Storage attachment (Azure Storage)
   ↓
9. Network configuration (SDN)
   ↓
10. OS boot sequence
    ↓
11. Guest agent initialization
    ↓
12. Extensions execution
    ↓
13. VM ready
    ↓
14. Your application runs
    ↓
15. CPU instructions execute on physical Intel/AMD processors
    ↓
16. Memory accesses hit physical DRAM
    ↓
17. Network packets traverse Azure fabric
    ↓
18. Storage I/O goes to Azure Storage stamps
```

### How This Helps Your Technical Assessment

**When reviewing Azure architectures, you can now**:

1. Evaluate VM sizing decisions
- Understand CPU/memory/storage requirements
- Recommend appropriate VM families
1. Design for high availability
- Use Availability Sets/Zones correctly
- Understand fault/update domains
1. Optimize costs
- Right-size VMs
- Use appropriate disk types
- Leverage auto-scaling
1. Troubleshoot performance
- Understand noisy neighbor impact
- Identify virtualization overhead
- Optimize disk caching
1. Design secure architectures
- Understand isolation boundaries
- Plan network segmentation
- Implement defense in depth

**Questions you can answer**:

- “Why did you choose this VM size?”
- “How does this scale?”
- “What happens if this host fails?”
- “Why are my disk IOPS limited?”
- “How is this VM isolated from others?”
- “What’s the impact of deallocating vs stopping?”

### Connection to Other Azure Services

**VM concepts apply to**:

- App Service (runs on VMs behind the scenes)
- Azure Kubernetes Service (VMs as nodes)
- Azure Batch (VM pools for batch computing)
- Azure Virtual Desktop (VMs for remote desktops)
- Azure Functions (serverless, but VMs underneath)

**Everything in Azure ultimately runs on VMs or similar compute**

-----

## Next Steps

Would you like me to reverse engineer another Azure service?

**Good candidates**:

1. **Azure Load Balancer** - How traffic distribution really works
1. **Azure Application Gateway** - Layer 7 load balancing and WAF
1. **Azure Storage** - Blob, disk, and file storage architecture
1. **Azure Kubernetes Service** - Container orchestration on VMs
1. **Azure SQL Database** - Managed database under the hood

Each breakdown will follow the same pattern:

- What you see in portal
- Components and structure
- How it works
- Underlying technology
- Physical implementation
- Practical examples

This approach gives you complete understanding from UI to silicon!
