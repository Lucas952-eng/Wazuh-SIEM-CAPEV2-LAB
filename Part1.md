# Part 1--Wazuh Setup in the Proxmox Environment

## 1.1 Create the internal lab network

1. In Proxmox, create a Linux bridge for the lab: **Datacenter → Node → System → Network → Create → Linux Bridge**.
 - Name: `vmbr0`
 - IPv4/CIDR: `10.10.10.1/24` (the Proxmox host is the lab gateway)
 - Bridge ports: `none` (internal-only, not attached to any physical NIC)
2. Give the lab internet access through the host with NAT. The resulting block in `/etc/network/interfaces`:
<img width="860" height="278" alt="image" src="https://github.com/user-attachments/assets/cf3ebb6a-8313-4007-a9ad-2a816d489651" />

- `bridge-ports none` keeps the lab isolated at Layer 2 from the home LAN (`192.168.1.0/24`).
- `ip_forward` + `MASQUERADE` let lab VMs reach the internet through the host's uplink (`wlp4s0` here;replace with your interface).

  1. Every lab VM/LXC gets its NIC on `vmbr0` with gateway `10.10.10.1`.
  2. Verify on the host: `ip -br a show vmbr0` shows `10.10.10.1/24`.

## 1.2 Creation/Specs of Wazuh VM

|Hardware/Software|Specs|
|---|---|
|OS|Ubuntu 24.04 Desktop ISO|
|CPU| `Proxmox` host CPU|
|RAM|**8 GB** minimum|
|Disk|40-80 GB (SSD-Reccomended)|
|Network|`vmbr0` bridge, VirtIO|
|Static IP| `10.10.10.9/24`|
     
