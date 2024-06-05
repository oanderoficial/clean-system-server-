<h1> clean-system-server </h1>


<p> Esse código tem como objetivo limpar a pasta de cache do SCCM (System Center Configuration Manager). Ele faz isso verificando os arquivos na pasta de cache e os objetos WMI associados, e excluindo aqueles que não são referenciados há mais de 30 dias. Além disso, ele remove pastas e objetos WMI órfãos que não têm correspondência na pasta de cache no disco. </p>

<strong> Obter o Caminho do Cache do SCCM: </strong>

```ps1
$SCCMCachePath = ([wmi]"ROOT\ccm\SoftMgmtAgent:CacheConfig.ConfigKey='Cache'").Location
```
<p> Esta linha usa o WMI (Windows Management Instrumentation) para recuperar a localização do cache do SCCM do registro. </p>

<strong> Verificar a Existência do Caminho: </strong>

```ps1
if (!(Test-Path $SCCMCachePath)) {...}
```

<p> Este bloco verifica se o caminho do cache recuperado existe e é acessível usando o cmdlet  <strong>Test-Path </strong>.</p>
<p> Se não, uma mensagem de erro é escrita e o script é encerrado.</p>

<strong> Obter Objetos de Cache Obsoletos: </strong>
```ps1
$StaleCacheObjects = Get-WmiObject -Query "SELECT * FROM CacheInfoEx" -Namespace "ROOT\ccm\SoftMgmtAgent" | Where-Object { ... }
```
<p> Esta linha recupera todas as informações do cache usando <strong>Get-WmiObject </strong> com a classe <strong>CacheInfoEx </strong> no namespace <strong>ROOT\ccm\SoftMgmtAgent </strong>. </p>
<p>A cláusula <strong>Where-Object </strong> subsequente filtra os resultados para objetos onde a diferença entre a data atual e a propriedade <strong>LastReferenced</strong>  (convertida em um objeto <strong>datetime</strong>) é maior que 30 dias. Essencialmente, ele encontra objetos de cache não acessados em mais de 30 dias.</p>

<strong> Excluir Objetos Obsoletos do Disco: </strong>
```ps1
foreach ($StaleCacheObject in $StaleCacheObjects) {... }
```
<p>Este loop itera por cada objeto de cache obsoleto encontrado na etapa anterior.</p>
<p>Dentro do loop:</p>

```ps1 
Remove-Item -Path $StaleCacheObject.Location -Recurse -Force 
```
<p>Tenta excluir a localização do objeto (o arquivo ou pasta) do disco.</p>

<p> <strong>-Recurse </strong> garante a exclusão de subpastas dentro da localização do objeto de cache.</p>
<p> <strong>-Force </strong> ignora solicitações de confirmação durante a exclusão.</p>
<p>O bloco <strong>try...catch </strong> lida com exceções potenciais (erros) durante a exclusão.</p>
<p>Se um erro ocorrer, ele grava uma mensagem de erro com a localização do objeto.</p>

<strong>  Excluir Objetos Obsoletos do WMI (Opcional): </strong>

<p>O bloco <strong>try...catch</strong> tenta excluir os objetos de cache obsoletos do repositório WMI usando <strong>Remove-WmiObject</strong>. </p>

<p>Esta etapa é opcional e pode não ser necessária dependendo de seus requisitos.</p>
<p>Se um erro ocorrer, ele grava uma mensagem de erro genérica sobre a falha na exclusão do WMI.</p>

<h2> <strong> Código </strong> </h2>

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

# Limpeza de Atualizações do Windows

<strong> Limpeza de Atualizações do Windows: </strong> 
<br>
<br>
<strong> Reduzir o Tamanho da Pasta WinSxS </strong> 
<br>
- Abra o Prompt de Comando como administrador.
- Execute os seguintes comandos para limpar as atualizações
<br>

<strong>  Analisar a pasta WinSxS e fornecer um relatório sobre o uso de espaço: </strong>

```cmd
Dism.exe /online /Cleanup-Image /AnalyzeComponentStore
```
<strong> Remover os componentes obsoletos e reduzir o tamanho da pasta: </strong>
```cmd
DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```
<strong> Remove pacotes de service pack substituídos: </strong> 
```cmd
dism.exe /online /Cleanup-Image /SPSuperseded
```
<strong> Referência: https://learn.microsoft.com/pt-br/windows-hardware/manufacture/desktop/clean-up-the-winsxs-folder?view=windows-11 </strong>
