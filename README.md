
# STIG Remediation Walkthrough – WN10-AU-000500

By Ramyar Daneshgar

---

## Step 1: Baseline Scan – Identifying the Finding

I started by provisioning a fresh Windows 10 VM in Azure. I disabled the Windows Firewall temporarily to ensure my scanner (Tenable.io, with LOCAL-SCAN-ENGINE-01) could connect without issues.

In Tenable, I created an **Advanced Network Scan** with the following configuration:

* Target: my VM’s private IP.
* Authentication: Windows credentials.
* Compliance Check: DISA Windows 10 STIG v3r1 (Policy Compliance).

I disabled all unnecessary plugins except **Windows Compliance Checks**, to make the scan faster and focused only on STIGs.

When the baseline scan completed, I reviewed the results and confirmed **WN10-AU-000500** was marked as **FAILED**. This meant the **Application Event Log “MaxSize” registry value was either missing or set below 32768 KB**.

My logic here was simple: establish a baseline → confirm the finding exists → document its state before remediation.

---

## Step 2: Manual Remediation

Next, I needed to fix this manually. Based on the DISA STIG documentation, the requirement is:

* **Registry Hive:** `HKEY_LOCAL_MACHINE`
* **Path:** `SOFTWARE\Policies\Microsoft\Windows\EventLog\Application`
* **Value Name:** `MaxSize`
* **Value Type:** `REG_DWORD`
* **Value:** `0x00008000` (decimal 32768, or greater)

I opened the Windows Registry Editor (`regedit`) on the VM and navigated to the above path. The `MaxSize` key wasn’t there by default.

My thought process was: *if the key doesn’t exist, I must create it; if it exists but is below 32768, I must raise it.*

So I added a new `DWORD (32-bit)` value named **MaxSize** and set its value to **32768** (decimal).

---

## Step 3: Verification via Rescan

After applying the manual registry change, I restarted the VM to ensure the change took effect.

Then, I re-ran the Tenable compliance scan. This time, the finding for **WN10-AU-000500** passed successfully.

This confirmed my manual remediation was correct.
<img width="1634" height="925" alt="Screenshot 2025-08-25 at 5 41 36 PM" src="https://github.com/user-attachments/assets/a44f3b45-aeba-4c44-aa26-05e8db4f8f3f" />



---

## Step 4: Undoing the Fix to Prove Control

To prove I fully understood the control, I then **deleted the MaxSize key** from the registry and rescanned.

As expected, the STIG failed again.

My thought process here was: *if I can make it fail and pass consistently, I’ve demonstrated I understand both the requirement and its enforcement mechanism.*

---

## Step 5: PowerShell Automation

Once I understood the manual process, I moved to automate it with PowerShell. I wanted a repeatable script that could:

1. Check if the registry path exists.
2. Create it if missing.
3. Set the `MaxSize` value to `32768` or greater.
4. Verify the applied value.

Here’s the script I wrote:

```powershell
<#
.SYNOPSIS
    Enforces Windows 10 STIG requirement (WN10-AU-000500) by setting
    Application event log size to at least 32 MB.

.NOTES
    Author: Ramyar Daneshgar
    Date  : 2025-08-25
    STIG-ID: WN10-AU-000500
#>

$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\EventLog\Application"

if (-not (Test-Path $regPath)) {
    Write-Output "Registry path not found. Creating: $regPath"
    New-Item -Path $regPath -Force | Out-Null
}

$desiredSize = 32768
Write-Output "Configuring MaxSize to $desiredSize KB (32 MB) per STIG requirement..."
Set-ItemProperty -Path $regPath -Name "MaxSize" -Value $desiredSize -Type DWord

$currentValue = (Get-ItemProperty -Path $regPath).MaxSize
Write-Output "Verification complete. Current MaxSize = $currentValue KB"
```

My reasoning here: using `Set-ItemProperty` ensures idempotence — the script works whether or not the value exists, and it sets it correctly every time.

---

## Step 6: Validation with Scan

I ran the script inside an elevated PowerShell session on my VM. Then I rescanned with Tenable. The finding flipped from **Fail → Pass** again.

This validated my script was working exactly like the manual fix, only automated.
<img width="1634" height="925" alt="Screenshot 2025-08-25 at 6 26 22 PM" src="https://github.com/user-attachments/assets/10235ca0-7eba-4bbf-af67-c0e30aa4098a" />

---

## Step 7: Publishing & Documentation

Finally, I pushed the script to my GitHub repository under a **“STIG Remediations”** folder, with the proper STIG-ID in the filename.

I also logged the experience in my internship **tracking spreadsheet**, writing in first person what I did, the logic I followed, and linking both the GitHub code and Tenable screenshots.

---

 **Lessons Learned:**

* Always establish a baseline before remediation.
* Understand what the STIG is checking (registry keys, values, permissions, etc.).
* Confirm by rescanning after manual and automated fixes.
* Document the fix, rollback, and automation process for audit and compliance readiness.

---

Would you like me to also draft the **rollback script** in PowerShell (to remove `MaxSize` or reset to default) so you can demonstrate the *fail → pass → fail → pass* control cycle in your GitHub repo?
