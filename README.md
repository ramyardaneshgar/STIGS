# STIGS
 <#
.SYNOPSIS
    This PowerShell script enforces the Windows 10 STIG requirement (WN10-AU-000500)
    by ensuring the maximum size of the Application event log is at least 32 MB (32768 KB).

.DESCRIPTION
    Creates or updates the "MaxSize" registry value for the Application event log
    to comply with DISA STIG guidance. Ensures compliance for audit logging capacity.

.NOTES
    Author          : Ramyar Daneshgar
    Email           : ramyarda@usc.edu
    LinkedIn        : linkedin.com/in/ramyardaneshgar
    GitHub          : github.com/ramyardaneshgar
    Date Created    : 2025-08-25
    Last Modified   : 2025-08-25
    Version         : 1.0
    STIG-ID         : WN10-AU-000500
    CVEs            : N/A
    Plugin IDs      : N/A

.TESTED ON
    Date(s) Tested  : 2025-08-25
    Tested By       : Ramyar Daneshgar
    Systems Tested  : Windows 10 (Latest Release)
    PowerShell Ver. : 5.1, 7.x

.USAGE
    Run in an elevated PowerShell session (Administrator privileges required).

    Example:
    PS C:\> .\Remediate-WN10-AU-000500.ps1
#>

# Registry path for Application event log
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\EventLog\Application"

# Ensure the registry path exists
if (-not (Test-Path $regPath)) {
    Write-Output "Registry path not found. Creating: $regPath"
    New-Item -Path $regPath -Force | Out-Null
}

# Set the MaxSize value to 0x00008000 (32768 decimal, 32 MB)
$desiredSize = 32768
Write-Output "Configuring MaxSize to $desiredSize KB (32 MB) per STIG requirement..."
Set-ItemProperty -Path $regPath -Name "MaxSize" -Value $desiredSize -Type DWord

# Verify the change
$currentValue = (Get-ItemProperty -Path $regPath).MaxSize
Write-Output "Verification complete. Current MaxSize = $currentValue KB"
 
