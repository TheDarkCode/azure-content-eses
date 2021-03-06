<properties
   pageTitle="Preguntas más frecuentes sobre el almacén de Servicios de recuperación | Microsoft Azure"
   description="Esta versión de las preguntas más frecuentes es compatible con la versión de vista previa pública del servicio Copia de seguridad de Azure. Respuestas a las preguntas más frecuentes sobre agente de copia de seguridad, copia de seguridad y retención, recuperación, seguridad y otras preguntas comunes sobre la solución de Copia de seguridad de Azure."
   services="backup"
   documentationCenter=""
   authors="markgalioto"
   manager="jwhit"
   editor=""
   keywords="solución de copia de seguridad; servicio de copia de seguridad"/>

<tags
   ms.service="backup"
   ms.workload="storage-backup-recovery"
	 ms.tgt_pltfrm="na"
	 ms.devlang="na"
	 ms.topic="get-started-article"
	 ms.date="08/21/2016"
	 ms.author="trinadhk; markgal; jimpark;"/>

# Almacén de Servicios de recuperación: preguntas más frecuentes

> [AZURE.SELECTOR]
- [Preguntas más frecuentes acerca del servicio Copia de seguridad para el modo clásico](backup-azure-backup-faq.md)
- [Preguntas más frecuentes acerca del servicio Copia de seguridad para el modo de Resource Manager](backup-azure-backup-ibiza-faq.md)

En este artículo se proporciona información específica acerca del almacén de Servicios de recuperación y complementa el artículo con las [preguntas más frecuentes acerca de Copia de seguridad de Azure](backup-azure-backup-faq). La sección de preguntas más frecuentes acerca de Copia de seguridad de Azure proporciona el conjunto completo de preguntas y respuestas sobre este servicio.

Las preguntas acerca de Copia de seguridad de Azure se pueden realizar en la sección Disqus de este artículo o de otros artículos relacionados. También se pueden publicar preguntas sobre el servicio Copia de seguridad de Azure en el [foro de debate](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup).

## Los almacenes de Servicios de recuperación se basan en Resource Manager, ¿aún se admiten los almacenes de Copia de seguridad (modo clásico)? <br/>
Sí, los almacenes de Copia de seguridad son todavía compatibles. Cree almacenes de Copia de seguridad en el [portal de clásico](https://manage.windowsazure.com). Cree almacenes de Servicios de recuperación en el [Portal de Azure](https://portal.azure.com). Sin embargo, se recomienda encarecidamente crear el almacén de Servicios de recuperación, ya que todas las futuras mejoras solo estarán disponibles en dicho almacén.

## ¿Se puede migrar un almacén de Copia de seguridad a un almacén de Servicios de recuperación? <br/>
Por desgracia no, en este momento no puede migrar el contenido de un almacén de Copia de seguridad a un almacén de Servicios de recuperación. Estamos trabajando para agregar esta funcionalidad, pero no está disponible como parte de la versión previa pública.

## ¿Admiten los almacenes de Servicios de recuperación máquinas virtuales implementadas con el modelo clásico o máquinas virtuales implementadas con Resource Manager? <br/>
Los almacenes de Servicios de recuperación admiten ambos modelos. Puede hacer copias de seguridad de una máquina virtual creada en el portal clásico (que son las máquinas virtuales del modo clásico) o una máquina virtual creada en el Portal de Azure (que se basan en Resource Manager) en un almacén de Servicios de recuperación.

## He realizado copias de seguridad de mis máquinas virtuales clásicas en el almacén de Copia de seguridad. Ahora deseo migrar mis máquinas virtuales del modo clásico al modo de Resource Manager. ¿Cómo puedo realizar una copia de seguridad de ellas en el almacén de Servicios de recuperación?
Las copias de seguridad de las máquinas virtuales clásicas del almacén de Copia de seguridad no se migrarán automáticamente al almacén de Servicios de recuperación al migrar las máquinas virtuales desde el modo clásico al de Resource Manager. Para realizar la migración de las copias de seguridad de máquinas virtuales, siga estos pasos:

1. En el almacén de Copia de seguridad, vaya a la pestaña **Elementos protegidos** y seleccione la máquina virtual. Haga clic en [Detener la protección](backup-azure-manage-vms-classic.md#stop-protecting-virtual-machines). Deje la opción *Eliminar los datos de copia de seguridad asociados* **desactivada**.
2. Migre la máquina virtual del modo clásico al modo de Resource Manager. Asegúrese de que tanto el almacenamiento como la red correspondientes a la máquina virtual también se migran al modo de Resource Manager.
3. Cree un almacén de Servicios de recuperación y configure la copia de seguridad de la máquina virtual migrada mediante la acción **Copia de seguridad** sobre el panel del almacén. Más información acerca de cómo [habilitar la copia de seguridad en el almacén de Servicios de recuperación](backup-azure-vms-first-look-arm.md)

<!---HONumber=AcomDC_0824_2016-->