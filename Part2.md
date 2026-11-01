

# Part 2--Linking Wazuh to Tailscale and Adding Agents

## 2.1 Tailscale LXC on Proxmox

 1. **Proxmox -> Tailscale -> Network -> Add**
    - Linux Bridge: `vmbr0`
    - IPv4: `10.10.10.2/24`
    - Default Gateway: `10.10.10.1`. Still reaches internet through bridge.

 2. Confirm from LXC terminal that it can see Wazuh: `ping -c 2 10.10.10.9`
 3. Next, advertise the LAN (`192.168.1.0/24`) and `vmbr0` ip addresses :
    ```
    tailscale set --advertise-routes=192.168.1.0/24,10.10.10.0/24
    tailscale debug prefs | grep -A4 AdvertiseRoutes 
    ```
 4. After these commands, go to tailscale.com -> **Admin Console -> Machines -> Tailscale LXC -> Edit Subnet Routes**

 5. Advertising the LAN and linux bridge gives other machines in the tailnet that aren't located locally on the server access via tailscale lxc.

## 2.2 Adding Wazuh VM to tailnet
Due to the CAPEv2 being remote means that it can directly communicate with the `10.10.10.0/24` addresses, so the easiest way to counter this is using the tailnet directly:

```
  curl -fsSL https://tailscale.com/install.sh | sh
  sudo tailscale up
  tailscale ip -4  
```
NOTE: Command `tailscale ip -4` shows the given tailscale ip on the machine (Wazuh VM)

## 2.3 Tailscale access policy (default-deny grants)

Tailscale as its own built-in **Access Control Lists** (ACLs) as well as **Grants**, that can help direct/block traffic on the tailnet.

In this section, I will go through my configurations in the access policy section located from **Admin console -> Access controls -> JSON Editor**. Make sure to click **Save** when done.

Below are my configurations:

```
{
  "tagOwners": {
    "tag:scanner": ["autogroup:admin"],
    "tag:cape":    ["autogroup:admin"],
  },

  "grants": [
    // Admin can reach everything
    {"src": ["autogroup:admin"], "dst": ["*"], "ip": ["*"]},

    // Invited members (the CAPE owner) only reach their own devices
    {"src": ["autogroup:member"], "dst": ["autogroup:self"], "ip": ["*"]},

    // Wazuh VM -> CAPE API only
    {"src": ["tag:scanner"], "dst": ["tag:cape"], "ip": ["tcp:8000"]},

    // CAPE host's Wazuh agent -> manager (enroll + events) only
    {"src": ["tag:cape"], "dst": ["tag:scanner"], "ip": ["tcp:1514", "tcp:1515"]},
  ],

  "ssh": [
    {
      "action": "check",
      "src":    ["autogroup:member"],
      "dst":    ["autogroup:self"],
      "users":  ["autogroup:nonroot", "root"],
    },
  ],
}
```
|Config Grants|Reasoning for Implementation|
|---|---|
|`autogroup:admin`|Grant gives admin access to all machines.|
|`autogroup:member`|CAPE member (added CAPEv2 remote user to add tailscale to CAPEv2 VM)can't reach Proxmox, Windows 11, etc.|
|`tag:scanner` -> `tag:scanner` tcp:8000| Wazuh VM may call the CAPE API and nothing else.|
|`tag:cape` -> `tag:scanner` tcp:1514,1515| The agent always **starts** the connection to the manager, so it needs its own one-way grant.

NOTE: Do **NOT** grant `10.10.10.0/24` to `autogroup:member` or `*` because CAPE host runs malware.

## 2.4 Tag the machines
**Machines -> ... -> Edit ACL tags (tags only appear after `tagOwners` is saved)**
  - Wazuh VM -> `tag:scanner`
  - CAPE host -> `tag:cape`

Once everything has been tagged, a ping from the Wazuh VM to CAPE **stops working** -- thats expected(only tcp:8000 is allowed). Add `"icmp:*"` to that grant if you want ping back.

## 2.5 Add agents
Here is the general rule with adding agents: **Agents on the lab bridge (`vmbr0`) point at `10.10.10.9`; agents off-site (not on server/local LAN) point at <WAZUH_TAILSCALE_IP>

## Windows 11 Specs/Configs
|Hardware/Software|Specs|
|---|---|
|OS|Windows 25H2 ISO|
|CPU|4 Cores (Proxmox Host)|
|Memory|8GB|
|Storage|40-80GB|
|Network|Virtio with `vmbr0`|
|IP Addresses| Static IP: `10.10.10.9`, Default Gateway: `10.10.10.1`|

1. To deploy a Wazuh agent go to **Wazuh Dashboard -> Agents -> Deploy new agent -> Windows,** manager `10.10.10.9`, name `WindowsHost4`, or whatever you prefer.
2. Then, Wazuh will give you the provided code (example):
   ```
      msiexec.exe /i wazuh-agent-4.14.7-1.msi /q `
     WAZUH_MANAGER="10.10.10.9" `
     WAZUH_AGENT_NAME="WindowsHost4" `
     WAZUH_REGISTRATION_SERVER="10.10.10.9"
     NET START WazuhSvc
   ```
3. To verify that the agent is currently on, go to the agents **Agents** in Wazuh and look for your newly added agent.
