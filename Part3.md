# Part 3--Connecting CAPEv2 to Wazuh

Here is how the data will flow:
```
Gmail inbox → new_analyzer.py (CAPE host) → CAPEv2 detonation
  → analyzer.json (one JSON line per event)
  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)
```
NOTE: In section **3.6 Manager rules** I go over the configured rules 100400-100405

CAPEv2 runs on the **Ubuntu host**. The actual "Detonation" goes on in the nested windows host within the Ubuntu machine.

## 3.1 Prerequisites on the CAPE host
Primarily in this repo, I will mainly only focus on the Wazuh configurations, if you want more information on how please go to this repo: **(link will be added shortly)**

  - CAPE host must join the tailnet if not already and tagged as `tag`(Part 2.3-2.4)

## Sandbox Isolation--before any real malware

These rules will stop the detonation guest from reaching the tailnet or any private network:
```
sudo iptables -I FORWARD -i virbr0 -o tailscale0 -j DROP
sudo iptables -I INPUT   -i virbr0 -d 100.64.0.0/10 -j DROP
sudo iptables -I FORWARD -i virbr0 -d 192.168.0.0/16 -j DROP
sudo iptables -I FORWARD -i virbr0 -d 10.0.0.0/8 -j DROP
sudo iptables -I FORWARD -i virbr0 -d 172.16.0.0/12 -j DROP
sudo apt install iptables-persistent && sudo netfilter-persistent save
```
These rules are to be made on the CAPEv2 Host, **NOT** the nested detonation VM.

- `/opt/CAPEv2/conf/routing.conf` -> `route = none`, then `sudo systemctl restart cape cape-rooter`.
  - Note: `route = none` applies **no** routing rules--the guest keeps internet access. Isolation from private networks comes from the iptable rules above.
- Never run `--advertise-routes` for the sandbox subnet or `--advertise-exit-node on the CAPE host.
- **Test:** from the CAPEv2 host, and ping any tailnet IP (`100.x`) and must fail.

## 3.3 Enable API endpoints (CAPE Host)
`/opt/CAPEv2/conf/api.conf` (everything else stays disabled -- least privilege):

```
[filecreate]
enabled = yes

[taskstatus]
enabled = yes
  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)

  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)

  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)

  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)

  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)

  → Wazuh agent (CAPEv2_Agent) → Tailscale :1514 → Wazuh manager
  → rules 100400–100405 → alerts (correlated with Windows YARA/FIM by SHA256)


[taskreport]
enabled = yes
```

```
sudo systemctl restart cape-web
sudo ss -tlnp | grep 8000      # must NOT be 127.0.0.1-only
```
From the Wazuh VM, confirm the API is reachable over Tailscale:

`curl http://<CAPE_TAILSCALE_IP>:8000/apiv2/cuckoo/status/`

If it outputs: `Cuckoo Status API is disabled` is a **network success**--CAPE answered.

## 3.4 Install the Wazuh agent on CAPE host
1. Wazuh Dashboard -> Agents -> Deploy new agent
|Agent Options|Config|
|---|---|
|OS| Linux, DEB amd64|
|Server Address|<WAZUH_TAILSCALE_IP>|
|Name|CAPEv2_Agent|
