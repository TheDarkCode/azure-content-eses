<properties
	pageTitle="Tutorial: Integración de Azure Active Directory con ImageRelay | Microsoft Azure"
	description="Aprenda a configurar el inicio de sesión único entre Azure Active Directory e ImageRelay."
	services="active-directory"
	documentationCenter=""
	authors="jeevansd"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/15/2016"
	ms.author="jeedes"/>


# Tutorial: Integración de Azure Active Directory con ImageRelay

El objetivo de este tutorial es mostrar cómo integrar ImageRelay con Azure Active Directory (Azure AD). La integración de ImageRelay con Azure AD proporciona las siguientes ventajas:

- Puede controlar en Azure AD quién tiene acceso a ImageRelay.
- Puede permitir que los usuarios inicien sesión automáticamente en ImageRelay (inicio de sesión único) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: el Portal de Azure clásico.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](active-directory-appssoaccess-whatis.md).

## Requisitos previos

Para configurar la integración de Azure AD con ImageRelay, se necesitan los siguientes elementos:

- Una suscripción de Azure AD
- Una suscripción habilitada para inicio de sesión único en ImageRelay


> [AZURE.NOTE] Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción.


Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No debe usar el entorno de producción, a menos que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).


## Descripción del escenario
El objetivo de este tutorial es permitirle probar el inicio de sesión único de Azure AD en un entorno de prueba. La situación descrita en este tutorial consta de dos bloques de creación principales:

1. Incorporación de ImageRelay desde la galería

2. Configuración y comprobación del inicio de sesión único de Azure AD


## Incorporación de ImageRelay desde la galería
Para configurar la integración de ImageRelay en Azure AD, es preciso agregar ImageRelay desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ImageRelay desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo del Portal de Azure clásico, haga clic en **Active Directory**.

	![Active Directory][1]

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para abrir la vista de aplicaciones, haga clic en **Applications**, en el menú superior de la vista de directorios.

	![Aplicaciones][2]

4. Haga clic en **Agregar** en la parte inferior de la página.

	![Aplicaciones][3]

5. En el cuadro de diálogo **¿Qué desea hacer?**, haga clic en **Agregar una aplicación de la galería**.

	![Aplicaciones][4]

6. En el cuadro de búsqueda, escriba **ImageRelay**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_01.png)

7. En el panel de resultados, seleccione **ImageRelay** y después haga clic en **Completar** para agregar la aplicación.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_02.png)

##  Configuración y comprobación del inicio de sesión único de Azure AD
El objetivo de esta sección es mostrar cómo configurar y probar el inicio de sesión único de Azure AD con ImageRelay con un usuario de prueba llamado "Britta Simon".

Para que el inicio de sesión único funcione, Azure AD necesita una cuenta de usuario que represente al usuario relacionado en ImageRelay. Es decir, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de ImageRelay. Esta relación de vínculo se establece mediante la asignación del valor del **nombre de usuario** en Azure AD como el valor del **nombre de usuario** en ImageRelay.

