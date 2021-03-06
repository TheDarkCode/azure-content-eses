<properties
	pageTitle="Tutorial: Integración de Azure Active Directory con The Funding Portal | Microsoft Azure"
	description="Aprenda a configurar el inicio de sesión único entre Azure Active Directory y The Funding Portal."
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
	ms.date="09/02/2016"
	ms.author="jeedes"/>


# Tutorial: integración de Azure Active Directory con The Funding Portal

En este tutorial, obtendrá información sobre cómo integrar The Funding Portal con Azure Active Directory (Azure AD).

La integración de The Funding Portal con Azure AD proporciona las siguientes ventajas:

- Puede controlar en Azure AD quién tiene acceso a The Funding Portal.
- Puede permitir que los usuarios inicien sesión automáticamente en The Funding Portal (inicio de sesión único) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: el Portal de Azure clásico.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](active-directory-appssoaccess-whatis.md).

## Requisitos previos

Para configurar la integración de Azure AD con The Funding Portal, necesita los siguientes elementos:

- Una suscripción de Azure AD
- Una suscripción habilitada para el inicio de sesión único en** The Funding Portal**


> [AZURE.NOTE] Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción.


Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No debe usar el entorno de producción, a menos que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).


## Descripción del escenario
En este tutorial, puede probar el inicio de sesión único de Azure AD en un entorno de prueba. La situación descrita en este tutorial consta de dos bloques de creación principales:

1. Adición de The Funding Portal desde la galería
2. Configuración y comprobación del inicio de sesión único de Azure AD


## Adición de The Funding Portal desde la galería
Para configurar la integración de The Funding Portal en Azure AD, deberá agregar esta solución desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar The Funding Portal desde la galería, realice los pasos siguientes:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**.

	![Active Directory][1]

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para abrir la vista de aplicaciones, haga clic en **Applications**, en el menú superior de la vista de directorios.

	![Aplicaciones][2]

4. Haga clic en **Agregar** en la parte inferior de la página.

	![Aplicaciones][3]

5. En el cuadro de diálogo **¿Qué desea hacer?**, haga clic en **Agregar una aplicación de la galería**.

	![Aplicaciones][4]

6. En el cuadro de búsqueda, escriba **The Funding Portal**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_01.png)

7. En el panel de resultados, seleccione **The Funding Portal** y, luego, haga clic en **Completar** para agregar la aplicación.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_02.png)

##  Configuración y comprobación del inicio de sesión único de Azure AD
En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con The Funding Portal utilizando un usuario de prueba llamado "Britta Simon".

Para que el inicio de sesión único funcione, Azure AD debe saber cuál es el usuario homólogo de The Funding Portal para un usuario de Azure AD. Es decir, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de The Funding Portal. Esta relación de vínculo se establece mediante la asignación del valor del **nombre de usuario** en Azure AD como el valor del **nombre de usuario** en The Funding Portal.

