# Azure Sentinel RDP Honeypot Project with Real-Time Attack Mapping


## 📌 Overview
This project demonstrates how to set up a Windows VM in Microsoft Azure as an **RDP honeypot** to attract brute-force attacks, and how to monitor those attacks in **real-time using Microsoft Sentinel** on a world map.

---

## 🧠 Objectives
- Simulate a real-world attack surface using a deliberately exposed RDP port.
- Capture failed login attempts.
- Geolocate attacking IP addresses.
- Visualize attacks on a dynamic map using Azure Sentinel.

---

## 🛠️ Setup Steps

### 1. Create a Windows VM in Azure
- Deploy a Windows 10 VM.
- Open **RDP port (3389)** to the public internet.
- Disable firewall and other security features to allow brute-force attempts.

### 2. Set Up Log Analytics Workspace
- Create a **Log Analytics workspace** in Azure.
- Connect the VM to the workspace via Azure Monitor.
- Ingest **SecurityEvent** logs (enable all events).

### 3. Install Microsoft Sentinel
- Add Microsoft Sentinel to the workspace.
- Set up a **custom connector** to ingest logs from your honeypot VM.

### 4. Log Failed RDP Attempts
- Use **Event Viewer** to monitor Event ID `4625` (failed logon).
- Run a PowerShell script to:
  - Extract source IPs from logs.
  - Call a geolocation API to fetch `latitude`, `longitude`, `country`, etc.
  - Output results into `failed_rdp.log`.

<details>
<summary>Example PowerShell Logic (simplified)</summary>

```powershell
# Pseudo-script logic
Get-EventLog -LogName Security -InstanceId 4625 |
  ForEach-Object {
    # Extract IP and call geolocation API
    # Append results to failed_rdp.log
  }
```
</details>

### 5. Upload Custom Log
- Upload `failed_rdp.log` to Log Analytics.
- Define a **Custom Log Table**: `FAILED_RDP_WITH_GEO_CL`.
- Extract fields:
  - `sourcehost_CF`
  - `latitude_CF`
  - `longitude_CF`
  - `country_CF`
  - `label_CF`
  - `destinationhost_CF`

### 6. Create Sentinel Workbook (Live Map)
- Use Kusto Query Language (KQL) to query the custom table:

```kql
FAILED_RDP_WITH_GEO_CL
| summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost" and sourcehost_CF != ""
```

- Visualize results using the **map widget** in Sentinel.
- Enable **auto-refresh** to simulate a live cyber threat map.

---

## 🌍 Example Output
> Live attacks will appear as bubbles on a global map in Sentinel.

![Alt Text](screenshots/global-map.png)


---

## 📌 Tools & Services Used
- 🧠 Microsoft Azure
- 🔍 Log Analytics
- 🛡 Microsoft Sentinel
- 💻 PowerShell
- 🌐 IP Geolocation API (e.g., ip-api.com, ipinfo.io)

---

## ⚠️ Disclaimer
> This project is **educational only**. Exposing RDP to the public internet is a major security risk. Do not use this setup in a production environment.

---

## ✅ To-Do
- [ ] Upload PowerShell script
- [ ] Add screenshots
- [ ] Document API rate limits
- [ ] Automate log shipping with cron/Task Scheduler
