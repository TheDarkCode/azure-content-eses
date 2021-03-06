<properties
	pageTitle="Habilitación del inicio de sesión único entre aplicaciones en Android mediante ADAL | Microsoft Azure"
	description="Cómo utilizar las características del SDK de ADAL para habilitar el inicio de sesión único entre las aplicaciones. "
	services="active-directory"
	documentationCenter=""
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="android"
	ms.devlang="java"
	ms.topic="article"
	ms.date="05/16/2016"
	ms.author="brandwe"/>


# Habilitación de SSO entre aplicaciones en Android mediante ADAL


Los clientes esperan que se proporcione un inicio de sesión único (SSO) para que los usuarios solo tengan que escribir sus credenciales una sola vez y esas credenciales automáticamente funcionen en otras aplicaciones. La dificultad de escribir el nombre de usuario y contraseña en una pantalla pequeña, a menudo combinado con un factor adicional (2FA) como una llamada de teléfono o un código de mensaje de texto, provoca una rápida insatisfacción si el usuario tiene que hacer esto más de una vez para su producto.

Además, si hace uso de una plataforma de identidad que puedan utilizar otras aplicaciones, como cuentas Microsoft o una cuenta de trabajo de Office 365, los clientes esperan que las credenciales estén disponibles para todas sus aplicaciones, independientemente del proveedor.

La plataforma de Microsoft Identity, junto con los SDK correspondientes, se ocupa de este trabajo ingrato y le ofrece la posibilidad de satisfacer a sus clientes con el inicio de sesión único, ya sea dentro de su propio conjunto de aplicaciones o, como sucede con nuestra funcionalidad de agente y las aplicaciones de Authenticator, en todo el dispositivo.

Este tutorial le indicará cómo configurar nuestro SDK dentro de la aplicación para proporcionar esta ventaja a sus clientes.

Este tutorial se aplica a:

* Azure Active Directory
* Azure Active Directory B2C
* Azure Active Directory B2B
* Acceso condicional de Azure Active Directory

