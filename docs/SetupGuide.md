# Setup Guide
# 🛠️ Setup Guide: SIEM RDP Honeypot Project

Welcome! This guide will walk you through setting up a secure and educational RDP honeypot using Microsoft Azure and Microsoft Sentinel (SIEM).

---

## 📦 Prerequisites

- Azure Subscription
- Basic understanding of PowerShell
- Familiarity with Microsoft Sentinel and Log Analytics

---

## 1. 🚀 Create the Honeypot (Windows VM)

1. Go to the [Azure Portal](https://portal.azure.com/)
2. Create a **Windows 10/11 VM**
3. In **Networking**, ensure port **3389 (RDP)** is open to the internet
4. Disable default security tools to allow login attempts
5. Note the **public IP address**

---

## 2. 🔍 Enable Log Collection

1. Create a **Log Analytics Workspace**
2. Link the VM to the workspace via **Azure Monitor Agent**
3. Enable the collection of **SecurityEvent** logs
4. Confirm that Event ID `4625` (Failed Login) is being captured

---

## 3. 📄 Extract RDP Failures

1. Run the PowerShell script `extract_failed_rdp.ps1`
2. The script should:
   - Parse failed login attempts from Event Viewer
   - Extract source IP addresses
   - Use a geolocation API (e.g., ip-api.com) to resolve country/lat/long
   - Save the output to `failed_rdp.log`

---

## 4. 📤 Upload to Sentinel

1. In Log Analytics, create a **Custom Log Table** named:  
   `FAILED_RDP_WITH_GEO_CL`
2. Upload the `failed_rdp.log` file
3. Parse custom fields:
   - `sourcehost_CF`
   - `latitude_CF`
   - `longitude_CF`
   - `country_CF`
   - `label_CF`
   - `destinationhost_CF`

---

## 5. 📈 Create the Real-Time Map in Sentinel

1. Open **Microsoft Sentinel > Workbooks**
2. Create a new workbook and use the following KQL:

```kql
FAILED_RDP_WITH_GEO_CL
| summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost" and sourcehost_CF != ""
```

3. Insert a **Map Widget**
4. Bind latitude/longitude for geovisualization
5. Enable **auto-refresh** for live updates

---

## ✅ Done!

You now have a working SIEM-integrated RDP honeypot showing real-time brute-force activity on a live world map.

> ⚠️ **Warning:** This is a honeypot. Do NOT reuse this setup in production environments.

---

## 📂 Project Structure

```
azure-sentinel-rdp-honeypot/
├── PowerShell/
│   └── extract_failed_rdp.ps1
├── screenshots/
│   └── (Add screenshots here)
├── docs/
│   └── SetupGuide.md
└── README.md
```

---

Happy Hunting! 🎯
