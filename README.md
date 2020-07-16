# AzureImageBuilder

VMs created from AIB images do not create successfully
By default, the AIB will also run ‘deprovision’ code at the end of each image customization phase, to ‘generalize’ the image. Generalize is a process where the image is setup, so it can be reused to create multiple VMs, and you can pass in VM settings, such as hostname, username etc. In Windows, AIB executes Sysprep, and in Linux AIB runs ‘waagent -deprovision’.

For Windows, we use a generic Sysprep command, however, it is understood, that this may not be suitable for every successful Windows generalization, so AIB will allow you to customize this command. Please note, AIB is an image automation tool, it is responsible for running Sysprep command successfully, but, you may need different Sysprep commands to make your image reusable. For Linux we use a generic 'waagent -deprovision+user' comand, see here for documentation.

If you are migrating existing customization, and you are using different Sysprep/waagent commands, you can try the image builder generic commands, and if the VM creates fail, use your previous Sysprep/waagent commands.

Note! If AIB creates a Windows custom image successfully, and you create a VM from it, then find the VM will not create successfully (i.e. the VM creation command does not complete/timeouts), you will need to review the Windows Server Sysprep documenation, or raise a support request with the Windows Server Sysprep Customer Services Support team, who can troubleshoot and advise on the correct Sysprep command.

Command Locations and Filenames
Windows: c:\DeprovisioningScript.ps1

Linux: /tmp/DeprovisioningScript.sh
Sysprep Command: Windows
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
Deprovision Command: Linux
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
Overriding the Commands
To override the commands, use the PowerShell or Shell script provisioners to create the command files with the exact file name, and put them in the directories above. AIB will read these commands, these are written out to the AIB logs, ‘customization.log’. See here on how to collect AIB logs:
