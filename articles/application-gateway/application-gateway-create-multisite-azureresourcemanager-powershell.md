<properties
   pageTitle="Creación de una puerta de enlace de aplicaciones para hospedar varios sitios | Microsoft Azure"
   description="Esta página proporciona instrucciones para crear y configurar una puerta de enlace de aplicaciones de Azure para hospedar varias aplicaciones web en la misma puerta de enlace."
   documentationCenter="na"
   services="application-gateway"
   authors="amsriva"
   manager="rossort"
   editor="amsriva"/>
<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="07/07/2016"
   ms.author="amsriva"/>


# Creación de una puerta de enlace de aplicaciones para hospedar varias aplicaciones web

El hospedaje de varios sitios permite implementar más de una aplicación web en la misma puerta de enlace de aplicaciones. Se basa en la presencia de un encabezado host en la solicitud HTTP entrante para determinar qué agente de escucha recibe el tráfico. Luego, el agente de escucha dirige el tráfico al grupo de back-end adecuado, tal como está configurado en la definición de las reglas de la puerta de enlace. En el caso de aplicaciones web con SSL habilitado, la puerta de enlace de aplicaciones depende de la extensión de la indicación de nombre de servidor (SNI) para elegir el agente de escucha correcto para el tráfico web.

Un uso común del hospedaje de varios sitios es el equilibrio de carga de las solicitudes de diferentes dominios web entre diferentes grupos de servidores de back-end. De forma similar, varios subdominios del mismo dominio raíz también se pueden hospedar en la misma puerta de enlace de aplicaciones.


## Escenario
En el siguiente ejemplo, la puerta de enlace de aplicaciones sirve el tráfico a contoso.com y a fabrikam.com con dos grupos de servidores back-end: el grupo de servidores de contoso y el grupo de servidores de fabrikam. Se puede usar una configuración similar para alojar subdominios como app.contoso.com y blog.contoso.com.


## Antes de empezar

