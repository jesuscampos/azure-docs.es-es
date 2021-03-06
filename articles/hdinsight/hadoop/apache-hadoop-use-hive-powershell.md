---
title: Uso de Hive de Hadoop con PowerShell en HDInsight (Azure) | Microsoft Docs
description: Use PowerShell para ejecutar consultas de Hive en Hadoop en HDInsight.
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: cb795b7c-bcd0-497a-a7f0-8ed18ef49195
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/26/2018
ms.author: larryfr
ms.openlocfilehash: fbd5ad2aedf0c3022e702a63f8e3d12735b41313
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/21/2018
---
# <a name="run-hive-queries-using-powershell"></a>Ejecución de consultas de Hive con PowerShell
[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

En este documento se ofrece un ejemplo de uso de Azure PowerShell en el modo Grupo de recursos de Azure para ejecutar consultas de Hive en un clúster Hadoop en HDInsight.

> [!NOTE]
> Este documento no ofrece una descripción detallada de las instrucciones de HiveQL que se usan en los ejemplos. Para obtener información sobre el lenguaje HiveQL que se usa en este ejemplo, vea [Uso de Hive con Hadoop en HDInsight](hdinsight-use-hive.md).

## <a name="prerequisites"></a>requisitos previos

* Un clúster de Hadoop en HDInsight basado en Linux versión 3.4 o posterior.

  > [!IMPORTANT]
  > Linux es el único sistema operativo que se usa en la versión 3.4 de HDInsight, o en las superiores. Consulte la información sobre la [retirada de HDInsight en Windows](../hdinsight-component-versioning.md#hdinsight-windows-retirement).

* Un cliente con Azure PowerShell.

[!INCLUDE [upgrade-powershell](../../../includes/hdinsight-use-latest-powershell.md)]

## <a name="run-a-hive-query"></a>Ejecución de una consulta de Hive

Azure PowerShell proporciona *cmdlets* que le permiten ejecutar de manera remota las consultas de Hive en HDInsight. Internamente, los cmdlets realizan llamadas de REST a la instancia de [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) del clúster de HDInsight.

Los siguientes cmdlets se utilizan al ejecutar las consultas de Hive en un clúster de HDInsight remoto:

* `Add-AzureRmAccount`: autentica a Azure PowerShell en la suscripción de Azure.
* `New-AzureRmHDInsightHiveJobDefinition`: crea una *definición de trabajo* con las instrucciones de HiveQL especificadas.
* `Start-AzureRmHDInsightJob`: envía la definición del trabajo a HDInsight e inicia el trabajo. Se devuelve un objeto *job*.
* `Wait-AzureRmHDInsightJob`: usa el objeto del trabajo para comprobar el estado del trabajo. Esperará hasta que el trabajo se complete o se supere el tiempo de espera.
* `Get-AzureRmHDInsightJobOutput`: se utiliza para recuperar la salida del trabajo.
* `Invoke-AzureRmHDInsightHiveJob`: se utiliza para ejecutar instrucciones de HiveQL. Este cmdlet bloquea toda la consulta y devuelve los resultados.
* `Use-AzureRmHDInsightCluster`: establece el clúster actual que se utilizará para el comando `Invoke-AzureRmHDInsightHiveJob`.

Los pasos siguientes muestran cómo usar estos cmdlets para ejecutar un trabajo en el clúster de HDInsight:

1. Mediante un editor, guarde el código siguiente como `hivejob.ps1`.

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=5-42)]

2. Abra un nuevo símbolo del sistema de **Azure PowerShell** . Cambie los directorios a la ubicación del archivo `hivejob.ps1` y, a continuación, utilice el siguiente comando para ejecutar el script:

        .\hivejob.ps1

    Cuando se ejecute el script, se le pedirá que escriba el nombre del clúster y las credenciales de la cuenta de administrador/HTTPS del clúster. Puede que también se le pida que inicie la sesión en su suscripción de Azure.

3. Cuando el trabajo se complete, la información devuelta será similar a la siguiente:

        Display the standard output...
        2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
        2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
        2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id

4. Como se mencionó anteriormente, `Invoke-Hive` se puede utilizar para ejecutar una consulta y esperar la respuesta. Use el siguiente script para ver cómo funciona Invoke-Hive:

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=50-71)]

    La salida tendrá un aspecto similar al siguiente:

        2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
        2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
        2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id

   > [!NOTE]
   > Para consultas de HiveQL más extensas, puede usar el cmdlet **Here-Strings** de Azure PowerShell o archivos de script de HiveQL. El siguiente fragmento de código indica cómo usar el cmdlet `Invoke-Hive` para ejecutar un archivo de script de HiveQL. El archivo de script de HiveQL debe cargarse en wasb://.
   >
   > `Invoke-AzureRmHDInsightHiveJob -File "wasb://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
   >
   > Para más información acerca de **Here-Strings**, consulte <a href="http://technet.microsoft.com/library/ee692792.aspx" target="_blank">Using Windows PowerShell Here-Strings</a> (Uso de Here-Strings de Windows PowerShell).

## <a name="troubleshooting"></a>solución de problemas

Si una vez completado el trabajo no se devuelve información, consulte los registros de errores. Para ver información de error para este trabajo, agregue lo siguiente al final del archivo `hivejob.ps1`, guárdelo y, a continuación, ejecútelo de nuevo.

```powershell
# Print the output of the Hive job.
Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Este cmdlet devuelve la información que se escribió en STDERR durante el procesamiento del trabajo.

## <a name="summary"></a>Resumen

Como puede ver, Azure PowerShell proporciona una manera fácil de ejecutar consultas de Hive en un clúster de HDInsight, de supervisar el estado del trabajo y de recuperar la salida.

## <a name="next-steps"></a>pasos siguientes

Para obtener información general acerca de Hive en HDInsight:

* [Uso de Hive con Hadoop en HDInsight](hdinsight-use-hive.md)

Para obtener información sobre otras maneras en que puede trabajar con Hadoop en HDInsight:

* [Uso de Pig con Hadoop en HDInsight](hdinsight-use-pig.md)
* [Uso de MapReduce con Hadoop en HDInsight](hdinsight-use-mapreduce.md)
