# clean-system-server
clean-system-server 

Esse código tem como objetivo limpar a pasta de cache do SCCM (System Center Configuration Manager). Ele faz isso verificando os arquivos na pasta de cache e os objetos WMI associados, e excluindo aqueles que não são referenciados há mais de 30 dias. Além disso, ele remove pastas e objetos WMI órfãos que não têm correspondência na pasta de cache no disco.

``` ps1

# Get the SCCM cache path
$SCCMCachePath = ([wmi]"ROOT\ccm\SoftMgmtAgent:CacheConfig.ConfigKey='Cache'").Location

# Check if the cache path exists and is accessible
if (!(Test-Path $SCCMCachePath))
{
    Write-Error "The SCCM cache path does not exist or is not accessible: $SCCMCachePath"
    return
}

# Get cache objects that have not been referenced for more than 30 days
$StaleCacheObjects = Get-WmiObject -Query "SELECT * FROM CacheInfoEx" -Namespace "ROOT\ccm\SoftMgmtAgent" | Where-Object { ([datetime](Date) - ([System.Management.ManagementDateTimeConverter]::ToDateTime($_.LastReferenced))).Days -gt 30  }

# Delete stale cache objects from disk
foreach ($StaleCacheObject in $StaleCacheObjects)
{
    try
    {
        Remove-Item -Path $StaleCacheObject.Location -Recurse -Force
    }
    catch [System.Exception]
    {
        Write-Error "Failed to delete stale cache object from disk: $($StaleCacheObject.Location)"
    }
}

# Delete stale cache objects from WMI
try
{
    $StaleCacheObjects | Remove-WmiObject
}
catch [System.Exception]
{
    Write-Error "Failed to delete stale cache objects from WMI."
}

# Get all cached folders from disk
$

```
