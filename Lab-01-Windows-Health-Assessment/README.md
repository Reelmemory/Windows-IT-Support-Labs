# Lab 01: Windows Performance & Health Assessment

## Objective

Perform a preventive assessment of a Windows workstation using built-in PowerShell tools. Inspect system resources, storage, startup applications, running services, disk health, Windows event logs, Secure Boot and TPM status.

This was a preventive assessment. The user did not report any performance problems.

## Environment

| Component | Details |
|---|---|
| Device | HP EliteBook 830 G5 |
| Processor | Intel Core i7-8650U |
| Memory | 16 GB RAM |
| Storage | 256 GB SSD |
| Operating System | Windows 11 Pro |
| Display Version | 26H2 |
| OS Build | 26300.9818 |
| Shell | Windows PowerShell |

## 1. Operating System Identification

Commands:

```powershell
Get-ComputerInfo |
Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
```

Additional verification:

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' |
Select-Object ProductName, DisplayVersion, CurrentBuild, EditionID
```

**Finding:** Windows identification commands returned inconsistent product-name information. `winver` identified the operating system as Windows 11 Pro, version 26H2, build 26300.9818. Other product-name fields reported Windows 10 Pro.

**Lesson:** Verify operating system information using multiple sources instead of relying on a single legacy field.

## 2. Memory Assessment

```powershell
Get-CimInstance Win32_OperatingSystem |
Select-Object Caption, TotalVisibleMemorySize, FreePhysicalMemory
```

**Results:**
- Total visible memory: approximately 15.85 GiB.
- Available memory: approximately 6.81 GiB.
- Memory in use: approximately 9.04 GiB at the time of inspection.

**Finding:** No immediate memory pressure was established from this snapshot.

## 3. Storage Assessment

```powershell
Get-Volume -DriveLetter C |
Select-Object DriveLetter, FileSystem, SizeRemaining, Size
```

**Results:**
- File system: NTFS.
- Total capacity: approximately 237.2 GiB.
- Available space: approximately 27.0 GiB.
- Free capacity: approximately 11.4%.

**Finding:** Storage headroom is relatively limited and should be monitored.

## 4. CPU Assessment

```powershell
Get-CimInstance Win32_Processor |
Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, LoadPercentage
```

CPU sampling:

```powershell
Get-Counter '\Processor(_Total)\% Processor Time' `
-SampleInterval 2 -MaxSamples 5
```

**Results:** Five samples showed CPU utilization between approximately 4.7% and 7.6%, averaging approximately 6.4%.

**Finding:** CPU utilization was low during the short sampling period. This does not establish long-term performance.

## 5. Top Memory-Consuming Processes

```powershell
Get-Process |
Sort-Object WorkingSet64 -Descending |
Select-Object -First 5 ProcessName,
@{Name="Memory_MB";Expression={[math]::Round($_.WorkingSet64 / 1MB, 2)}}
```

**Finding:** Browser processes and Windows Memory Compression appeared among the largest memory consumers. This was an observation, not evidence of a fault.

## 6. Startup Application Inventory

```powershell
Get-CimInstance Win32_StartupCommand |
Select-Object Name, User, Location, Command |
Format-List
```

**Finding:** Startup entries included OneDrive, Teams, Discord, Chrome, Edge and Docker Desktop, alongside OneDrive setup entries.

No startup applications were disabled.

## 7. Windows Services Inventory

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.State -eq "Running"} |
Select-Object Name, DisplayName, StartMode |
Sort-Object DisplayName
```

**Finding:** Running services were inventoried, including Windows services and development-related services such as WSL and Hyper-V components.

No services were stopped or disabled.

## 8. Physical Disk Health

```powershell
Get-PhysicalDisk |
Select-Object FriendlyName, MediaType, HealthStatus, OperationalStatus, Size |
Format-Table -AutoSize
```

**Results:**
- Media type: SSD.
- Health status: Healthy.
- Operational status: OK.

**Finding:** Windows reported the SSD as healthy. This is not a complete assessment of SSD wear or remaining lifespan.

## 9. Windows Event Log Review

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Level = 1,2,3
    StartTime = (Get-Date).AddDays(-7)
} -MaxEvents 15 |
Select-Object TimeCreated, ProviderName, Id, LevelDisplayName, Message |
Format-List
```

Events identified:

| Provider | Event ID | Observation |
|---|---:|---|
| DistributedCOM | 10016 | Application permission warnings |
| DistributedCOM | 10010 | DCOM server registration timeouts |
| DNS-Client | 1014 | DNS resolution timeout for chatgpt.com |
| TPM-WMI | 1797 | Secure Boot certificate update issue |
| TPM-WMI | 1802 | Firmware-related Secure Boot update restrictions |

**Analysis:**
- DCOM 10016 warnings are commonly encountered and should not be modified blindly.
- DCOM 10010 requires investigation to identify the affected component and determine whether it causes an actual application problem.
- DNS 1014 may indicate a transient DNS resolution issue. Recurrence should be monitored.
- TPM-WMI events indicate a firmware-related Secure Boot certificate-update issue that warrants investigation.

## 10. Secure Boot and TPM Verification

Secure Boot:

```powershell
Confirm-SecureBootUEFI
```

**Result:** True.

TPM:

```powershell
Get-Tpm
```

**Results:**
- TPM present: True.
- TPM ready: True.
- TPM enabled: True.
- TPM activated: True.
- TPM manufacturer: Infineon.
- TPM manufacturer version: 7.63.3353.0.

BIOS information:

```powershell
Get-CimInstance Win32_BIOS |
Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate
```

**Results:**
- Manufacturer: HP.
- BIOS version: Q78 Ver. 01.31.00.
- Reported release date: March 10, 2025.

**Finding:** Secure Boot and TPM are operational. The Event Viewer warnings indicate a firmware-related certificate-update issue, not proof that Secure Boot is disabled.

## Overall Assessment

The workstation showed no established performance fault during this preventive assessment.

| Area | Assessment |
|---|---|
| CPU | Low utilization during sampling |
| RAM | No immediate pressure established |
| Storage | Limited free-space headroom |
| SSD | Healthy according to Windows |
| Startup | Inventory completed |
| Services | Inventory completed |
| Secure Boot | Enabled |
| TPM | Ready and enabled |
| Event Logs | Several events require further investigation |

## Recommendations

1. Monitor available storage and investigate safe opportunities to reclaim space.
2. Check HP's official firmware guidance for the Secure Boot certificate-update issue.
3. Investigate recurring DNS resolution failures if they continue.
4. Identify the components associated with DCOM 10010 events before considering corrective action.
5. Avoid disabling Windows services or modifying security settings without a confirmed reason.

## Conclusion

This lab demonstrates preventive Windows workstation assessment using native PowerShell and Windows diagnostic utilities.

No corrective changes were made because no confirmed performance fault was identified.

**Next lab:** Windows Event Log Investigation and Error Remediation.

---

*Portfolio note: This is a self-directed practical lab performed on a personal workstation. Findings reflect the system's state at the time of assessment.*
