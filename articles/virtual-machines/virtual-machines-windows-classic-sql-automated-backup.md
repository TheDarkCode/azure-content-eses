<properties
	pageTitle="Copia de seguridad automatizada para máquinas virtuales SQL Server (implementación clásica) | Microsoft Azure"
	description="Explica la característica Copia de seguridad automatizada para SQL Server que se ejecuta en Máquinas virtuales de Azure mediante Resource Manager. "
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management" />
<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="07/14/2016"
	ms.author="jroth" />

# Copia de seguridad automatizada para SQL Server en máquinas virtuales de Azure (implementación clásica)

> [AZURE.SELECTOR]
- [Resource Manager](virtual-machines-windows-sql-automated-backup.md)
- [Clásico](virtual-machines-windows-classic-sql-automated-backup.md)

Copia de seguridad automatizada configura automáticamente [Copia de seguridad administrada para Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx) para todas las bases de datos existentes y nuevas en una máquina virtual de Azure que ejecuta SQL Server 2014 Standard y Enterprise. Esto le permite configurar copias de seguridad de datos normales que utilizan el almacenamiento de blobs de Azure. Copia de seguridad automatizada se basa en la [Extensión Agente de IaaS de SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] 
Para ver la versión de Resource Manager de este artículo, consulte [Automated Backup for SQL Server in Azure Virtual Machines (Resource Manager)](virtual-machines-windows-sql-automated-backup.md) (Copia de seguridad automatizada para SQL Server en máquinas virtuales de Azure [Resource Manager]).

## Requisitos previos

Para utilizar Copia de seguridad automatizada, tenga en cuenta los siguientes requisitos previos:

**Sistema operativo**:

- Windows Server 2012
- Windows Server 2012 R2

**Edición/versión de SQL Server**:

- SQL Server 2014 Standard
- SQL Server 2014 Enterprise

**Configuración de base de datos**:

- Las bases de datos de destino deben utilizar el modelo de recuperación completa

**Azure PowerShell**:

- [Instale los comandos de Azure PowerShell más recientes](../powershell-install-configure.md) si planea configurar Copia de seguridad automatizada con PowerShell.

