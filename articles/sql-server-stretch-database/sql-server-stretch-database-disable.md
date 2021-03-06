<properties
	pageTitle="Deshabilitación de Stretch Database y devolución de datos remotos | Microsoft Azure"
	description="Obtenga información sobre cómo deshabilitar Stretch Database para una tabla y, opcionalmente, recuperar datos remotos."
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/05/2016"
	ms.author="douglasl"/>

# Deshabilitación de Stretch Database y devolución de datos remotos

Para deshabilitar Stretch Database para una tabla, seleccione **Stretch** para una tabla en SQL Server Management Studio. Seleccione una de las siguientes opciones.

-   **Deshabilitar| Bring data back from Azure**. Copia los datos remotos para la tabla de Azure a SQL Server y, a continuación, deshabilita Stretch Database para la tabla. Esta operación provoca costos de transferencia de datos y no se puede cancelar.

-   **Deshabilitar| Leave data in Azure**. Deshabilita Stretch Database para la tabla.  Abandona los datos remotos para la tabla en Azure.

También puede utilizar Transact-SQL para deshabilitar Stretch Database para una tabla o una base de datos.

Después de deshabilitar Stretch Database para una tabla, se detiene la migración de datos y los resultados de la consulta dejan de incluir los resultados de la tabla remota.

Si simplemente desea pausar la migración de datos, consulte [Pausa y reanudación de Stretch Database](sql-server-stretch-database-pause.md).

>   [AZURE.NOTE] Al deshabilitar la característica Stretch Database para una tabla o una base de datos, no se elimina el objeto remoto. Si quiere eliminar la tabla remota o la base de datos remota, debe hacerlo mediante el Portal de administración de Azure. Los objetos remotos seguirán generando costos de Azure hasta que se eliminan. Para obtener más información, consulte la página [Precios SQL Server Stretch Database](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/).

## Deshabilitación de Stretch Database para una tabla

### Uso de SQL Server Management Studio para deshabilitar Stretch Database para una tabla

1.  En SQL Server Management Studio, en el Explorador de objetos, seleccione la tabla para la que desee deshabilitar Stretch Database.

2.  Haga clic con el botón derecho y seleccione **Stretch** y, a continuación, seleccione una de las siguientes opciones.

    -   **Deshabilitar| Bring data back from Azure**. Copia los datos remotos para la tabla de Azure a SQL Server y, a continuación, deshabilita Stretch Database para la tabla. Este comando no se puede cancelar.

        >   [AZURE.NOTE] Al volver a copiar los datos remotos de la tabla de Azure en el SQL Server, se generan costos de transferencia de datos. Para obtener más información, consulte [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/).

        Después de que todos los datos remotos se hayan copiado de Azure a SQL Server, Stretch se deshabilita para la tabla.

    -   **Deshabilitar| Leave data in Azure**. Deshabilita Stretch Database para la tabla.  Abandona los datos remotos para la tabla en Azure.

    >   [AZURE.NOTE] Al deshabilitar la característica Stretch Database para una tabla, no se eliminan los datos remotos ni la tabla remota. Si quiere eliminar la tabla remota, debe hacerlo mediante el Portal de administración de Azure. La tabla remota seguirá generando costos de almacenamiento de Azure hasta que la elimine. Para obtener más información, consulte la página [Precios SQL Server Stretch Database](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/).

### Uso de Transact-SQL para deshabilitar Stretch Database para una tabla

-   Si quiere deshabilitar Stretch para una tabla y copiar los datos remotos de ella de Azure a SQL Server, ejecute el siguiente comando. Después de que todos los datos remotos se hayan copiado de Azure a SQL Server, Stretch se deshabilita para la tabla.

    Este comando no se puede cancelar.

    ```tsql
	USE <Stretch-enabled database name>;
    GO
    ALTER TABLE <Stretch-enabled table name>  
       SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = INBOUND ) ) ;
    GO
    ```
    >   [AZURE.NOTE] Al volver a copiar los datos remotos de la tabla de Azure en el SQL Server, se generan costos de transferencia de datos. Para obtener más información, consulte [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/).

-   Para deshabilitar Stretch para una tabla y abandonar los datos remotos, ejecute el siguiente comando.

    ```tsql
    ALTER TABLE <table_name>
       SET ( REMOTE_DATA_ARCHIVE = OFF_WITHOUT_DATA_RECOVERY ( MIGRATION_STATE = PAUSED ) ) ;
    ```

>   [AZURE.NOTE] Al deshabilitar la característica Stretch Database para una tabla, no se eliminan los datos remotos ni la tabla remota. Si quiere eliminar la tabla remota, debe hacerlo mediante el Portal de administración de Azure. La tabla remota seguirá generando costos de almacenamiento de Azure hasta que la elimine. Para obtener más información, consulte la página [Precios SQL Server Stretch Database](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/).

## Deshabilitación de Stretch Database para una base de datos
Para poder deshabilitar Stretch Database para una base de datos, tendrá que deshabilitar Stretch Database en las tablas habilitadas para Stretch individuales en la base de datos.

### Uso de SQL Server Management Studio para deshabilitar Stretch Database para una base de datos

1.  En SQL Server Management Studio, en el Explorador de objetos, seleccione la base de datos para la que desee deshabilitar Stretch Database.

2.  Haga clic con el botón derecho y seleccione **Tareas**, **Stretch** y **Deshabilitar**.

>   [AZURE.NOTE] Al deshabilitar la característica Stretch Database para una base de datos, no se elimina la base de datos remota. Si quiere eliminar la base de datos remota, debe hacerlo mediante el Portal de administración de Azure. La base de datos remota seguirá generando costos de almacenamiento de Azure hasta que la elimine. Para obtener más información, consulte la página [Precios SQL Server Stretch Database](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/).

### Uso de Transact-SQL para deshabilitar Stretch Database para una base de datos
Ejecute el siguiente comando.

```tsql
ALTER DATABASE <database name>
    SET REMOTE_DATA_ARCHIVE = OFF ;
```

>   [AZURE.NOTE] Al deshabilitar la característica Stretch Database para una base de datos, no se elimina la base de datos remota. Si quiere eliminar la base de datos remota, debe hacerlo mediante el Portal de administración de Azure. La base de datos remota seguirá generando costos de almacenamiento de Azure hasta que la elimine. Para obtener más información, consulte la página [Precios SQL Server Stretch Database](https://azure.microsoft.com/pricing/details/sql-server-stretch-database/).

## Otras referencias

[Opciones de ALTER DATABASE SET (Transact-SQL)](https://msdn.microsoft.com/library/bb522682.aspx)

[Pausa y reanudación Stretch Database](sql-server-stretch-database-pause.md)

<!---HONumber=AcomDC_0810_2016-->
