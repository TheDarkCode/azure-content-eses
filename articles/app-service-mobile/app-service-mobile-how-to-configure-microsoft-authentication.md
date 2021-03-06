<properties
	pageTitle="Configuración de la autenticación mediante la cuenta Microsoft para la aplicación de Servicios de aplicaciones"
	description="Obtenga información acerca de cómo configurar la autenticación mediante la cuenta Microsoft para la aplicación de Servicios de aplicaciones."
	authors="mattchenderson"
	services="app-service"
	documentationCenter=""
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="mahender"/>

# Configuración de la aplicación Servicio de aplicaciones para usar el inicio de sesión de la cuenta Microsoft

[AZURE.INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este tema se muestra cómo configurar Servicio de aplicaciones de Azure para usar la cuenta Microsoft como proveedor de autenticación.

## <a name="register-microsoft-account"> </a>Registro de la aplicación con la cuenta de Microsoft

1. Inicie sesión en el [Portal de Azure] y vaya a la aplicación. Copie la **URL**, que posteriormente utilizará para configurar la aplicación con su cuenta Microsoft.

2. Vaya a la página [Mis aplicaciones] del Centro para desarrolladores de la cuenta Microsoft e inicie sesión con su cuenta Microsoft, si procede.

3. Haga clic en **Agregar una aplicación**, escriba el nombre de la aplicación y haga clic en **Crear aplicación**.

4. Tome nota del **Id. de aplicación**, ya que lo necesitará más adelante.

5. En "Plataformas", haga clic en **Agregar plataforma** y seleccione "Web".

6. En "URI de redirección" proporcione el punto de conexión de la aplicación y haga clic en **Guardar**.
 
	>[AZURE.NOTE]El URI de redireccionamiento es la dirección URL de la aplicación anexada a la ruta de acceso _/.auth/login/microsoftaccount/callback_. Por ejemplo: `https://contoso.azurewebsites.net/.auth/login/microsoftaccount/callback`. Asegúrese de que está utilizando el esquema HTTPS.

7. En "Secretos de aplicación", haga clic en **Generar nueva contraseña**. Anote el valor que aparece. Una vez que salga de esta página, la contraseña no se volverá a mostrar.


    > [AZURE.IMPORTANT] La contraseña es una credencial de seguridad importante. No la comparta con nadie ni la distribuya en una aplicación cliente.

## <a name="secrets"> </a>Incorporación de información de la cuenta de Microsoft a la aplicación de Servicio de aplicaciones

1. En el [Portal de Azure], vaya a la aplicación, haga clic en **Configuración** > **Autenticación/autorización**.

2. Si esta característica no está habilitada, **actívela**.

3. Haga clic en **Cuenta Microsoft**. Pegue los valores de identificador de la aplicación y de contraseña que obtuvo previamente y habilite opcionalmente los ámbitos que requiere la aplicación. y, a continuación, haga clic en **Aceptar**.

    ![][1]

	De forma predeterminada, el Servicio de aplicaciones ofrece autenticación pero no restringe el acceso autorizado al contenido del sitio y a las API. Debe autorizar a los usuarios en el código de la aplicación.

4. (Opcional) Para restringir el acceso al sitio solo a los usuarios autenticados mediante la cuenta Microsoft, establezca **Acción por realizar cuando no se autentique la solicitud** en **Cuenta Microsoft**. Esto requiere que todas las solicitudes se autentiquen y que todas las solicitudes no autenticadas se redirijan a la cuenta Microsoft para la autenticación.

5. Haga clic en **Guardar**.

De este modo ya estará listo para usar la cuenta Microsoft para realizar la autenticación en la aplicación.

## <a name="related-content"> </a>Contenido relacionado

[AZURE.INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]


<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/app-service-microsoftaccount-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/mobile-app-microsoftaccount-settings.png

<!-- URLs. -->

[Mis aplicaciones]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Portal de Azure]: https://portal.azure.com/

<!---HONumber=AcomDC_0824_2016-->