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


