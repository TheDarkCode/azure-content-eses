<properties
   pageTitle="Información general de Configuración de estado deseado de Azure | Microsoft Azure"
   description="Información general para usar el controlador de extensiones de Microsoft Azure en la Configuración de estado deseado de PowerShell. Incluidos los requisitos previos, arquitectura, cmdlets, etc."
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="08/24/2016"
   ms.author="zachal"/>

# Introducción al controlador de extensiones de configuración de estado deseado de Azure #

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

El agente de máquina virtual de Azure y las extensiones asociadas forman parte de los servicios de infraestructura de Microsoft Azure. Las extensiones de máquina virtual son componentes de software que amplían la funcionalidad de la máquina virtual y simplifican diversas operaciones de administración de máquinas virtuales. Por ejemplo, puede utilizar la extensión VMAccess para restablecer una contraseña de administrador o la extensión de script personalizado para ejecutar un script en la máquina virtual.

Este artículo presenta la extensión de configuración de estado deseado (DSC) de PowerShell para las máquinas virtuales de Azure como parte del SDK de Azure PowerShell. Puede usar cmdlets nuevos para cargar y aplicar una configuración de DSC de PowerShell en una máquina virtual de Azure habilitada con la extensión DSC de PowerShell. La extensión DSC de PowerShell llama a DSC de PowerShell para aplicar la configuración de DSC recibida en la máquina virtual. Esta funcionalidad también está disponible a través del Portal de Azure.

## Requisitos previos ##
**Máquina local** Para interactuar con la extensión de máquina virtual de Azure, será necesario usar el Portal de Azure o el SDK de Azure PowerShell.

