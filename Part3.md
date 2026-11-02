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
1. Wazuh Dashboard -> **Agents -> Deploy new agent**
   
|Agent Options|Config|
|---|---|
|OS| Linux, DEB amd64|
|Server Address|<WAZUH_TAILSCALE_IP>|
|Name|CAPEv2_Agent|
  - Copy the generated install command into the CAPEv2 Host terminal and run it.
2. After running the install command:
```
   sudo systemctl daemon-reload
   sudo systemctl enable --now wazuh-agent
```
1. Test agent paths:
   ```
   nc -zv <WAZUH_TAILSCALE_IP> 1515     # succeeded
   nc -zv <WAZUH_TAILSCALE_IP> 1514     # succeeded
   nc -zv <WAZUH_TAILSCALE_IP> 22       # must TIME OUT
   nc -zv <WAZUH_TAILSCALE_IP> 443      # must TIME OUT
   sudo tail -f /var/ossec/logs/ossec.log   # "Valid key received" → "Connected to the server"
   ```

## 3.5 Ship the analyzer log to Wazuh
On the CAPE host, get the real path (run **without** sudo):

`echo ~/analyzer/logs/analyzer.json`

Add `/var/ossec/etc/ossec.conf`, above the final `<ossec_config>`:

```
<localfile>
  <log_format>json</log_format>
  <location>/home/<CAPE_USER>/analyzer/logs/analyzer.json</location>
</localfile>
```

After adding above to the config file, do `sudo systemctl restart wazuh-agent` to confirm.
  - `loge_format json` auto-decodes every key into `data.` fields--no custom decoder needed.
  - Each event must be **one JSON object per line**.
  - The agent only ships lines written **after** it starts watching--add this before testing.

## 3.6 Manager Rules

Command to make sure ID range is free: `sudo grep -E 'id="1004[0-9][0-9]"' /var/ossec/etc/rules/local_rules.xml`

Append to `/var/ossec/etc/rules/local_rules.xml`:
```
<group name="email_analyzer,">
  <!-- Parent. Hangs off Suricata's JSON parent 86600, which otherwise captures
       any JSON with an event_type field. The event_type list keeps real
       Suricata events out. -->
  <rule id="100400" level="0">
    <if_sid>86600</if_sid>
    <field name="event_type">^attachment_scan$|^email_triage$|^auth_failed$|^quarantine$</field>
    <description>Email analyzer event</description>
  </rule>

  <rule id="100401" level="12">
    <if_sid>100400</if_sid>
    <field name="event_type">^attachment_scan$</field>
    <field name="verdict">^malicious$</field>
    <description>Email attachment $(filename) is MALICIOUS (CAPE malscore $(cape_malscore), VT $(vt_malicious))</description>
  </rule>

  <rule id="100402" level="8">
    <if_sid>100400</if_sid>
    <field name="event_type">^attachment_scan$</field>
    <field name="verdict">^suspicious$</field>
    <description>Email attachment $(filename) is suspicious (CAPE malscore $(cape_malscore))</description>
  </rule>

  <rule id="100403" level="3">
    <if_sid>100400</if_sid>
    <field name="event_type">^attachment_scan$</field>
    <field name="verdict">^clean$</field>
    <description>Email attachment $(filename) scanned clean</description>
  </rule>

  <rule id="100404" level="10">
    <if_sid>100400</if_sid>
    <field name="event_type">^email_triage$</field>
    <field name="verdict">^malicious$|^suspicious$</field>
    <description>Email "$(email_subject)" from $(email_from) quarantined: $(verdict)</description>
  </rule>

  <rule id="100405" level="10">
    <if_sid>100400</if_sid>
    <field name="event_type">^auth_failed$</field>
    <description>Email analyzer cannot log in to Gmail: scanning has stopped</description>
  </rule>
</group>
```

