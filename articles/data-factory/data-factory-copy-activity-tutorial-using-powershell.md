<properties 
	pageTitle="Tutorial: crear una canalización con la actividad de copia con Azure PowerShell | Microsoft Azure" 
	description="En este tutorial, creará una canalización de Data Factory de Azure con una actividad de copia mediante Azure PowerShell." 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="08/01/2016" 
	ms.author="spelluru"/>

# Tutorial: Crear una canalización con la actividad de copia con Azure PowerShell
> [AZURE.SELECTOR]
- [Información general del tutorial](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
- [Uso del Editor de Data Factory.](data-factory-copy-activity-tutorial-using-azure-portal.md)
- [Uso de PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
- [Uso de Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
- [Uso de la API de REST](data-factory-copy-activity-tutorial-using-rest-api.md)
- [Uso de la API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
- [Uso del Asistente para copia](data-factory-copy-data-wizard-tutorial.md)

En el tutorial [Copia de datos de Almacenamiento de blobs en Base de datos SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) se muestra cómo crear y supervisar una factoría de datos de Azure con el [Portal de Azure][azure-portal]. En este tutorial, creará y supervisará una factoría de datos de Azure con cmdlets de Azure PowerShell. La canalización en la factoría de datos que cree en este tutorial usa una actividad de copia para copiar los datos de un blob de Azure a una base de datos SQL de Azure.

La actividad de copia realiza el movimiento de datos en Data Factory de Azure. La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Consulte el artículo [Actividades de movimiento de datos](data-factory-data-movement-activities.md) para obtener más información sobre la actividad de copia.

> [AZURE.IMPORTANT] 
Lea el tema [Información general del tutorial](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) y realice los pasos de requisitos previos antes de completar este tutorial.
>   
> Este artículo no abarca todos los cmdlets de Factoría de datos. Vea [Referencia de cmdlets de factoría de datos](https://msdn.microsoft.com/library/dn820234.aspx) para obtener la documentación completa sobre los cmdlets de la factoría de datos.
  

##Requisitos previos
Además de los requisitos previos que se enumeran en el tema Información general del tutorial, es preciso instalar lo siguiente:

- **Azure PowerShell**. Siga las instrucciones del artículo [Cómo instalar y configurar Azure PowerShell](../powershell-install-configure.md) para instalar Azure PowerShell en su equipo.

##Apartados de este tutorial
En la tabla siguiente se enumeran los pasos que se llevarán a cabo como parte de este tutorial y sus descripciones.

Paso | Description
-----| -----------
[Crear una factoría de datos de Azure](#create-data-factory) | En este paso, creará una factoría de datos de Azure denominada **ADFTutorialDataFactoryPSH**. 
[Crear servicios vinculados](#create-linked-services) | En este paso, creará dos servicios vinculados: **StorageLinkedService** y **AzureSqlLinkedService**. StorageLinkedService vincula el Almacenamiento de Azure y AzureSqlLinkedService vincula la Base de datos SQL de Azure a ADFTutorialDataFactoryPSH.
[Crear conjuntos de datos de entrada y salida](#create-datasets) | En este paso, se definen dos conjuntos de datos (**EmpTableFromBlob** y **EmpSQLTable**). Estos conjuntos de datos se usan como tablas de entrada y salida de la **actividad de copia** en la canalización ADFTutorialPipeline que creará en el siguiente paso.
[Creación y ejecución de una canalización](#create-pipeline) | En este paso, creará una canalización denominada **ADFTutorialPipeline** en la factoría de datos **ADFTutorialDataFactoryPSH**. La canalización tiene una **actividad de copia** que copia datos de un blob de Azure en una tabla de base de datos de Azure de salida.
[Supervisar conjuntos de datos y canalización](#monitor-pipeline) | En este paso, supervisará los conjuntos de datos y la canalización con Azure PowerShell.

## Creación de Data Factory
En este paso, use PowerShell de Azure para crear una factoría de datos de Azure llamada **ADFTutorialDataFactoryPSH**.

1. Inicie **PowerShell** y ejecute el comando siguiente. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.
	1. Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en el Portal de Azure.
	
			Login-AzureRmAccount   
	2. Ejecute el siguiente comando para ver todas las suscripciones para esta cuenta.

			Get-AzureRmSubscription 
	3. Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Reemplace **& lt; NameOfAzureSubscription**& gt; por el nombre de su suscripción de Azure.

			Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext

3. Cree un grupo de recursos de Azure con el nombre: **ADFTutorialResourceGroup** ejecutando el siguiente comando.
   
		New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"

	En algunos de los pasos de este tutorial se supone que se usa el grupo de recursos denominado **ADFTutorialResourceGroup**. Si usa un otro grupo de recursos, deberá usarlo en lugar de ADFTutorialResourceGroup en este tutorial.
4. Ejecute el cmdlet **New-AzureRmDataFactory** para crear una factoría de datos con el nombre: **ADFTutorialDataFactoryPSH**.

		New-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"

	
Tenga en cuenta lo siguiente:
 
- El nombre del generador de datos de Azure debe ser único global. Si recibe el siguiente error, cambie el nombre (por ejemplo, yournameADFTutorialDataFactoryPSH). Use este nombre en lugar de ADFTutorialFactoryPSH mientras lleva a cabo los pasos de este tutorial. Consulte el tema [Data Factory: reglas de nomenclatura](data-factory-naming-rules.md) para conocer las reglas de nomenclatura para los artefactos de Data Factory.
	
		Data factory name “ADFTutorialDataFactoryPSH” is not available
- Para crear instancias de Data Factory, debe ser administrador o colaborador de la suscripción de Azure.
- El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, hacerse públicamente visible.
- Si recibe el error: "**La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**", realice una de las acciones siguientes e intente publicarla de nuevo:

	- En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory.
		
			Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
	
		Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory.
	
			Get-AzureRmResourceProvider
	- Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.

## Crear servicios vinculados
Los servicios vinculados vinculan almacenes de datos o servicios de proceso con una factoría de datos de Azure. Un almacén de datos puede ser Almacenamiento de Azure, Base de datos SQL de Azure o Base de datos SQL Server local que contenga los datos de entrada o almacene los datos de salida de una canalización de Factoría de datos. Un servicio de proceso es el servicio que procesa datos de entrada y genera datos de salida.

En este paso, creará dos servicios vinculados: **StorageLinkedService** y **AzureSqlLinkedService**. El servicio vinculado StorageLinkedService vincula una cuenta de almacenamiento de Azure y AzureSqlLinkedService vincula una base de datos SQL de Azure a la factoría de datos: **ADFTutorialDataFactoryPSH**. Más adelante en este tutorial creará una canalización que copia datos de un contenedor de blobs de StorageLinkedService en una tabla SQL de AzureSqlLinkedService.

### Creación de un servicio vinculado para una cuenta de almacenamiento de Azure
1.	Cree un archivo JSON con el nombre **StorageLinkedService.json** en **C:\\ADFGetStartedPSH** con el siguiente contenido. Si todavía no existe, cree la carpeta ADFGetStartedPSH.

			{
		  		"name": "StorageLinkedService",
		  		"properties": {
	    			"type": "AzureStorage",
		    		"typeProperties": {
		      			"connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
		    		}
		  		}
			}

	Reemplace **accountname** y **accountkey** por el nombre y la clave de su cuenta de Almacenamiento de Azure.
2.	En **PowerShell de Azure**, cambie a la carpeta **ADFGetStartedPSH**.
3.	Use el cmdlet **New-AzureRmDataFactoryLinkedService** para crear un servicio vinculado. Tanto cmdlet como otros cmdlets de Data Factory que usa en este tutorial requieren que se pasen los valores de los parámetros **ResourceGroupName** y **DataFactoryName**. Como alternativa, puede usar **Get-AzureRmDataFactory** para obtener un objeto DataFactory y pasarlo sin necesidad de escribir ResourceGroupName y DataFactoryName cada vez que ejecuta un cmdlet. Ejecute el comando siguiente para asignar el resultado del cmdlet **Get-AzureRmDataFactory** a una variable: **$df**.

		$df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH

4.	Ahora, ejecute el cmdlet **New-AzureRmDataFactoryLinkedService** para crear el servicio vinculado: **StorageLinkedService**.

		New-AzureRmDataFactoryLinkedService $df -File .\StorageLinkedService.json

	Si no hubiera ejecutado el cmdlet **Get-AzureRmDataFactory** y asignado a la salida a la variable **$df**, tendría que especificar valores para los parámetros ResourceGroupName y DataFactoryName de la siguiente forma.
		
		New-AzureRmDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName ADFTutorialDataFactoryPSH -File .\StorageLinkedService.json

	Si cierra Azure PowerShell en medio del tutorial, tendrá que ejecutar el cmdlet Get-AzureRmDataFactory la próxima vez que ejecute Azure PowerShell para completar el tutorial.

### Creación de un servicio vinculado para una base de datos SQL de Azure
1.	Cree un archivo JSON con el nombre AzureSqlLinkedService.json con el siguiente contenido.

			{
				"name": "AzureSqlLinkedService",
				"properties": {
					"type": "AzureSqlDatabase",
					"typeProperties": {
				      	"connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=user@server;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
					}
		  		}
			}

	Reemplace **servername**, **databasename**, **username@servername** y **password** por los nombres del servidor SQL Azure, de la base de datos, de la cuenta de usuario y de la contraseña.

2.	Ejecute el siguiente comando para crear un servicio vinculado.
	
		New-AzureRmDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json

	Confirme que la opción **Permitir el acceso a servicios de Azure** esté activada para el servidor SQL de Azure. Para comprobarla y activarla, realice la siguiente tarea:

	1. Haga clic en el concentrador **EXAMINAR** a la izquierda y haga clic en **Servidores SQL**.
	2. Seleccione un servidor y haga clic en **CONFIGURACIÓN** en la hoja SQL SERVER.
	3. En la hoja **CONFIGURACIÓN**, haga clic en **Firewall**.
	4. En la hoja **Configuración de firewall**, haga clic en **ACTIVADA** para **Permitir el acceso a los servicios de Azure**.
	5. Haga clic en el concentrador **ACTIVO** a la izquierda para cambiar a la hoja **Factoría de datos** en la que está.
	

## Creación de conjuntos de datos

En el paso anterior, creó los servicios vinculados **StorageLinkedService** y **AzureSqlLinkedService** para vincular una cuenta de almacenamiento de Azure y una base de datos SQL de Azure con la factoría de datos: **ADFTutorialDataFactoryPSH**. En este paso, creará conjuntos de datos que representan los datos de entrada y salida de la actividad de copia en la canalización que va a crear en el paso siguiente.

Una tabla es un conjunto de datos rectangular y es el único tipo de conjunto de datos que se admite ahora. La tabla de entrada de este tutorial hace referencia a un contenedor de blobs de Almacenamiento de Azure que apunta a StorageLinkedService. Este conjunto de datos hace referencia a una tabla SQL de la base de datos SQL de Azure que apunta a AzureSqlLinkedService.

### Preparación del almacenamiento de blobs de Azure y la base de datos SQL de Azure para el tutorial
Omita este paso si ha realizado el tutorial del artículo [Copia de datos de Almacenamiento de blobs en Base de datos SQL mediante Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Tiene que realizar los siguientes pasos para preparar el Almacenamiento de blobs de Azure y Base de datos SQL de Azure para este tutorial.
 
* Cree un contenedor de blobs con el nombre **adftutorial** en el almacenamiento de blobs de Azure al que señala **StorageLinkedService**.
* Cree y cargue un archivo de texto, **emp.txt**, como un blob para el contenedor **adftutorial**.
* Cree una tabla llamada **emp** en la Base de datos SQL de Azure a la que apunta **AzureSqlLinkedService**.


1. Inicie el Bloc de notas, pegue el texto siguiente y guárdelo como **emp.txt** en la carpeta **C:\\ADFGetStartedPSH** de su disco duro.

        John, Doe
		Jane, Doe
				
2. Use herramientas como el [Explorador de almacenamiento de Azure](https://azurestorageexplorer.codeplex.com/) para crear el contenedor **adftutorial** y cargar el archivo **emp.txt** en el contenedor.

    ![Explorador de almacenamiento de Azure](media/data-factory-copy-activity-tutorial-using-powershell/getstarted-storage-explorer.png)
3. Use el siguiente script de SQL para crear la tabla **emp** en su Base de datos SQL de Azure.


        CREATE TABLE dbo.emp 
		(
			ID int IDENTITY(1,1) NOT NULL,
			FirstName varchar(50),
			LastName varchar(50),
		)
		GO

		CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID); 

	Si tiene SQL Server 2014 instalado en el equipo: siga las instrucciones del artículo [Paso 2: Conexión con la base de datos SQL de la base de datos de administración de SQL de Azure con SQL Server Management Studio](../sql-database/sql-database-manage-azure-ssms.md) para conectarse al servidor SQL de Azure y ejecutar el script de SQL.

	Si el cliente no tiene permiso para acceder al servidor SQL de Azure, tendrá que configurar el firewall de su servidor SQL de Azure para permitir el acceso desde su máquina (dirección IP). Consulte [este artículo](../sql-database/sql-database-configure-firewall-settings.md) para conocer los pasos que deben darse para configurar el firewall de un servidor SQL de Azure.
		
### Creación de un conjunto de datos de entrada 
Una tabla es un conjunto de datos rectangular y tiene un esquema. En este paso, creará una tabla denominada **EmpBlobTable** que apunta a un contenedor de blobs en Almacenamiento de Azure, representado por el servicio vinculado **StorageLinkedService**. Este contenedor de blobs (**adftutorial**) contiene los datos de entrada en el archivo: **emp.txt**.

1.	Cree un archivo JSON llamado **EmpBlobTable.json** en la carpeta **C:\\ADFGetStartedPSH** con el siguiente contenido:

			{
			  "name": "EmpTableFromBlob",
			  "properties": {
			    "structure": [
			      {
			        "name": "FirstName",
			        "type": "String"
			      },
			      {
			        "name": "LastName",
			        "type": "String"
			      }
			    ],
			    "type": "AzureBlob",
			    "linkedServiceName": "StorageLinkedService",
			    "typeProperties": {
				  "fileName": "emp.txt",
			      "folderPath": "adftutorial/",
			      "format": {
			        "type": "TextFormat",
			        "columnDelimiter": ","
			      }
			    },
			    "external": true,
			    "availability": {
			      "frequency": "Hour",
			      "interval": 1
			    }
			  }
			}
	
	Tenga en cuenta lo siguiente:
	
	- **type** de conjunto de datos está establecido en **AzureBlob**.
	- **linkedServiceName** se establece en **StorageLinkedService**.
	- **folderPath** se establece en el contenedor **adftutorial**.
	- **fileName** se establece en **emp.txt**. Si no especifica el nombre del blob, los datos de todos los blobs del contenedor se consideran datos de entrada.
	- el **tipo** de formato se establece en **TextFormat**
	- Hay dos campos en el archivo de texto: **FirstName** y **LastName** separados por un carácter de coma (**columnDelimiter**)
	- El elemento **availability** está establecido en **hourly** (**frequency** está establecido en **hour** e **interval** en **1**). Por consiguiente, Data Factory busca los datos de entrada cada hora en la carpeta raíz del contenedor de blobs (**adftutorial**) que se ha especificado.

	Si no especifica **fileName** para una **tabla** de **entrada**, todos los archivos o blobs de la carpeta de entrada (**folderPath**) se consideran entradas. Si especifica un nombre de archivo en JSON, solo el archivo o blob especificado se consideran una entrada.
 
	Si no especifica un valor **fileName** para una **tabla de salida**, los archivos generados en la **ruta de la carpeta** se denominan con el siguiente formato: Data.<Guid>.txt (ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.).

	Para establecer **folderPath** y **fileName** de forma dinámica según la hora de **SliceStart**, use la propiedad **partitionedBy**. En el ejemplo siguiente, folderPath usa Year, Month y Day de SliceStart (hora de inicio del segmento que se está procesando) y fileName usa Hour de SliceStart. Por ejemplo, si se está produciendo una división de 2014-10-20T08:00:00, el nombre de carpeta se establece en wikidatagateway/wikisampledataout/2014/10/20 y el nombre de archivo se establece en 08.csv.

			"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
	        "fileName": "{Hour}.csv",
	        "partitionedBy": 
	        [
	        	{ "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
	            { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
	            { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
	            { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 
	        ],

	Para obtener más información sobre las propiedades JSON, vea [Referencia de scripting JSON](http://go.microsoft.com/fwlink/?LinkId=516971).

2.	Ejecute el comando siguiente para crear el conjunto de datos de Factoría de datos.

		New-AzureRmDataFactoryDataset $df -File .\EmpBlobTable.json

### Creación del conjunto de datos de salida
En este paso se crea un conjunto de datos de salida denominado **EmpSQLTable**. Dicho conjunto de datos apunta a una tabla SQL (**emp**) de la base de datos SQL de Azure representada por **AzureSqlLinkedService**. La canalización copia los datos del blob de entrada a la tabla **emp**.

1.	Cree un archivo JSON llamado**EmpSQLTable.json** en la carpeta **C:\\ADFGetStartedPSH** con el siguiente contenido:
		
			{
			  "name": "EmpSQLTable",
			  "properties": {
			    "structure": [
			      {
			        "name": "FirstName",
			        "type": "String"
			      },
			      {
			        "name": "LastName",
			        "type": "String"
			      }
			    ],
			    "type": "AzureSqlTable",
			    "linkedServiceName": "AzureSqlLinkedService",
			    "typeProperties": {
			      "tableName": "emp"
			    },
			    "availability": {
			      "frequency": "Hour",
			      "interval": 1
			    }
			  }
			}

     Tenga en cuenta lo siguiente:
	
	* **type** de conjunto de datos está establecido en **AzureSqlTable**.
	* **linkedServiceName** se establece en **AzureSqlLinkedService**.
	* **tablename** está establecido en **emp**.
	* Hay tres columnas (**ID**, **FirstName** y **LastName**) en la tabla emp de la base de datos. ID es una columna de identidad, por lo que en ella solo se deben especificar **FirstName** y **LastName**.
	* El elemento **availability** está establecido en **hourly** (**frequency** está establecido en **hour** e **interval** en **1**). El servicio Data Factory generará un segmento de datos de salida cada hora en la tabla **emp** de la base de datos SQL de Azure.

2.	Ejecute el comando siguiente para crear el conjunto de datos de Factoría de datos.
	
		New-AzureRmDataFactoryDataset $df -File .\EmpSQLTable.json


## Creación de una canalización
En este paso, creará una canalización con una **actividad de copia** que usa **EmpTableFromBlob** como entrada y **EmpSQLTable** como salida.

1.	Cree un archivo JSON llamado **ADFTutorialPipeline.json** en la carpeta **C:\\ADFGetStartedPSH** con el siguiente contenido:
	
			 {
			  "name": "ADFTutorialPipeline",
			  "properties": {
			    "description": "Copy data from a blob to Azure SQL table",
			    "activities": [
			      {
			        "name": "CopyFromBlobToSQL",
			        "description": "Push Regional Effectiveness Campaign data to Azure SQL database",
			        "type": "Copy",
			        "inputs": [
			          {
			            "name": "EmpTableFromBlob"
			          }
			        ],
			        "outputs": [
			          {
			            "name": "EmpSQLTable"
			          }
			        ],
			        "typeProperties": {
			          "source": {
			            "type": "BlobSource"
			          },
			          "sink": {
			            "type": "SqlSink"
			          }
			        },
			        "Policy": {
			          "concurrency": 1,
			          "executionPriorityOrder": "NewestFirst",
			          "style": "StartOfInterval",
			          "retry": 0,
			          "timeout": "01:00:00"
			        }
			      }
			    ],
			    "start": "2016-08-09T00:00:00Z",
			    "end": "2016-08-10T00:00:00Z",
			    "isPaused": false
			  }
			}

	Tenga en cuenta lo siguiente:

	- En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**.
	- La entrada de la actividad está establecida en **EmpTableFromBlob** y la salida en **EmpSQLTable**.
	- En la sección **transformación**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor.

	Reemplace el valor de la propiedad **start** por el día actual y el valor **end** por el próximo día. Las fechas y horas de inicio y de finalización deben estar en [formato ISO](http://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2014-10-14T16:32:41Z. La hora de finalización (**end**) es opcional, pero se utilizará en este tutorial.
	
	Si no especifica ningún valor para la propiedad **end**, se calcula como "**start + 48 horas**". Para ejecutar la canalización indefinidamente, especifique **9/9/9999** como valor de la propiedad **end**.
	
	En el ejemplo, hay 24 segmentos de datos, ya que cada segmento de datos se produce cada hora.
	
	Para obtener más información sobre las propiedades JSON, vea [Referencia de scripting JSON](http://go.microsoft.com/fwlink/?LinkId=516971).
2.	Ejecute el comando siguiente para crear la tabla Factoría de datos.
		
		New-AzureRmDataFactoryPipeline $df -File .\ADFTutorialPipeline.json

**¡Enhorabuena!** Ha creado correctamente una factoría de datos de Azure, servicios vinculados, tablas y una canalización, y ha programado la canalización.

## Supervisión de la canalización
En este paso, usará Azure PowerShell para supervisar lo que está ocurriendo en una factoría de datos de Azure.

1.	Ejecute **Get-AzureRmDataFactory** y asigne el resultado a la variable $df.

		$df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH
 
2.	Ejecute **Get-AzureRmDataFactorySlice** para obtener la información sobre todos los segmentos de **EmpSQLTable**, que es la tabla de salida de la canalización.

		Get-AzureRmDataFactorySlice $df -DatasetName EmpSQLTable -StartDateTime 2015-03-03T00:00:00

	Reemplace la parte del año, mes y fecha del parámetro **StartDateTime** por el año, mes y fecha actuales. Esta configuración debe coincidir con el valor de **Inicio** del JSON de canalización.

	Debe ver 24 segmentos, uno para cada hora de 12 A.M. del día actual hasta las 12 A.M. del día siguiente.
	
	**Primer segmento:**

		ResourceGroupName : ADFTutorialResourceGroup
		DataFactoryName   : ADFTutorialDataFactoryPSH
		TableName         : EmpSQLTable
		Start             : 3/3/2015 12:00:00 AM
		End               : 3/3/2015 1:00:00 AM
		RetryCount        : 0
		Status            : Waiting
		LatencyStatus     :
		LongRetryCount    : 0

	**Último segmento:**

		ResourceGroupName : ADFTutorialResourceGroup
		DataFactoryName   : ADFTutorialDataFactoryPSH
		TableName         : EmpSQLTable
		Start             : 3/4/2015 11:00:00 PM
		End               : 3/4/2015 12:00:00 AM
		RetryCount        : 0
		Status            : Waiting
		LatencyStatus     : 
		LongRetryCount    : 0

3.	Ejecute **Get-AzureRmDataFactoryRun** para obtener la información de la actividad que se ejecuta para un segmento **específico**. Cambie el valor del parámetro **StartDateTime** para que coincida con la hora de **Inicio** del segmento desde la salida. El valor de **StartDateTime** debe estar en [formato ISO](http://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2014-03-03T22:00:00Z.

		Get-AzureRmDataFactoryRun $df -DatasetName EmpSQLTable -StartDateTime 2015-03-03T22:00:00

	Debería ver una salida similar a la siguiente:

		Id                  : 3404c187-c889-4f88-933b-2a2f5cd84e90_635614488000000000_635614524000000000_EmpSQLTable
		ResourceGroupName   : ADFTutorialResourceGroup
		DataFactoryName     : ADFTutorialDataFactoryPSH
		TableName           : EmpSQLTable
		ProcessingStartTime : 3/3/2015 11:03:28 PM
		ProcessingEndTime   : 3/3/2015 11:04:36 PM
		PercentComplete     : 100
		DataSliceStart      : 3/8/2015 10:00:00 PM
		DataSliceEnd        : 3/8/2015 11:00:00 PM
		Status              : Succeeded
		Timestamp           : 3/8/2015 11:03:28 PM
		RetryAttempt        : 0
		Properties          : {}
		ErrorMessage        :
		ActivityName        : CopyFromBlobToSQL
		PipelineName        : ADFTutorialPipeline
		Type                : Copy

Vea [Referencia de cmdlets de factoría de datos][cmdlet-reference] para obtener la documentación completa sobre los cmdlets de la factoría de datos.

## Resumen
En este tutorial, ha creado una factoría de datos de Azure para copiar datos de un blob de Azure en una base de datos SQL de Azure. Ha usado PowerShell para crear la factoría de datos, los servicios vinculados, los conjuntos de datos y una canalización. Estos son los pasos de alto nivel que realizó en este tutorial:

1.	Ha creado una **factoría de datos** de Azure.
2.	Ha creado **servicios vinculados**:
	1. Un servicio vinculado **Almacenamiento de Azure** para vincular la cuenta de Almacenamiento de Azure que contiene datos de entrada.
	2. Un servicio vinculado **SQL Azure** para vincular la base de datos SQL de Azure que contiene los datos de salida.
3.	Ha creado **conjuntos de datos** que describen los datos de entrada y salida de las canalizaciones.
4.	Ha creado una **canalización** con una **actividad de copia** con un origen **BlobSource** y un receptor **SqlSink**.

## Otras referencias
| Tema. | Description |
| :---- | :---- |
| [Actividades de movimiento de datos](data-factory-data-movement-activities.md) | En este artículo se proporciona información detallada sobre la actividad de copia que se usa en el tutorial. |
| [Programación y ejecución con Data Factory](data-factory-scheduling-and-execution.md) | En este artículo se explican los aspectos de programación y ejecución del modelo de aplicación de Factoría de datos de Azure. |
| [Procesos](data-factory-create-pipelines.md) | Este artículo le ayuda a conocer las canalizaciones y actividades de Data Factory de Azure. |
| [Conjuntos de datos](data-factory-create-datasets.md) | Este artículo le ayuda a comprender los conjuntos de datos de Data Factory de Azure.
| [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md) | En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante la aplicación de supervisión y administración. 



[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908

[cmdlet-reference]: https://msdn.microsoft.com/library/azure/dn820234.aspx
[old-cmdlet-reference]: https://msdn.microsoft.com/library/azure/dn820234(v=azure.98).aspx
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[azure-portal]: http://portal.azure.com
[download-azure-powershell]: ../powershell-install-configure.md
[data-factory-introduction]: data-factory-introduction.md

[image-data-factory-get-started-storage-explorer]: ./media/data-factory-copy-activity-tutorial-using-powershell/getstarted-storage-explorer.png

[sql-management-studio]: ../sql-database/sql-database-manage-azure-ssms.md
 

<!---HONumber=AcomDC_0824_2016-->