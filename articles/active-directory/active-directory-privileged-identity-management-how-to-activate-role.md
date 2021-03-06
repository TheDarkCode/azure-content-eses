<properties
   pageTitle="Activación o desactivación de un rol | Microsoft Azure"
   description="Aprenda a activar roles para identidades con privilegios con la aplicación Privileged Identity Management de Azure."
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/02/2016"
   ms.author="kgremban"/>

# Activación y desactivación de roles en Privileged Identity Management de Azure AD

Privileged Identity Management de Azure Active Directory (AD) simplifica el modo en que las empresas administra el acceso con privilegios a los recursos de Azure AD y en otros servicios en línea de Microsoft como Office 365 o Microsoft Intune.

Si se le ha considerado apto para un rol administrativo, significa que puede activar el rol cuando tenga la necesidad de realizar una tarea para la que le haga falta. Por ejemplo, si administra de vez en cuando características de Office 365, los administradores de roles con privilegios de su organización puede que no le hayan asignado el rol de administrador global permanente, ya que ese rol afecta también a otros servicios. En su lugar, pueden asignarle roles de Azure AD como administrador de Exchange Online. Puede solicitar una asignación temporal de ese rol cuando necesite sus privilegios y tendrá control administrativo en Office 365 durante un periodo predeterminado.

Este artículo va dirigido a los administradores que necesitan activar su rol en Privileged Identity Management (PIM) de Azure AD. Le llevará por los pasos para activar un rol cuando necesite los permisos y desactivar el rol cuando haya terminado.


## Incorporación de la aplicación Privileged Identity Management

Use la aplicación Privileged Identity Management de Azure AD en el [Portal de Azure](https://portal.azure.com/) para solicitar una activación del rol aunque vaya a trabajar en otro portal o mediante PowerShell. Si no tiene la aplicación en el Portal de Azure, siga estos pasos para comenzar:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).
2. Seleccione su nombre de usuario en la esquina superior derecha del Portal de Azure y elija el directorio donde va a trabajar.
3. Seleccione **Nuevo** > **Seguridad e identidad** > **Privileged Identity Management de Azure AD**.

    ![Habilitación de PIM en el portal][1]

4. Active **Anclar al panel** y haga clic en **Crear**. Se abre la aplicación Privileged Identity Management.

## Activación de un rol

Cuando necesite asumir un rol, puede solicitar la activación mediante el botón **Activar mis roles** de la aplicación Privileged Identity Management de Azure AD.


1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/) y seleccione el icono de Privileged Identity Management de Azure AD.
2. Seleccione **Activar mis roles**. Aparecerá una lista de los roles que se le asignaron.
3. Seleccione el rol que quiera activar.
4. Seleccione **Activar**. Aparecerá la hoja **Solicitar activación del role**.
5. Algunos roles requieren Multi-Factor Authentication (MFA) para poder activarlos. Solo tiene que autenticarse una vez por sesión.

    ![Comprobación con MFA antes de la activación del rol: captura de pantalla][2]

6. En el campo de texto, escriba el motivo de la solicitud de activación. Algunos roles requieren que proporcione un número de vale de problema.
7. Seleccione **Aceptar**. El rol se activará y el cambio de rol se hará visible en Microsoft Online Services.

## Desactivación de un rol

Cuando se ha activado un rol, si alcanza su límite de tiempo, se desactiva automáticamente.

Si termina pronto, también puede desactivar un rol de forma manual en la aplicación Privileged Identity Management de Azure AD. Seleccione **Activar mis roles**, elija el rol que haya usado y seleccione **Desactivar**.


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## Pasos siguientes

Si le interesa aprender más sobre Privileged Identity Management de Azure AD, puede encontrar más información en los siguientes vínculos.

[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/PIM_EnablePim.png
[2]: ./media/active-directory-privileged-identity-management-how-to-activate-role/PIM_activation_MFA.png

<!---HONumber=AcomDC_0907_2016-->