<properties
   pageTitle="Cmdlets de PowerShell para Almacenamiento de datos SQL de Azure"
   description="Busque los principales cmdlets de PowerShell para Almacenamiento de datos SQL de Azure, incluidos aquellos para pausar y reanudar una base de datos."
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="08/16/2016"
   ms.author="sonyama;barbkess;mausher"/>

# Cmdlets de PowerShell y API de REST para Almacenamiento de datos SQL

Muchas tareas de administración de Almacenamiento de datos SQL se pueden administrar mediante los cmdlets de Azure PowerShell o las API de REST. A continuación se muestran algunos ejemplos de cómo usar comandos de PowerShell para automatizar tareas comunes en Almacenamiento de datos SQL. Para ver algunos buenos ejemplos de REST, consulte el artículo [Administración de la potencia de proceso en Almacenamiento de datos SQL de Azure (REST)][].

> [AZURE.NOTE]  Para usar Azure PowerShell con Almacenamiento de datos SQL, es necesaria la versión 1.0.3 o posterior de Azure PowerShell. Puede comprobar la versión ejecutando **Get-Module -ListAvailable -Name Azure**. Se puede instalar la versión más reciente desde el [Instalador de plataforma web de Microsoft][]. Para más información sobre cómo instalar la versión más reciente, consulte [Cómo instalar y configurar Azure PowerShell][].

## Introducción a los cmdlets de Azure PowerShell

1. Abra Windows PowerShell.
2. En el símbolo del sistema de PowerShell, ejecute estos comandos para iniciar sesión en Azure Resource Manager y seleccione su suscripción.

    ```PowerShell
    Login-AzureRmAccount
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName "MySubscription"
    ```

## Ejemplo de pausa de Almacenamiento de datos SQL

Pausa una base de datos denominada "Database02" que está hospedada en un servidor cuyo nombre es "Server01". El servidor está en un grupo de recursos de Azure denominado "ResourceGroup1."

```Powershell
Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
```
Este ejemplo, que es una variación, canaliza el objeto recuperado a [Suspend-AzureRmSqlDatabase][]. El resultado es que se pausa la base de datos. El comando final muestra los resultados.

```Powershell
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Suspend-AzureRmSqlDatabase
$resultDatabase
```

## Ejemplo de inicio de Almacenamiento de datos SQL

Reanuda el funcionamiento de una base de datos denominada "Database02" que está hospedada en un servidor cuyo nombre es "Server01". El servidor está en un grupo de recursos denominado "ResourceGroup1."

```Powershell
Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" -DatabaseName "Database02"
```

Este ejemplo, que es una variación, recupera una base de datos denominada "Database02" desde un servidor cuyo nombre es "Server01" que está incluido en un grupo de recursos denominado "ResourceGroup1". Canaliza el objeto recuperado a [Resume-AzureRmSqlDatabase][].

```Powershell
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Resume-AzureRmSqlDatabase
```

> [AZURE.NOTE] Tenga en cuenta que si el servidor es foo.database.windows.net, debe usar "foo" como -ServerName en los cmdlets de Powershell.

## Cmdlets de PowerShell usados con frecuencia

Estos cmdlets de PowerShell se usan con frecuencia con Almacenamiento de datos SQL de Azure.

- [Get-AzureRmSqlDatabase][]
- [Get-AzureRmSqlDeletedDatabaseBackup][]
- [Get-AzureRmSqlDatabaseRestorePoints][]
- [New-AzureRmSqlDatabase][]
- [Remove-AzureRmSqlDatabase][]
- [Restore-AzureRmSqlDatabase][]
- [Resume-AzureRmSqlDatabase][]
- [Select-AzureRmSubscription][]
- [Set-AzureRmSqlDatabase][]
- [Suspend-AzureRmSqlDatabase][]

## Pasos siguientes
Para obtener más ejemplos de PowerShell, consulte:

- [Creación de Almacenamiento de datos SQL con PowerShell][]
- [Restauración de base de datos][]

Para ver una lista de todas las tareas que se pueden automatizar con PowerShell, consulte [Azure SQL Database Cmdlets][] (Cmdlets de Base de datos SQL de Azure). Para ver una lista de todas las tareas que se pueden automatizar con PowerShell, consulte [Operaciones para bases de datos SQL de Azure][].

<!--Image references-->

<!--Article references-->
[Cómo instalar y configurar Azure PowerShell]: ./powershell-install-configure.md
[Creación de Almacenamiento de datos SQL con PowerShell]: ./sql-data-warehouse-get-started-provision-powershell.md
[Restauración de base de datos]: ./sql-data-warehouse-restore-database-powershell.md
[Administración de la potencia de proceso en Almacenamiento de datos SQL de Azure (REST)]: ./sql-data-warehouse-manage-compute-rest-api.md

<!--MSDN references-->
[Azure SQL Database Cmdlets]: https://msdn.microsoft.com/library/mt574084.aspx
[Operaciones para bases de datos SQL de Azure]: https://msdn.microsoft.com/library/azure/dn505719.aspx
[Get-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt603648.aspx
[Get-AzureRmSqlDeletedDatabaseBackup]: https://msdn.microsoft.com/library/mt693387.aspx
[Get-AzureRmSqlDatabaseRestorePoints]: https://msdn.microsoft.com/library/mt603642.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt619339.aspx
[Remove-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt619368.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt693390.aspx
[Resume-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt619347.aspx
<!-- It appears that Select-AzureRmSubscription isn't documented, so this points to Select-AzureSubscription -->
[Select-AzureRmSubscription]: https://msdn.microsoft.com/library/dn722499.aspx
[Set-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt619433.aspx
[Suspend-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt619337.aspx

<!--Other Web references-->
[Instalador de plataforma web de Microsoft]: https://aka.ms/webpi-azps

<!---HONumber=AcomDC_0817_2016-->