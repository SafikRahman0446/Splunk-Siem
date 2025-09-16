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
