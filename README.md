# CrowdStrike-removal-and-monitoring
This repository contains a PowerShell script designed to monitor and remove CrowdStrike driver files that may be targeted or tampered with by attackers. The script provides real-time file integrity monitoring, alerts administrators if changes are detected, and can shut down the system to prevent further damage. 
Repository Description
This repository contains a PowerShell script designed to monitor and remove CrowdStrike driver files that may be targeted or tampered with by attackers. The script provides real-time file integrity monitoring, alerts administrators if changes are detected, and can shut down the system to prevent further damage. The script logs all actions for review and operates in the background to ensure continuous protection.

About CrowdStrike
CrowdStrike is a cybersecurity technology company renowned for its cloud-delivered endpoint protection platform, which leverages AI, machine learning, and behavioral analysis to detect and prevent cyber threats. CrowdStrike's primary product, Falcon, is designed to stop breaches by providing advanced threat detection, prevention, and response capabilities.

What Attack It Is
Attackers often target security solutions like CrowdStrike to disable or circumvent their protection mechanisms. By deleting or modifying CrowdStrike's driver files, an attacker can potentially disable the endpoint protection, leaving the system vulnerable to further attacks, data breaches, or other malicious activities.

How It Works
File Monitoring: The script continuously monitors specific CrowdStrike driver files for any changes in their hash values, which indicate modifications.
Deletion of Malicious Files: If the script detects any unwanted CrowdStrike driver files, it will attempt to delete them.
Alert and Shutdown: If changes to the files are detected, the script will alert the user with a popup message and then shut down the system to prevent further tampering.
Logging: All actions and detected issues are logged for further analysis and review by IT administrators.
Script Usage
Clone the Repository:

sh
Copy code
git clone https://github.com/yourusername/CrowdStrike-removal-and-monitoring.git
cd CrowdStrike-removal-and-monitoring
Run the Script in PowerShell:

Ensure the script is run with administrative privileges to allow it to monitor and modify system files.
sh
Copy code
.\monitoring-script.ps1
Log File:

The script logs actions to C:\logs\SafeModeScript.log for review and auditing.
Enhanced PowerShell Script
powershell
Copy code
# Function to log messages
function Log-Message {
    param (
        [string]$message
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "$timestamp - $message" | Out-File -FilePath "C:\logs\SafeModeScript.log" -Append
    Write-Host $message
}

# Function to calculate file hash
function Get-Hash {
    param (
        [string]$filePath
    )
    if (Test-Path $filePath) {
        return (Get-FileHash -Algorithm SHA256 -Path $filePath).Hash
    } else {
        return $null
    }
}

# Function to delete CrowdStrike driver files
function Delete-CrowdStrikeFiles {
    $directory = "C:\Windows\System32\drivers\CrowdStrike"
    $pattern = "C-00000291*.sys"

    Log-Message "Attempting to delete CrowdStrike driver files in $directory matching pattern $pattern..."

    if (Test-Path $directory) {
        $files = Get-ChildItem -Path $directory -Filter $pattern
        if ($files.Count -eq 0) {
            Log-Message "No files matching pattern $pattern found in $directory."
        } else {
            foreach ($file in $files) {
                try {
                    Remove-Item -Path $file.FullName -Force
                    Log-Message "Deleted file: $($file.FullName)"
                } catch {
                    Log-Message "Error deleting file $($file.FullName): $_"
                }
            }
        }
    } else {
        Log-Message "Directory not found: $directory"
    }
}

# Function to show popup message and shut down system
function Alert-And-Shutdown {
    Add-Type -AssemblyName PresentationFramework
    [System.Windows.MessageBox]::Show('An attacker is trying to attack with CrowdStrike. Please shut down the system and call your IT Admin.', 'Security Alert', [System.Windows.MessageBoxButton]::OK, [System.Windows.MessageBoxImage]::Warning)
    Log-Message "Alert: An attacker is trying to attack with CrowdStrike. Shutting down the system."
    Stop-Computer -Force
}

# Background monitoring function
function Monitor-Files {
    $directory = "C:\Windows\System32\drivers\CrowdStrike"
    $pattern = "C-00000291*.sys"
    $initialHashes = @{}

    if (Test-Path $directory) {
        $files = Get-ChildItem -Path $directory -Filter $pattern
        foreach ($file in $files) {
            $initialHashes[$file.FullName] = Get-Hash -filePath $file.FullName
        }
    }

    Log-Message "Initial file hashes calculated."

    # Periodic check
    while ($true) {
        Start-Sleep -Seconds 60  # Adjust the interval as needed

        $currentHashes = @{}
        if (Test-Path $directory) {
            $files = Get-ChildItem -Path $directory -Filter $pattern
            foreach ($file in $files) {
                $currentHashes[$file.FullName] = Get-Hash -filePath $file.FullName
            }
        }

        foreach ($filePath in $currentHashes.Keys) {
            if ($initialHashes[$filePath] -ne $currentHashes[$filePath]) {
                Alert-And-Shutdown
            }
        }
    }
}

# Main script
try {
    Log-Message "Script started by TeamX."
    Start-Job -ScriptBlock {
        Monitor-Files
    }
    Log-Message "Background monitoring started."
    
    Delete-CrowdStrikeFiles
    Log-Message "Finished deleting CrowdStrike files."
    Log-Message "Thank you from TeamX."
} catch {
    Log-Message "Script encountered an error: $_"
    exit 1
}
This repository provides a robust solution for monitoring and securing systems against tampering with CrowdStrike driver files. The script ensures continuous protection by operating in the background and provides administrators with alerts and logs to take necessary actions.