>[AZURE.NOTE] Copia de seguridad automatizada se basa en la Extensión Agente de IaaS de SQL Server. Las imágenes actuales de la galería de máquinas virtuales de SQL agregan esta extensión de manera predeterminada. Para más información, consulte la [extensión Agente de IaaS de SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

## Settings

En la siguiente tabla se describen las opciones que pueden configurarse para Copia de seguridad automatizada. Los pasos de configuración reales varían si usa el Portal de Azure o comandos de Windows PowerShell de Azure.

|Configuración|Intervalo (valor predeterminado)|Descripción|
|---|---|---|
|**Copia de seguridad automatizada**|Habilitar/deshabilitar (deshabilitado)|Habilita o deshabilita la copia de seguridad automatizada para una máquina virtual de Azure que ejecuta SQL Server 2014 Standard o Enterprise.|
|**Período de retención**|1-30 días (30 días)|El número de días para retener una copia de seguridad.|
|**Storage Account**|Cuenta de almacenamiento de Azure (la cuenta de almacenamiento creada para la máquina virtual especificada)|Una cuenta de almacenamiento de Azure que usar para almacenar archivos de Copia de seguridad automatizada en Almacenamiento de blobs. Se crea un contenedor en esta ubicación para guardar todos los archivos de copia de seguridad. La convención de nomenclatura del archivo de copia de seguridad incluye la fecha, la hora y el nombre de máquina.|
|**Cifrado**|Habilitar/deshabilitar (deshabilitado)|Habilita o deshabilita el cifrado. Cuando se habilita el cifrado, los certificados usados para restaurar la copia de seguridad se ubican en la cuenta de almacenamiento especificada en el mismo contenedor de copia de seguridad automatizada con la misma convención de nomenclatura. Si la contraseña cambia, se genera un nuevo certificado con esa contraseña, pero el certificado antiguo permanece para restaurar copias de seguridad anteriores.|
|**Password**|Texto de contraseña (ninguno)|Una contraseña para claves de cifrado. Esto solo es necesario si se habilita el cifrado. Para restaurar una copia de seguridad cifrada, debe disponer de la contraseña correcta y del certificado relacionado que se usó en el momento en el que se realizó la copia de seguridad.|

## Configuración en el Portal
Puede usar el Portal de Azure para configurar Copia de seguridad automatizada durante el aprovisionamiento o para las máquinas virtuales existentes.

### Nuevas máquinas virtuales
Use el Portal de Azure para configurar la opción Copia de seguridad automatizada cuando cree una nueva máquina virtual de SQL Server 2014 en el modelo de implementación clásica.

La siguiente captura del Portal de Azure muestra estas opciones en **CONFIGURACIÓN OPCIONAL** | **COPIA DE SEGURIDAD AUTOMATIZADA DE SQL**.

![Configuración de Copia de seguridad automática de SQL en el Portal de Azure](./media/virtual-machines-windows-classic-sql-automated-backup/IC778483.jpg)

### Máquinas virtuales existentes
Para las máquinas virtuales de SQL Server 2014, seleccione la configuración **Copia de seguridad automática** en la sección **Configuración** de las propiedades de la máquina virtual. En la ventana **Copia de seguridad automatizada** puede habilitar la característica, configurar el período de retención, seleccionar la cuenta de almacenamiento y configurar el cifrado. Esto se muestra en la siguiente captura de pantalla.

![Configuración de Copia de seguridad automatizada en el Portal de Azure](./media/virtual-machines-windows-classic-sql-automated-backup/IC792133.jpg)

>[AZURE.NOTE] Cuando habilita Copia de seguridad automatizada por primera vez, Azure configura el agente de Iaas de SQL Server en segundo plano. Durante este tiempo, es posible que el Portal de Azure no muestre que se ha configurado Copia de seguridad automatizada. Espere unos minutos hasta que el agente se instale y configure. Después, el Portal de Azure mostrará la nueva configuración.

## Configuración con PowerShell

En el siguiente ejemplo de PowerShell, se configura Copia de seguridad automatizada para una máquina virtual de SQL Server 2014 existente. El comando **New-AzureVMSqlServerAutoBackupConfig** configura los valores de Copia de seguridad automatizada para almacenar copias de seguridad en la cuenta de almacenamiento de Azure especificada por la variable $storageaccount. Estas copias de seguridad se conservarán durante 10 días. El comando **Set-AzureVMSqlServerExtension** actualiza la máquina virtual de Azure especificada con esta configuración.

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM

La instalación y configuración del agente de Iaas de SQL Server puede tardar algunos minutos.

Para habilitar el cifrado, modifique el script anterior para pasar el parámetro EnableEncryption junto con una contraseña (cadena segura) para el parámetro CertificatePassword. El siguiente script habilita la configuración de Copia de seguridad automatizada en el ejemplo anterior y agrega cifrado.

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $password = "P@ssw0rd"
    $encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10 -EnableEncryption -CertificatePassword $encryptionpassword

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM

Para deshabilitar la copia de seguridad automática, ejecute el script sin el parámetro **-Enable** en **New-AzureVMSqlServerAutoBackupConfig**. Al igual que la instalación, la deshabilitación de Copia de seguridad automatizada puede tardar algunos minutos.

>[AZURE.NOTE] La deshabilitación y la desinstalación del agente de Iaas de SQL Server no quita los ajustes de Copia de seguridad administrada configurados anteriormente. Debe deshabilitar Copia de seguridad automatizada antes de deshabilitar o desinstalar el agente de Iaas de SQL Server.

## Pasos siguientes

Copia de seguridad automatizada configura Copia de seguridad administrada en máquinas virtuales de Azure. Por lo tanto, es importante [revisar la documentación de la Copia de seguridad administrada](https://msdn.microsoft.com/library/dn449496.aspx) para comprender el comportamiento y las implicaciones.

Puede encontrar directrices adicionales sobre la copia de seguridad y la restauración para SQL Server en máquinas virtuales de Azure en el siguiente tema: [Copias de seguridad y restauración para SQL Server en Máquinas virtuales de Azure](virtual-machines-windows-sql-backup-recovery.md).

Para más información acerca de otras tareas de automatización disponibles, consulte la [extensión Agente de IaaS de SQL Server](virtual-machines-windows-classic-sql-server-agent-extension.md).

Para más información sobre cómo ejecutar SQL Server en Máquinas virtuales de Azure, consulte [Información general sobre SQL Server en máquinas virtuales de Azure](virtual-machines-windows-sql-server-iaas-overview.md).

<!---HONumber=AcomDC_0720_2016-->