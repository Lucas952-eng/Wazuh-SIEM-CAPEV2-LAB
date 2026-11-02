# Table of Contents
Per user convienence, I have created a table of contents that will show all parts, sections, and Lab Components.

The lab components are listed here: [Components](Lab_Components.md)

## [Part 1--Wazuh Setup in the Proxmox Environment](Part1.md#part-1--wazuh-setup-in-the-proxmox-environment)
 - [1.1 Creating the internal lab network](Part1.md#11-create-the-internal-lab-network)
 - [1.2 Create the Wazuh VM](Part1.md#12-creationspecs-of-wazuh-vm)
 - [1.3 Install Wazuh](Part1.md#13-install-wazuh-all-in-one)
 - [1.4 Open the indexer to the lab network](Part1.md#14-open-the-indexer-to-the-lab-network-needed-for-grafana)
 - [1.5 Implementing ufw firewall on Wazuh VM](Part1.md#15-implementing-ufw-firewall-on-wazuh-vm)

## [Part 2--Linking Wazuh to Tailscale and Adding Agents](Part2.md#part-2--linking-wazuh-to-tailscale-and-adding-agents)
 - [2.1 Tailscale LXC on Proxmox](Part2.md#21-tailscale-lxc-on-proxmox)
 - [2.2 Adding Wazuh VM to tailnet](Part2.md#22-adding-wazuh-vm-to-tailnet)
 - [2.3 Tailscale access policy (default-deny grants)](Part2.md#23-tailscale-access-policy-default-deny-grants)
 - [2.4 Tag the machines](Part2.md#24-tag-the-machines)
 - [2.5 Add agents](Part2.md#25-add-agents)
   - [Windows 11 Specs/Configs](Part2.md#windows-11-specsconfigs)

## [Part 3--Connecting CAPEv2 to Wazuh](Part3.md#part-3--connecting-capev2-to-wazuh)
 - [3.1 Prerequisites on the CAPE host](Part3.md#31-prerequisites-on-the-cape-host)
 - [3.2 Sandbox isolation--before any real malware](Part3.md#sandbox-isolation--before-any-real-malware)
 - [3.3 Enable API endpoints (CAPE Host)](Part3.md#33-enable-api-endpoints-cape-host)
 - [3.4 Install the Wazuh agent on CAPE host](Part3.md#34-install-the-wazuh-agent-on-cape-host)
 - [3.5 Ship the analyzer log to Wazuh](Part3.md#35-ship-the-analyzer-log-to-wazuh)
 - [3.6 Manager Rules](Part3.md#36-manager-rules)
