---
title: "VMware para la replicación de la arquitectura en Azure con Azure Site Recovery | Microsoft Docs"
description: "Este artículo proporciona información general sobre los componentes y la arquitectura que se utilizan para replicar máquinas virtuales locales de VMware en Azure con Azure Site Recovery."
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 02/27/2018
ms.author: raynew
ms.openlocfilehash: 3d20ce1da2ed9b6e3213c9689b49cc2d759e5376
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/28/2018
---
# <a name="vmware-to-azure-replication-architecture"></a>Arquitectura de replicación de VMware a Azure

En este artículo, se describe la arquitectura y los procesos que se usan al replicar, conmutar por error y recuperar máquinas virtuales de VMware entre un sitio de VMware local y Azure mediante [Azure Site Recovery](site-recovery-overview.md).


## <a name="architectural-components"></a>Componentes de la arquitectura

En la siguiente tabla y gráfico se proporciona una visión general de los componentes que se usaron para la replicación de VMware en Azure.

**Componente** | **Requisito** | **Detalles**
--- | --- | ---
**Las tablas de Azure** | Una suscripción a Azure, una cuenta de Azure Storage y una red de Azure. | Los datos replicados desde las máquinas virtuales locales se almacenan en la cuenta de almacenamiento. Las máquinas virtuales de Azure se crean con los datos replicados cuando se ejecuta una conmutación por error desde el entorno local en Azure. Las máquinas virtuales de Azure se conectan a la red virtual de Azure cuando se crean.
**Equipo del servidor de configuración** | Una sola máquina local. Se recomienda ejecutarla como una máquina virtual de VMware que pueda implementarse desde una plantilla de OVF descargada.<br/><br/> La máquina ejecuta todos los componentes locales de Site Recovery, incluido el servidor de configuración, el servidor de procesos y el servidor de destino maestro. | **Servidor de configuración**: coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.<br/><br/> **Servidor de procesos**: se instala de forma predeterminada en el servidor de configuración. Recibe los datos de la replicación; los optimiza mediante almacenamiento en caché, compresión y cifrado, y los envía a Azure Storage. El servidor de procesos también instala Azure Site Recovery Mobility Service en las máquinas virtuales que se van a replicar y realiza la detección automática de las máquinas locales. A medida que crece la implementación, puede agregar más servidores de procesos independientes para controlar mayores volúmenes de tráfico de replicación.<br/><br/> **Servidor de destino maestro**: instalado en el servidor de configuración de forma predeterminada. Controla los datos de replicación durante la conmutación por recuperación desde Azure. En el caso de las implementaciones de gran tamaño, puede agregar un servidor de destino maestro independiente adicional para la conmutación por recuperación.
**Servidores de VMware** | Las máquinas virtuales VMware se hospedan en servidores ESXi de vSphere locales. Se recomienda un servidor vCenter para administrar los hosts. | Durante la implementación de Site Recovery, se agregan servidores VMware al almacén de Recovery Services.
**Máquinas replicadas** | Mobility Service está instalado en cada una de las máquinas virtuales de VMware que se van a replicar. | Se recomienda permitir la instalación automática desde el servidor de procesos. Si lo desea, también puede instalar manualmente el servicio o usar un método de implementación automatizada, como System Center Configuration Manager.

**Arquitectura de VMware a Azure**

![Componentes](./media/vmware-azure-architecture/arch-enhanced.png)

## <a name="replication-process"></a>Proceso de replicación

