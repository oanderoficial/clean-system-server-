# clean-system-server
clean-system-server 

``` ps1

#get CCMCache path
$Cachepath = ([wmi]"ROOT\ccm\SoftMgmtAgent:CacheConfig.ConfigKey='Cache'").Location

#Get Items not referenced for more than 30 days
$OldCache = get-wmiobject -query "SELECT * FROM CacheInfoEx" -namespace "ROOT\ccm\SoftMgmtAgent" | Where-Object { ([datetime](Date) - ([System.Management.ManagementDateTimeConverter]::ToDateTime($_.LastReferenced))).Days -gt 30  }

#delete Items on Disk
$OldCache | % { Remove-Item -Path $_.Location -Recurse -Force -ea SilentlyContinue }
#delete Items on WMI
$OldCache | Remove-WmiObject

#Get all cached Items from Disk
$CacheFoldersDisk = (Get-ChildItem $Cachepath).FullName
#Get all cached Items from WMI
$CacheFoldersWMI = get-wmiobject -query "SELECT * FROM CacheInfoEx" -namespace "ROOT\ccm\SoftMgmtAgent"

#Remove orphaned Folders from Disk
$CacheFoldersDisk | % { if($_ -notin $CacheFoldersWMI.Location) { remove-item -path $_ -recurse -force -ea SilentlyContinue} }

#Remove orphaned WMI Objects
$CacheFoldersWMI| % { if($_.Location -notin $CacheFoldersDisk) { $_ | Remove-WmiObject }}

```