1. Instale la versión más reciente de los cmdlets de Azure PowerShell mediante el Instalador de plataforma web. Puede descargar e instalar la versión más reciente desde la sección **Windows PowerShell** de la [página Descargas](https://azure.microsoft.com/downloads/).
2. Creará una red virtual y una subred para la Puerta de enlace de aplicaciones. Asegúrese de que ninguna máquina virtual o implementación en la nube usan la subred. La Puerta de enlace de aplicaciones debe encontrarse en una subred de red virtual.
3. Los servidores agregados al grupo de back-end para usar la puerta de enlace de aplicaciones deben existir, o bien sus puntos de conexión deben haberse creado en la red virtual o tener una dirección IP/VIP pública asignada.



## ¿Qué se necesita para crear una Puerta de enlace de aplicaciones?


- **Grupo de servidores back-end:** lista de direcciones IP de los servidores back-end. Las direcciones IP que se enumeran deben pertenecer a la subred de la red virtual o ser una IP/VIP pública. También puede utilizarse el FQDN.
- **Configuración del grupo de servidores back-end:** cada grupo tiene una configuración en la que se incluye el puerto, el protocolo y la afinidad basada en cookies. Estos valores están vinculados a un grupo y se aplican a todos los servidores del grupo.
- **Puerto front-end:** este puerto es el puerto público que se abre en la puerta de enlace de aplicaciones. El tráfico llega a este puerto y después se redirige a uno de los servidores back-end.
- **Agente de escucha:** tiene un puerto front-end, un protocolo (Http o Https, que distinguen mayúsculas de minúsculas) y el nombre del certificado SSL (si se configura la descarga de SSL). En el caso de las puertas de enlace de aplicaciones con sitios múltiples habilitados, también se agregan el nombre de host y los indicadores de SNI.
- **Regla**: enlaza el agente de escucha y el grupo de servidores back-end, y define a qué grupo de servidores back-end se debe redireccionar el tráfico que llegue a un agente de escucha concreto.

## Creación de una nueva puerta de enlace de aplicaciones

Estos son los pasos necesarios para crear una puerta de enlace de aplicaciones:

1. Cree un grupo de recursos para el Administrador de recursos.
2. Cree una red virtual, una subred y una IP pública para la puerta de enlace de aplicaciones.
3. Cree un objeto de configuración de la Puerta de enlace de aplicaciones.
4. Cree un recurso de Puerta de enlace de aplicaciones.

## Creación de un grupo de recursos para el Administrador de recursos

Asegúrese de que está usando la versión más reciente de Azure PowerShell. Hay más información disponible en [Uso de Windows PowerShell con Resource Manager](../powershell-azure-resource-manager.md).

### Paso 1
Inicio de sesión en Login-AzureRmAccount de Azure

Se le solicitará que se autentique con sus credenciales.<BR>

### Paso 2
Compruebe las suscripciones para la cuenta.

		Get-AzureRmSubscription

### Paso 3
Elija qué suscripción de Azure va a utilizar.<BR>


		Select-AzureRmSubscription -SubscriptionName "Name of subscription"

### Paso 4
Cree un grupo de recursos nuevo (omita este paso si usa uno existente).

    New-AzureRmResourceGroup -Name appgw-RG -location "East Asia"
También puede crear etiquetas para un grupo de recursos de la puerta de enlace de aplicaciones:
	
	$resourceGroup = New-AzureRmResourceGroup -Name appgw-RG -Location "East Asia" -Tags @{Name = "testtag"; Value = "Application Gateway multiple site"} 

El Administrador de recursos de Azure requiere que todos los grupos de recursos especifiquen una ubicación. Esta se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Asegúrese de que todos los comandos para crear una puerta de enlace de aplicaciones se van a usar en el mismo grupo de recursos.

En el ejemplo anterior, creamos un grupo de recursos denominado "appgw-RG" y la ubicación "Asia Oriental".

>[AZURE.NOTE] Si necesita configurar un sondeo personalizado para la puerta de enlace de aplicaciones, consulte [Creación de una puerta de enlace de aplicaciones con sondeos personalizados mediante PowerShell](application-gateway-create-probe-ps.md). Para más información, consulte [Información general sobre la supervisión de estado de la puerta de enlace de aplicaciones](application-gateway-probe-overview.md).

## Creación de una red virtual y una subred para la puerta de enlace de aplicaciones

En el ejemplo siguiente se muestra cómo crear una red virtual con el Administrador de recursos.

### Paso 1
Asigne el intervalo de direcciones 10.0.0.0/24 a la variable de subred que se utilizará para crear una red virtual.

	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24


### Paso 2
Cree una red virtual denominada "appgwvnet" en el grupo de recursos "appgw-rg" para la región Oeste de EE. UU. con el prefijo 10.0.0.0/16 y la subred 10.0.0.0/24.

	$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-RG -Location "East Asia" -AddressPrefix 10.0.0.0/16 -Subnet $subnet


### Paso 3
Asignación de una variable de subred en los pasos siguientes para crear una puerta de enlace de aplicaciones

	$subnet=Get-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -VirtualNetwork $vnet

## Creación de una dirección IP pública para la configuración del front-end
Cree un recurso IP público "publicIP01" en el grupo de recursos "appgw-rg" para la región Oeste de EE. UU..

	$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -name publicIP01 -location "East Asia" -AllocationMethod Dynamic

Se asignará una dirección IP a la puerta de enlace de aplicaciones cuando se inicia el servicio.

## Creación de una configuración de puerta de enlace de aplicaciones

Debe configurar todos los elementos de configuración antes de crear la Puerta de enlace de aplicaciones de la aplicación. En los pasos siguientes, se crean los elementos de configuración necesarios para un recurso de puerta de enlace de aplicaciones.

### Paso 1
Cree una configuración de IP de puerta de enlace de aplicaciones denominada "gatewayIP01". Cuando se inicie Puerta de enlace de aplicaciones, elegirá una dirección IP de la subred configurada y redirige el tráfico de red a las direcciones IP en el grupo IP de back-end. Tenga en cuenta que cada instancia tomará una dirección IP.


	$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet


### Paso 2
Configuración del grupo de direcciones IP de back-end llamado "pool01"y "pool2" con las direcciones IP "10.0.0.100, 10.0.0.101,10.0.0.102" para "pool1" y "10.0.0.103, 10.0.0.104, 10.0.0.105" para "pool2".


	$pool1 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 10.0.0.100, 10.0.0.101,10.0.0.102 
	$pool2 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool02 -BackendIPAddresses 10.0.0.103, 10.0.0.104, 10.0.0.105

En este ejemplo habrá dos grupos de back-end para enrutar el tráfico de red en función del sitio solicitado. Un grupo recibe el tráfico del sitio "contoso.com" y otro lo recibe del sitio "fabrikam.com". Tiene que reemplazar las direcciones IP anteriores para agregar sus propios puntos de conexión de dirección IP de aplicación. Tenga en cuenta que en lugar de las direcciones IP internas, también se pueden usar direcciones IP públicas, el FQDN o la NIC de una máquina virtual para las instancias de back-end. Use el parámetro "-BackendFQDNs" en PowerShell para especificar FQDN, en lugar de direcciones IP.

### Paso 3
Configuración de la opción de la puerta de enlace de aplicaciones "poolsetting01" y "poolsetting02" para el tráfico de red con carga equilibrada en el grupo de back-end En este ejemplo se configura otro grupo de back-end para los grupos de back-end. Cada grupo de back-end puede tener su propia configuración de grupo de back-end.


	$poolSetting01 = New-AzureRmApplicationGatewayBackendHttpSettings -Name "besetting01" -Port 80 -Protocol Http -CookieBasedAffinity Disabled -RequestTimeout 120
	$poolSetting02 = New-AzureRmApplicationGatewayBackendHttpSettings -Name "besetting02" -Port 80 -Protocol Http -CookieBasedAffinity Enabled -RequestTimeout 240

### Paso 4
Configuración de la dirección IP de front-end con el punto de conexión de IP pública

	$fipconfig01 = New-AzureRmApplicationGatewayFrontendIPConfig -Name "frontend1" -PublicIPAddress $publicip

### Paso 5 
Configuración del puerto front-end de una puerta de enlace de aplicaciones

	$fp01 = New-AzureRmApplicationGatewayFrontendPort -Name "fep01" -Port 443

### Paso 6
Configuración de dos certificados SSL para los dos sitios web que vamos a admitir en este ejemplo. Uno de ellos es para el tráfico de contoso.com y el otro es para el de fabrikam.com. Debería haber una entidad de certificación que generara certificados para sus sitios web. Los certificados autofirmados se admiten, pero no se recomiendan para el tráfico de producción.

	$cert01 = New-AzureRmApplicationGatewaySslCertificate -Name  contosocert -CertificateFile <file path> -Password <password>
	$cert02 = New-AzureRmApplicationGatewaySslCertificate -Name fabrikamcert -CertificateFile <file path> -Password <password>


### Paso 7
Configuración de dos agentes de escucha para los dos sitios web del ejemplo. En este paso se configuran los agentes de escucha para la dirección IP pública, el puerto y el host usados para recibir el tráfico entrante. El parámetro HostName es necesario para lograr compatibilidad con varios sitios y se debe establecer en el sitio web adecuado para el que se recibirá el tráfico. El parámetro RequireServerNameIndication debe establecerse en true para los sitios web que necesitan compatibilidad con SSL en un escenario de varios hosts. Si se requiere compatibilidad con SSL, también será preciso especificar el certificado SSL que se utilizará para proteger el tráfico de la aplicación web. La combinación de FrontendIPConfiguration, FrontendPort y HostName debe ser única en cada agente de escucha. Cada agente de escucha puede admitir solo un certificado.
 
	$listener01 = New-AzureRmApplicationGatewayHttpListener -Name "listener01" -Protocol Https -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -HostName "contoso11.com" -RequireServerNameIndication true  -SslCertificate $cert01
	$listener02 = New-AzureRmApplicationGatewayHttpListener -Name "listener02" -Protocol Https -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -HostName "fabrikam11.com" -RequireServerNameIndication true -SslCertificate $cert02


### Paso 8 
Creación de la configuración de dos reglas para las dos aplicaciones web de este ejemplo. Una regla une los agentes de escucha, los grupos de back-end y la configuración de http. En este paso se configura la puerta de enlace de aplicaciones para que use una regla de enrutamiento básica, una para cada sitio web. El tráfico para cada sitio web lo recibe el agente de escucha que tenga configurado y, luego, se dirige a su grupo de back-end configurado, para lo que se usan las propiedades especificadas en BackendHttpSettings.

	$rule01 = New-AzureRmApplicationGatewayRequestRoutingRule -Name "rule01" -RuleType Basic -HttpListener $listener01 -BackendHttpSettings $poolSetting01 -BackendAddressPool $pool1
	$rule02 = New-AzureRmApplicationGatewayRequestRoutingRule -Name "rule02" -RuleType Basic -HttpListener $listener02 -BackendHttpSettings $poolSetting02 -BackendAddressPool $pool2

### Paso 9: 
Configuración del número de instancias y el tamaño de la puerta de enlace de aplicaciones

	$sku = New-AzureRmApplicationGatewaySku -Name "Standard_Medium" -Tier Standard -Capacity 2

## Creación de una puerta de enlace de aplicaciones
Cree una puerta de enlace de aplicaciones con todos los objetos de configuración de los pasos anteriores.

	$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-RG -Location "East Asia" -BackendAddressPools $pool1,$pool2 -BackendHttpSettingsCollection $poolSetting01, $poolSetting02 -FrontendIpConfigurations $fipconfig01 -GatewayIpConfigurations $gipconfig -FrontendPorts $fp01 -HttpListeners $listener01, $listener02 -RequestRoutingRules $rule01, $rule02 -Sku $sku -SslCertificates $cert01, $cert02 	


>[AZURE.IMPORTANT] El aprovisionamiento de Puerta de enlace de aplicaciones es una operación larga y puede tardar algún tiempo en completarse.

## Obtención del nombre DNS de una puerta de enlace de aplicaciones
Recupere los detalles de la puerta de enlace de aplicaciones y su nombre DNS o IP asociados, para lo que debe usar el elemento PublicIPAddress asociado a la puerta de enlace de aplicaciones. El nombre DNS de la puerta de enlace de aplicaciones se debe utilizar para crear un registro CNAME que apunte las dos aplicaciones web a este nombre DNS. No se recomienda el uso de registros A, ya que la VIP puede cambiar al reiniciarse la puerta de enlace de aplicaciones.
	
	Get-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -name publicIP01
		
	Name                     : publicIP01
	ResourceGroupName        : appgw-RG
	Location                 : eastasia
	Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
	Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
	ResourceGuid             : 00000000-0000-0000-0000-000000000000
	ProvisioningState        : Succeeded
	Tags                     : 
	PublicIpAllocationMethod : Dynamic
	IpAddress                : xx.xx.xxx.xx
	PublicIpAddressVersion   : IPv4
	IdleTimeoutInMinutes     : 4
	IpConfiguration          : {
	                             "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
	                           Configurations/frontend1"
	                           }
	DnsSettings              : {
	                             "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
	                           }

<!---HONumber=AcomDC_0810_2016-->