**Agente invitado** Será preciso que la máquina virtual de Azure afectada por la configuración del DSC tenga un SO que admita Windows Management Framework (WMF) 4.0 o 5.0. La lista completa de las versiones compatibles del SO puede encontrarse en [Release history for the Azure DSC Extension](https://blogs.msdn.microsoft.com/powershell/2014/11/20/release-history-for-the-azure-dsc-extension/) (Historial de versiones de la extensión DSC de Azure).

## Términos y conceptos ##
Esta guía presupone que está familiarizado con los conceptos siguientes:

Configuración: un documento de configuración de DSC.

Nodo: un destino para una configuración de DSC. En este documento, "nodo" siempre hace referencia a una máquina virtual de Azure.

Datos de configuración: un archivo .psd1 que contiene datos del entorno de una configuración

## Información general acerca de la arquitectura ##

La extensión DSC de Azure usa el marco del agente de máquina virtual de Azure para entregar y aplicar las configuraciones de DSC que se ejecutan en las máquinas virtuales de Azure, así como informar sobre estas. La extensión DSC espera un archivo .zip que contiene al menos un documento de configuración y un conjunto de parámetros proporcionados a través del SDK de Azure PowerShell o del Portal de Azure.

La primera vez que se llama a la extensión, se ejecuta un proceso de instalación. Dicho proceso instala una versión de Windows Management Framework (WMF) como se define a continuación:

1. Si el SO de la máquina virtual de Azure es Windows Server 2016, no se realiza ninguna acción. Windows Server 2016 ya tiene instalada la versión más reciente de PowerShell.
2. Si está especificada la propiedad `wmfVersion`, se instala esta versión de WMF, salvo que sea incompatible con el SO de la máquina virtual.
3. Si no hay ninguna propiedad `wmfVersion` especificada, se instala la versión aplicable más reciente de WMF.

La instalación de WMF requiere un reinicio. Después del reinicio, la extensión descarga el archivo .zip especificado en la propiedad `modulesUrl`. Si esta ubicación se encuentra en Almacenamiento de blobs de Azure, se puede especificar un token de SAS en la propiedad `sasToken` para acceder al archivo. Después de que el archivo .zip se descargue y descomprima, se ejecuta la función de configuración definida en `configurationFunction` para generar el archivo MOF. Luego, la extensión ejecuta `Start-DscConfiguration -Force` en el archivo MOF generado. La extensión de captura de salida y la escribe de nuevo en el canal de estado de Azure. A partir de ese momento, el LCM de DSC controla la supervisión y la corrección de la forma habitual.

## Cmdlets de PowerShell ##

Los cmdlets de PowerShell se pueden utilizar con ARM o ASM para empaquetar, publicar y supervisar las implementaciones de la extensión DSC. Los siguientes cmdlets enumerados son los módulos ASM, pero "Azure" se puede reemplazar por "AzureRm" para utilizar el modelo ARM. Por ejemplo, `Publish-AzureVMDscConfiguration` utiliza ASM, mientras que `Publish-AzureRmVMDscConfiguration` usa ARM.

`Publish-AzureVMDscConfiguration` toma un archivo de configuración, lo examina en busca de recursos de DSC dependientes y crea un archivo .zip que contiene la configuración y los recursos de DSC necesarios para aplicar la configuración. Puede crear también el paquete localmente mediante el parámetro `-ConfigurationArchivePath`. En caso contrario, publica el archivo .zip en Almacenamiento de blobs de Azure y lo protege con un token de SAS.

El archivo .zip que crea este cmdlet tiene el script de configuración. ps1 en la raíz de la carpeta de archivos. Los recursos tienen la carpeta del módulo colocada en la carpeta de archivos.

`Set-AzureVMDscExtension` inserta la configuración que necesita la extensión DSC de PowerShell en un objeto de configuración de máquina virtual, que posteriormente puede aplicarse a una máquina virtual de Azure con `Update-AzureVM`.

`Get-AzureVMDscExtension` recupera el estado de la extensión DSC de una máquina virtual concreta.

`Get-AzureVMDscExtensionStatus` recupera el estado de la configuración de DSC aplicada por el controlador de extensiones DSC. Esta acción puede realizarse en una sola máquina virtual o en un grupo de máquinas virtuales.

`Remove-AzureVMDscExtension` quita el controlador de extensiones de una máquina virtual dada. Este cmdlet **no** quita la configuración, ni desinstala WMF ni cambia la configuración aplicada en la máquina virtual. Solo quita el controlador de extensiones.

**Principales diferencias entre los cmdlets de ASM y ARM**

- Los cmdlets de ARM son sincrónicos. Los cmdlets de ASM son asincrónicos.
- ResourceGroupName, VMName, ArchiveStorageAccountName, Version y Location son parámetros nuevos necesarios.
- ArchiveResourceGroupName es un nuevo parámetro opcional de ARM. Se puede especificar este parámetro cuando la cuenta de almacenamiento pertenezca a un grupo de recursos que no sea el mismo que en el que se crea la máquina virtual.
- ConfigurationArchive se llama ArchiveBlobName en ARM
- ContainerName se llama ArchiveContainerName en ARM
- StorageEndpointSuffix se llama ArchiveStorageEndpointSuffix in ARM
- El modificador AutoUpdate se ha agregado a ARM para habilitar la actualización automática del controlador de extensiones a la versión más reciente cuando esté disponible, y tal como esté disponible. Tenga en cuenta que este parámetro puede provocar que la máquina virtual se reinicie cuando se lance una nueva versión de WMF.


## Funcionalidad del Portal de Azure ##
Vaya a una máquina virtual clásica. En Configuración -> General, haga clic en "Extensiones". Se crea un nuevo panel. Haga clic en "Agregar" y seleccione DSC de PowerShell.

El portal necesita una entrada. **Script o módulos de configuración**: este campo es obligatorio. Requiere un archivo .ps1 que contenga un script de configuración o un archivo .zip con un script de configuración .ps1 en la raíz y todos los recursos dependientes en las carpetas de módulos dentro del archivo .zip. Se puede crear con el cmdlet `Publish-AzureVMDscConfiguration -ConfigurationArchivePath` que se incluye en el SDK de Azure PowerShell. El archivo .zip se carga en el almacenamiento de blobs de usuario protegido por un token de SAS.

**Archivo PSD1 de datos de configuración**: este campo es opcional. Si la configuración requiere un archivo de datos de configuración en .psd1, utilice este campo para seleccionarlo y cargarlo en el almacenamiento de blobs de usuario, donde está protegido por un token de SAS.
 
**Nombre completo del módulo de configuración**: los archivos .ps1 pueden tener varias funciones de configuración. Escriba el nombre del script .ps1 de configuración seguido de un '' y del nombre de la función de configuración. Por ejemplo, si su script .ps1 tiene el nombre "configuration.ps1" y la configuración es "IisInstall", escriba: `configuration.ps1\IisInstall`

**Argumentos de configuración**: si la función de configuración adopta argumentos, escríbalos aquí con el formato `argumentName1=value1,argumentName2=value2`. Tenga en cuenta que se trata de un formato distinto a cómo se aceptan argumentos de configuración a través de los cmdlets de PowerShell o las plantillas de Resource Manager.

## Introducción ##

La extensión DSC de Azure toma documentos de configuración de DSC y los aplica a máquinas virtuales de Azure. A continuación encontrará un ejemplo sencillo de una configuración. Guárdelo localmente como "IisInstall.ps1":

```powershell
configuration IISInstall 
{ 
    node "localhost"
    { 
        WindowsFeature IIS 
        { 
            Ensure = "Present" 
            Name = "Web-Server"                       
        } 
    } 
}
```

Los siguientes pasos colocan el script IisInstall.ps1 en la máquina virtual especificada, ejecutan la configuración y devuelven información acerca del estado.
 
```powershell
#Azure PowerShell cmdlets are required
Import-Module Azure

#Use an existing Azure Virtual Machine, 'DscDemo1'
$demoVM = Get-AzureVM DscDemo1

#Publish the configuration script into user storage.
Publish-AzureVMDscConfiguration -ConfigurationPath ".\IisInstall.ps1" -StorageContext $storageContext -Verbose -Force

#Set the VM to run the DSC configuration
Set-AzureVMDscExtension -VM $demoVM -ConfigurationArchive "demo.ps1.zip" -StorageContext $storageContext -ConfigurationName "runScript" -Verbose

#Update the configuration of an Azure Virtual Machine
$demoVM | Update-AzureVM -Verbose

#check on status
Get-AzureVMDscExtensionStatus -VM $demovm -Verbose
```

## Registro ##

Los registros se colocan en:

C:\\WindowsAzure\\Logs\\Plugins\\Microsoft.Powershell.DSC[número de versión]

## Pasos siguientes ##

Para más información sobre DSC de PowerShell, [visite el centro de documentación de PowerShell](https://msdn.microsoft.com/powershell/dsc/overview).

Para buscar otras funcionalidades que se puedan administrar con DSC de PowerShell, [examine la Galería de PowerShell](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0) para encontrar más recursos de DSC.

Para más información sobre cómo pasar parámetros confidenciales a configuraciones, consulte [Cómo pasar las credenciales al controlador de extensiones de la DSC de Azure](virtual-machines-windows-extensions-dsc-credentials.md).

<!---HONumber=AcomDC_0824_2016-->