Para configurar y probar el inicio de sesión único de Azure AD con The Funding Portal, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configuring-azure-ad-single-single-sign-on)**: para permitir a los usuarios usar esta característica.
2. **[Creación de un usuario de prueba de Azure AD](#creating-an-azure-ad-test-user)**: para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Creación de un usuario de prueba de The Funding Portal](#creating-a-the-funding-portal-test-user)**: para tener un homólogo de Britta Simon en The Funding Portal que esté vinculado a la representación de ella en Azure AD.
5. **[Asignación del usuario de prueba de Azure AD](#assigning-the-azure-ad-test-user)**: para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Prueba del inicio de sesión único](#testing-single-sign-on)**: para comprobar si funciona la configuración.

### Configuración del inicio de sesión único de Azure AD

El objetivo de esta sección es habilitar el inicio de sesión único de Azure AD en el Portal de Azure clásico y configurar el inicio de sesión único en la aplicación The Funding Portal.

La aplicación The Funding Portal espera que las aserciones SAML contengan un atributo denominado "externalId1". El valor de externalId1 debe ser un elemento studentID reconocido. Configure las notificaciones de externalId1 de esta aplicación. Puede administrar el valor de estos atributos desde la pestaña **"Atributo"** de la aplicación. La siguiente captura de pantalla le muestra un ejemplo de esto.

![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_03.png)

**Para configurar el inicio de sesión único de Azure AD con The Funding Portal, realice los pasos siguientes:**

1. En el Portal de Azure clásico, en la página de integración de la aplicación **The Funding Portal**, en el menú de la parte superior, haga clic en **Atributos**.
     
    ![Configurar inicio de sesión único][5]

2. En el cuadro de diálogo de atributos de token SAML, agregue el atributo externalId1.

	a. Haga clic en **agregar atributo de usuario** para abrir el cuadro de diálogo **Agregar atributo de usuario**.
	
	![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_05.png)

	b. En el cuadro de texto **Nombre de atributo**, escriba el nombre de atributo externalId1.

	c. En la lista **Valor de atributo**, seleccione el atributo de usuario que quiere usar en su implementación. Por ejemplo, si ha almacenado el valor de StudentID en ExtensionAttribute1, seleccione después user.extensionattribute1.

	d. Haga clic en **Completo**. Luego, haga clic en **Aplicar cambios**.

1. En el menú de la parte superior, haga clic en **Inicio rápido**.

	![Configurar inicio de sesión único][6]

2. En el portal clásico, en la página de integración de la aplicación **The Funding Portal**, haga clic en **Configurar inicio de sesión único** para abrir el cuadro de diálogo **Configurar inicio de sesión único**.

	![Configurar inicio de sesión único][7]

3. En la página **¿Cómo desea que los usuarios inicien sesión en The Funding Portal?**, seleccione **Inicio de sesión único de Azure AD** y después haga clic en **Siguiente**.
 	
	![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_06.png)

4. En la página de diálogo **Configurar las opciones de la aplicación**, realice los pasos siguientes:

	![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_07.png)


    a. En el cuadro de texto URL de inicio de sesión, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.regenteducation.net/`.

	b. Haga clic en **Next**.

5. En la página **Configuración de inicio de sesión único en The Funding Portal**, haga clic en **Descargar metadatos** y, luego, guarde el archivo en el equipo.

	![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_08.png)

6. Si quiere configurar el inicio de sesión único para la aplicación, póngase en contacto con el equipo de soporte técnico de The Funding Portal. Ellos le ayudarán con el modo adecuado de configurar el inicio de sesión único. Tenga en cuenta que tendrá que enviar un correo electrónico y adjuntar el archivo de metadatos descargado a info@regenteducation.com.

7. En el portal clásico, seleccione la confirmación de la configuración de inicio de sesión único y haga clic en **Siguiente**.
	
	![Inicio de sesión único de Azure AD][10]

8. En la página **Confirmación del inicio de sesión único**, haga clic en **Completar**.
  	
	![Inicio de sesión único de Azure AD][11]

### Creación de un usuario de prueba de Azure AD
En esta sección, creará un usuario de prueba llamado Britta Simon en el portal clásico.

![Creación de un usuario de Azure AD][20]

**Siga estos pasos para crear un usuario de prueba en Azure AD:**

1. En el **Portal de Azure clásico**, en el panel de navegación izquierdo, haga clic en **Active Directory**.
	
	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_09.png)

2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.

3. Para mostrar la lista de usuarios, en el menú de la parte superior, haga clic en **Usuarios**.
	
	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_03.png)

4. Para abrir el diálogo **Agregar usuario**, en la barra de herramientas de la parte inferior, haga clic en **Agregar usuario**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_04.png)

5. En la página de diálogo **Proporcione información sobre este usuario**, realice los pasos siguientes:
 
	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_05.png)

    a. En Tipo de usuario, seleccione Nuevo usuario de la organización.

    b. En el cuadro de texto **Nombre de usuario**, escriba **BrittaSimon**.

    c. Haga clic en **Next**.

6.  En la página de diálogo **Perfil de usuario**, realice los siguientes pasos:

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_06.png)

    a. En el cuadro de texto **Nombre**, escriba **Britta**.

    b. En el cuadro de texto **Apellidos**, escriba **Simon**.

    c. En el cuadro de texto **Nombre para mostrar**, escriba **Britta Simon**.

    d. En la lista **Rol**, seleccione **Usuario**.

    e. Haga clic en **Siguiente**.

7. En la página de diálogo **Obtener contraseña temporal**, haga clic en **Crear**.

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_07.png)

8. En la página de diálogo **Obtener contraseña temporal**, realice los pasos siguientes:

	![Creación de un usuario de prueba de Azure AD](./media/active-directory-saas-thefundingportal-tutorial/create_aaduser_08.png)

    a. Anote el valor del campo **Nueva contraseña**.

    b. Haga clic en **Completo**.



### Creación de un usuario de prueba de The Funding Portal

En esta sección, creará un usuario llamado "Britta Simon" en The Funding Portal. En esta sección, creará un usuario llamado "Britta Simon" en The Funding Portal. Si no sabe cómo agregar Britta Simon a The Funding Portal, colabore con el equipo de soporte técnico de dicha plataforma para agregar el usuario de prueba y habilitar SSO. Póngase en contacto con ellos en info@regenteducation.com.

### Asignación del usuario de prueba de Azure AD

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a The Funding Portal.

![Asignar usuario][200]

**Para asignar el usuario Britta Simon a The Funding Portal, realice los pasos siguientes:**

1. En el portal clásico, para abrir la vista de aplicaciones, en la vista del directorio, haga clic en **Aplicaciones** en el menú superior.

	![Asignar usuario][201]

2. En la lista de aplicaciones, seleccione **The Funding Portal**.

	![Configurar inicio de sesión único](./media/active-directory-saas-thefundingportal-tutorial/tutorial_thefundingportal_09.png)

1. En el menú de la parte superior, haga clic en **Usuarios**.

	![Asignar usuario][203]

1. En la lista Todos los usuarios, seleccione **Britta Simon**.

2. En la barra de herramientas de la parte inferior, haga clic en **Asignar**.

	![Asignar usuario][205]


### Prueba del inicio de sesión único

El objetivo de esta sección es probar la configuración del inicio de sesión único de Azure AD mediante el panel de acceso.

Al hacer clic en el icono de The Funding Portal del panel de acceso, debería iniciar sesión automáticamente en la aplicación The Funding Portal.

## Recursos adicionales

* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](active-directory-saas-tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_04.png


[5]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_05.png
[6]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_06.png
[7]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_050.png
[10]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_060.png
[11]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_070.png
[20]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-thefundingportal-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0907_2016-->