Tenga en cuenta que en el documento siguiente se considera que tiene conocimientos acerca de cómo [aprovisionar aplicaciones en el portal heredado para Azure Active Directory](active-directory-how-to-integrate.md) y que ha integrado su aplicación con el [SDK de Android de Microsoft Identity](https://github.com/AzureAD/azure-activedirectory-library-for-android).

## Conceptos de inicio de sesión único en la plataforma de Microsoft Identity

### Agentes de Microsoft Identity

Microsoft proporciona aplicaciones para todas las plataformas móviles que permiten el traspaso de credenciales entre aplicaciones de distintos proveedores, al tiempo que habilita características mejoradas especiales que requieren un único lugar seguro desde el que validar las credenciales. A estas aplicaciones las llamamos **agentes**. En iOS y Android, estos agentes se consiguen a través de aplicaciones descargables que los clientes instalan de forma independiente o bien se insertan en el dispositivo gracias a una empresa que administra el dispositivo de manera total o parcial para sus empleados. Gracias a estos agentes, es posible administrar la seguridad de solo unas aplicaciones o bien del dispositivo en su totalidad, en función de lo que decidan los administradores de TI. En Windows, esta funcionalidad se proporciona mediante un selector de cuentas integrado en el sistema operativo, que se conoce técnicamente como agente de autenticación web.

Siga leyendo este artículo para entender el uso que hacemos de estos agentes y la percepción que tendrán de ellos sus clientes durante el proceso de inicio de sesión para la plataforma Microsoft Identity.

### Patrones para iniciar sesión en dispositivos móviles

El acceso a credenciales en los dispositivos sigue dos patrones básicos para la plataforma Microsoft Identity:

* Inicios de sesión no asistidos por agente
* Inicios de sesión asistidos por agente

#### Inicios de sesión no asistidos por agente

Los inicios de sesión no asistidos por agente son experiencias de inicio de sesión que ocurren en línea con la aplicación y usan el almacenamiento local en el dispositivo para esa aplicación. Este almacenamiento puede compartirse entre las aplicaciones, pero las credenciales están estrechamente vinculadas a la aplicación o al conjunto de aplicaciones que usan esa credencial. Esta es la experiencia que probablemente ha experimentado en muchas aplicaciones móviles en las que escribe un nombre de usuario y una contraseña en la propia aplicación.

Estos inicios de sesión tienen las siguientes ventajas:

-  La experiencia de usuario se desarrolla por completo dentro de la aplicación.
-  Las credenciales se pueden compartir entre aplicaciones que estén firmadas por el mismo certificado, proporcionando una experiencia de inicio de sesión único para el conjunto de aplicaciones.
-  Se proporciona el control de la experiencia de inicio de sesión a la aplicación antes y después del inicio de sesión.

Estos inicios de sesión tienen las siguientes desventajas:

- El usuario no puede experimentar el inicio de sesión único entre todas las aplicaciones que usan una identidad de Microsoft Identity, solo en aquellas identidades de Microsoft Identity que su aplicación posee y ha configurado.
- La aplicación no se puede utilizar con las características empresariales más avanzadas, como el acceso condicional, ni utilizar el conjunto de productos de InTune.
- La aplicación no admite la autenticación basada en certificados para los usuarios empresariales.

A continuación se representa el funcionamiento de los SDK de Microsoft Identity con el almacenamiento compartido de sus aplicaciones para habilitar el inicio de sesión único:

```
+------------+ +------------+  +-------------+
|            | |            |  |             |
|   App 1    | |   App 2    |  |   App 3     |
|            | |            |  |             |
|            | |            |  |             |
+------------+ +------------+  +-------------+
| Azure SDK  | | Azure SDK  |  | Azure SDK   |
+------------+-+------------+--+-------------+
|                                            |
|            App Shared Storage              |
+--------------------------------------------+
```

#### Inicios de sesión asistidos por agente

Los inicios de sesión asistidos por agente son experiencias de inicio de sesión que se producen dentro de la aplicación del agente y usan el almacenamiento y la seguridad del agente para compartir las credenciales en todas las aplicaciones del dispositivo que utilizan la plataforma Microsoft Identity. Esto significa que las aplicaciones se basarán en el agente para que los usuarios inicien sesión. En iOS y Android, estos agentes se consiguen a través de aplicaciones descargables que los clientes instalan de forma independiente o bien se insertan en el dispositivo por gracias a una empresa que administra el dispositivo para sus usuarios. Un ejemplo de este tipo de aplicación es la aplicación Azure Authenticator en iOS. En Windows, esta funcionalidad se proporciona mediante un selector de cuentas integrado en el sistema operativo, que se conoce técnicamente como agente de autenticación web. La experiencia varía según la plataforma y a veces puede ser problemática para los usuarios si no se administra correctamente. Quizá este modelo le resulte más familiar si tiene instalada la aplicación Facebook y utiliza la funcionalidad de inicio de sesión de Facebook en otra aplicación. La plataforma Microsoft Identity utiliza este mismo modelo.

En el caso de iOS, esto da lugar a una animación de "transición" en la que la aplicación se envía a un segundo plano y las aplicaciones de Azure Authenticator vienen al primer plano para que el usuario seleccione la cuenta en la que desea iniciar sesión.

En el caso de Android y Windows, el selector de cuentas se muestra sobre la aplicación, lo cual interrumpe menos la experiencia del usuario.

#### ¿Cómo se invoca el agente?

Si se instala un agente compatible en el dispositivo, como la aplicación Azure Authenticator, los SDK de Microsoft Identity se ocuparán automáticamente de la tarea de invocar al agente cuando un usuario indique que desea iniciar sesión con alguna de las cuentas de la plataforma Microsoft Identity. Podría tratarse de una cuenta personal de Microsoft, una cuenta profesional o educativa o una cuenta que especifique y hospede en Azure mediante nuestros productos B2C y B2B. Garantizamos la protección de las credenciales en el proceso de solicitud y devolución a la aplicación mediante el uso de algoritmos y sistemas de cifrado sumamente seguros. Los detalles técnicos exactos de estos mecanismos no están publicados, pero se han desarrollado en colaboración con Apple y Google.

**El desarrollador tiene la opción de elegir si el SDK de Microsoft Identity llama al agente o usa el flujo sin asistencia del agente.** Sin embargo, si el desarrollador decide no utilizar el flujo con asistencia del agente, no podrá beneficiarse del uso de las credenciales de SSO que el usuario puede haber añadido en el dispositivo; además, impide que la aplicación se use con las características empresariales que Microsoft proporciona a sus clientes, como el acceso condicional, las funcionalidades de administración de Intune y la autenticación basada en certificados.

Estos inicios de sesión tienen las siguientes ventajas:

-  El usuario experimenta el inicio de sesión único en todas sus aplicaciones, con independencia del proveedor.
-  La aplicación puede aprovechar las características empresariales más avanzadas, como el acceso condicional, o utilizar el conjunto de productos de InTune.
-  La aplicación puede admitir la autenticación basada en certificados para los usuarios empresariales.
- La experiencia de inicio de sesión es mucho más segura, ya que tanto la identidad de la aplicación como el usuario se verifican por parte de la aplicación de agente con algoritmos de seguridad y sistemas de cifrado adicionales.

Estos inicios de sesión tienen las siguientes desventajas:

- En iOS, el usuario pasa por un proceso externo a la experiencia de la aplicación mientras se eligen las credenciales.
- Se pierde la capacidad de administrar la experiencia de inicio de sesión para los clientes dentro de la aplicación.



A continuación se representa el funcionamiento de los SDK de Microsoft Identity con las aplicaciones de agente para habilitar el inicio de sesión único:

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    Else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
| Azure SDK  | | Azure SDK  |   | Azure SDK   |
+-----+------+-+-----+------+-  +-------+-----+
      |              |                  |
      |       +------v------+           |
      |       |             |           |
      |       | Microsoft   |           |
      +-------> Broker      |^----------+
              | Application
              |             |
              +-------------+
              |             |
              |   Broker    |
              |   Storage   |
              |             |
              +-------------+

```
              
Una vez que disponga de toda esta información, probablemente pueda entender e implementar mejor el inicio de sesión único en la aplicación mediante los SDK y la plataforma Microsoft Identity.


## Habilitación del SSO entre aplicaciones mediante ADAL

Ahora vamos a usar el SDK de Android de ADAL para:

- Activar el SSO no asistido por agente para su conjunto de aplicaciones
- Activar la compatibilidad para el SSO asistido por agente


### Activación del SSO para el SSO no asistido por agente

En el caso del SSO entre aplicaciones no asistido por agente, los SDK de Microsoft Identity administran automáticamente gran parte de la complejidad que entraña este proceso. Esto incluye la búsqueda del usuario adecuado en la memoria caché y el mantenimiento de una lista de usuarios que han iniciado sesión en la que se pueden realizar consultas.

Para habilitar el SSO entre aplicaciones de las que es el propietario debe hacer lo siguiente:

1. Garantizar que todas las aplicaciones utilizan el mismo identificador de cliente o aplicación
* Garantizar que todas las aplicaciones tienen el mismo conjunto de SharedUserID.
* Garantizar que todas las aplicaciones comparten el mismo certificado de firma de Google Play Store para que puedan compartir el almacenamiento.

#### Paso 1: Uso del mismo identificador de cliente o aplicación para todas las aplicaciones de su conjunto

A fin de que la plataforma Microsoft Identity sepa que puede compartir tokens entre las aplicaciones, es necesario que todas ellas compartan el mismo identificador de cliente o aplicación. Se trata del identificador único que se le suministró al registrar su primera aplicación en el portal.

Quizás se pregunte cómo se puede identificar a las diferentes aplicaciones en el servicio Microsoft Identity si todas utilizan el mismo identificador. Pues bien, esto es posible gracias a los **URI de redirección**. Cada aplicación puede tener varios URI de redirección registrados en el portal de incorporación. Cada una de las aplicaciones de su conjunto tendrá diferentes URI de redirección. A continuación se muestra un ejemplo típico de su aspecto:

URI de redirección de la aplicación 1: `msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D`

URI de redirección de la aplicación 2: `msauth://com.example.userapp1/KmB7PxIytyLkbGHuI%2UitkW%2Fejk%4E`

URI de redirección de la aplicación 3: `msauth://com.example.userapp2/Pt85PxIyvbLkbKUtBI%2SitkW%2Fejk%9F`

....

Estos están anidados en el mismo identificador de cliente o aplicación y se consultan en función del URI de redirección que nos devuelve en su configuración del SDK.

```
+-------------------+
|                   |
|  Client ID        |
+---------+---------+
          |
          |           +-----------------------------------+
          |           |  App 1 Redirect URI               |
          +----------^+                                   |
          |           +-----------------------------------+
          |
          |           +-----------------------------------+
          +----------^+  App 2 Redirect URI               |
          |           |                                   |
          |           +-----------------------------------+
          |
          +----------^+-----------------------------------+
                      |  App 3 Redirect URI               |
                      |                                   |
                      +-----------------------------------+

```


*Tenga en cuenta que el formato de estos URI de redirección se explica a continuación. Puede usar cualquier URI de redirección, a menos que desee utilizar el agente, en cuyo caso debe tener un aspecto como el anterior*


#### Paso 2: Configuración del almacenamiento compartido en Android

La definición de `SharedUserID` está fuera del ámbito de este documento, pero se puede aprender en la documentación de Google Android, en el [Manifiesto](http://developer.android.com/guide/topics/manifest/manifest-element.html). Lo importante es que decida cómo desea llamar a su sharedUserID y usar ese nombre en todas sus aplicaciones.

Una vez que tiene `SharedUserID` en todas las aplicaciones, estará listo para utilizar SSO.

> [AZURE.WARNING] 
Cuando se comparte un almacenamiento entre aplicaciones, cualquiera de ellas puede eliminar usuarios o, peor aún, eliminar todos los tokens de la aplicación. Esto puede resultar especialmente problemático si tiene aplicaciones que dependen de los tokens para hacer el trabajo en segundo plano. Compartir un almacenamiento implica ser muy cuidadoso en todas y cada una de las operaciones de eliminación realizadas a través de los SDK de Microsoft Identity.

Eso es todo. El SDK de Microsoft Identity ahora compartirá las credenciales entre todas sus aplicaciones. La lista de usuarios también se compartirá entre las instancias de la aplicación.

### Activación del SSO para el SSO asistido por agente

La capacidad de una aplicación de usar cualquier agente que esté instalado en el dispositivo está **desactivada de forma predeterminada**. Para poder utilizar la aplicación con el agente debe realizar algunos pasos de configuración adicionales y agregar código a la aplicación.

Los pasos que debe seguir son los siguientes:

1. Habilitar el modo de agente en la llamada al código de la aplicación para el SDK de Microsoft
2. Establecer un nuevo URI de redirección e indicarlo tanto en la aplicación como en el registro de la aplicación
3. Configurar los permisos correctos en el manifiesto de Android


#### Paso 1: Habilitar el modo de agente en la aplicación
La capacidad de la aplicación de utilizar el agente se activa al crear la configuración inicial de la instancia de autenticación. Para ello, configure el tipo de ApplicationSettings en el código:

```
AuthenticationSettings.Instance.setUseBroker(true);
```


#### Paso 2: Establecer un nuevo URI de redirección con el esquema de dirección URL

Para garantizar que siempre se devuelvan los tokens de credencial a la aplicación correcta, es necesario asegurarse de que se llama a la aplicación de tal manera que el sistema operativo Android pueda verificarla. El sistema operativo Android utiliza el hash del certificado en Google Play Store. Este sistema no se puede suplantar por ninguna aplicación malintencionada. Por lo tanto, se aprovecha este elemento junto con el URI de la aplicación de agente para garantizar que los tokens se devuelven a la aplicación correcta. Es necesario establecer este URI de redirección único en la aplicación y configurarlo como URI de redirección en nuestro portal para desarrolladores.

Para ser correcto, el URI de redirección debe presentar el formato siguiente:

`msauth://packagename/Base64UrlencodedSignature`

ejemplo: *msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D*

Este URI de redirección debe especificarse en el registro de la aplicación mediante el [Portal de Azure clásico](https://manage.windowsazure.com/). Para más información sobre el registro de aplicaciones de Azure AD, consulte [Integración con Azure Active Directory](active-directory-how-to-integrate.md).


#### Paso 3: Configurar los permisos correctos en la aplicación

Nuestra aplicación de agente en Android utiliza la característica de administrador de cuentas del SO Android para administrar las credenciales entre aplicaciones. Para utilizar el agente en Android el manifiesto de la aplicación debe tener permisos para usar cuentas del administrador de cuentas. Esto se explica en detalle en la [documentación de Google para el administrador de cuentas que puede encontrar aquí](http://developer.android.com/reference/android/accounts/AccountManager.html).

En concreto, estos permisos son:

```
GET_ACCOUNTS
USE_CREDENTIALS
MANAGE_ACCOUNTS
```

### Ya ha configurado el SSO.

Ahora, el SDK de Microsoft Identity compartirá automáticamente las credenciales entre las aplicaciones e invocará al agente si está presente en su dispositivo.

<!---HONumber=AcomDC_0817_2016-->