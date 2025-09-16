# Splunk-Siem
Splunk-based SOC project: log collection, dashboards, alerts, and reporting from Linux &amp; Windows servers.

# Splunk SIEM Project – Log Monitoring & Visualization

This project demonstrates how to set up a Splunk SIEM solution for monitoring logs from both Windows and Ubuntu servers. It includes connecting agents, ingesting logs, building dashboards for visualization, and configuring alerts and reports for proactive security monitoring.

This project sets up Splunk Enterprise on an Ubuntu server and connects two Splunk Universal Forwarders (agents) — one from a Windows machine and one from another Ubuntu machine. Logs are collected centrally, visualized with dashboards, and analyzed through alerts and reports.

# 🔹 Requirements

Before starting, make sure you have the following:

# System Requirements

Ubuntu Server (Splunk Enterprise host)

CPU: 4 cores (minimum)
RAM: 8 GB (recommended)
Disk: 50+ GB (for logs & indexing)
OS: Ubuntu 20.04 LTS or later

Windows Agent (Splunk Universal Forwarder)

OS: Windows 10 / Windows Server 2016+
CPU: 2 cores
RAM: 4 GB
Disk: 20 GB free

Ubuntu Agent (Splunk Universal Forwarder)

CPU: 2 cores
RAM: 4 GB
Disk: 20 GB free
OS: Ubuntu 20.04 LTS or later

# Software Requirements

Splunk Enterprise (installed on Ubuntu server)
Splunk Universal Forwarder (installed on Windows & Ubuntu agents)
Python 3 (optional, for custom scripts/log forwarding)
OpenSSH / WinRM (optional, for remote management)

# Network Requirements

Open port 9997/tcp on Ubuntu server (for agent → server log forwarding).
Ensure all agents can resolve and reach the Splunk server IP.
Proper firewall rules allowing Splunk communication.

Nice — here’s a clear, **copy-pasteable step-by-step procedure** to set up **Splunk Enterprise on an Ubuntu server** and connect **Windows + Ubuntu Universal Forwarders**, then create dashboards, alerts and reports. Replace `<SPLUNK_SERVER_IP>`, `<SPLUNK_FILE>.deb` and `<PASSWORD>` with your values.

# Step-by-step procedure

---

## 0) Prep / prerequisites (quick checks)

1. Ensure Ubuntu server has public/private IP and reachable from agents.
2. Ensure time sync (NTP / systemd-timesyncd) on all machines.
3. Open required ports (or add cloud security group rules):

   * Splunk Web: `8000/tcp`
   * Forwarder → Indexer: `9997/tcp`
   * SSH: `22/tcp` (for management)
4. On Ubuntu server:

```bash
sudo apt update && sudo apt install -y wget curl netcat ufw
```


---

## 1) Install Splunk Enterprise on the Ubuntu server

1. Download the Splunk `.deb` from Splunk website to your Ubuntu server (use browser or `wget` if you have URL).
2. Install the package (example):

```bash
sudo dpkg -i splunk-<VERSION>-linux-amd64.deb
```

3. Start Splunk and accept license:

```bash
sudo /opt/splunk/bin/splunk start --accept-license
```

4. Access Splunk Web at: `http://<SPLUNK_SERVER_IP>:8000` — set the admin password when prompted.
5. (Optional) Enable Splunk to start at boot:

```bash
sudo /opt/splunk/bin/splunk enable boot-start
```

---

## 2) Enable Splunk to receive data (on Ubuntu server)

1. From server shell:

```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:<YOUR_ADMIN_PASSWORD>
```

2. Ensure firewall allows port 9997 and 8000:

```bash
sudo ufw allow 8000/tcp
sudo ufw allow 9997/tcp
sudo ufw enable    # if not already enabled
```

3. Confirm Splunk is listening:

```bash
sudo ss -tlnp | grep 9997
```

---

## 3) Install Splunk Universal Forwarder — Ubuntu agent

1. Download the Universal Forwarder `.deb` to the Ubuntu agent.
2. Install:

```bash
sudo dpkg -i splunkforwarder-<VER>-linux-amd64.deb
```

3. Start UF and accept license:

```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

4. Set a UF admin password (if prompted) or change default:

```bash
sudo /opt/splunkforwarder/bin/splunk edit user admin -password <UF_PASSWORD> -auth admin:changeme
```

5. Configure forwarding to Splunk server:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_SERVER_IP>:9997 -auth admin:<UF_PASSWORD>
```

