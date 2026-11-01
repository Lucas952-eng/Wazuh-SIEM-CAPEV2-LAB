# Table of Contents
Per user convienence, I have created a table of contents that will show all parts, sections, and Lab Components.

The lab components are listed here: [Components](Lab_Components.md)

## Part 1--Wazuh Setup in the Proxmox Environment
 - [1.1 Creating the internal lab network](Part1.md#11-create-the-internal-network)
 - [1.2 Create the Wazuh VM](Part1.md#12-creation-/-specs-of-wazuh-vm)
 - [1.3 Install Wazuh (all-in-one)](Part1.md#13-install-wazuh)
 - [1.4 Open the indexer to the lab network]
 - [1.5 Implementing ufw firewall on Wazuh VM]

## Part 2--Linking Wazuh to Tailscale and Adding Agents
 - 2.1 Tailscale subnet router (LXC on Proxmox)
 - 2.2 Put the Wazuh VM on the tailnet
 - 2.3 Tailscale access policy (default-deny grants)
 - 2.4 Tag the machines
 - 2.5 Add agents
   - Windows 11 (WindowsHost4)
   - Omarchy/Arch laptop
   - Verification 

## Part 3--Connecting CAPEv2 to Wazuh
 - 3.1 Prerequisites on the CAPE host
 - 3.2 Sandbox isolation--before any real malware
 - 3.3 Enable only the API endpoints needed
 - 3.4 Install the Wazuh agent on the CAPE host
 - 3.5 Ship the analyzer log to Wazuh
 - 3.6 Manager rules
 - 3.7 End-to-end test
 - 3.8 Known issues/to-do 
