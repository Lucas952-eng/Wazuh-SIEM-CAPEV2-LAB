# Wazuh-SIEM-CAPEV2-LAB
This is a blue-team detection lab: a Wazuh SIEM on Proxmox, connected over Tailscale to a CAPEV2 malware sandbox that detonates email attachments and reports verdict back into Wazuh

# Lab Components

## Proxmox Host

 - This host is my home server that I have setup. Used in my home LAN and its role is a hypervisor. Contains all lxc containers, vms, and more.
 - Inside home router private ip address 192.168.0/24.
 - Everything in this lab, except for the CAPEV2 agent is within this.

## Tailscale Subnet Router
 - LXC on Proxmox Server.
 - This container advertises the home router (192.168.1.0/24) & vmbr0 to the tailnet.
 - Uses a default linux bridge called vmbr0 and within 10.10.10.0/24 (lab-side) subnet mask.
 - IP address: 10.10.10.2

## Wazuh Manager 
 - Ubuntu 26.0.4 LTS VM on Proxmox.
 - This is the main VM that contains the Wazuh Manager, indexer, and dashboard.
 - IP address: 10.10.10.9 + <TAILSCALE_IP>.

## WindowHost4
 - Windows 11 VM on Proxmox.
 - Has Wazuh Agent implemented as well as FIM (File Integrity Monitoring). This was tested in the demo. YARA, and VirusTotal are also configured on this VM to watch users /Downloads folder.
 - IP address: 10.10.10.10

## Grafana
 - LXC on Proxmox
 - Visualization and analytics tool that visualizes wazuh indexer logs. Used for custom dashboards.
 - IP address: 10.10.10.11

## CAPEv2 Host
 - Seperate Ubuntu VM on tailnet.
 - The VM contains a windows 10 host that is the detonation sandbox. All email attachments are sent to this VM and viewed with email analyzer python script.
 - IP address: <CAPE_TAILSCALE_IP>
 
 

