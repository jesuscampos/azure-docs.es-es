---
title: "Inscripción de un dispositivo de TPM al servicio Azure Device Provisioning con Java | Microsoft Docs"
description: "Inicio rápido de Azure: Inscripción de un dispositivo de TPM al servicio IoT Hub Device Provisioning mediante el SDK del servicio de Java"
services: iot-dps
keywords: 
author: dsk-2015
ms.author: dkshir
ms.date: 12/20/2017
ms.topic: hero-article
ms.service: iot-dps
documentationcenter: 
manager: timlt
ms.devlang: na
ms.custom: mvc
ms.openlocfilehash: 79b12c276dae3cf1e480ec8d3f12d2d4b3d8fea7
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="enroll-tpm-device-to-iot-hub-device-provisioning-service-using-java-service-sdk"></a>Inscripción de un dispositivo de TPM al servicio IoT Hub Device Provisioning mediante el SDK del servicio de Java

[!INCLUDE [iot-dps-selector-quick-enroll-device-tpm](../../includes/iot-dps-selector-quick-enroll-device-tpm.md)]


En estos pasos se muestra cómo inscribir un dispositivo de TPM simulado mediante programación en el servicio Azure IoT Hub Device Provisioning con el [SDK del servicio de Java](https://azure.github.io/azure-iot-sdk-java/service/) con la ayuda de una aplicación de Java de ejemplo. Aunque el SDK del servicio de Java funciona en máquinas Windows y Linux, este artículo usa una máquina de desarrollo de Windows para guiarle por el proceso de inscripción.

Asegúrese de [configurar un servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) y de [simular un dispositivo de TPM](quick-create-simulated-device.md#simulatetpm) antes de continuar.

<a id="setupdevbox"></a>

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo 

1. Asegúrese de que tiene [Java SE Development Kit 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) instalado en su máquina. 

2. Establezca las variables de entorno para la instalación de Java. La variable `PATH` debería incluir la ruta de acceso completa al directorio *jdk1.8.x\bin*. Si esta es la primera instalación de Java de su máquina, cree una nueva variable de entorno con el nombre `JAVA_HOME` y haga que señale a la ruta de acceso completa del directorio *jdk1.8.x*. En una máquina con Windows, este directorio se encuentra normalmente en la carpeta *C:\\Archivos de programa\\Java\\*. Puede crear o modificar las variables de entorno si busca **Editar las variables de entorno del sistema** en el **Panel de control** de su máquina Windows. 

  Para comprobar si Java está correctamente configurado en la máquina, ejecute el comando siguiente en la ventana Comandos:

    ```cmd\sh
    java -version
    ```

3. Descargue y extraiga [Maven 3](https://maven.apache.org/download.cgi) en su máquina. 

4. Modifique la variable de entorno `PATH` para que apunte a la carpeta *apache-maven-3.x.x\\bin* dentro de la carpeta donde se extrae Maven. Para confirmar que Maven está instalado correctamente, ejecute este comando en la ventana Comandos:

    ```cmd\sh
    mvn --version
    ```

5. Asegúrese de que [GIT](https://git-scm.com/download/) esté instalado en su máquina y de que se agregue a la variable de entorno `PATH`. 


<a id="javasample"></a>

## <a name="download-and-modify-the-java-sample-code"></a>Descarga y modificación del código de ejemplo de Java

En esta sección se muestra cómo agregar los detalles de aprovisionamiento del dispositivo de TPM al código de ejemplo. 

1. Abra el símbolo del sistema. Clone el repositorio de GitHub para código de ejemplo de inscripción de dispositivos mediante el SDK del servicio de Java:
    
    ```cmd\sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```

2. En el código fuente descargado, navegue hasta la carpeta de ejemplo **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-sample_**. Abra el archivo  **_/src/main/java/samples/com/microsoft/azure/sdk/iot/ServiceEnrollmentSample.java_**  en el editor que prefiera y agregue los detalles siguientes:

    1. Desde el portal, agregue la cadena `[Provisioning Connection String]` para su servicio de aprovisionamiento como se muestra a continuación:
        1. Vaya al servicio de aprovisionamiento en [Azure Portal](https://portal.azure.com). 
        2. Abra las **directivas de acceso compartido** y seleccione una directiva que tenga el permiso *EnrollmentWrite*.
        3. Copie la **cadena de conexión de clave principal**. 

            ![Obtención de la cadena de conexión de aprovisionamiento desde el portal](./media/quick-enroll-device-tpm-java/provisioning-string.png)  

        4. En el archivo de código de ejemplo **_ServiceEnrollmentSample.java_**, reemplace la cadena `[Provisioning Connection String]` con la **cadena de conexión de clave principal**.
    
            ```Java
            private static final String PROVISIONING_CONNECTION_STRING = "[Provisioning Connection String]";
            ```

    2. Agregue los detalles del dispositivo de TPM:
        1. Obtenga el *id. de registro* y la *clave de aprobación del TPM* para simular el dispositivo de TPM. Para ello, siga los pasos que conducen a la sección [Simulate TPM device](quick-create-simulated-device.md#simulatetpm) (Simulación del dispositivo de TPM).
        2. Use el **_id. de registro_** y la **_clave de aprobación_** desde la salida del paso anterior para reemplazar los elementos `[RegistrationId]` y `[TPM Endorsement Key]` del archivo de código de ejemplo **_ServiceEnrollmentSample.java_**:
        
            ```Java
            private static final String REGISTRATION_ID = "[RegistrationId]";
            private static final String TPM_ENDORSEMENT_KEY = "[TPM Endorsement Key]";
            ```

    3. Si lo desea, puede configurar el servicio de aprovisionamiento con el código de ejemplo:
        - Para agregar esta configuración al ejemplo, siga estos pasos:
            1. Vaya a la instancia de IoT Hub enlazada con el servicio de aprovisionamiento en [Azure Portal](https://portal.azure.com). Abra la pestaña **Información general** de dicha instancia y copie el **nombre de host**. Asigne este **nombre de host** al parámetro *IOTHUB_HOST_NAME*.
                ```Java
                private static final String IOTHUB_HOST_NAME = "[Host name].azure-devices.net";
                ```
            2. Asigne un nombre descriptivo al parámetro *DEVICE_ID* y mantenga el elemento *PROVISIONING_STATUS* con el valor predeterminado *HABILITADO*. 
    
        - Si decide no configurar el servicio de aprovisionamiento, asegúrese de que convierte en comentario o elimina las siguientes instrucciones en el archivo _ServiceEnrollmentSample.java_:
            ```Java
            // The following parameters are optional. Remove it if you don't need.
            individualEnrollment.setDeviceId(DEVICE_ID);
            individualEnrollment.setIotHubHostName(IOTHUB_HOST_NAME);
            individualEnrollment.setProvisioningStatus(PROVISIONING_STATUS);
            ```

    4. Estudie el código de ejemplo. Crea, actualiza, consulta y elimina una inscripción de un dispositivo de TPM individual. Para comprobar si la inscripción se realizó correctamente en el portal, convierta temporalmente en comentario las siguientes líneas de código al final del archivo _ServiceEnrollmentSample.java_:
    
        ```Java
        // *********************************** Delete info of individualEnrollment ************************************
        System.out.println("\nDelete the individualEnrollment...");
        provisioningServiceClient.deleteIndividualEnrollment(REGISTRATION_ID);
        ```

    5. Guarde el archivo _ServiceEnrollmentSample.java_.

<a id="runjavasample"></a>

## <a name="build-and-run-the-java-sample-code"></a>Compilación y ejecución del código de ejemplo de Java

1. Abra una ventana de comandos y navegue hasta la carpeta **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-sample_**.

2. Compile el código de ejemplo con este comando:

    ```cmd\sh
    mvn install -DskipTests
    ```

   Este comando descarga el paquete de Maven [`com.microsoft.azure.sdk.iot.provisioning.service`](https://www.mvnrepository.com/artifact/com.microsoft.azure.sdk.iot.provisioning/provisioning-service-client) a su máquina. El paquete incluye los archivos binarios para el SDK del servicio de Java que debe compilar el código de ejemplo. 

3. Para ejecutar el ejemplo, use estos comandos en la ventana de comandos:

    ```cmd\sh
    cd target
    java -jar ./service-enrollment-sample-{version}-with-deps.jar
    ```

4. Observe la ventana de salida para comprobar si la inscripción se realizó correctamente. 

5. Vaya al servicio de aprovisionamiento en Azure Portal. Haga clic en **Administrar inscripciones** y seleccione la pestaña **Inscripciones individuales**. Tenga en cuenta que ahora aparece en la lista el *id. de registro* del TPM simulado. 

    ![Comprobación de si la inscripción de TPM se realizó correctamente en el portal](./media/quick-enroll-device-tpm-java/verify-tpm-enrollment.png)  

## <a name="clean-up-resources"></a>Limpieza de recursos
Si tiene previsto explorar el ejemplo del servicio de Java, no elimine los recursos que se crearon con esta guía de inicio rápido. Si no va a continuar, use el siguiente comando para eliminar todos los recursos creados.

1. Cierre la ventana de salida de ejemplo de Java en su máquina.
1. Cierre la ventana del simulador de TPM que haya creado para simular el dispositivo de TPM.
1. Navegue a su servicio de Device Provisioning en Azure Portal, haga clic en **Administrar inscripciones** y, a continuación, seleccione la pestaña **Inscripciones individuales**. Seleccione el *id. de registro* del dispositivo que inscribió en esta guía de inicio rápido y haga clic en el botón **Eliminar** situado en la parte superior de la hoja. 

## <a name="next-steps"></a>pasos siguientes
En esta guía de inicio rápido, inscribió un dispositivo de TPM simulado al servicio de Device Provisioning. Para más información acerca del aprovisionamiento de dispositivos, continúe con el tutorial para instalar el servicio Device Provisioning en Azure Portal. 

> [!div class="nextstepaction"]
> [Tutoriales del servicio Azure IoT Hub Device Provisioning](./tutorial-set-up-cloud.md)
