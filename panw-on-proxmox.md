# Palo Alto VM Series & Panorama Proxmox Installation

Special thanks to Zach Forsyth

## Order of operations

1. SSH into your Proxmox server

```bash
ssh root@pve1
```

2. Download QCOW2 image from PANW Support Center

2.1. Panorama Virtual Appliance
```bash
wget https://download.somewhere.com/Panorama-KVM-11.1.2.qcow2
```

2.2. VM-Series Virtual Appliance
```bash
wget https://download.somewhere.com/PA-VM-KVM-11.1.0.qcow2
```

3. Create the VM

Make sure to update accordingly with your environment.

3.1. Panorama VM
Note: Panorama virtual appliance supports only one network interface

```bash
qm create 104 \
--name panorama \
--agent enabled=1 \
--machine q35 \
--bios seabios \
--numa 0 \
--cpu host \
--ostype l26 \
--cores 2 \
--sockets 2 \
--memory 8192  \
--serial0 socket \
--scsihw virtio-scsi-pci \
--boot order='virtio0' \
--hotplug disk,network,usb,cpu \
--net0 virtio,bridge=vmbr0 \
```

3.2. VM-Series
Note: Add as many interfaces as you need

```bash
qm create 101 \
--name palo-fw-vm-100 \
--agent enabled=1 \
--machine q35 \
--bios seabios \
--numa 0 \
--cpu host \
--ostype l26 \
--cores 2 \
--sockets 1 \
--memory 6144  \
--serial0 socket \
--scsihw virtio-scsi-pci \
--boot order='virtio0' \
--hotplug disk,network,usb,cpu \
--net0 virtio,bridge=vmbr0 \
--net1 virtio,bridge=vmbr0.66 \
--net2 virtio,bridge=vmbr0 \
--net3 virtio,bridge=vmbr0
```

4. Import the VM-Series disk

Update your storage path accordingly, in this example I am using `local-lvm`

4.1. Panorama VM
```bash
qm importdisk 104 Panorama-KVM-11.1.2.qcow2 local-lvm --format qcow2
```

4.2. VM-Series
```bash
qm importdisk 101 PA-VM-KVM-11.1.0.qcow2 local-lvm --format qcow2
```

5. Attach the disk to your VM

5.1. Panorama VM
```bash
qm set 104 --virtio0 local-lvm:vm-104-disk-0,discard=on,cache=writeback
```

5.2. VM-Series
```bash
qm set 101 --virtio0 local-lvm:vm-101-disk-0,discard=on,cache=writeback
```

6. Start your VM

6.1. Panorama VM
```bash
qm start 104
```

6.2. VM-Series
```bash
qm start 101
```