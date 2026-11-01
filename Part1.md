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

## 1.3 Install Wazuh 

1. Wazuh installation assistant: `curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh`
2. After this command, run `bash wazuh-install.sh --wazuh-server wazuh-1` to actually run the assistant with the `--wazuh-server` option followed by node name. `Wazuh-1` is a example.
 - Everything is installed including **manager**,**indexer**, and **dashboard** on one VM.
 - MAKE SURE TO SAVE `admin` PASSWORD.
 - Default Wazuh Dashboard IP: `https://10.10.10.9:443`.

## 1.4 Open the indexer to the lab network (needed for Grafana)

The all-in-one install binds the wazuh indexer to `127.0.0.1`(loopback), so nothing outside the VM can reach port 9200.

```
sudo cp /etc/wazuh-indexer/opensearch.yml /etc/wazuh-indexer/opensearch.yml.bak
sudo sed -i 's/^network.host:.*/network.host: "0.0.0.0"/' /etc/wazuh-indexer/opensearch.yml
grep network.host /etc/wazuh-indexer/opensearch.yml     # should show "0.0.0.0"
sudo systemctl restart wazuh-indexer                    # 30–60 seconds

ss -tlnp | grep 9200                 # 0.0.0.0:9200
curl -k https://localhost:9200       # "Unauthorized" = working
```
- Changing network host from loopback to `0.0.0.0` means that Opensearch listens on **every** ipv4 address on this VM.
- If the indexer doesn't start correctly: `journalctl -u wazuh-indexer -n 50 --no-pager`.

**NOTE**: Setting the indexer to `0.0.0.0` is a security risk. In the next section, we will configure the ufw firewall to only allow **Grafana** to be accepted and view indexer logs.

## 1.5 Firewall the Wazuh VM(ufw)

  - This will be run from the **Proxmox console**. This is in case a bad ufw rules locks you out.
 1. Indexer Rules: Grafana Only
    ```
    ufw allow from 10.10.10.11 to any port 9200 proto tcp
    ufw deny 9200/tcp
    ```
