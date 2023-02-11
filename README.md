# clean-system-server
clean-system-server 

Esse código tem como objetivo limpar a pasta de cache do SCCM (System Center Configuration Manager). Ele faz isso verificando os arquivos na pasta de cache e os objetos WMI associados, e excluindo aqueles que não são referenciados há mais de 30 dias. Além disso, ele remove pastas e objetos WMI órfãos que não têm correspondência na pasta de cache no disco.

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
