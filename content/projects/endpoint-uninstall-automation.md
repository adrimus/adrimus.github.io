---
date: '2025-05-17T07:18:22+02:00'
draft: false
title: '🛠️ Automating Sophos Uninstall Across Hundreds of PCs—Without Disrupting Users'
---

Migrating from Sophos to Defender for Endpoint came with a major challenge: tamper protection. To tackle it, I built a PowerShell module that interacts with the [Sophos Central API](https://developer.sophos.com/docs/endpoint-v1/1/overview) to disable tamper protection remotely. You can check it out on [GitHub](https://github.com/adrimus/pssophoscentral).

## TL;DR

This setup saved countless hours, avoided manual intervention, and kept users blissfully unaware—just the way it should be. If you’re facing a similar migration, feel free to explore or fork the modules, I didn't get a chance to finish the module and did not include all the necessary components like paging as I didn't have to deal with more than 200 endpoints. Always happy to connect with others tackling endpoint automation!

## 🔧 “The Orchestrator: Control Script”

Then I created a control script that:

- 🔓 Disables tamper protection via the API
- 🧹 Runs the uninstall script on the target PC
- 📁 Logs results to a file so each PC is only processed once
- 🕒 Runs after hours and checks that no user is signed in—ensuring zero disruption

```powershell
Start-Transcript -Path c:\transcript.txt

param (
    # Array of servers to connect to
    $servers = 'C:\POWERSHELL2\MDM\server.txt',

    # Specifies the path to save the results. If the script is interrupted and rerun, it will skip servers that have already been processed.
    $CsvFile = '~\Results.csv',

    # The script file which is executed remotely on computer
    $ScriptFile = '.\Uninstall-Sohpos.ps1',

    # Another CSV file to record connection errors
    $ConnectionErrors = '~\Errors.csv'
)

Import-Module PSSophosCentral -Force

#region API credentials

# Get password for vault
$passwordPath = Join-Path (Split-Path $profile) SecretStore.vault.credential
$passwordPath
$password = Import-CliXml -Path $passwordPath

# Get credentials
Unlock-SecretStore -Password $password
$clientid = Get-Secret -Name TamperProtection-ClientID -AsPlainText
$clientsecret = Get-Secret -Name TamperProtection-Secret -AsPlainText

#endregion

Get-Content -path $servers

#region Authenticate to Sophos Central and get metadata
try {

    $response = Connect-SophosCentral -clientid $clientid -clientsecret $clientsecret -ErrorAction stop
    $token = $response.access_token

}
catch {

    throw "connection error"

} #try/catch

$metadata = Get-SophosCentralContext -token $token
$TenantId = $metadata.TenantId
$dataregion = $metadata.dataRegion

#endregion

# Get hash table with device ID's
$computerHashTable = Get-EndpointIDHashTable

# Test whether the CSV file exists; if it does, exclude the servers already scanned
if (Test-Path -Path $CsvFile) {
    $csvData = Import-Csv -Path $CsvFile | 
    Select-Object -ExpandProperty PSComputerName -Unique
    $servers = $servers | Where-Object { $_ -notin $csvData }
}

[System.Collections.Generic.List[PSObject]] $Sessions = @()
# Connect to each server and add the session to the $Sessions array list
foreach ($s in $servers) {

    

    $id = $computerHashTable[$s]
    

    if ($null -ne $id) {
        #region Disable Tamper Protection

        Write-output "Disabling Tamper protection for [$s] - Device ID: [$id]"
        Set-TamperProtection -dataregion $dataregion -tenantID $TenantId -token $token -enable:$false -id $id 

        #endregion

        $PSSession = @{
            ComputerName = $s
        }
        try {
            $session = New-PSSession @PSSession -ErrorAction Stop
            $Sessions.Add($session) 
        }
        catch {
            # Add any errors to the connection error CSV file
            [pscustomobject]@{
                ComputerName = $s
                Date         = Get-Date
                ErrorMsg     = $_
            } | Export-Csv -Path $ConnectionErrors -Append
        } #try/catch

    }
    else {
        Write-Output "Device ID not found for [$s]"
        # Output empty object with just the Computername into the results
        $obj = [PSCustomObject]@{
            ComputerName       = $s
            SophosStatus       = "Not found"
            ExitCode           = ""
            PSComputerName     = ""
            RunspaceId         = ""
            PSShowComputerName = ""
        } 
        $obj | Export-Csv -Path $CsvFile -Append
    } #if/else
} #foreach $s

# Execute the script on all remote sessions at once
$Command = @{
    Session  = $Sessions
    FilePath = $ScriptFile
}
$Results = Invoke-Command @Command

# Export the results to CSV
$Results | Export-Csv -Path $CsvFile -Append

# Close and remove the remote sessions
Remove-PSSession -Session $Sessions

Stop-Transcript
```

## 🧼 “The Cleanup Crew: Uninstall Script”

The uninstall script is designed to cleanly remove Sophos components before the Defender rollout. It checks for the presence of Sophos, attempts to uninstall it, and handles tamper protection gracefully. It also has a timer in place to ensure that the process does not hang indefinitely and keeps trying to uninstall within a 90-second window.

If tamper protection is still active, it will log the status and exit without disrupting the user. It will restart once the uninstall process is complete and there is no user logged in.

```powershell
<#
.SYNOPSIS
Uninstall the Sophos Suite
.DESCRIPTION
This script is intended to be run on the client workstation. It would be executed by the control script which would create a session to each workstation
.NOTES
For Core Agent 2022.3 and later, run:     
C:\Program Files\Sophos\Sophos Endpoint Agent\uninstallgui.exe --quiet. 
 
For Windows 10 (x64) and Windows Server 2016 and later running Core Agent 2022.4 and later, run:      
C:\Program Files\Sophos\Sophos Endpoint Agent\SophosUninstall.exe --quiet.
#>
[CmdletBinding()]

param (
    # PUT PARAMETER DEFINITIONS HERE AND DELETE THIS COMMENT.
)

#region verbose

Write-Verbose "[BEGIN ] Starting: $($MyInvocation.Mycommand)"
Write-Verbose "Execution Metadata:"
Write-Verbose "User = $($env:userdomain)\$($env:USERNAME)"

$id = [System.Security.Principal.WindowsIdentity]::GetCurrent()
$IsAdmin = [System.Security.Principal.WindowsPrincipal]::new($id).IsInRole('administrators')

Write-Verbose "Is Admin = $IsAdmin"
Write-Verbose "Computername = $env:COMPUTERNAME"
Write-Verbose "OS = $((Get-CimInstance Win32_Operatingsystem).Caption)"
Write-Verbose "Host = $($host.Name)"
Write-Verbose "PSVersion = $($PSVersionTable.PSVersion)"
Write-Verbose "Runtime = $(Get-Date)"

#endregion

#region Information

Write-Information "Command = $($myinvocation.mycommand)" -Tags Meta
Write-Information "PSVersion = $($PSVersionTable.PSVersion)" -Tags Meta
Write-Information "User = $env:userdomain\$env:username" -tags Meta
Write-Information "Computer = $env:computername" -tags Meta
Write-Information "PSHost = $($host.name)" -Tags Meta
Write-Information "Test Date = $(Get-Date)" -tags Meta

#endregion

#region code

$myObject = [PSCustomObject]@{
    ComputerName = $env:COMPUTERNAME
    SophosStatus = ""
    ExitCode     = ""
}

try {

    if (Test-Path -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\Sophos Endpoint Agent") {

        #region Uninstall 
        # Get the Uninstall string
        $UninstallString = Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\Sophos Endpoint Agent" -Name UninstallString | 
        Select-Object -ExpandProperty UninstallString

        # start timer to allow tamper protection to take effect  
        $timer = [system.diagnostics.stopwatch]::StartNew()

        # Keep trying if exitcode is 5 which means status is still active
        do {
            Write-Verbose "Uninstall using [$UninstallString]"
            $Process = Start-Process -FilePath $UninstallString -ArgumentList "--quiet" -PassThru -Wait
            $exitCode = $Process.ExitCode
            Write-Verbose "Exit code: [$exitCode]"

            # Pause if not successfull
            if ($exitCode -ne 1) {
                Write-Verbose "Waiting 5 Seconds"
                Start-Sleep -Seconds 5
            }
            Write-Verbose "Timer: [$($timer.Elapsed.TotalSeconds)]"
            
        } while ($timer.Elapsed.TotalSeconds -lt 90 -and $exitCode -eq 5) #do/while

        $myObject.ExitCode = $exitCode
        
        # Decide what to do
        switch ($exitcode) {
            1 { $action = "Restart" } # Uninstallation was successful but a reboot is required.  
            2 { $myObject.SophosStatus = "Uninstall Failed" }
            5 { $myObject.SophosStatus = "Uninstallation failed because tamper protection is active" }
            8 { 
                $myObject.SophosStatus = "UNINSTALL PENDING REBOOT"
                $action = "Restart" 
            }
        } #Switch
        #endregion

        #region Restart
        if ($action -eq "Restart") {
            $user = Get-CimInstance -Class Win32_ComputerSystem | Select-Object -ExpandProperty Username -ErrorAction stop

            if (-not ($user)) {

                $myObject.SophosStatus = "Uninstalled"
                Restart-Computer
                
            }
            else {
                $myObject.SophosStatus = "Could not Restart, User logged in"
            }
        } #if
        #endregion
        
    }
    else {
        $myObject.SophosStatus = "Uninstalled" 
    } #if

}
catch {

    $myObject.SophosStatus = "Error: [$($PSItem.Exception.Message)]"
    
} #try/catch

# Output at the end
$myObject

#endregion
```