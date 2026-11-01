# Part 2--Linking Wazuh to Tailscale and Adding Agents

## 2.1 Tailscale LXC on Proxmox

 1. **Proxmox -> Tailscale -> Network -> Add**
    - Linux Bridge: `vmbr0`
    - IPv4: `10.10.10.2/24`
    - Default Gateway: `10.10.10.1`. Still reaches internet through bridge.

 2. Confirm from LXC terminal that it can see Wazuh: `ping -c 2 10.10.10.9`
 3. Next, advertise both LAN and linux bridge subnets through tailscale:
    ```
    tailscale set --advertise-routes=192.168.1.0/24,10.10.10.0/24
    tailscale debug prefs | grep -A4 AdvertiseRoutes 
    ```
    