1. Prepare los recursos de Azure y los componentes locales.
2. En el almacén de Recovery Services, establezca la configuración de replicación de origen. Durante este proceso, configure también el servidor de configuración local. Si desea implementar este servidor como una máquina virtual de VMware, descargue una plantilla de OVF preparada e impórtela en VMware para crear la máquina virtual.
3. Especifique la configuración de replicación de destino, cree una directiva de replicación y habilite la replicación en las máquinas virtuales de VMware.
4. Las máquinas se replican con arreglo a la directiva de replicación y, en Azure Storage, se replica una copia inicial de los datos de la máquina virtual.
5. Una vez terminada la replicación inicial, comienza la de los cambios incrementales en Azure. Las marcas de revisión de una máquina se conservan en un archivo .hrl.

    * Las máquinas se comunican con el servidor de configuración en el puerto HTTPS 443 entrante para la administración de la replicación.

    * Las máquinas envían los datos de replicación al servidor de procesos en el puerto HTTPS 9443 entrante (se puede modificar).

    * El servidor de configuración organiza la administración de la replicación con Azure a través del puerto HTTPS 443 saliente.

    * El servidor de procesos recibe datos de las máquinas de origen, los optimiza y los cifra para enviarlos después a Azure Storage a través del puerto 443 de salida.

    * Si habilita la coherencia entre varias máquinas virtuales, las máquinas del grupo de replicación se comunican entre sí a través del puerto 20004. La implementación de varias máquinas virtuales se usa si agrupa varias máquinas en grupos de replicación que tienen puntos de recuperación coherentes con los bloqueos y coherentes con la aplicación cuando conmutan por error. Este método resulta útil si las máquinas ejecutan la misma carga de trabajo y deben ser coherentes.

6. El tráfico se replica en los puntos de conexión públicos de Azure Storage a través de Internet. Como alternativa, puede usar el [emparejamiento público](../expressroute/expressroute-circuit-peerings.md#azure-public-peering) de Azure ExpressRoute. No se admite la replicación del tráfico entre un sitio local y Azure a través de una red privada virtual (VPN) de sitio a sitio.


**Proceso de replicación de VMware a Azure**

![Proceso de replicación](./media/vmware-azure-architecture/v2a-architecture-henry.png)

## <a name="failover-and-failback-process"></a>Proceso de conmutación por error y conmutación por recuperación

Una vez que la replicación está configurada y tras ejecutar una exploración de la recuperación ante desastres (conmutación por error de prueba) para comprobar que todo funciona según lo previsto, puede ejecutar la conmutación por error y la conmutación por recuperación, según sea necesario.

1. Puede conmutar por error una única máquina o crear planes de recuperación para conmutar por error varias máquinas virtuales.

2. Cuando se ejecuta una conmutación por error, se crean máquinas virtuales de Azure a partir de los datos replicados en Azure Storage.

3. Después de desencadenar la conmutación por error inicial, debe confirmarla para que se inicie. Para ello, acceda a la carga de trabajo desde la máquina virtual de Azure.

Cuando el sitio local principal esté disponible de nuevo, podrá realizar una conmutación por recuperación.
1. Debe configurar una infraestructura de conmutación por recuperación, que incluya:

    * **Servidor de procesos temporal de Azure**: para realizar una conmutación por recuperación desde Azure, debe configurar una máquina virtual de Azure para que actúe como servidor de procesos y controle la replicación desde Azure. Dicha máquina virtual se puede eliminar cuando finalice la conmutación por recuperación.

    * **Conexión VPN**: para realizar una conmutación por recuperación, necesita una conexión VPN (o ExpressRoute) entre la red de Azure y el sitio local.

    * **Servidor de destino maestro independiente**: de manera predeterminada, el servidor de destino maestro que se instaló con el servidor de configuración en la máquina virtual local de VMware controla la conmutación por recuperación. Si necesita realizar la conmutación por recuperación en grandes volúmenes de tráfico, debe configurar un servidor de destino maestro local diferente para este propósito.

    * **Directiva de conmutación por recuperación**: para replicar de nuevo en el sitio local, necesita una directiva de conmutación por recuperación. Esta directiva se creó automáticamente junto con la directiva de replicación entre el entorno local y Azure.
2. Una vez instalados los componentes, la conmutación por recuperación se produce en tres fases:

    a. Fase 1: Vuelva a proteger las máquinas virtuales de modo que realicen la replicación desde Azure de vuelta a las máquinas virtuales VMware locales.

    b. Fase 2: Ejecute una conmutación por error en el sitio local.

    c. Fase 3: Una vez que las cargas de trabajo han conmutado por recuperación, debe habilitar de nuevo la replicación de las máquinas virtuales locales.

**Conmutación por recuperación VMware desde Azure**

![Conmutación por recuperación](./media/vmware-azure-architecture/enhanced-failback.png)


## <a name="next-steps"></a>pasos siguientes

Siga [este tutorial](vmware-azure-tutorial.md) para habilitar la replicación de VMware en Azure.
