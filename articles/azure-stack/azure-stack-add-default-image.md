---
title: "Adición de la imagen de máquina virtual predeterminada a Marketplace de Azure Stack | Microsoft Docs"
description: "Agregue la imagen predeterminada de máquina virtual de Windows Server 2016 a Marketplace de Azure Stack."
services: azure-stack
documentationcenter: 
author: mattbriggs
manager: femila
editor: 
ms.assetid: 2849E53F-3D58-48A5-8007-3238FC39F630
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 1/23/2018
ms.author: mabrigg
ms.openlocfilehash: be4a61f185435238db68e4dc43c323a30a754f03
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="add-the-windows-server-2016-vm-image-to-the-azure-stack-marketplace"></a>Adición de la imagen de máquina virtual de Windows Server 2016 a Marketplace de Azure Stack

De forma predeterminada, no hay ninguna imagen de máquina virtual disponible en Marketplace de Azure Stack. Un operador de Azure Stack debe agregar una imagen a Marketplace para que los usuarios tengan acceso. Puede agregar la imagen de Windows Server 2016 a Marketplace de Azure Stack mediante uno de los métodos siguientes:

* [Descargar la imagen de Azure Marketplace](#add-the-image-by-downloading-it-from-the-azure-marketplace). Use esta opción si trabaja en un escenario conectado y ha registrado su instancia de Azure Stack en Azure.

* [Agregar la imagen mediante PowerShell](#add-the-image-by-using-powershell). Use esta opción si ha implementado Azure Stack en un escenario sin conexión o en escenarios con conectividad limitada.

## <a name="add-the-image-by-downloading-it-from-the-azure-marketplace"></a>Agregación de la imagen mediante la descarga de Azure Marketplace

1. Implemente Azure Stack y luego inicie sesión en su instancia del Kit de desarrollo de Azure Stack.

2. Seleccione **Más servicios** > **Marketplace Management** (Administración de Marketplace)  > **Add from Azure** (Agregar desde Azure). 

3. Busque la imagen **Windows Server 2016 Datacenter** y luego seleccione **Descargar**.

   ![Descarga de la imagen desde Azure](media/azure-stack-add-default-image/download-image.png)

Cuando finalice la descarga, la imagen estará disponible en **Marketplace Management** (Administración de Marketplace). La imagen también está disponible bajo **Proceso** y se utiliza para crear nuevas máquinas virtuales.

## <a name="add-the-image-by-using-powershell"></a>Agregación de la imagen mediante el uso de PowerShell

### <a name="prerequisites"></a>requisitos previos 

Implemente los siguientes requisitos previos desde el [kit de desarrollo](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop) o desde un cliente externo basado en Windows, si está [conectado a través de VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn):

1. Instale los [módulos de Azure PowerShell compatibles con Azure Stack](azure-stack-powershell-install.md).  

2. Descargue las [herramientas necesarias para trabajar con Azure Stack](azure-stack-powershell-download.md).  

3. En la página de evaluaciones de Windows Server, [descargue la evaluación de Windows Server 2016](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016). Cuando se le solicite, seleccione la versión ISO de la descarga. Registre la ruta de acceso a la ubicación de descarga, que se usará posteriormente en los pasos descritos en este artículo. Este paso requiere conectividad a Internet.  

### <a name="add-the-image-to-the-azure-stack-marketplace"></a>Adición de la imagen a Marketplace de Azure Stack
   
1. Importe los módulos Connect y ComputeAdmin de Azure Stack mediante los comandos siguientes:

   ```powershell
   Set-ExecutionPolicy RemoteSigned

   # Import the Connect and ComputeAdmin modules.   
   Import-Module .\Connect\AzureStack.Connect.psm1
   Import-Module .\ComputeAdmin\AzureStack.ComputeAdmin.psm1

   ```

2. Inicie sesión en el entorno de Azure Stack. Ejecute uno de los siguientes scripts, según si implementó su entorno de Azure Stack mediante Azure Active Directory (Azure AD) o Servicios de federación de Active Directory (AD FS). (Reemplace el valor de `tenantName` de Azure AD, el punto de conexión `GraphAudience` y el valor de `ArmEndpoint` para que reflejen la configuración de su entorno).  

   * **Azure Active Directory**. Use el siguiente cmdlet:

    ```PowerShell
    # For Azure Stack Development Kit, this value is set to https://adminmanagement.local.azurestack.external. To get this value for Azure Stack integrated systems, contact your service provider.
    $ArmEndpoint = "<Resource Manager endpoint for your environment>"

    # For Azure Stack Development Kit, this value is set to https://graph.windows.net/. To get this value for Azure Stack integrated systems, contact your service provider.
    $GraphAudience = "<GraphAuidence endpoint for your environment>"
    
    # Create the Azure Stack operator's Azure Resource Manager environment by using the following cmdlet:
    Add-AzureRMEnvironment `
      -Name "AzureStackAdmin" `
      -ArmEndpoint $ArmEndpoint

    Set-AzureRmEnvironment `
      -Name "AzureStackAdmin" `
      -GraphAudience $GraphAudience

    $TenantID = Get-AzsDirectoryTenantId `
      -AADTenantName "<myDirectoryTenantName>.onmicrosoft.com" `
      -EnvironmentName AzureStackAdmin

    Login-AzureRmAccount `
      -EnvironmentName "AzureStackAdmin" `
      -TenantId $TenantID 
    ```

   * **Servicios de federación de Active Directory**. Use el siguiente cmdlet:
    
    ```PowerShell
    # For Azure Stack Development Kit, this value is set to https://adminmanagement.local.azurestack.external. To get this value for Azure Stack integrated systems, contact your service provider.
    $ArmEndpoint = "<Resource Manager endpoint for your environment>"

    # For Azure Stack Development Kit, this value is set to https://graph.local.azurestack.external/. To get this value for Azure Stack integrated systems, contact your service provider.
    $GraphAudience = "<GraphAuidence endpoint for your environment>"

    # Create the Azure Stack operator's Azure Resource Manager environment by using the following cmdlet:
    Add-AzureRMEnvironment `
      -Name "AzureStackAdmin" `
      -ArmEndpoint $ArmEndpoint

    Set-AzureRmEnvironment `
      -Name "AzureStackAdmin" `
      -GraphAudience $GraphAudience `
      -EnableAdfsAuthentication:$true

    $TenantID = Get-AzsDirectoryTenantId `
     -ADFS `
     -EnvironmentName "AzureStackAdmin" 

    Login-AzureRmAccount `
      -EnvironmentName "AzureStackAdmin" `
      -TenantId $TenantID 
    ```
   
3. Agregue la imagen de Windows Server 2016 a Marketplace de Azure Stack. (Reemplace *fully_qualified_path_to_ISO* por la ruta de acceso a la instancia de Windows Server 2016 ISO que ha descargado).

    ```PowerShell
    $ISOPath = "<fully_qualified_path_to_ISO>"

    # Add a Windows Server 2016 Evaluation VM image.
    New-AzsServer2016VMImage `
      -ISOPath $ISOPath

    ```

Para asegurarse de que la imagen de máquina virtual Windows Server 2016 tiene la actualización acumulativa más reciente, incluya el parámetro `IncludeLatestCU` al ejecutar el cmdlet `New-AzsServer2016VMImage`. Para más información sobre los parámetros permitidos para el cmdlet `New-AzsServer2016VMImage`, consulte [Parámetros](#parameters). Se tarda aproximadamente una hora en publicar la imagen en Marketplace de Azure Stack. 

## <a name="parameters-for-new-azsserver2016vmimage"></a>Parámetros de New-AzsServer2016VMImage

### <a name="new-azsserver2016vmimage"></a>New-AzsServer2016VMImage 

Crea y carga una nueva instancia de Server 2016 Core, o una imagen completa, y crea un elemento en Marketplace para ella.

| Parámetros | Obligatorio | Ejemplo | DESCRIPCIÓN |
|-----|-----|------|---- |
|ISOPath|Sí| N:\ISO\en_windows_16_x64_dvd | La ruta de acceso completa a la imagen ISO de Windows Server 2016 descargada.|
|Net35|Sin | True | Instale el entorno de tiempo de ejecución de .NET 3.5 en la imagen de Windows Server 2016. De forma predeterminada, este valor se establece en **true**.|
|Versión|Sin | Completo |  Especifica la imagen de Windows Server 2016, que puede ser **Core** (Principal), **Full** (Completa) o **Both** (Ambas). De forma predeterminada, este valor se establece en **Full** (Completa).|
|VHDSizeInMB|Sin | 40 960 | Establece el tamaño (en MB) de la imagen de disco duro virtual que se va a agregar a su entorno de Azure Stack. De forma predeterminada, este valor se establece en 40 960 MB.|
|CreateGalleryItem|Sin | True | Especifica si se debe crear un elemento de Marketplace para la imagen de Windows Server 2016. De forma predeterminada, este valor se establece en **true**.|
|location |Sin  | D:\ | Especifica la ubicación en la que se debe publicar la imagen de Windows Server 2016.|
|IncludeLatestCU|Sin | False | Aplica la actualización acumulativa más reciente de Windows Server 2016 al nuevo disco duro virtual. Compruebe el script para asegurarse de que apunta a la actualización más reciente o utilice una de las dos opciones siguientes. |
|CUUri |Sin  | https://yourupdateserver/winservupdate2016 | Establece la actualización acumulativa de Windows Server 2016 para que se ejecute desde un identificador URI específico. |
|CUPath |Sin  | C:\winservupdate2016 | Establece la actualización acumulativa de Windows Server 2016 para que se ejecute desde una ruta de acceso local. Esta opción es útil si ha implementado la instancia de Azure Stack en un entorno desconectado.|

## <a name="next-steps"></a>pasos siguientes

[Aprovisionamiento de una máquina virtual](azure-stack-provision-vm.md)
