---
title: "Administración de almacenes y servidores de Azure Recovery Services | Microsoft Docs"
description: "Use este artículo para administrar los almacenes y servidores de Azure Recovery Services."
services: backup
documentationcenter: 
author: markgalioto
manager: carmonm
editor: tysonn
ms.assetid: 4eea984b-7ed6-4600-ac60-99d2e9cb6d8a
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/23/2018
ms.author: markgal
ms.openlocfilehash: 2e5fd9e7e3cae1665519e4f08604fddf7834fd51
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/24/2018
---
# <a name="monitor-and-manage-azure-recovery-services-vaults-and-servers-for-windows-machines"></a>Supervisión y administración de almacenes y servidores de los Servicios de recuperación de Azure para máquinas Windows

Este artículo incluye información general sobre las tareas de administración y supervisión de copias de seguridad disponibles en Azure Portal y en el agente de Microsoft Azure Backup. En este artículo se asume que ya tiene una suscripción de Azure y que ha creado al menos un almacén de Recovery Services.

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]


## <a name="open-a-recovery-services-vault"></a>Apertura de un almacén de Recovery Services

El panel de almacén de Recovery Services muestra los detalles o los atributos de un almacén de Recovery Services.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante la suscripción de Azure.
2. Haga clic en **Todos los servicios**. 

3. Quiere abrir un almacén de Recovery Services. En el cuadro de diálogo, comience a escribir **Recovery Services**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Haga clic en **Almacenes de Recovery Services** para mostrar la lista de almacenes de Recovery Services de la suscripción.

     ![Apertura de la lista de almacenes de Recovery Services paso 1.](./media/backup-azure-manage-windows-server/open-rs-vault-list.png) <br/>

    Se abre la lista de almacenes de Recovery Services.

    ![Creación del almacén de Recovery Services, paso 1](./media/backup-azure-manage-windows-server/list-of-rs-vaults.png) <br/>

4. En la lista de almacenes, seleccione el nombre del almacén de Recovery Services que quiere abrir. Se abre el menú del panel del almacén de Recovery Services.

    ![panel del almacén de Servicios de recuperación](./media/backup-azure-manage-windows-server/rs-vault-blade.png) <br/>

    Ahora que ha abierto el almacén de Recovery Services, pruebe cualquiera de las tareas de supervisión o administración.

## <a name="monitor-backup-jobs-and-alerts"></a>Supervisión de trabajos y alertas de copia de seguridad

Puede supervisar los trabajos y alertas desde el panel del almacén de Recovery Services, en el que encontrará:

* Detalles de las alertas de copia de seguridad
* Archivos y carpetas, así como las máquinas virtuales de Azure protegidas en la nube
* Almacenamiento total usado en Azure
* Estado del trabajo de copia de seguridad

![Copia de seguridad de las tareas del panel](./media/backup-azure-manage-windows-server/dashboard-tiles.png)

Si hace clic en la información de cada uno de estos iconos, se abrirá el menú asociado en el que podrá administrar las tareas relacionadas.

En la parte superior del panel:

* El icono de configuración: proporciona acceso a las tareas de copia de seguridad disponibles.
* El icono de copia de seguridad: ayuda a realizar la copia de seguridad de nuevos archivos y carpetas (o máquinas virtuales de Azure) en el almacén de Recovery Services.
* El icono de eliminación: en caso de que ya no se use un almacén de Recovery Services, elimínelo para liberar espacio de almacenamiento. Este icono solo estará habilitado después de que todos los servidores protegidos se hayan eliminado del almacén.

![Copia de seguridad de las tareas del panel](./media/backup-azure-manage-windows-server/dashboard-tasks.png)

## <a name="alerts-for-backups-using-azure-backup-agent"></a>Alertas de copias de seguridad mediante el agente de Azure Backup:
| Nivel de alerta | Alertas enviadas |
| --- | --- |
| Crítico | Error de copia de seguridad, error de recuperación y eliminación aplazada, es decir, cuando alguien detiene la protección con eliminar datos |
| Warning (Advertencia) | Copia de seguridad completada con advertencias (cuando menos de 100 archivos no se copian debido a problemas de daños y más de un millón de archivos se copian correctamente) |
| Informativo | Actualmente, no hay alertas informativas disponibles para el agente de Azure Backup |

## <a name="manage-backup-alerts"></a>Administración de alertas de copia de seguridad
Haga clic en el icono de **Alertas de copias de seguridad** para abrir el menú **Alertas de copias de seguridad** y administrar las alertas.

