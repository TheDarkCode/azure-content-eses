<properties
	pageTitle="Integración del SDK de Windows Universal"
	description="Integración del SDK de Windows Universal para Azure Mobile Engagement" 									
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="dwrede"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="08/12/2016"
	ms.author="piyushjo;ricksal" />

#Integración del SDK de Windows Universal para Azure Mobile Engagement

Este documento describe todas las opciones de integración y configuración disponibles para el SDK de Windows Universal para Azure Mobile Engagement.

## Requisitos previos

Antes de comenzar este tutorial, primero debe completar la [Introducción a Azure Mobile Engagement para aplicaciones Android](mobile-engagement-windows-store-dotnet-get-started.md), un tutorial de 15 minutos.

## Características avanzadas

### Características de informes
Puede agregar estas características:

1. [Opciones de informes avanzados](mobile-engagement-windows-store-advanced-reporting.md)

2. [Opciones de configuración avanzada](mobile-engagement-windows-store-advanced-configuration.md)

### Notificaciones

[Integración de Reach (notificaciones) en su aplicación universal de Windows](mobile-engagement-windows-store-integrate-engagement-reach.md)

### Implementación del plan de etiquetas:

[Uso de la API de etiquetado avanzado de Mobile Engagement en su aplicación universal de Windows](mobile-engagement-windows-store-use-engagement-api.md)

##Notas de la versión

###3\.4.0 (04/19/2016)

-   Mejoras de superposición de Reach.
-   API "TestLogLevel" agregada para habilitar, deshabilitar y filtrar registros de consola emitidos por el SDK.
-   Notificaciones en actividad corregidas dirigidas a la primera actividad que no aparecen en el inicio de la aplicación.

Para versiones anteriores, consulte las [notas de la versión completas](mobile-engagement-windows-store-release-notes.md).

##Procedimientos de actualización

Si ya integró una versión anterior de Engagement en la aplicación, debería tener en cuenta los siguientes puntos a la hora de actualizar el SDK.

Es posible que tenga que seguir varios procedimientos si se perdió varias versiones del SDK. Consulte los [procedimientos de actualización](mobile-engagement-windows-store-upgrade-procedure.md) completos. Por ejemplo, si migra desde 0.10.1 a 0.11.0, primero debe seguir el procedimiento "de 0.9.0 a 0.10.1" y luego el procedimiento "de 0.10.1 a 0.11.0".

###De 3.3.0 a 3.4.0

####Registros de prueba

Los registros de consola generados por el SDK ahora se pueden habilitar, deshabilitar o filtrar. Para personalizarlo, actualice la propiedad `EngagementAgent.Instance.TestLogEnabled` a uno de los valores disponibles en la enumeración `EngagementTestLogLevel`, por ejemplo:

			EngagementAgent.Instance.TestLogLevel = EngagementTestLogLevel.Verbose;
			EngagementAgent.Instance.Init();

####Recursos

Se ha mejorado la superposición de Reach. Forma parte de los recursos del paquete NuGet del SDK.

Al actualizar a la nueva versión del SDK puede elegir si desea conservar los archivos existentes de la carpeta de superposición de los recursos:

* Si la superposición anterior funciona o va a integrar los elementos `WebView` manualmente, puede decidir mantener los archivos de salida. El funcionamiento no se verá afectado.
* Para actualizar a la nueva superposición, reemplace toda la carpeta `overlay` de sus recursos por la nueva desde el paquete del SDK (aplicaciones de UWP: tras la actualización, puede obtener la nueva carpeta de superposición desde %USERPROFILE%\\.nuget\\packages\\MicrosoftAzure.MobileEngagement\\3.4.0\\content\\win81\\Resources).

> [AZURE.WARNING] El uso de la nueva superposición sobrescribirá todas las personalizaciones realizadas en la versión anterior.

### Actualizar de versiones anteriores

Consulte [Procedimientos de actualización](mobile-engagement-windows-store-upgrade-procedure.md)

<!---HONumber=AcomDC_0817_2016-->