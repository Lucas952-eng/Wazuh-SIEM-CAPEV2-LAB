# Part 1--Wazuh Setup in the Proxmox Environment

## 1.1 Create the internal lab network

1. In Proxmox, create a Linux bridge for the lab: **Datacenter → Node → System → Network → Create → Linux Bridge**.
 - Name: `vmbr0`
 - IPv4/CIDR: `10.10.10.1/24` (the Proxmox host is the lab gateway)
 - Bridge ports: `none` (internal-only, not attached to any physical NIC)
2. Give the lab internet access through the host with NAT. The resulting block in `/etc/network/interfaces`:

