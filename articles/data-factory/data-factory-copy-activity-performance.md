<properties
	pageTitle="Guía de optimización y rendimiento de la actividad de copia | Microsoft Azure"
	description="Conozca los factores más importantes que afectan al rendimiento del movimiento de datos en Data Factory de Azure cuando se usa la actividad de copia."
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
	ms.topic="article"
	ms.date="08/25/2016"
	ms.author="spelluru"/>


# Guía de optimización y rendimiento de la actividad de copia
En este artículo se describen los factores principales que afectan al rendimiento del movimiento de datos cuando se usa Data Factory de Azure con la actividad de copia. También se muestra el rendimiento que hemos observado en nuestras propias pruebas y se comentan las diversas maneras de optimizar el rendimiento de la actividad de copia.

Mediante la actividad de copia, puede obtener un gran rendimiento en el movimiento de datos:

- Ingesta de 1 TB de datos en Almacenamiento de blobs de Azure desde un sistema de archivos local en menos de tres horas (a 100 MBps)
- Ingesta de 1 TB de datos en Azure Data Lake Store desde un sistema de archivos local y Almacenamiento de blobs de Azure en menos de tres horas (a 100 MBps)
- Ingesta de 1 TB de datos en Almacenamiento de datos SQL de Azure desde Almacenamiento de blobs en menos de tres horas (a 100 MBps)

Siga leyendo para saber más sobre el rendimiento de la actividad de copia y para obtener sugerencias de optimización para mejorar esta herramienta.

> [AZURE.NOTE] Si no está familiarizado con la actividad de copia, consulte [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md) antes de leer este artículo.

## Pasos de optimización del rendimiento
Para optimizar el rendimiento del servicio Data Factory con la actividad de copia, sugerimos que realice estos pasos:

1.	**Establezca una línea base.** Durante la fase de desarrollo, pruebe la canalización usando la actividad de copia en un ejemplo de datos representativo. Puede usar el [modelo de segmentación](data-factory-scheduling-and-execution.md#time-series-datasets-and-data-slices) de Data Factory para limitar la cantidad de datos con los que trabaja.

	Recopile características de tiempo de ejecución y rendimiento mediante la **Aplicación de supervisión y administración**. Elija **Supervisión y administración** en la página de inicio de Data Factory. En la vista de árbol, elija el **conjunto de datos de salida**. En la lista **Activity Windows** (Ventanas de actividad), elija la ejecución de la actividad de copia. En **Activity Windows** (Ventanas de actividad) se muestra la duración de la actividad de copia y el tamaño de los datos que se copian. El rendimiento se muestra en **Activity Window Explorer** (Explorador de ventanas de actividad). Para más información sobre la aplicación, consulte [Supervisión y administración de canalizaciones de Data Factory de Azure mediante la nueva Aplicación de supervisión y administración](data-factory-monitor-manage-app.md).

	![Detalles de ejecución de actividad](./media/data-factory-copy-activity-performance/mmapp-activity-run-details.png)

	Más adelante en este artículo, puede comparar el rendimiento y la configuración de su escenario con la [referencia de rendimiento](#performance-reference) de la actividad de copia de nuestras pruebas.
2. **Diagnostique y optimice el rendimiento**. Si el rendimiento que observa no satisface sus expectativas, deberá identificar los cuellos de botella. A continuación, optimice el rendimiento para eliminar o reducir el efecto de los cuellos de botella. Aunque la descripción completa del diagnóstico de rendimiento escapa del ámbito de este artículo, aquí hay algunos aspectos comunes a tener en cuenta:
	- [Origen](#considerations-for-the-source)
	- [Sink](#considerations-for-the-sink)
	- [Serialización y deserialización](#considerations-for-serialization-and-deserialization)
	- [Compresión](#considerations-for-compression)
	- [Asignación de columnas](#considerations-for-column-mapping)
	- [Data Management Gateway](#considerations-for-data-management-gateway)
	- [Otras consideraciones](#other-considerations)
	- [Copia paralela](#parallel-copy)
	- [Unidades de movimiento de datos de nube](#cloud-data-movement-units)

3. **Expanda la configuración a todo el conjunto de datos**. Cuando esté satisfecho con los resultados de la ejecución y el rendimiento, puede expandir la definición y el período activo de canalización para cubrir todo el conjunto de datos.

## Referencia de rendimiento

![Matriz de rendimiento](./media/data-factory-copy-activity-performance/CopyPerfRef.png)

Puntos a tener en cuenta:

- La capacidad de proceso se calcula con la siguiente fórmula: [tamaño de los datos leídos del origen]/[duración de la ejecución de Copiar actividad].
- Usaremos el conjunto de datos [TPC-H](http://www.tpc.org/tpch/) para calcular las cifras de la tabla anterior.
-	Para copiar entre almacenes de datos en la nube, establezca **cloudDataMovementUnits** en 1 y 4 para compararlos. **parallelCopies** no se especifica. Consulte la sección [Copia en paralelo](#parallel-copy) para más información sobre estas características.
- En los almacenes de datos de Azure, el origen y el receptor se encuentran en la misma región de Azure.
- Para movimientos de datos híbridos (del entorno local a la nube o de la nube al entorno loca), se requiere una sola instancia de Data Management Gateway en una máquina que esté separada del almacén de datos local. La configuración se muestra en la tabla siguiente. Si se ha ejecutado una única actividad en la puerta de enlace, la operación de copia solo ha consumido una pequeña parte de los recursos de CPU o memoria de la máquina de prueba y una pequeña parte de su ancho de banda de red.
	<table>
	<tr>
		<td>CPU</td>
		<td>Intel Xeon E5-2660 v2 de 32 núcleos a 2,20 GHz</td>
	</tr>
	<tr>
		<td>Memoria</td>
		<td>128 GB</td>
	</tr>
	<tr>
		<td>Red</td>
		<td>Interfaz de Internet: 10 Gbps; interfaz de intranet: 40 Gbps</td>
	</tr>
	</table>

## Copia paralela
Puede leer datos del origen o escribir datos en el destino **en paralelo dentro de una ejecución de actividad de copia**. Esta característica mejora el rendimiento de una operación de copia y reduce el tiempo necesario para mover datos.

Esta configuración es diferente de la propiedad **concurrency** de la definición de actividad. La propiedad **concurrency** determina el número de **ejecuciones simultáneas de la actividad de copia** para procesar datos de diferentes ventanas de actividad (de 1 a.m. a 2 a.m., de 2 a.m. a 3 a.m. y de 3 a.m. a 4 a.m., etc.). Esta funcionalidad es útil cuando se realiza una carga histórica. La funcionalidad de copia en paralelo se aplica a la **ejecución de una única actividad**.

Echemos un vistazo a un escenario de ejemplo. En el ejemplo siguiente, se van a procesar varios sectores del pasado. Data Factory ejecuta una única instancia de actividad de copia (una ejecución de actividad) para cada segmento:

- El segmento de datos de la primera ventana de actividad ( de 1 a.m. a 2 a.m.) == > ejecución de actividad 1
- El segmento de datos de la segunda ventana de actividad (de 2 a.m. a 3 a.m.) == > ejecución de actividad 2
- El segmento de datos de la tercera ventana de actividad (de 3 a.m. a 4 a.m.) == > ejecución de actividad 3

y así sucesivamente.

En este ejemplo, cuando el valor de **concurrency** está establecido en 2, la **ejecución de actividad 1** y la **ejecución de actividad 2** copian datos de dos ventanas de actividad a la vez para mejorar el rendimiento del movimiento de datos. Sin embargo, si hay varios archivos asociados con la ejecución de actividad 1, el servicio de movimiento de datos copia de uno en uno los archivos del origen al destino.

### parallelCopies
Puede usar la propiedad **parallelCopies** para indicar el paralelismo que quiere que use la actividad de copia. Esta propiedad se puede considerar como el número máximo de subprocesos dentro de la actividad de copia que se pueden leer del origen o escribir en los almacenes de datos receptores en paralelo.

Para cada ejecución de actividad de copia, Data Factory determina el número de copias en paralelo que se usarán para copiar datos del almacén de datos de origen al almacén de datos de destino. El número predeterminado de copias en paralelo que use dependerá del tipo de origen y receptor que vaya a emplear.

Origen y receptor |	Recuento predeterminado de copias en paralelo determinado por servicio
------------- | -------------------------------------------------
Copia de datos entre almacenes basados en archivos (Almacenamiento de blobs, Data Lake Store, un sistema de archivos local, un sistema de archivos distribuido de Hadoop local, o HDFS) | Entre 1 y 32. Depende del tamaño de los archivos y del número de unidades de movimiento de datos (DMU) de nube usadas para copiar datos entre dos almacenes de datos de nube; o de la configuración física de la máquina de puerta de enlace usada para una copia híbrida (para copiar datos a o desde un almacén de datos local).
Copia de datos desde **cualquier almacén de datos a Almacenamiento de tablas de Azure** | 4
Todos los demás pares de origen y receptor | 1

En la mayoría de los casos, el comportamiento predeterminado debe proporcionar el mejor rendimiento. Sin embargo, para controlar la carga en las máquinas que hospedan los almacenes de datos, o para optimizar el rendimiento de la copia, puede optar por reemplazar el valor predeterminado y especificar un valor para la propiedad **parallelCopies**. El valor debe estar entre 1 y 32 (ambos inclusive). En tiempo de ejecución, y para obtener el mejor rendimiento, la actividad de copia usa un valor inferior o igual al valor que ha establecido.

	"activities":[  
	    {
	        "name": "Sample copy activity",
	        "description": "",
	        "type": "Copy",
	        "inputs": [{ "name": "InputDataset" }],
	        "outputs": [{ "name": "OutputDataset" }],
	        "typeProperties": {
	            "source": {
	                "type": "BlobSource",
	            },
	            "sink": {
	                "type": "AzureDataLakeStoreSink"
	            },
	            "parallelCopies": 8
	        }
	    }
	]

Puntos a tener en cuenta:

- Cuando se copian datos entre almacenes basados en archivos, se produce paralelismo en el nivel de archivo. No hay fragmentación dentro de un único archivo. El número real de copias en paralelo que usa el servicio de movimiento de datos para la operación de copia en tiempo de ejecución no es superior al número de archivos que tenga. Si el comportamiento de copia es **mergeFile**, la actividad de copia no puede aprovechar las ventajas del paralelismo.
- Cuando especifique un valor para la propiedad **parallelCopies**, tenga en cuenta el aumento de la carga en los almacenes de datos de origen y receptor, y en la puerta de enlace si se trata de una copia híbrida. Esto es especialmente cierto si tiene varias actividades o ejecuciones simultáneas de las mismas actividades que se ejecutan en el mismo almacén de datos. Si observa que el almacén de datos o la puerta de enlace están sobrecargados, disminuya el valor de la propiedad **parallelCopies** para aliviar la carga.
- Cuando se copian datos de almacenes no basados en archivos en almacenes basados en archivos, el servicio de movimiento de datos ignora la propiedad **parallelCopies**. Aunque se especifica el paralelismo, no se aplica en este caso.

> [AZURE.NOTE] Debe usar Data Management Gateway versión 1.11 o superior para emplear la característica **parallelCopies** cuando realice una copia híbrida.

### Unidades de movimiento de datos de nube
Una **unidad de movimiento de datos (DMU) de nube** es una medida que representa la eficacia (una combinación de CPU, memoria y asignación de recursos de red) de una única unidad en Data Factory. Una DMU podría utilizarse en una operación de copia de una nube a otra, pero no en una copia híbrida.

De forma predeterminada, Data Factory usa una única DMU para realizar una única ejecución de la actividad de copia. Para reemplazar esta configuración predeterminada, especifique un valor para la propiedad **cloudDataMovementUnits** de la manera siguiente. Para más información sobre el nivel de ganancia de rendimiento que puede obtener al configurar más unidades para un origen y un receptor de copia específicos, consulte la [referencia de rendimiento](#performance-reference).

	"activities":[  
	    {
	        "name": "Sample copy activity",
	        "description": "",
	        "type": "Copy",
	        "inputs": [{ "name": "InputDataset" }],
	        "outputs": [{ "name": "OutputDataset" }],
	        "typeProperties": {
	            "source": {
	                "type": "BlobSource",
	            },
	            "sink": {
	                "type": "AzureDataLakeStoreSink"
	            },
	            "cloudDataMovementUnits": 4
	        }
	    }
	]

Los **valores admitidos** para la propiedad **cloudDataMovementUnits** son 1 (predeterminado), 2, 4 y 8. El **número real de DMS de nube** que usa la operación de copia en tiempo de ejecución es igual o inferior al valor configurado, según el patrón de datos. Si necesita más DMU de nube para aumentar el rendimiento, póngase en contacto con el [servicio técnico de Azure](https://azure.microsoft.com/support/). La configuración de 8 o más solo funciona actualmente cuando se copian varios archivos de Almacenamiento de blobs a Almacenamiento de blobs o a una instancia de Data Lake Store que sea mayor o igual que 16 MB de forma individual.

Para un uso mejor de estas dos propiedades, y para mejorar el rendimiento del movimiento de datos, consulte los [casos de uso de ejemplo](#case-study-use-parallel-copy). No es necesario configurar **parallelCopies** para aprovechar el comportamiento predeterminado. Si configura **parallelCopies** y lo hace en un valor demasiado pequeño, es posible que varias DMU de nube no se utilicen completamente.

Es **importante** recordar que se cobra en función del tiempo total de la operación de copia. Si un trabajo de copia solía tardar una hora con una unidad de nube y ahora tarda 15 minutos con cuatro unidades de nube, la factura general será casi igual. Por ejemplo, va a utilizar cuatro unidades de nube. La primera gasta 10 minutos, la segunda 10 minutos, la tercera 5 minutos y la cuarta 5 minutos, todas ellas en una única ejecución de actividad de copia. Se le cobra por el tiempo total de copia (movimiento de datos), que es 10 + 10 + 5 + 5 = 30 minutos. El uso de **parallelCopies** no afecta a la facturación.

## Copias almacenadas provisionalmente
Al copiar datos de un almacén de datos de origen a un almacén de datos receptor, podría elegir usar Almacenamiento de blobs como almacenamiento provisional. El almacenamiento provisional es especialmente útil en los siguientes casos:

-	**En ocasiones, realizar un movimiento de datos híbridos lleva tiempo (es decir, copiar de un almacén de datos local a un almacén de datos en la nube o viceversa) a través de una conexión de red lenta**. Para mejorar el rendimiento, puede comprimir los datos locales de modo que se tarde menos tiempo en mover datos al almacén de datos provisional en la nube. Luego, puede descomprimir los datos en el almacenamiento provisional antes de cargarlos en el almacén de datos de destino.
2.	**No quiere abrir otros puertos que no sean el 80 y el 443 en el firewall, debido a las directivas de TI corporativas**. Por ejemplo, al copiar datos de un almacén de datos local a un receptor de Base de datos SQL de Azure o a un receptor de Almacenamiento de datos SQL de Azure, debe activar la comunicación TCP saliente en el puerto 1433 tanto para el firewall de Windows como para el firewall corporativo. En ese escenario, puede aprovechar Data Management Gateway para copiar primero los datos en una instancia de Almacenamiento de blobs provisional mediante HTTP o HTTPS en el puerto 443. Luego, puede cargar los datos en Base de datos SQL o en Almacenamiento de datos SQL desde el Almacenamiento de blobs provisional. En este flujo, no es necesario habilitar el puerto 1433.
3.	**Realiza la ingesta de datos desde diversos almacenes de datos en Almacenamiento de datos SQL mediante PolyBase**. Almacenamiento de datos SQL emplea PolyBase como mecanismo de alto rendimiento para cargar una gran cantidad de datos en Almacenamiento de datos SQL. Sin embargo, los datos de origen deben estar en Almacenamiento de blobs y deben satisfacer criterios adicionales. Al cargar datos desde un almacén de datos distinto de Almacenamiento de blobs, puede activar la copia de datos mediante el Almacenamiento de blobs provisional. En ese caso, Data Factory realiza las transformaciones de datos necesarias para garantizar que se cumplen los requisitos de PolyBase. A continuación, se usa PolyBase para cargar datos en Almacenamiento de datos SQL. Consulte [Uso de PolyBase para cargar datos en el Almacenamiento de datos SQL](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-sql-data-warehouse) para más información y ver ejemplos.

### Funcionamiento de las copias almacenadas provisionalmente
Al activar la característica de almacenamiento provisional, primero se copian los datos desde el almacén de datos de origen al almacén de datos provisional (el suyo propio). A continuación, los datos se copian desde el almacén de datos provisional al almacén de datos receptor. Data Factory administra automáticamente el flujo de las dos fases. Data Factory también limpia los datos temporales del almacenamiento provisional una vez finalizado el movimiento de los datos.

En el escenario de copia en la nube, en el que los almacenes de datos de origen y receptor están en la nube y no usan Data Management Gateway, Data Factory se encarga de las operaciones de copia.

![Copias almacenadas provisionalmente: escenario de modelo en la nube](media/data-factory-copy-activity-performance/staged-copy-cloud-scenario.png)

En el escenario de copia híbrida, en el que el origen se encuentra en el entorno local y el receptor está en la nube, Data Management Gateway mueve los datos desde el almacén de datos de origen a un almacén de datos provisional. Data Factory también mueve los datos desde el almacén de datos provisional al almacén de datos receptor. Con el flujo invertido también se permite copiar datos de un almacén de datos en la nube a uno local a través del almacenamiento provisional.

![Copias almacenadas provisionalmente: escenario de modelo híbrido](media/data-factory-copy-activity-performance/staged-copy-hybrid-scenario.png)

Cuando activa el movimiento de datos mediante un almacenamiento provisional, puede especificar si quiere que los datos se compriman antes de moverlos del almacén de datos de origen al provisional y luego descomprimirlos antes de moverlos desde un almacenamiento de datos provisional a un almacén de datos receptor.

Actualmente, no se pueden copiar datos entre dos almacenes de datos locales mediante almacenamiento provisional. Esperamos que esta opción esté pronto disponible.

### Configuración
Puede configurar la opción **enableStaging** de la actividad de copia para especificar si quiere que los datos se almacenen provisionalmente en Almacenamiento de blobs antes de cargarlos en un almacén de datos de destino. Al establecer **enableStaging** en TRUE, especifique las propiedades adicionales enumeradas en la siguiente tabla. Si no tiene un servicio vinculado a la firma de acceso compartido de Almacenamiento o de Almacenamiento de Azure para el almacenamiento provisional, deberá crear uno.

Propiedad | Description | Valor predeterminado | Obligatorio
--------- | ----------- | ------------ | --------
**enableStaging** | Especifique si desea copiar los datos a través de un almacén provisional. | False | No
**linkedServiceName** | Especifique el nombre de un servicio vinculado [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service) o [AzureStorageSas](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service), que haga referencia a la instancia de Almacenamiento que se usa como almacenamiento provisional. <br/><br/> No puede usar Almacenamiento con una firma de acceso compartido para cargar datos en Almacenamiento de datos SQL mediante PolyBase. Puede usarlo en todos los demás casos. | N/D | Sí, cuando el valor de **enableStaging** está establecido en True.
**path** | Especifique la ruta de acceso de Almacenamiento de blobs que quiere que contenga los datos almacenados provisionalmente. Si no se proporciona una ruta de acceso, el servicio creará un contenedor para almacenar los datos temporales. <br/><br/> Especifique una ruta de acceso solo si usa Almacenamiento con una firma de acceso compartido o si necesita que los datos temporales estén en una ubicación específica. | N/D | No
**enableCompression** | Para reducir el volumen de datos que se van a transferir, especifique si se deben comprimir los datos cuando el servicio de movimiento de datos los mueva de un almacén de datos de origen a un almacén de datos receptor. | False | No

Este es un ejemplo de definición de actividad de copia con las propiedades que se han descrito en la tabla anterior:

	"activities":[  
	{
		"name": "Sample copy activity",
		"type": "Copy",
		"inputs": [{ "name": "OnpremisesSQLServerInput" }],
		"outputs": [{ "name": "AzureSQLDBOutput" }],
		"typeProperties": {
			"source": {
				"type": "SqlSource",
			},
			"sink": {
				"type": "SqlSink"
			},
	    	"enableStaging": true,
			"stagingSettings": {
				"linkedServiceName": "MyStagingBlob",
				"path": "stagingcontainer/path",
				"enableCompression": true
			}
		}
	}
	]


### Impacto en la facturación
Los cargos que se le realizan se basan en dos elementos: duración de la copia y tipo de copia.

- Al utilizar almacenamiento provisional durante una copia de nube (copia de datos de un almacén de datos en la nube a otro de este tipo, por ejemplo, de Data Lake Store a Almacenamiento de datos SQL), se le cobrará de la siguiente forma: [suma de la duración de la copia de los pasos 1 y 2] x [precio unitario de la copia de nube].
- Al utilizar almacenamiento provisional durante una copia híbrida (copia de datos de un almacén de datos local a uno en la nube, por ejemplo, de una base de datos de SQL Server local a Almacenamiento de datos SQL), se le cobrará de la siguiente forma: [duración de la copia híbrida] x [precio unitario de la copia híbrida] + [duración de la copia de nube] x [precio unitario de la copia de nube].


## Consideraciones sobre el origen
### General
Asegúrese de que el almacén de datos subyacente no esté saturado con otras cargas de trabajo que se ejecutan en él o contra él.

Para almacenes de datos de Microsoft, consulte los [temas sobre supervisión y optimización](#performance-reference) específicos de los almacenes de datos para que le ayuden a comprender las características de rendimiento del almacén de datos, a reducir los tiempos de respuesta y a maximizar la capacidad de proceso.

Si copia los datos de Almacenamiento de blobs a Almacenamiento de datos SQL, considere el uso de **PolyBase** para mejorar el rendimiento. Consulte [Uso de PolyBase para cargar datos en el Almacenamiento de datos SQL](data-factory-azure-sql-data-warehouse-connector.md###use-polybase-to-load-data-into-azure-sql-data-warehouse) para obtener más información.


### Almacenes de datos basados en archivos
*(Incluye Almacenamiento de blobs, Data Lake Store y sistemas de archivos locales)*

- **Tamaño de archivo medio y número de archivos**: la actividad de copia transfiere datos de archivo en archivo. Siendo la misma cantidad de datos la que se va a mover, el rendimiento general es menor si los datos constan de muchos archivos pequeños en lugar de algunos grandes, debido a la fase de arranque de cada archivo. Por lo tanto, si es posible, combine archivos pequeños en archivos de mayor tamaño para obtener una capacidad de proceso mayor.
- **Formato de archivo y compresión**: para ver más formas de mejorar el rendimiento, consulte las secciones [Consideraciones sobre serialización y deserialización](#considerations-for-serialization-and-deserialization) y [Consideraciones sobre la compresión](#considerations-for-compression).
- Para el escenario de **sistema de archivos local**, en el que se necesita **Data Management Gateway**, consulte la sección [Consideraciones sobre Data Management Gateway](#considerations-for-data-management-gateway).

### Almacenes de datos relacionales
*(Incluye Base de datos SQL, Almacenamiento de datos SQL, bases de datos SQL Server, y bases de datos Oracle, MySQL, DB2, Teradata, Sybase y PostgreSQL)*

- **Patrón de datos**: el esquema de tabla afecta al rendimiento de la copia. Para copiar la misma cantidad de datos, un tamaño de fila grande proporcionará un mejor rendimiento que un tamaño de fila pequeño, ya que la base de datos recupera de forma más eficaz menos lotes de datos que contienen un menor número de filas.
- **Consulta o procedimiento almacenado**: optimice la lógica de la consulta o del procedimiento almacenado que se especifica en el origen de la actividad de copia para que capture los datos de forma más eficiente.
- Además, para el escenario de **bases de datos relacionales locales**, como SQL Server y Oracle, que requieren el uso de **Data Management Gateway**, consulte la sección [Consideraciones sobre Data Management Gateway](#considerations-on-data-management-gateway).

## Consideraciones sobre el receptor

### General
Asegúrese de que el almacén de datos subyacente no esté saturado con otras cargas de trabajo que se ejecutan en él o contra él.

Para información sobre los almacenes de datos de Microsoft, consulte los [temas sobre supervisión y optimización](#performance-reference) que son específicos de los almacenes de datos. Estos temas pueden ayudarle a comprender las características de rendimiento de los almacenes de datos y a saber cómo reducir los tiempos de respuesta y aumentar la capacidad de proceso.

Si va a copiar datos de **Almacenamiento de blobs** a **Almacenamiento de datos SQL**, considere la posibilidad de usar **PolyBase** para aumentar el rendimiento. Consulte [Uso de PolyBase para cargar datos en el Almacenamiento de datos SQL](data-factory-azure-sql-data-warehouse-connector.md###use-polybase-to-load-data-into-azure-sql-data-warehouse) para obtener más información.


### Almacenes de datos basados en archivos
*(Incluye Almacenamiento de blobs, Data Lake Store y sistemas de archivos locales)*

- **Comportamiento de copia**: si copia datos de un almacén de datos basado en archivos diferente, la actividad de copia presenta tres opciones mediante la propiedad **copyBehavior**. Conserva la jerarquía, aplana la jerarquía o combina archivos. Conservar o aplanar la jerarquía supone poca o ninguna sobrecarga sobre el rendimiento, mientras que combinar archivos hace que aumente dicha sobrecarga.
- **Formato de archivo y compresión**: consulte las secciones [Consideraciones sobre serialización y deserialización](#considerations-for-serialization-and-deserialization) y [Consideraciones sobre la compresión](#considerations-for-compression) para conocer más formas de mejorar el rendimiento.
- **Almacenamiento de blobs**: actualmente, Almacenamiento de blobs solo admite blobs en bloques para optimizar la transferencia de datos y el rendimiento.
- En escenarios de **sistemas de archivos locales** que requieran el uso de **Data Management Gateway**, consulte la sección [Consideraciones sobre Data Management Gateway](#considerations-for-data-management-gateway).

### Almacenes de datos relacionales
*(Incluye Base de datos SQL, Almacenamiento de datos SQL y bases de datos SQL Server)*

- **Comportamiento de copia**: según las propiedades que haya establecido para **sqlSink**, la actividad de copia escribe datos en la base de datos de destino de diferentes formas.
	- De forma predeterminada, el servicio de movimiento de datos usa la API de copia masiva para insertar datos en modo de anexión, lo que proporciona el mejor rendimiento.
	- Si configura un procedimiento almacenado en el receptor, la base de datos aplica los datos de fila en fila en lugar de como una carga masiva. El rendimiento disminuye considerablemente. Si el conjunto de datos es grande, considere la posibilidad de cambiar al uso de la propiedad **sqlWriterCleanupScript**, cuando sea conveniente.
	- Si configura la propiedad **sqlWriterCleanupScript** para cada ejecución de la actividad de copia, el servicio desencadena el script y luego el usuario usa la API de copia masiva para insertar los datos. Por ejemplo, para sobrescribir toda la tabla con los datos más recientes, puede especificar un script para eliminar primero todos los registros antes de realizar la carga masiva de los nuevos datos desde el origen.
- **Patrón de datos y tamaño de lote**:
	- El esquema de tabla afecta al rendimiento de la copia. Para copiar la misma cantidad de datos, un tamaño de fila grande proporcionará un mejor rendimiento que un tamaño de fila pequeño porque la base de datos puede confirmar de forma más eficiente menos lotes de datos.
	- La actividad de copia inserta datos en una serie de lotes. Puede establecer el número de filas en un lote mediante la propiedad **writeBatchSize**. Si los datos tienen filas pequeñas, puede establecer la propiedad **writeBatchSize** con un valor más alto para beneficiarse de una menor sobrecarga de los lotes y un mayor rendimiento. Si el tamaño de fila de los datos es grande, tenga cuidado al aumentar **writeBatchSize**. Un valor alto podría provocar un error de copia provocado por la sobrecarga de la base de datos.
- Para **bases de datos relacionales locales** como SQL Server y Oracle, que requieren el uso de **Data Management Gateway**, consulte la sección [Consideraciones sobre Data Management Gateway](#considerations-for-data-management-gateway).


### Almacenes NoSQL
*(Incluye Almacenamiento de tablas y Azure DocumentDB)*

- Para **Almacenamiento de tablas**:
	- **Partición**: escribir datos en particiones intercaladas degrada considerablemente el rendimiento. Puede ordenar los datos de origen por la clave de partición para que los datos se inserten en una partición después de otra de forma eficiente, o bien, puede ajustar la lógica para escribir los datos en una sola partición.
- Para **DocumentDB**:
	- **Tamaño de lote**: la propiedad **writeBatchSize** indica el número de solicitudes en paralelo al servicio de DocumentDB para crear documentos. Puede esperar un rendimiento mejor si aumenta el valor de **writeBatchSize** porque se envían más solicitudes en paralelo a DocumentDB. Sin embargo, tenga cuidado con las limitaciones cuando escriba en DocumentDB (el mensaje de error es "La tasa de solicitudes es grande"). Son varios los factores que pueden provocar limitaciones, como el tamaño del documento, el número de términos que contiene y la directiva de indexación de la colección de destino. Para lograr un mayor rendimiento de la actividad de copia, considere la posibilidad de usar una colección mejor (por ejemplo, S3).

## Consideraciones sobre serialización y deserialización
La serialización y deserialización pueden tener lugar cuando el conjunto de datos de entrada o el conjunto de datos de salida es un archivo. Actualmente, la actividad de copia admite los formatos de datos Avro y texto (por ejemplo, CSV y TSV).

**Comportamiento de copia**:

-	Al copiar archivos entre almacenes de datos basados en archivos:
	- Cuando los conjuntos de datos de entrada y salida tienen la misma configuración de formato de archivo o carecen de dicha configuración, el servicio de movimiento de datos ejecuta una copia binaria sin ninguna serialización o deserialización. En este caso, el rendimiento es mayor en comparación con el escenario en el que la configuración del formato de archivo de origen y receptor es diferente.
	- Cuando los conjuntos de datos de entrada y salida están en formato de texto y solo es diferente el tipo de codificación, el servicio de movimiento de datos solo realiza la conversión de la codificación. No se realiza ninguna serialización y deserialización, lo que provoca cierta sobrecarga en el rendimiento en comparación con una copia binaria.
	- Cuando los conjuntos de datos de entrada y salida tienen formatos de archivo diferentes o configuraciones diferentes, por ejemplo, los delimitadores, el servicio de movimiento de datos deserializa los datos de origen para transmitirlos, transformarlos y luego serializarlos en el formato de salida indicado. Esta operación da como resultado una sobrecarga mucho mayor sobre el rendimiento en comparación con otros escenarios.
- Al copiar archivos a y desde un almacén de datos que no se basa en archivos (por ejemplo, desde un almacén basado en archivos hasta un almacén relacional), se requiere el paso de serialización o deserialización. Este paso produce una sobrecarga considerable sobre el rendimiento.

**Formato de archivo**: el formato de archivo que elija puede afectar al rendimiento de la copia. Por ejemplo, Avro es un formato binario compacto que almacena metadatos con datos. Es ampliamente admitido para el procesamiento y la realización de consultas en el ecosistema de Hadoop. Sin embargo, Avro resulta más caro para serialización y deserialización, lo que da lugar a un menor rendimiento de la copia en comparación con el formato de texto. Elija el formato de archivo para el flujo de procesamiento de forma holística. Para comenzar, piense en qué formato se almacenan los datos, los almacenes de datos de origen o si se extraerán de sistemas externos; el mejor formato para el almacenamiento, el procesamiento analítico y la realización de consultas; y en qué formato se deben exportar los datos a data marts para poder usar herramientas de visualización y generación de informes. A veces, un formato de archivo que no es óptimo desde el punto de vista del rendimiento de lectura y escritura podría ser una buena opción al considerar el proceso analítico en general.

## Consideraciones sobre la compresión
Cuando el conjunto de datos de entrada o salida es un archivo, puede configurar la actividad de copia para realizar la compresión y descompresión de los datos a medida que se escriben en el destino. Si elige compresión, está buscando el equilibrio entre entrada/salida (E/S) y CPU. La compresión de los datos supone un costo adicional en recursos de proceso. Pero, a cambio, se reduce la E/S de red y el almacenamiento. Según los datos, puede que observe un aumento en el rendimiento general de la actividad de copia.

**Códec**: la actividad de copia admite los tipos de compresión gzip, bzip2 y Deflate. HDInsight de Azure puede consumir los tres tipos de procesamiento. Cada códec de compresión tiene sus ventajas. Por ejemplo, bzip2 tiene el rendimiento de copia más bajo, pero obtiene el mejor rendimiento de consulta de Hive porque puede dividirlo para el procesamiento. Gzip es la opción más equilibrada y es la que se usa con mayor frecuencia. Elija el códec que mejor se adapte a su escenario de principio a fin.

**Nivel:** para cada códec de compresión, puede elegir entre dos opciones: compresión más rápida y compresión más óptima. La operación de compresión más rápida comprime los datos tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima. La opción de compresión óptima dedica más tiempo a la compresión y produce una cantidad mínima de datos. Puede probar ambas opciones para ver cuál proporciona un mejor rendimiento general en su caso.

**Una consideración**: para copiar una gran cantidad de datos entre un almacén local y la nube, considere el uso de Almacenamiento de blobs provisional con compresión. El uso de almacenamiento provisional resulta de utilidad cuando el ancho de banda de la red corporativa y los servicios de Azure son el factor limitador, y si quiere que el conjunto de datos de entrada y el conjunto de datos de salida estén ambos en formato sin comprimir. En concreto, puede dividir una única actividad de copia en dos. La primera copia del origen en un blob de almacenamiento provisional en formato comprimido. La segunda copia los datos comprimidos del almacenamiento provisional y los descomprime mientras se escriben en el receptor.

## Consideraciones sobre la asignación de columnas
La propiedad **columnMappings** se puede establecer en la actividad de copia para asignar todas las columnas de entrada, o un subconjunto de ellas, a las columnas de salida. Después de que el servicio de movimiento de datos lee los datos del origen, debe realizar la asignación de columnas en los datos antes de escribirlos en el receptor. Este procesamiento adicional reduce la capacidad de proceso de la copia.

Si el almacén de datos de origen es consultable, por ejemplo, si es un almacén relacional como Base de datos SQL o SQL Server, o es un almacén NoSQL como Almacenamiento de tablas o DocumentDB, considere la posibilidad de insertar la lógica de filtrado y reordenación de columnas en la propiedad **query** en lugar de usar la asignación de columnas. De esta forma, la proyección se produce mientras el servicio de movimiento de datos lee los datos del almacén de datos de origen, donde es mucho más eficaz.

## Consideraciones sobre Data Management Gateway
Para ver las recomendaciones de configuración de la puerta de enlace, consulte [Consideraciones sobre el uso de Data Management Gateway](data-factory-move-data-between-onprem-and-cloud.md#Considerations-for-using-Data-Management-Gateway).

**Entorno de la máquina de puerta de enlace:** recomendamos usar una máquina dedicada para hospedar Data Management Gateway. Use herramientas como PerfMon para examinar el uso de CPU, memoria y ancho de banda durante una operación de copia en la máquina de puerta de enlace. Cambie a un equipo con más capacidad si la CPU, la memoria o el ancho de banda de red se convierten en un cuello de botella.

**Ejecuciones simultáneas de actividad de copia**: una única instancia de Data Management Gateway puede atender varias ejecuciones de actividad de copia al mismo tiempo, o de manera simultánea. El número máximo de trabajos simultáneos se calcula en función de la configuración de hardware de la máquina de puerta de enlace. Los trabajos de copia adicionales se ponen a la cola hasta que los selecciona la puerta de enlace o hasta que se agota el tiempo de espera de otro trabajo. Para evitar la contención de recursos en la máquina de puerta de enlace, puede organizar la programación de la actividad de copia para reducir el número de trabajos de copia que pueden estar en la cola a la vez, o bien dividir la carga entre varias máquinas de puerta de enlace.


## Otras consideraciones
Si el tamaño de los datos que quiere copiar es grande, puede ajustar la lógica empresarial para particionar los datos aún más mediante el mecanismo de fragmentación de Data Factory. A continuación, programe la actividad de copia para ejecutarse con mayor frecuencia y así reducir el tamaño de los datos con cada ejecución de actividad de copia.

Tenga cuidado con el número de conjuntos de datos y actividades de copia que necesitan Data Factory para conectarse al mismo almacén de datos al mismo tiempo. Muchos trabajos de copia simultáneos podrían limitar un almacén de datos y llevar a una degradación en el rendimiento, reintentos internos de trabajos de copia y, en ocasiones, errores de ejecución.

## Caso práctico: Copia de una instancia local de SQL Server a Almacenamiento de blobs
**Escenario**: se crea una canalización para copiar datos de una instancia de SQL Server local a Almacenamiento de blobs en formato CSV. Para acelerar el trabajo de copia, los archivos CSV se deben comprimir en formato bzip2.

**Análisis y pruebas**: el rendimiento de la actividad de copia es inferior a 2 MBps, que es mucho más lento que la referencia de rendimiento.

**Análisis y ajuste del rendimiento**: para solucionar el problema de rendimiento, primero veremos cómo se procesan y se mueven los datos.

1.	**Lectura de datos**: la puerta de enlace abre una conexión a SQL Server y envía la consulta. SQL Server responde enviando el flujo de datos a la puerta de enlace a través de la intranet.
2.	**Serialización y compresión de datos**: la puerta de enlace serializa el flujo de datos en formato CSV y comprime los datos en una secuencia bzip2.
3.	**Escritura de datos**: la puerta de enlace carga la secuencia bzip2 en Almacenamiento de blobs a través de Internet.

Como puede ver, los datos se procesan y se mueven en streaming de forma secuencial: SQL Server > LAN > Puerta de enlace > WAN > Almacenamiento de blobs. **El rendimiento general viene determinado por el rendimiento mínimo a través de la canalización**.

![flujo de datos](./media/data-factory-copy-activity-performance/case-study-pic-1.png)

Puede que uno o varios de los siguientes factores provoquen el cuello de botella en el rendimiento:

-	**Origen:** el propio SQL Server tiene un rendimiento bajo debido a cargas intensas.
-	**Data Management Gateway**
	-	**LAN**: la puerta de enlace se encuentra lejos de la máquina de SQL Server y tiene una conexión de ancho de banda bajo.
	-	**Puerta de enlace**: la puerta de enlace ha alcanzado sus limitaciones de carga para llevar a cabo lo siguiente:
		-	**Serialización:** la serialización del flujo de datos a formato CSV tiene un rendimiento bajo.
		-	**Compresión**: ha elegido un códec de compresión lenta (por ejemplo, bzip2, a 2,8 MBps con Core i7).
	-	**WAN**: el ancho de banda entre la red corporativa y los servicios de Azure es bajo (por ejemplo, T1 = 1544 kbps. T2 = 6312 kbps).
-	**Receptor**: el Almacenamiento de blobs tiene un rendimiento bajo. (Esta situación es improbable porque su SLA garantiza un mínimo de 60 MBps).

En este caso, la compresión de datos bzip2 podría estar ralentizando la canalización entera. El cambio a un códec de compresión gzip podría aliviar este cuello de botella.


## Caso práctico: Uso de copias en paralelo  

**Escenario I:** se copian 1000 archivos de 1 MB del sistema de archivos local a Almacenamiento de blobs.

**Análisis y optimización del rendimiento**: por ejemplo, si ha instalado Data Management Gateway en una máquina de cuatro núcleos, Data Factory usa 16 copias en paralelo para mover los archivos del sistema de archivos a Almacenamiento de blobs de manera simultánea. Esta ejecución en paralelo debe tener como resultado un alto rendimiento. También puede especificar explícitamente el número de copias en paralelo. Al copiar muchos archivos pequeños, las copias en paralelo ayudan considerablemente al rendimiento al usar los recursos de forma más eficaz.

![Escenario 1.](./media/data-factory-copy-activity-performance/scenario-1.png)

**Escenario II**: se copian 20 blobs de 500 MB cada uno de Almacenamiento de blobs a Data Lake Store Analytics y luego se optimiza el rendimiento.

**Análisis y optimización del rendimiento**: en este escenario, Data Factory copia los datos de Almacenamiento de blobs a Data Lake Store mediante una única copia (**parallelCopies** se establece en 1) y unidades de movimiento de datos de nube única. El rendimiento que observe estará próximo al que se describe en la [sección de referencia de rendimiento](#performance-reference).

![Escenario 2.](./media/data-factory-copy-activity-performance/scenario-2.png)

**Escenario III**: el tamaño de cada archivo es mayor que decenas de MB y el volumen total es grande.

**Análisis y optimización del rendimiento**: el aumento del valor de **parallelCopies** no da lugar a un rendimiento mejor de la copia debido a las limitaciones de recursos de una DMU de nube única. En su lugar, debe especificar más DMU de nube para obtener más recursos de cara a realizar el movimiento de los datos. No especifique un valor para la propiedad **parallelCopies**. Data Factory controla el paralelismo automáticamente. En este caso, si establece **cloudDataMovementUnits** en 4, se produce una capacidad de proceso cuádruple.

![Escenario 3.](./media/data-factory-copy-activity-performance/scenario-3.png)

## Referencia
Estas son algunas referencias para la supervisión y la optimización del rendimiento para algunos de los almacenes de datos admitidos:

- Almacenamiento de Azure (que incluye Almacenamiento de blobs y Almacenamiento de tablas): [Objetivos de escalabilidad de Almacenamiento de Azure](../storage/storage-scalability-targets.md) y [Lista de comprobación de rendimiento y escalabilidad de Almacenamiento de Microsoft Azure](../storage//storage-performance-checklist.md)
- Base de datos SQL de Azure: puede [supervisar el rendimiento](../sql-database/sql-database-service-tiers.md#monitoring-performance) y comprobar el porcentaje de unidades de transacción de base de datos (DTU).
- Almacenamiento de datos SQL de Azure: su capacidad se mide en unidades de almacenamiento de datos (DWU); consulte [Administración de la potencia de proceso en Almacenamiento de datos SQL de Azure (información general)](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)
- Azure DocumentDB: [Niveles de rendimiento en DocumentDB](../documentdb/documentdb-performance-levels.md)
- Instancia de SQL Server local: [Supervisión y optimización del rendimiento](https://msdn.microsoft.com/library/ms189081.aspx)
- Servidor de archivos local: [Performance Tuning for File Servers](https://msdn.microsoft.com/library/dn567661.aspx) (Optimización del rendimiento para servidores de archivos)

<!---HONumber=AcomDC_0831_2016-->