![Alertas de copias de seguridad](./media/backup-azure-manage-windows-server/manage-backup-alerts.png)

El icono de alertas de copia de seguridad le muestra el número de:

* alertas críticas sin resolver en las últimas 24 horas
* alertas de advertencia sin resolver en las últimas 24 horas

Haga clic en el vínculo para ver el menú **Alertas de copias de seguridad**, con una vista filtrada de estas alertas (críticas o de advertencia).

En el menú Alertas de copias de seguridad, puede:

* Elegir la información adecuada para incluir con sus alertas.

    ![Elegir columnas](./media/backup-azure-manage-windows-server/choose-alerts-colunms.png)
* Filtrar alertas por gravedad, estado, y hora de inicio y finalización.

    ![Filtrar alertas](./media/backup-azure-manage-windows-server/filter-alerts.png)
* Configure las notificaciones según la gravedad, la frecuencia y los destinatarios, y active o desactive las alertas.

    ![Filtrar alertas](./media/backup-azure-manage-windows-server/configure-notifications.png)

Si está seleccionada la opción **Por alerta** como frecuencia en **Notificar**, no se producirá ninguna agrupación ni reducción de los correos electrónicos. Todas las alertas generan una notificación (configuración predeterminada) y se envía inmediatamente un correo electrónico de resolución.

Si se selecciona la opción **Resumen cada hora** como frecuencia en **Notificar**, se envía un correo electrónico al usuario explicándole las alertas sin resolver que se generaron en la última hora. Se envía un correo electrónico de resolución al final del período de una hora.

Se pueden enviar alertas para los siguientes niveles de gravedad:

* Crítico
* Warning (Advertencia)
* informativo

Desactive la alerta con el botón **Desactivar** en el menú de detalles del trabajo. Al hacer clic en Desactivar, puede proporcionar notas de resolución.

Elija las columnas que desea que aparezcan como parte de la alerta con el botón **Elegir columnas** .

> [!NOTE]
> En el menú **Configuración**, puede administrar alertas de copias de seguridad seleccionando **Supervisión e informes > Alertas y eventos > Alertas de copias de seguridad** y haciendo clic en **Filtrar** o en **Configurar notificaciones**.
>
>

## <a name="manage-backup-items"></a>Administración de elementos de copia de seguridad
La administración de copias de seguridad locales ya está disponible en el Portal de administración. En la sección Copia de seguridad del panel, el icono **Elementos de copia de seguridad** muestra el número de elementos de copia de seguridad protegidos en el almacén.

Haga clic en **Archivos y carpetas** en el icono Elementos de copia de seguridad.

![Icono Elementos de copia de seguridad](./media/backup-azure-manage-windows-server/backup-items-tile.png)

Se abre el menú Elementos de copia de seguridad con el filtro establecido en Archivos y carpetas, en el que podrá ver cada elemento de copia de seguridad específico en una lista.

![Elementos de copia de seguridad](./media/backup-azure-manage-windows-server/backup-item-list.png)

Si selecciona un elemento de una copia de seguridad específica de la lista, podrá ver los detalles esenciales de ese elemento.

> [!NOTE]
> Desde el menú **Configuración**, puede administrar los archivos y carpetas seleccionando **Elementos protegidos > Elementos de copia de seguridad** y, después, **Archivos y carpetas** en el menú desplegable.
>
>

![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/backup-files-and-folders.png)

## <a name="manage-backup-jobs"></a>Administración de trabajos de copia de seguridad
Los trabajos de copia de seguridad locales (cuando el servidor local realiza la copia de seguridad en Azure) y los de copias de seguridad de Azure aparecen en el panel.

En la sección Copia de seguridad del panel, el icono Trabajo de copia de seguridad muestra el número de trabajos:

* En curso
* Con error en las últimas 24 horas.

Para administrar los trabajos de copia de seguridad, haga clic en el icono **Trabajos de copia de seguridad**, que permite abrir el menú del mismo nombre.

![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/backup-jobs.png)

Puede modificar la información disponible en el menú Trabajos de copia de seguridad con el botón **Elegir columnas** situado en la parte superior de la página.

Use el botón **Filtrar** para seleccionar entre Archivos y carpetas y Copia de seguridad de la máquina virtual de Azure.

Si no ve los archivos y carpetas de los que se ha realizado la copia de seguridad, haga clic en el botón **Filtrar** situado en la parte superior de la página y seleccione **Archivos y carpetas** en el menú Tipo de elemento.

> [!NOTE]
> En el menú **Configuración**, puede administrar los trabajos de copia de seguridad mediante **Supervisión e informes > Trabajos > Trabajos de copia de seguridad** y, después, seleccionar **Archivos y carpetas** en el menú desplegable.
>
>