Para configurar y probar el inicio de sesión único de Azure AD con ImageRelay, es necesario completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-single-sign-on)**: para permitir a los usuarios usar esta característica.
2. **[Creación de un usuario de prueba de Azure AD](#creating-an-azure-ad-test-user)**: para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Creación de un usuario de prueba de ImageRelay](#creating-a-userecho-test-user)**: para tener un homólogo de Britta Simon en ImageRelay que esté vinculado a la representación de ella en Azure AD.
5. **[Asignación del usuario de prueba de Azure AD](#assigning-the-azure-ad-test-user)**: para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Prueba del inicio de sesión único](#testing-single-sign-on)**: para comprobar si funciona la configuración.

### Configuración del inicio de sesión único de Azure AD

El objetivo de esta sección es habilitar el inicio de sesión único de Azure AD en el Portal de Azure clásico y configurar el inicio de sesión único en la aplicación ImageRelay.


**Para configurar el inicio de sesión único de Azure AD con ImageRelay, realice los pasos siguientes:**

1. En el Portal de Azure clásico, en la página de integración de la aplicación **ImageRelay**, haga clic en **Configurar inicio de sesión único** para abrir el cuadro de diálogo **Configurar inicio de sesión único**.

     ![Configurar inicio de sesión único][6]

2. En la página **¿Cómo desea que los usuarios inicien sesión en ImageRelay?**, seleccione **Inicio de sesión único de Azure AD** y después haga clic en **Siguiente**.

    ![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_03.png)

3. En la página de diálogo **Configurar las opciones de la aplicación**, realice los pasos siguientes:

     ![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_04.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL que utilizan los usuarios para iniciar sesión en la aplicación ImageRelay (por ejemplo, *https://fabrikam.ImageRelay.com/*).

    b. Haga clic en **Next**.

4. En la página **Configurar inicio de sesión único en ImageRelay**, siga estos pasos:

    ![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_05.png)

    a. Haga clic en **Descargar certificado** y después guarde el archivo en el equipo.

    b. Haga clic en **Next**.

5. En otra ventana del explorador, inicie sesión en su sitio de la compañía de ImageRelay como administrador.

1. En la barra de herramientas de la parte superior, haga clic en la carga de trabajo **Users & Permissions** (Usuarios y permisos).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_06.png)

1. Haga clic en **Create New Permission** (Crear permiso nuevo).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_08.png)

1. En la carga de trabajo **Single Sign On Settings** (Configuración de inicio de sesión único), active la casilla **This Group can only sign-in via Single Sign On** (Este grupo solo puede iniciar sesión mediante inicio de sesión único) y haga clic en **Save** (Guardar).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_09.png)

1. Vaya a **Account Settings** (Configuración de cuenta).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_10.png)

1. Vaya a la carga de trabajo **Single Sign On Settings** (Configuración de inicio de sesión único).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_11.png)

1. En el cuadro de diálogo **SAML Settings** (Configuración de SAML), siga estos pasos:

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_12.png)

	a. En el Portal de Azure clásico, copie el valor de **Dirección URL del servicio de inicio de sesión único** y péguelo en el cuadro de texto **Login URL** (Dirección URL de inicio de sesión).


	b. En el Portal de Azure clásico, copie el valor de **Dirección URL del servicio de cierre de sesión único** y péguelo en el cuadro de texto **Logout URL** (Dirección URL de cierre de sesión).

	c. En **Name Id Format** (Formato de id. de nombre), seleccione **urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress**.

	
    d. En **Binding Options for Requests from the Service Provider (Image Relay)** [Opciones de enlace para solicitudes del proveedor de servicio (ImageRelay)], seleccione **POST Binding** (Enlace POST).
   

	e. En **x.509 Certificate** (Certificado x.509), haga clic en **Update Certificate** (Actualizar certificado).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_17.png)

    f. Abra el certificado descargado en el Bloc de notas, copie el contenido y péguelo en el cuadro de texto x.509 Certificate (Certificado X.509).
  
	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_18.png)

    g. En **Just-In-Time User Provisioning** (Aprovisionamiento de usuarios Just-In-Time), active la casilla **Enable Just-In-Time User Provisioning** (Habilitar aprovisionamiento de usuarios Just-In-Time).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_19.png)

    h. Seleccione el grupo de permisos, por ejemplo, **SSO Basic** (SSO básico), que puede iniciar sesión solo mediante inicio de sesión único.

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_20.png)

    i. Haga clic en **Save**.

6. En el Portal de Azure clásico, seleccione la confirmación de la configuración de inicio de sesión único y haga clic en **Siguiente**.

    ![Inicio de sesión único de Azure AD][10]

7. En la página **Confirmación del inicio de sesión único**, haga clic en **Completar**.

    ![Inicio de sesión único de Azure AD][11]


