---
title: Uso de firmas de acceso compartido (SAS) en Azure Storage | Microsoft Docs
description: "Obtenga información acerca de cómo usar firmas de acceso compartido (SAS) para delegar recursos de Azure Storage, incluidos blobs, colas, tablas y archivos."
services: storage
documentationcenter: 
author: tamram
manager: timlt
editor: tysonn
ms.assetid: 46fd99d7-36b3-4283-81e3-f214b29f1152
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 04/18/2017
ms.author: tamram
ms.openlocfilehash: 32e92e6ffc376d27297810596691f0371770e86d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/11/2017
---
# <a name="using-shared-access-signatures-sas"></a>Uso de firmas de acceso compartido (SAS)

Una firma de acceso compartido (SAS) ofrece una manera de conceder acceso limitado a los objetos de la cuenta de almacenamiento a otros clientes sin exponer la clave de cuenta. En este artículo, se proporciona una introducción del modelo SAS, se analizan las prácticas recomendadas de SAS y se revisan algunos ejemplos.

Para obtener ejemplos de código adicionales mediante SAS, aparte de los aquí presentados, consulte [Introducción a Azure Blob Storage en .NET](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/) y otros ejemplos disponibles en la biblioteca [Ejemplos de código de Azure](https://azure.microsoft.com/documentation/samples/?service=storage). Puede descargar las aplicaciones de ejemplo y ejecutarlas, así como ver el código en GitHub.

## <a name="what-is-a-shared-access-signature"></a>¿Qué es una firma de acceso compartido?
Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Con una SAS, puede conceder a los clientes acceso a los recursos de su cuenta de almacenamiento sin compartir las claves de la cuenta. Este es el aspecto clave de usar las firmas de acceso compartido en las aplicaciones: una SAS es una forma segura de compartir los recursos de almacenamiento sin poner en peligro las claves de cuenta.

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

Una SAS le ofrece control granular el tipo de acceso que se concede a los clientes que tienen la SAS, como por ejemplo:

* El intervalo durante el que la SAS es válida, incluida la hora de inicio y la hora de caducidad.
* Los permisos concedidos por la SAS. Por ejemplo, una SAS para un blob podría conceder permisos de lectura y escritura para el blob, pero no permisos de eliminación.
* Una dirección IP opcional o un intervalo de direcciones IP de las que Azure Storage aceptará la SAS. Por ejemplo, podría especificar un intervalo de direcciones IP que pertenezcan a su organización.
* El protocolo a través del cual Azure Storage aceptará la SAS. Puede usar este parámetro opcional para restringir el acceso a los clientes mediante HTTPS.

## <a name="when-should-you-use-a-shared-access-signature"></a>¿Cuándo debe usar una firma de acceso compartido?
Puede usar una SAS cuando desee proporcionar acceso a los recursos de la cuenta de almacenamiento a cualquier cliente que no posea sus claves de acceso a la cuenta de almacenamiento. La cuenta de almacenamiento incluye tanto una clave de acceso primaria y como una secundaria, que conceden acceso administrativo a la cuenta y a todos los recursos contenidos en ella. La exposición de alguna de estas claves posibilita que se haga un uso negligente o malintencionado de la cuenta. Las firmas de acceso compartido ofrecen una alternativa segura que permite a los clientes leer, escribir y eliminar datos en la cuenta de almacenamiento según los permisos que se le hayan concedido explícitamente y sin que sea necesaria una clave de cuenta.

Un escenario común en el que es útil una SAS es un servicio en el que los usuarios leen y escriben sus propios datos en la cuenta de almacenamiento. Existen dos patrones de diseño típicos en los escenarios en los que una cuenta de almacenamiento guarda datos de usuario:

1. Los clientes cargan y descargan datos mediante un servicio de proxy front-end que realiza la autenticación. Este servicio de proxy front-end cuenta con la ventaja de permitir la validación de reglas de negocio, pero para grandes cantidades de datos o transacciones de gran volumen, la creación de un servicio que pueda escalarse para satisfacer la demanda puede ser complicada o costosa.

  ![Diagrama del escenario: servicio de proxy de front-end](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png)   

1. Un servicio ligero realiza la autenticación del cliente según sea necesario y, luego, genera una SAS. Una vez que el cliente recibe la SAS, puede obtener acceso a los recursos de la cuenta de almacenamiento directamente con los permisos definidos por la SAS y para el intervalo permitido por ella. La SAS mitiga la necesidad de enrutar todos los datos a través del servicio de proxy front-end.

  ![Diagrama del escenario: servicio de proveedor de SAS](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png)   

Muchos servicios reales pueden usar una combinación de estos dos enfoques. Por ejemplo, se pueden procesar y validar algunos datos mediante el proxy de front-end, mientras que otros datos se guardan o leen directamente mediante SAS.

Además, deberá usar una SAS para autenticar el objeto de origen en una operación de copia en ciertos escenarios:

* Cuando copia un blob en otro blob que reside en otra cuenta de almacenamiento, debe usar una SAS para autenticar el blob de origen. También puede usar una SAS para autenticar el blob de destino.
* Cuando copia un archivo en otro archivo que reside en otra cuenta de almacenamiento, debe usar una SAS para autenticar el archivo de origen. También puede usar una SAS para autenticar el archivo de destino.
* Si va a copiar un blob en un archivo, o un archivo en un blob, tiene que usar una firma de acceso compartido (SAS) para autenticar el objeto de origen, incluso si los objetos de origen y destino residen dentro la misma cuenta de almacenamiento.

## <a name="types-of-shared-access-signatures"></a>Tipos de firmas de acceso compartido
Puede crear dos tipos de firmas de acceso compartido:

* **SAS de servicio.** SAS de servicio delega el acceso a un recurso en solo uno de los servicios de almacenamiento: el servicio Blob, Cola, Tabla o Archivo. Consulte [Creación de una SAS de servicio](https://msdn.microsoft.com/library/dn140255.aspx) y [Ejemplos de SAS de servicio](https://msdn.microsoft.com/library/dn140256.aspx), para obtener información detallada acerca de cómo construir el token de SAS de servicio.
* **SAS de cuenta.** SAS de cuenta delega el acceso a los recursos en uno o varios de los servicios de almacenamiento. Todas las operaciones disponibles con una SAS de servicio están también disponibles con una SAS de cuenta. Además, con la SAS de cuenta, puede delegar el acceso a las operaciones que se aplican a un servicio determinado como **Get/Set Service Properties** y **Get Service Stats**. También puede delegar el acceso para leer, escribir y eliminar operaciones en contenedores de blobs, tablas, colas y recursos compartidos de archivos que no están permitidos con SAS de servicio. Consulte [Creación de una SAS de cuenta](https://msdn.microsoft.com/library/mt584140.aspx) para obtener información detallada sobre cómo crear el token de SAS de cuenta.

## <a name="how-a-shared-access-signature-works"></a>Funcionamiento de una firma de acceso compartido
Una firma de acceso compartido es un URI firmado que señala a uno o más recursos de almacenamiento e incluye un token que contiene un conjunto especial de parámetros de consulta. El token indica cómo puede el cliente tener acceso a los recursos. Uno de los parámetros de consulta, la firma, se construye a partir de parámetros SAS y se firma con la clave de cuenta. Almacenamiento de Azure usa esa firma para autenticar la SAS.

Este es un ejemplo de un URI de SAS, que muestra el URI de recurso y el token de SAS:

![Componentes de un identificador URI de SAS](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-uri.png)   

El token de SAS es una cadena que genera en el *cliente* (consulte la sección de [Ejemplos de SAS](#sas-examples) para ver ejemplos de código). Por ejemplo, Azure Storage no realiza el seguimiento de un token de SAS que haya generado con la biblioteca de cliente de almacenamiento. Puede crear un número ilimitado de tokens de SAS en el cliente.

Cuando un cliente proporciona un URI de SAS para Azure Storage como parte de una solicitud, el servicio comprueba los parámetros de SAS y la firma para comprobar que es válido para autenticar la solicitud. Si el servicio confirma la firma es válida, la solicitud se autentica. De lo contrario, la solicitud se rechaza con el código de error 403 (prohibido).

## <a name="shared-access-signature-parameters"></a>Parámetros de la firma de acceso compartido
Los tokens de SAS de cuenta y de SAS de servicio incluyen algunos parámetros comunes, pero también usan otros que son diferentes.

### <a name="parameters-common-to-account-sas-and-service-sas-tokens"></a>Parámetros comunes para tokens de SAS de cuenta y de SAS de servicio
* **Versión de API.** Un parámetro opcional que especifica la versión del servicio de almacenamiento que se usa para ejecutar la solicitud.
* **Versión del servicio.** Es un parámetro necesario que especifica la versión del servicio de almacenamiento que se usa para autenticar la solicitud.
* **Hora de inicio.** Es la hora en la que la SAS comienza a ser válida. La hora de inicio de una firma de acceso compartido es opcional. Si se omite una hora de inicio, la SAS entra en vigor de inmediato. La hora de inicio se debe expresar en UTC (Hora universal coordinada) con un designador de hora UTC especial ("Z"), por ejemplo `1994-11-05T13:15:30Z`.
* **Hora de expiración.** Es la hora a partir de la que la SAS deja de ser válida. En las prácticas recomendadas se aconseja especificar una hora de expiración para una SAS o asociarla a una directiva de acceso almacenada. La hora de expiración se debe expresar en UTC (Hora universal coordinada) con un designador de hora UTC especial ("Z"), por ejemplo `1994-11-05T13:15:30Z` (vea más información a continuación).
* **Permisos.** Los permisos especificados en una SAS indican qué operaciones puede realizar el cliente en el recurso de almacenamiento con la SAS. Los permisos disponibles son diferentes para SAS de cuenta y SAS de servicio.
* **Dirección Dirección IP.** Es un parámetro opcional que especifica una dirección IP o un intervalo de direcciones IP fuera de Azure (consulte la sección [Estado de la configuración de la sesión de enrutamiento](../../expressroute/expressroute-workflows.md#routing-session-configuration-state) de Express Route), desde el cual puede aceptar solicitudes.
* **Protocolo.** Un parámetro opcional que especifica el protocolo permitido para una solicitud. Los valores posibles son HTTPS y HTTP (`https,http`), que es el valor predeterminado, o solo HTTPS solo (`https`). Tenga en cuenta que HTTP solo no es un valor permitido.
* **Firma.** La firma se construye a partir de los parámetros especificados como parte del token y, a continuación, se cifra. Se usa para autenticar la SAS.

### <a name="parameters-for-a-service-sas-token"></a>Parámetros para un token de SAS de servicio
* **Recurso de almacenamiento.** Los recursos de almacenamiento para los que puede delegar el acceso con una SAS de servicio incluyen:
  * Contenedores y blobs
  * Archivos y recursos compartidos de archivo
  * Colas
  * Las tablas y los intervalos de las entidades de tabla.

### <a name="parameters-for-an-account-sas-token"></a>Parámetros para un token de SAS de cuenta
* **Servicio o servicios.** SAS de cuenta puede delegar el acceso a uno o varios de los servicios de almacenamiento. Por ejemplo, puede crear una SAS de cuenta que delega el acceso al servicio Blob y Archivo. O bien, puede crear una SAS que delega el acceso a los cuatro servicios (Blob, Cola, Tabla y Archivo).
* **Tipos de recursos de almacenamiento.** SAS de cuenta se aplica a una o más clases de recursos de almacenamiento, más que a un recurso específico. Puede crear una SAS de cuenta que delega el acceso a:
  * Las API de nivel de servicio, a las que se llaman en el recurso de la cuenta de almacenamiento. Algunos ejemplos incluyen **Get/Set Service Properties**, **Get Service Stats**, and **List Containers/Queues/Tables/Shares**.
  * Las API de nivel de contenedor, a las que se llaman en los objetos de contenedor para cada servicio: contenedores de blobs, colas, tablas y recursos compartidos de archivos. Algunos ejemplos incluyen **Create/Delete Container**, **Create/Delete Queue**, **Create/Delete Table**, **Create/Delete Share** y **List Blobs/Files y Directories**.
  * Las API de nivel de objeto, a las que se llaman en blobs, mensajes de colas, entidades de tablas y archivos. Por ejemplo, **Put Blob**, **Query Entity**, **Get Messages** y **Create File**.

## <a name="examples-of-sas-uris"></a>Ejemplos de URI de SAS

### <a name="service-sas-uri-example"></a>Ejemplo de identificador URI de SAS de servicio

A continuación se muestra un ejemplo de un URI de SAS de servicio que ofrece permisos de lectura y escritura en un blob. En la tabla siguiente se divide cada parte del URI para saber cómo contribuye a la SAS:

```
https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
```

| Nombre | Parte de SAS | Description |
| --- | --- | --- |
| URI de blobs |`https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt` |La dirección del blob. Tenga en cuenta que se recomienda fehacientemente el uso de HTTPS. |
| Versión de servicios de almacenamiento |`sv=2015-04-05` |En la versión de servicios de almacenamiento 2012-02-12 y superiores, este parámetro indica qué versión usar. |
| Hora de inicio |`st=2015-04-29T22%3A18%3A26Z` |Se especifica en hora UTC. Si desea que la SAS sea válida de inmediato, omita la hora de inicio. |
| Hora de expiración |`se=2015-04-30T02%3A23%3A26Z` |Se especifica en hora UTC. |
| Recurso |`sr=b` |El recurso es un blob. |
| Permisos |`sp=rw` |Los permisos que concede la SAS son de lectura y escritura. |
| Intervalo de IP |`sip=168.1.5.60-168.1.5.70` |El intervalo de direcciones IP desde el que se aceptará una solicitud. |
| Protocolo |`spr=https` |Solo se permiten solicitudes con HTTPS. |
| Firma |`sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D` |Se usa para autenticar el acceso al blob. La firma es un HMAC que se procesa mediante una cadena para firmar y una clave con el algoritmo SHA256, y que se codifica mediante Base64. |

### <a name="account-sas-uri-example"></a>Ejemplo de identificador URI de SAS de cuenta

Este es un ejemplo de una SAS de cuenta que usa los mismos parámetros comunes en el token. Como estos parámetros aparecen descritos anteriormente, no los vamos a describir aquí. Solo los parámetros que son específicos de la SAS de cuenta se describen en la tabla siguiente.

```
https://myaccount.blob.core.windows.net/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B
```

| Nombre | Parte de SAS | Description |
| --- | --- | --- |
| URI de recurso |`https://myaccount.blob.core.windows.net/?restype=service&comp=properties` |Extremo de servicio BLOB, con parámetros para obtener propiedades de servicio (cuando se llama con GET) o para establecer propiedades de servicio (cuando se llama con SET). |
| Services |`ss=bf` |La SAS se aplica a los servicios Blob y Archivo |
| Tipos de recursos |`srt=s` |La SAS se aplica a las operaciones de nivel de servicio. |
| Permisos |`sp=rw` |Los permisos conceden acceso para operaciones de lectura y escritura. |

Dado que los permisos están restringidos al nivel de servicio, las operaciones accesibles con esta SAS son **Get Blob Service Properties** (lectura) y **Set Blob Service Properties** (escritura). Sin embargo, con un URI de recurso diferente, el mismo token de SAS también se puede usar para delegar el acceso a **Get Blob Service Stats** (lectura).

## <a name="controlling-a-sas-with-a-stored-access-policy"></a>Control de una SAS con una directiva de acceso almacenada
Una firma de acceso compartido puede presentar una de estas dos formas:

* **SAS ad-hoc**: cuando cree una SAS ad-hoc, la hora de inicio, la hora de expiración y los permisos para la SAS se especifican en el URI de SAS (o se encuentran implícitos en el caso de que se omita la hora de inicio). Este tipo de SAS puede crearse como SAS de cuenta o como SAS de servicio.
* **SAS con directiva de acceso almacenada**: se define una directiva de acceso almacenada en un contenedor de recursos (un contenedor de blobs, un archivo compartido, un archivo, una tabla o una cola) y se puede usar para administrar las restricciones de una o varias firmas de acceso compartido. Cuando asocia una SAS a una directiva de acceso almacenada, la SAS hereda las restricciones (hora de inicio, hora de expiración y permisos) definidas para la directiva de acceso almacenada.

> [!NOTE]
> Actualmente, una SAS de cuenta debe ser una SAS ad hoc. Las directivas de acceso almacenadas ya no son compatibles para SAS de cuenta.

La diferencia entre las dos formas es importante para un escenario principal: revocación. Dado que un URI de SAS es una dirección URL, cualquier persona que obtenga la SAS puede utilizarla, independientemente de quién la creó originalmente. Si una SAS se encuentra disponible públicamente, cualquier persona del mundo puede usarla. Una SAS concede acceso a los recursos a cualquier persona que cuente con ella hasta que se sucede uno de las siguientes situaciones:

1. Se alcanza el tiempo de expiración especificado en la SAS.
2. Se alcanza el tiempo de expiración especificado en la directiva de acceso almacenada al que hace referencia la SAS (si se hace referencia a una directiva de acceso almacenada y si especifica una hora de expiración). Esto puede producirse ya sea porque transcurra el intervalo o porque haya modificado la directiva de acceso almacenada para tener una hora de expiración pasada, que es una forma de revocar la SAS.
3. Se elimina la directiva de acceso almacenada a la que hace referencia la SAS, que es otra forma de revocar la SAS. Tenga en cuenta que si vuelve a crear la directiva de acceso almacenada exactamente con el mismo nombre, todos los tokens de la SAS volverán a ser válidos según los permisos asociados a la directiva de acceso almacenada (si se presupone que la hora de expiración en la SAS no ha pasado). Si prevé revocar la SAS, asegúrese de usar un nombre distinto si vuelve a crear la directiva de acceso con una hora de expiración futura.
4. Se vuelve a generar la clave de cuenta que se usó para crear la SAS. La regeneración de una clave de cuenta provocará que todos los componentes de la aplicación que usen esa clave no puedan autenticarse hasta que se actualicen para usar la otra clave de cuenta válida o la clave de cuenta que se acaba de regenerar.

> [!IMPORTANT]
> Los URI de firma de acceso compartido están asociados a la clave de la cuenta que se utiliza para crear la firma y a la directiva de acceso almacenada correspondiente (en su caso). Si no se especifica una directiva de acceso almacenada, la única forma de revocar una firma de acceso compartido es cambiar la clave de la cuenta.

## <a name="authenticating-from-a-client-application-with-a-sas"></a>Autenticación desde una aplicación cliente con una SAS
Un cliente que esté en posesión de una SAS puede usarla para autenticar una solicitud en una cuenta de almacenamiento para la que no posee las claves de cuenta. Una SAS se puede incluir en una cadena de conexión, o usar directamente desde el método o constructor adecuados.

### <a name="using-a-sas-in-a-connection-string"></a>Uso de SAS en una cadena de conexión
[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

### <a name="using-a-sas-in-a-constructor-or-method"></a>Uso de una SAS en un constructor o método
Varios sobrecargas de métodos y constructores de biblioteca de cliente de Azure Storage ofrecen un parámetro SAS, para que pueda autenticar una solicitud al servicio con una SAS.

Por ejemplo, aquí se usa un URI de SAS para crear una referencia a un blob en bloques. La SAS proporciona las únicas credenciales necesarias para la solicitud. La referencia de blob en bloques se usa luego para una operación de escritura:

```csharp
string sasUri = "https://storagesample.blob.core.windows.net/sample-container/" +
    "sampleBlob.txt?sv=2015-07-08&sr=b&sig=39Up9JzHkxhUIhFEjEH9594DJxe7w6cIRCg0V6lCGSo%3D" +
    "&se=2016-10-18T21%3A51%3A37Z&sp=rcw";

CloudBlockBlob blob = new CloudBlockBlob(new Uri(sasUri));

// Create operation: Upload a blob with the specified name to the container.
// If the blob does not exist, it will be created. If it does exist, it will be overwritten.
try
{
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
    {
        await blob.UploadFromStreamAsync(msWrite);
    }

    Console.WriteLine("Create operation succeeded for SAS {0}", sasUri);
    Console.WriteLine();
}
catch (StorageException e)
{
    if (e.RequestInformation.HttpStatusCode == 403)
    {
        Console.WriteLine("Create operation failed for SAS {0}", sasUri);
        Console.WriteLine("Additional error information: " + e.Message);
        Console.WriteLine();
    }
    else
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}

```

## <a name="best-practices-when-using-sas"></a>Procedimientos recomendados al usar SAS
Cuando use firmas de acceso compartido en sus aplicaciones, debe tener en cuenta dos posibles riesgos:

* Si se perdió una SAS, cualquier persona que la consiga puede usarla, lo que puede poner en riesgo su cuenta de almacenamiento.
* Si una SAS proporcionada para una aplicación cliente expira y la aplicación no puede recuperar una nueva SAS del servicio, la funcionalidad de la aplicación puede verse afectada.

Las siguientes recomendaciones para el uso de firmas de acceso compartido pueden ayudar a mitigar estos riesgos:

1. **Siempre use HTTPS** para crear una SAS o distribuirla. Si se pasa una SAS a través de HTTP y se intercepta, un atacante que realice un ataque de tipo "Man in the middle" puede leer la SAS y, luego, usarla como lo podría hacer el usuario previsto. Esto puede poner en riesgo la información confidencial o permitir que un usuario malintencionado provoque daños en los datos.
2. **Haga referencia a las directivas de acceso almacenadas cuando sea posible.** Las directivas de acceso almacenadas le ofrecen la posibilidad de revocar permisos sin tener que volver a generar las claves de cuenta de almacenamiento. Establezca su expiración en un futuro muy lejano (o infinito) y asegúrese de que se actualiza regularmente para trasladarla a un punto posterior en el tiempo.
3. **Use horas de expiración a corto plazo en una SAS ad-hoc.** De esta manera, incluso si una SAS está en peligro, es válida solo durante un breve período. Esta práctica es especialmente importante si no puede hacer referencia a una directiva de acceso almacenada. Las expiraciones a corto plazo también limitan la cantidad de datos que puede escribirse en un blob mediante la limitación del tiempo disponible para cargarlos.
4. **Haga que los clientes renueven automáticamente la SAS si fuese necesario.** Los clientes deben renovar la SAS correctamente antes de la expiración para que exista tiempo para los reintentos si el servicio que ofrece la SAS no está disponible. Si la SAS se ha creado para usarse en un número reducido de operaciones de corta duración e inmediatas, que se espera que se va a completar dentro del tiempo de expiración determinado, es posible que no sea necesario ese procedimiento, ya que no se espera que la SAS se renueve. Sin embargo, si dispone de un cliente que realice solicitudes de forma rutinaria a través de la SAS, existe la posibilidad de la expiración. La consideración clave es equilibrar la necesidad de que la SAS sea de corta duración (como se indicó anteriormente) con el requisito de garantizar que el cliente solicitan la renovación con la suficiente antelación como para evitar la interrupción debida a la expiración de SAS antes de una renovación correcta.
5. **Tenga cuidado con la hora de inicio de la SAS.** Si establece la hora de inicio de la SAS en **now**, pueden producirse errores intermitentes durante los primeros minutos debido al desplazamiento del reloj (diferencias en la hora actual según las distintas máquinas). En general, establezca la hora de inicio sea al menos 15 minutos en el pasado. O, no establezca esta opción en absoluto, lo que hará que sea válido inmediatamente en todos los casos. Normalmente se aplica lo mismo a la hora de expiración. Recuerde que debe tener en cuenta hasta 15 minutos de desplazamiento del reloj en cualquier dirección en una solicitud. Para los clientes con una versión REST anterior a 2012-02-12: la duración máxima de una SAS que no hace referencia a una directiva de acceso almacenada es de 1 hora y se producirá un error en las directivas que especifican un período más largo.
6. **Sea específico con el recurso al que se va a tener acceso.** Un procedimiento recomendado de seguridad es proporcionar al usuario los privilegios mínimos necesarios. Si un usuario solo necesita acceso de lectura en una única entidad, concédale acceso de lectura a esa única entidad y no acceso de lectura, escritura o eliminación a todas las entidades. Esto también ayuda a reducir los daños si se pone en peligro una SAS porque la SAS tiene menor menos poder en manos de un atacante.
7. **Comprenda que se le hará un cargo en la cuenta por cualquier uso, incluido el realizado con la SAS.** Si proporciona acceso de escritura a un blob, el usuario puede seleccionar cargar un blob de 200 GB. Si le proporciona también acceso de lectura, puede seleccionar descargarlo 10 veces, lo que le supone 2 TB de costos de salida. Proporcione de nuevo permisos limitados para ayudar a mitigar acciones potenciales de usuarios malintencionados. Use una SAS de corta duración para reducir esa amenaza (pero tenga en cuenta el desplazamiento del reloj y la hora final).
8. **Valide los datos escritos mediante la SAS.** Cuando una aplicación cliente escribe datos en la cuenta de almacenamiento, tenga en cuenta que pueden existir problemas con esos datos. Si la aplicación requiere que se validen o autoricen los datos antes de que estén listos para usar, debe realizar la validación después de que se escriban los datos y antes de que la aplicación los use. Esta práctica también le protege frente a los datos erróneos o malintencionados que se escriben en la cuenta, ya sea mediante un usuario que adquirió correctamente la SAS o un usuario que aproveche una SAS errónea.
9. **No use siempre la SAS.** En ocasiones, los riesgos asociados a una operación determinada en la cuenta de almacenamiento superan a las ventajas del uso de la SAS. Para esas operaciones, cree un servicio de nivel medio que escriba en la cuenta de almacenamiento después de llevar a cabo una auditoría, autenticación o validación de la regla de negocio. A veces también es más sencillo administrar el acceso de otras formas. Por ejemplo, si desea que todos los blobs de un contenedor puedan leerse públicamente, puede hacer que el contenedor sea público en lugar de proporcionar un SAS a cada cliente para obtener acceso.
10. **Use el análisis de almacenamiento para supervisar la aplicación.** Puede hacer uso de registros y métricas para observar cualquier pico en los errores de autenticación producidos por la interrupción del servicio del proveedor de SAS o la eliminación involuntaria de una directiva de acceso almacenada. Consulte el [blog del equipo de almacenamiento de Azure](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx) (en inglés) para obtener más información.

## <a name="sas-examples"></a>Ejemplos de SAS
A continuación figuran algunos ejemplos de ambos tipos de firmas de acceso compartido, SAS de cuenta y SAS de servicio.

Para ejecutar estos ejemplos de C#, debe hacer referencia a los siguientes paquetes de NuGet en el proyecto:

* Versión 6.x o posterior de la [Biblioteca de cliente de Almacenamiento de azure para .NET](http://www.nuget.org/packages/WindowsAzure.Storage) (para usar la cuenta SAS).
* [Administrador de configuración Azure](http://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager)

Para obtener ejemplos adicionales que muestran cómo crear y probar SAS, vea [Ejemplos de código de Azure para Storage](https://azure.microsoft.com/documentation/samples/?service=storage).

### <a name="example-create-and-use-an-account-sas"></a>Ejemplo: Crear y utilizar una SAS de cuenta
En el ejemplo de código siguiente se crea una SAS de cuenta que es válida para los servicios Blob y Archivo y da al cliente permisos de lectura, escritura y lista para acceder a las API de nivel de servicio. La SAS de cuenta restringe el protocolo a HTTPS, por lo que la solicitud se debe realizar con HTTPS.

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use your shared key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

Para usar la SAS de cuenta a fin de acceder a las API de nivel de servicio para el servicio Blob, construya un objeto de cliente Blob con la SAS y el extremo de almacenamiento de Blob para la cuenta de almacenamiento.

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "account-name", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

### <a name="example-create-a-stored-access-policy"></a>Ejemplo: Crear una directiva de acceso almacenada
El código siguiente crea una directiva de acceso almacenada en un contenedor. Puede usar la directiva de acceso para especificar restricciones para una SAS de servicio en el contenedor o sus blobs.

```csharp
private static async Task CreateSharedAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new shared access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
        // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Delete
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

### <a name="example-create-a-service-sas-on-a-container"></a>Ejemplo: Crear una SAS de servicio en un contenedor
El código siguiente crea una SAS en un contenedor. Si se proporciona el nombre de una directiva de acceso almacenada existente, esa directiva se asocia con la SAS. Si no se proporciona ninguna directiva de acceso almacenada, el código crea una SAS ad-hoc en el contenedor.

```csharp
private static string GetContainerSasUri(CloudBlobContainer container, string storedPolicyName = null)
{
    string sasContainerToken;

    // If no stored policy is specified, create a new access policy and define its constraints.
    if (storedPolicyName == null)
    {
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocPolicy = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List
        };

        // Generate the shared access signature on the container, setting the constraints directly on the signature.
        sasContainerToken = container.GetSharedAccessSignature(adHocPolicy, null);

        Console.WriteLine("SAS for blob container (ad hoc): {0}", sasContainerToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the container. In this case, all of the constraints for the
        // shared access signature are specified on the stored access policy, which is provided by name.
        // It is also possible to specify some constraints on an ad-hoc SAS and others on the stored access policy.
        sasContainerToken = container.GetSharedAccessSignature(null, storedPolicyName);

        Console.WriteLine("SAS for blob container (stored access policy): {0}", sasContainerToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return container.Uri + sasContainerToken;
}
```

### <a name="example-create-a-service-sas-on-a-blob"></a>Ejemplo: Crear una SAS de servicio en un blob
El código siguiente crea una SAS en un blob. Si se proporciona el nombre de una directiva de acceso almacenada existente, esa directiva se asocia con la SAS. Si no se proporciona ninguna directiva de acceso almacenada, el código crea una SAS ad-hoc en el blob.

```csharp
private static string GetBlobSasUri(CloudBlobContainer container, string blobName, string policyName = null)
{
    string sasBlobToken;

    // Get a reference to a blob within the container.
    // Note that the blob may not exist yet, but a SAS can still be created for it.
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    if (policyName == null)
    {
        // Create a new access policy and define its constraints.
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocSAS = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create
        };

        // Generate the shared access signature on the blob, setting the constraints directly on the signature.
        sasBlobToken = blob.GetSharedAccessSignature(adHocSAS);

        Console.WriteLine("SAS for blob (ad hoc): {0}", sasBlobToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the blob. In this case, all of the constraints for the
        // shared access signature are specified on the container's stored access policy.
        sasBlobToken = blob.GetSharedAccessSignature(null, policyName);

        Console.WriteLine("SAS for blob (stored access policy): {0}", sasBlobToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return blob.Uri + sasBlobToken;
}
```

## <a name="conclusion"></a>Conclusión
Las firmas de acceso compartido son útiles para ofrecer permisos limitados a su cuenta de almacenamiento a clientes que no deben tener la clave de cuenta. Por ese motivo, son una parte fundamental del modelo de seguridad para cualquier aplicación que use Almacenamiento de Azure. Si sigue las prácticas recomendadas descritas aquí, puede usar la SAS para ofrecer una mayor flexibilidad de acceso a los recursos en la cuenta de almacenamiento sin que se ponga en riesgo la seguridad de la aplicación.

## <a name="next-steps"></a>Pasos siguientes
* [Firmas de acceso compartido, Parte 2: Creación y uso de una SAS con Blob Storage](../blobs/storage-dotnet-shared-access-signature-part-2.md)
* [Administración del acceso de lectura anónimo a contenedores y blobs](../blobs/storage-manage-access-to-resources.md)
* [Delegación de acceso con una firma de acceso compartido](http://msdn.microsoft.com/library/azure/ee395415.aspx)
* [Introducción a las firmas de acceso compartido de tabla y cola](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
