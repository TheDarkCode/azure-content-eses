<properties 
   pageTitle="Uso de Apache Phoenix y SQuirreL en HDInsight | Microsoft Azure" 
   description="Aprenda a usar Apache Phoenix en HDInsight y cómo instalar y configurar SQuirreL en su estación de trabajo para conectarse a un clúster de HBase en HDInsight." 
   services="hdinsight" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="09/02/2016"
   ms.author="jgao"/>

# Uso de Apache Phoenix con clústeres de HBase basados en Linux en HDInsight  

Aprenda a utilizar [Apache Phoenix](http://phoenix.apache.org/) en HDInsight y SQLLine. Para obtener más información acerca de Phoenix, consulte [Phoenix en 15 minutos o menos](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html). Para la gramática de Phoenix, vea [Gramática de Phoenix](http://phoenix.apache.org/language/index.html).

>[AZURE.NOTE] Para obtener información de la versión de Phoenix en HDInsight, consulte [Novedades en las versiones de clústeres de Hadoop proporcionadas por HDInsight][hdinsight-versions].

##Uso de SQLLine
[SQLLine](http://sqlline.sourceforge.net/) es una utilidad de línea de comandos para ejecutar SQL.

###Requisitos previos
Antes de usar SQLLine, debe tener lo siguiente:

- **Un clúster de HBase en HDInsight**. Para obtener información sobre el aprovisionamiento del clúster de HBase, consulte [Introducción a HBase Apache en HDInsight][hdinsight-hbase-get-started].
- **Conexión al clúster de HBase a través del protocolo de escritorio remoto**. Para conocer las instrucciones, consulte [Administración de clústeres de Hadoop en HDInsight mediante el Portal de Azure clásico][hdinsight-manage-portal].


Cuando se conecte a un clúster de HBase, deberá conectarse a uno de los ZooKeepers. Cada clúster de HDInsight tiene 3 Zookeepers.

**Para averiguar el nombre de host de ZooKeeper**

1. Vaya a **https://<nombreDelClúster>.azurehdinsight.net/** para abrir Ambari.
2. Escriba el nombre de usuario HTTP (clúster) y la contraseña para iniciar sesión.
3. En el menú izquierdo, haga clic en **ZooKeeper**. Verá una lista con 3 **servidores de ZooKeeper**.
4. Haga clic en uno de los **servidores de ZooKeeper** que aparecen. En el panel Resumen, busque el **Nombre de host**. Es similar a *zk1-jdolehb.3lnng4rcvp5uzokyktxs4a5dhd.bx.internal.cloudapp.net*.

**Para usar SQLLine**

1. Conéctese al clúster con SSH. Para obtener instrucciones, consulte [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux, Unix u OS X](hdinsight-hadoop-linux-use-ssh-unix.md) o [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Windows](hdinsight-hadoop-linux-use-ssh-windows.md), según el sistema operativo de su equipo cliente.

2. Desde SSH, introduzca los siguientes comandos para ejecutar SQLLine:

        cd /usr/hdp/2.2.9.1-7/phoenix/bin
        ./sqlline.py <ClusterName>:2181:/hbase-unsecure

2. Ejecute los siguientes comandos para crear una tabla de HBase e insertar algunos datos:

		CREATE TABLE Company (COMPANY_ID INTEGER PRIMARY KEY, NAME VARCHAR(225));
	
		!tables
		
		UPSERT INTO Company VALUES(1, 'Microsoft');
		
		SELECT * FROM Company;
        
        !quit

Para obtener más información, vea [Manual de SQLLine](http://sqlline.sourceforge.net/#manual) y [Gramática de Phoenix](http://phoenix.apache.org/language/index.html).


 
##Pasos siguientes
En este artículo, ha aprendido cómo utilizar Phoenix Apache en HDInsight. Para obtener más información, consulte:

- [Información general de HBase de HDInsight][hdinsight-hbase-overview]\: HBase es una base de datos NoSQL de código abierto Apache basada en Hadoop que proporciona acceso aleatorio y una coherencia sólida para grandes cantidades de datos no estructurados y semiestructurados.
- [Aprovisionamiento de clústeres de HBase en Red virtual de Azure][hdinsight-hbase-provision-vnet]\: con la integración de redes virtuales, los clústeres de HBase se pueden implementar en la misma red virtual que sus aplicaciones para que estas puedan comunicarse directamente con HBase.
- [Configuración de la replicación de HBase en HDInsight](hdinsight-hbase-geo-replication.md): aprenda a configurar la replicación de HBase entre dos centros de datos de Azure.
- [Análisis de opiniones de Twitter con HBase en HDInsight][hbase-twitter-sentiment]\: descubra cómo realizar [análisis de opinión](http://en.wikipedia.org/wiki/Sentiment_analysis) en tiempo real de grandes volúmenes de datos con HBase en un clúster de Hadoop en HDInsight.

[azure-portal]: https://portal.azure.com
[vnet-point-to-site-connectivity]: https://msdn.microsoft.com/library/azure/09926218-92ab-4f43-aa99-83ab4d355555#BKMK_VNETPT

[hdinsight-versions]: hdinsight-component-versioning.md
[hdinsight-hbase-get-started]: hdinsight-hbase-tutorial-get-started.md
[hdinsight-manage-portal]: hdinsight-administer-use-management-portal.md#connect-to-hdinsight-clusters-by-using-rdp
[hdinsight-hbase-provision-vnet]: hdinsight-hbase-provision-vnet.md
[hdinsight-hbase-overview]: hdinsight-hbase-overview.md
[hbase-twitter-sentiment]: hdinsight-hbase-analyze-twitter-sentiment.md

[hdinsight-hbase-phoenix-sqlline]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-phoenix-sqlline.png
[img-certificate]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vpn-certificate.png
[img-vnet-diagram]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vnet-point-to-site.png
[img-squirrel-driver]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-driver.png
[img-squirrel-alias]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-alias.png
[img-squirrel]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel.png
[img-squirrel-sql]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-sql.png


 

<!---HONumber=AcomDC_0907_2016-->