### Creación de un usuario de prueba de Azure AD
El objetivo de esta sección es crear un usuario de prueba en el Portal de Azure clásico llamado Britta Simon.

![Creación de un usuario de Azure AD][20]

**Siga estos pasos para crear un usuario de prueba en Azure AD:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_09.png)

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para mostrar la lista de usuarios, en el menú de la parte superior, haga clic en **Usuarios**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_03.png)

4. Para abrir el diálogo **Agregar usuario**, en la barra de herramientas de la parte inferior, haga clic en **Agregar usuario**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_04.png)

5. En la página de diálogo **Proporcione información sobre este usuario**, realice los pasos siguientes:

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_05.png)

    a. En Tipo de usuario, seleccione Nuevo usuario de la organización.

    b. En el cuadro de texto **Nombre de usuario**, escriba **BrittaSimon**.

    c. Haga clic en **Next**.

6.  En la página de diálogo **Perfil de usuario**, realice los siguientes pasos:

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_06.png)

    a. En el cuadro de texto **Nombre**, escriba **Britta**.

    b. En el cuadro de texto **Apellidos**, escriba **Simon**.

    c. En el cuadro de texto **Nombre para mostrar**, escriba **Britta Simon**.

    d. En la lista **Rol**, seleccione **Usuario**.

    e. Haga clic en **Siguiente**.

7. En la página de diálogo **Obtener contraseña temporal**, haga clic en **Crear**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_07.png)

8. En la página de diálogo **Obtener contraseña temporal**, realice los pasos siguientes:

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-imagerelay-tutorial/create_aaduser_08.png)

    a. Anote el valor del campo **Nueva contraseña**.

    b. Haga clic en **Complete**.



### Creación de un usuario de prueba de ImageRelay

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en ImageRelay.

**Para crear un usuario llamado Britta Simon en ImageRelay, realice los pasos siguientes:**

1. Inicie sesión en su sitio de la compañía de ImageRelay como administrador.

1. Vaya a **Users & Permissions** (Usuarios y permisos) y seleccione **Create SSO User** (Crear usuario de SSO).

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_21.png)

1. Escriba los valores de los campos **Email** (Correo electrónico), **First Name** (Nombre), **Last Name** (Apellidos) y **Company** (Compañía) del usuario que desea aprovisionar y seleccione el grupo de permisos [por ejemplo, SSO Basic (SSO básico)], que es el grupo que solo puede iniciar sesión mediante inicio de sesión único.

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_22.png)

1. Haga clic en **Crear**.

### Asignación del usuario de prueba de Azure AD

El objetivo de esta sección es permitir que Britta Simon use el inicio de sesión único de Azure, para lo que se le concederá acceso a ImageRelay.

![Asignar usuario][200]

**Para asignar Britta Simon a ImageRelay, realice los pasos siguientes:**

1. En el Portal de Azure clásico, para abrir la vista de aplicaciones, en la vista del directorio, haga clic en **Aplicaciones** en el menú superior.

	![Asignar usuario][201]

2. En la lista de aplicaciones, seleccione **ImageRelay**.

	![Configurar inicio de sesión único](./media/active-directory-saas-imagerelay-tutorial/tutorial_imagerelay_23.png)

1. En el menú de la parte superior, haga clic en **Usuarios**.

	![Asignar usuario][203]

1. En la lista Usuarios, seleccione **Britta Simon**.

2. En la barra de herramientas de la parte inferior, haga clic en **Asignar**.

	![Asignar usuario][205]


### Prueba del inicio de sesión único

El objetivo de esta sección es probar la configuración del inicio de sesión único de Azure AD mediante el panel de acceso. Al hacer clic en el icono de ImageRelay en el panel de acceso, debería iniciar sesión automáticamente en su aplicación ImageRelay.


## Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](active-directory-saas-tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)


<!--Image references-->

[1]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-imagerelay-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0817_2016-->