6. Add log monitors (example: system logs):

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log -auth admin:<UF_PASSWORD>
sudo /opt/splunkforwarder/bin/splunk restart
```

**Alternative:** put `inputs.conf` and `outputs.conf` under `/opt/splunkforwarder/etc/system/local/` (examples below).

---

## 4) Install Splunk Universal Forwarder — Windows agent

1. Download `splunkforwarder-<ver>-x64.msi` to Windows machine.
2. Install via GUI or silent install:

```powershell
msiexec /i splunkforwarder-<ver>-x64.msi AGREETOLICENSE=Yes /quiet
```

3. Start the forwarder (from elevated CMD/PowerShell):

```powershell
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" start --accept-license
```

4. Add forward-server and monitor Windows event logs:

```powershell
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" add forward-server <SPLUNK_SERVER_IP>:9997 -auth admin:changeme
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" add monitor "C:\Windows\System32\winevt\Logs" -auth admin:changeme
```

5. Restart UF service:

```powershell
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" restart
```

(Replace auth if you changed the UF admin password.)

---

## 5) Example `outputs.conf` and `inputs.conf` (for repo / automation)

**outputs.conf** (on forwarders — `/opt/splunkforwarder/etc/system/local/outputs.conf`)

```
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <SPLUNK_SERVER_IP>:9997
autoLBFrequency = 30
```

**inputs.conf** (example monitors)

```
[monitor:///var/log]
disabled = 0
index = main
sourcetype = syslog

[WinEventLog://Security]
disabled = 0
index = main
```

After creating these files restart the forwarder.

---

## 6) Verify forwarders are connected

1. On forwarder (Ubuntu):

```bash
/opt/splunkforwarder/bin/splunk list forward-server
# shows: <SPLUNK_SERVER_IP>:9997 (Active)
```

2. From Windows (PowerShell):

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
```

3. From Splunk Web (on the indexer): `Settings → Forwarder Management` or Search:

```spl
index=_internal sourcetype=splunkd component=TcpOutputProc | stats count by host
```

4. Or test network connectivity from agents:

* Linux: `nc -vz <SPLUNK_SERVER_IP> 9997`
* Windows: `Test-NetConnection -ComputerName <SPLUNK_SERVER_IP> -Port 9997`

---

## 7) Create indexes & sourcetypes (on Splunk Web)

1. Splunk Web → `Settings → Indexes → New Index` → create `linux_syslog`, `windows_events`, etc.
2. Optionally configure inputs to send to those indexes (or set via `inputs.conf` on forwarders).

---

## 8) Example SPL searches to use in dashboards / alerts

**Linux failed SSH attempts**

```spl
index=linux_syslog sourcetype=syslog "Failed password" OR "authentication failure"
| stats count by host, user
| sort -count
```

**Windows failed logins (EventCode 4625)**

```spl
index=windows_events sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Account_Name, host
| sort -count
```

**Combined timeline of failed logins**

```spl
(index=linux_syslog sourcetype=syslog "Failed password")
OR
(index=windows_events sourcetype="WinEventLog:Security" EventCode=4625)
| timechart span=1h count
```



---

## 9) Create dashboards (Splunk Web)

1. In Splunk Web go to **Search & Reporting**.
2. Run a saved search (e.g. combined timeline).
3. Click **Save As → Dashboard Panel** → select an existing or **New Dashboard**.
4. Configure visualization type (timechart, table, single value) and panel title.
5. Repeat for multiple panels (failed logins, top hosts, CPU/memory from metric logs).



---

## 10) Create alerts

1. From a search that detects suspicious activity (e.g., failed logins), click **Save As → Alert**.
2. Configure:

   * **Title**: e.g., `Multiple Failed Logins`
   * **Alert type**: Real-time (for immediate) or Scheduled (run every 5 min)
   * **Trigger condition**: `If number of results > 5` (example) or `per-result`
   * **Actions**: Email, run a script, webhook, trigger a notable event in ES, etc.
3. Add alert throttling if you want to suppress repetitive notifications.
4. Save and test (force the condition if needed).

**Example alert SPL** (trigger when > 5 failed logins in 10 minutes):

```spl
(index=linux_syslog sourcetype=syslog "Failed password") OR (index=windows_events sourcetype="WinEventLog:Security" EventCode=4625)
| bin _time span=10m
| stats count by _time, host
| where count > 5
```

---

## 11) Create scheduled reports

1. Save a useful search as a **Report** (`Save As → Report`).
2. In the Report settings click **Schedule**: pick `Daily` at 08:00 or cron.
3. Configure distribution: email recipients, attach PDF/CSV, or save to Splunk.
4. Save and verify the first run.

---

## 12) Export dashboards / saved searches to the repo

1. In Splunk Web use Dashboard **Edit → Source** to copy XML/JSON and save to `dashboards/` in your repo.
2. Export saved searches via **Settings → Searches, reports, and alerts** → view the saved search → **Export** or copy the search definition into `alerts/` or `reports/` folder as `.conf` snippets.
3. Keep screenshots in `docs/images/` (add sample dashboard screenshots).

---

## 13) Troubleshooting (common issues & fixes)

* **Agent not connecting**: check `nc -vz <server> 9997` from agent; ensure UFW/SG rules allow 9997.
* **No events visible**: confirm forwarder added `monitor` paths and restarted; check forwarder logs: `/opt/splunkforwarder/var/log/splunk/splunkd.log`.
* **Time mismatch**: verify NTP — event timestamps will be messy otherwise.
* **Auth errors**: ensure correct admin credentials when using CLI `-auth user:pass`.
* **High disk usage**: set retention / index size limits under Index settings.

---