## <a name="monitor-backup-usage"></a>Supervisión del uso de Copia de seguridad
En la sección Copia de seguridad del panel, el icono Uso de copia de seguridad muestra el almacenamiento consumido en Azure. El uso de almacenamiento se proporciona para:

* Uso de almacenamiento LRS en la nube asociado con el almacén
* Uso de almacenamiento GRS en la nube asociado con el almacén

## <a name="manage-your-production-servers"></a>Administración de los servidores de producción
Para administrar los servidores de producción, haga clic en **Configuración**.

En Administrar, haga clic en **Infraestructura de copia de seguridad > Servidores de producción**.

El menú Servidores de producción muestra todos los servidores de producción disponibles. Haga clic en un servidor de la lista para abrir los detalles del servidor.

![Elementos protegidos](./media/backup-azure-manage-windows-server/production-server-list.png)


## <a name="open-the-azure-backup-agent"></a>Apertura del agente de Azure Backup
Abra el **agente de Microsoft Azure Backup** (puede encontrarlo si busca en su equipo *Microsoft Azure Backup*).

![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/snap-in-search.png)

Con las **acciones** disponibles en la parte derecha de la consola del agente de Copia de seguridad, podrá llevar a cabo las siguientes tareas de administración:

* Registrar un servidor
* Programación de una copia de seguridad
* Realizar una copia de seguridad en ese momento
* Cambiar propiedades

![Acciones de la consola del agente de Microsoft Azure Backup](./media/backup-azure-manage-windows-server/console-actions.png)

> [!NOTE]
> Para **recuperar datos**, consulte [Restauración de archivos en una máquina de Windows Server o del cliente de Windows](backup-azure-restore-windows-server.md).
>
>

## <a name="modify-the-backup-schedule"></a>Modificación de la programación de copias de seguridad
1. En el agente de Microsoft Azure Backup, haga clic en **Programar copia de seguridad**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/schedule-backup.png)
2. En el **Asistente para programar copias de seguridad**, deje activada la opción **Cambiar la hora o las horas de las copias de seguridad** y haga clic en **Siguiente**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)
3. Si quiere agregar o cambiar elementos, en la pantalla **Seleccionar elementos de los que realizar copia de seguridad**, haga clic en **Agregar elementos**.

    También puede establecer preferencias en **Configuración de exclusión** , en esta página del asistente. Si quiere excluir archivos o tipos de archivos, lea el procedimiento para agregar [configuración de exclusión](#manage-exclusion-settings).
4. Seleccione los archivos y las carpetas de los que quiere realizar la copia de seguridad y haga clic en **Aceptar**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/add-items-modify.png)
5. Especifique la **programación de copia de seguridad** y haga clic en **Siguiente**.

    Puede programar copias de seguridad diarias (un máximo de 3 veces al día) o semanales.

    ![Elementos para la copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/specify-backup-schedule-modify-close.png)

   > [!NOTE]
   > En este [artículo](backup-azure-backup-cloud-as-tape.md)se explica detalladamente cómo especificar la programación de copias de seguridad.
   >

6. Seleccione la **directiva de retención de la copia de seguridad** y haga clic en **Siguiente**.

    ![Elementos para la copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/select-retention-policy-modify.png)
7. En la pantalla **Confirmación**, revise la información y haga clic en **Finalizar**.
8. Cuando el asistente finalice la creación de la **programación de copia de seguridad**, haga clic en **Cerrar**.

    Tras modificar la protección, puede confirmar que las copias de seguridad se estén activando correctamente; para ello, vaya a la pestaña **Trabajos** y confirme que se hayan reflejado los cambios en los trabajos de copia de seguridad.

## <a name="enable-network-throttling"></a>Habilitación de la limitación de la red

En el agente de Azure Backup se incluye la pestaña Limitación, donde podrá controlar cómo se utiliza el ancho de banda de red durante la transferencia de datos. Este control puede resultar de ayuda si necesita realizar una copia de seguridad de los datos durante las horas de trabajo, pero no quiere que el proceso interfiera con otro tráfico de Internet. La limitación de la transferencia de datos se aplica a las actividades de copia de seguridad y restauración.  

Para habilitar la limitación, siga estos pasos:

1. En el **agente de Backup**, haga clic en **Cambiar propiedades**.
2. En la pestaña **Limitación, seleccione la opción **Habilitar límite de uso del ancho de banda de Internet para las operaciones de copia de seguridad**.

    ![Limitación de la red](./media/backup-azure-manage-windows-server/throttling-dialog.png)

    Una vez que se ha habilitado la limitación, especifique el ancho de banda permitido para la transferencia de datos de copia de seguridad durante la **jornada laboral** y las **horas de descanso**.

    Los valores de ancho de banda comienzan en 512 kilobytes por segundo (Kbps) y pueden subir hasta 1023 megabytes por segundo (Mbps). También puede designar el inicio y el final de la **jornada laboral**, así como qué días de la semana se consideran laborables. El tiempo no comprendido en la jornada laboral designada se considera horas de descanso.
3. Haga clic en **OK**.

## <a name="manage-exclusion-settings"></a>Administración de la configuración de exclusión
1. Abra el **agente de Microsoft Azure Backup** (puede encontrarlo si busca en su equipo *Microsoft Azure Backup*).

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/snap-in-search.png)
2. En el agente de Microsoft Azure Backup, haga clic en **Programar copia de seguridad**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/schedule-backup.png)
3. En el Asistente para programar copias de seguridad deje activada la opción **Cambiar la hora o las horas de las copias de seguridad** y haga clic en **Siguiente**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)
4. Haga clic en **Configuración de exclusión**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/exclusion-settings.png)
5. Haga clic en **Agregar exclusión**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/add-exclusion.png)
6. Seleccione la ubicación y, después, haga clic en **Aceptar**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/exclusion-location.png)
7. Agregue la extensión de archivo en el campo **Tipo de archivo** .

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/exclude-file-type.png)

    Adición de una extensión .mp3

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/exclude-mp3.png)

    Para agregar otra extensión, haga clic en **Agregar exclusión** y especifique otra extensión de tipo de archivo (por ejemplo, .jpeg).

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/exclude-jpg.png)
8. Cuando haya agregado todas las extensiones, haga clic en **Aceptar**.
9. Avance por las distintas pantallas del Asistente para programar copias de seguridad haciendo clic en **Siguiente** hasta llegar a la página **Confirmación** y, después, en **Finalizar**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-azure-manage-windows-server/finish-exclusions.png)

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
**P1. El estado del trabajo de copia de seguridad aparece como completado en el agente de Azure Backup, ¿por qué no se ve reflejado inmediatamente en el portal?**

R1. Hay un retraso máximo de 15 minutos entre que el estado del trabajo de copia de seguridad se refleja en el agente de Azure Backup y en Azure Portal.

**P.2 Cuando se produce un error en un trabajo de copia de seguridad, ¿cuánto tiempo se tarda en generar una alerta?**

R.2 Se genera una alerta en menos de 20 minutos desde que se produce el error de copia de seguridad de Azure.

**P3. ¿Hay algún caso en el que no se envíe ningún correo electrónico si se configuran las notificaciones?**

R3. A continuación figuran los casos en los que no se enviará la notificación con el fin de reducir el ruido de las alertas:

* Si se configuran las notificaciones por horas y se genera una alerta y esta se resuelve en menos de una hora.
* Se canceló el trabajo.
* Se produjo un error en el segundo trabajo de copia de seguridad porque el trabajo de copia de seguridad original está en curso.

## <a name="troubleshooting-monitoring-issues"></a>Solución de problemas de supervisión
**Problema**: los trabajos y las alertas del agente de Azure Backup no aparecen en el portal.

**Pasos para solucionar problemas:** el proceso ```OBRecoveryServicesManagementAgent```, envía los datos de alerta y del trabajo para al servicio de Azure Backup. En ocasiones este proceso puede quedar atascado o apagado.

1. Para comprobar que el proceso no se está ejecutando, abra **Administrador de tareas** y compruebe si el proceso ```OBRecoveryServicesManagementAgent``` se está ejecutando.
2. Suponiendo que el proceso no se está ejecutando, abra el **Panel de Control** y examine la lista de servicios. Inicie o reinicie el **agente de administración de Microsoft Azure Recovery Services**.

    Para más información, examine los registros en:<br/>
   Por ejemplo: `<AzureBackup_agent_install_folder>\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider*`<br/>
   `C:\Program Files\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider0.errlog`

## <a name="next-steps"></a>pasos siguientes
* [Restauración de Windows Server o el cliente de Windows desde Azure](backup-azure-restore-windows-server.md)
* Para obtener más información sobre Azure Backup, consulte [Información general de Azure Backup](backup-introduction-to-azure-backup.md)
* Visite el [Foro de Azure Backup](http://go.microsoft.com/fwlink/p/?LinkId=290933)
