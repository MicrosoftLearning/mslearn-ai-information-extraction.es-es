---
lab:
  title: Creación de una solución de minería de conocimientos
  description: "Use Búsqueda de Azure\_AI para extraer información clave de los documentos y facilitar la búsqueda y el análisis."
---

# Creación de una solución de minería de conocimientos

En este ejercicio, usará Búsqueda de Azure AI para indexar un conjunto de documentos mantenidos por Margie's Travel, una agencia de viajes ficticia. El proceso de indexación implica el uso de aptitudes de inteligencia artificial para extraer información clave para que se puedan realizar búsquedas y generar un almacén de conocimiento que contenga recursos de datos para su posterior análisis.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares mediante varios SDK específicos del lenguaje; incluidos los siguientes:

- [Biblioteca cliente de Búsqueda de Azure AI para Python](https://pypi.org/project/azure-search-documents/)
- [Biblioteca cliente de Búsqueda de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.Search.Documents)
- [Biblioteca cliente de Búsqueda de Azure AI para JavaScript](https://www.npmjs.com/package/@azure/search-documents)

Este ejercicio dura aproximadamente **40** minutos.

## Creación de recursos de Azure

Para la solución que creará para Margie's Travel necesita los siguientes recursos en la suscripción de Azure. En este ejercicio, los creará directamente en Azure Portal. También puede crearlos mediante un script o una plantilla de ARM o BICEP; o bien, podría crear un proyecto de Fundición de IA de Azure que incluya un recurso de Búsqueda de Azure AI.

> **Importante**: Los recursos de Azure deben crearse en la misma ubicación.

### Creación de un recurso de Búsqueda de Azure AI

1. En un explorador web, abra [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicie sesión con las credenciales de Azure.
1. Seleccione el botón **&#65291;Crear un recurso**, busque `Azure AI Search` y cree un recurso de **Búsqueda de Azure AI** con la siguiente configuración:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Nombre del servicio**: *Un nombre válido para el recurso de búsqueda*
    - **Ubicación**: *cualquier ubicación disponible*
    - **Plan de tarifa**: Gratis

1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Revise la página **Información general** del panel del recurso de Búsqueda de Azure AI en Azure Portal. Aquí puede usar una interfaz visual para crear, probar, administrar y supervisar los distintos componentes de una solución de búsqueda, incluidos los orígenes de datos, los índices, los indexadores y los conjuntos de aptitudes.

### Cree una cuenta de almacenamiento

1. Vuelva a la página principal y, después, cree un recurso de **cuenta de almacenamiento** con la siguiente configuración:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *El mismo grupo de recursos que el de los recursos de Búsqueda de Azure AI y Servicios de Azure AI*
    - **Nombre de la cuenta de almacenamiento**: *Un nombre válido para el recurso de almacenamiento*
    - **Región**: *la misma región que el recurso de Búsqueda de Azure AI*
    - **Servicio principal**: Azure Blob Storage o Azure Data Lake Storage Gen 2
    - **Rendimiento**: Estándar
    - **Redundancia**: almacenamiento con redundancia local (LRS)

1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.

    > **Sugerencia**: Mantenga abierta la página del portal de la cuenta de almacenamiento; la usará en el siguiente procedimiento.

## Carga de documentos en Azure Storage

La solución de minería de conocimiento extraerá información de documentos de folletos de viaje en un contenedor de blobs de Azure Storage.

1. En una nueva pestaña del explorador, descargue [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) desde `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` y guárdelo en una carpeta local.
1. Extraiga el archivo *documents.zip* descargado y vea los archivos de folleto de viajes que contiene. Extraerá e indexará información de estos archivos.
1. En la pestaña del explorador que contiene la página de Azure Portal de la cuenta de almacenamiento, en el panel de navegación de la izquierda, seleccione **Explorador de almacenamiento**.
1. En el explorador de almacenamiento, seleccione **Contenedores de blob**.

    Actualmente, la cuenta de almacenamiento debe contener solo el contenedor **$logs** predeterminado.

1. En la barra de herramientas, seleccione **+ Contenedor** y cree un contenedor con la siguiente configuración:
    - **Nombre**: `documents`
    - **Nivel de acceso anónimo**: Privado (sin acceso anónimo)\*

    > **Nota**: \*A menos que haya habilitado la opción para permitir el acceso anónimo al contenedor al crear la cuenta de almacenamiento, no podrá seleccionar ningún otro valor.

1. Seleccione el contenedor **documents** para abrirlo y, después, use el **botón Cargar** de la barra de herramientas para cargar los archivos .pdf extraídos de **documents.zip** anteriormente en la raíz del contenedor, como se muestra aquí:

    ![Recorte de pantalla del explorador de Azure Storage con el contenedor de documentos y su contenido de archivos.](./media/blob-containers.png)

## Creación y ejecución de un indexador

Ahora que tiene los documentos, puede crear un indexador para extraer información de ellos.

1. En Azure Portal, navegue hasta el recurso de Búsqueda de Azure AI. A continuación, en la página **Información general**, seleccione **Importar datos**.
1. En la página **Conexión a los datos**, en la lista **Origen de datos**, seleccione **Azure Blob Storage**. A continuación, complete los detalles del almacén de datos con los valores siguientes:
    - **Origen de datos**: Azure Blob Storage
    - **Nombre de origen de datos**: `margies-documents`
    - **Datos que se extraerán**: contenido y metadatos
    - **Modo de análisis**: predeterminado
    - **Suscripción**: *suscripción a Azure*
    - **Cadena de conexión:** 
        - Seleccione **Elegir una conexión existente**
        - Selección de la cuenta de almacenamiento
        - Selección del contenedor **documents**
    - **Autenticación de identidad administrada**: ninguna
    - **Nombre del contenedor**: documents
    - **Carpeta de blobs**: *dejar en blanco*
    - **Descripción**: `Travel brochures`
1. Continúe con el paso siguiente (**Agregar aptitudes cognitivas**), que tiene tres secciones expandibles para completar.
1. En la sección **Adjuntar servicios de Azure AI**, seleccione **Gratis (enriquecimientos limitados**).\*

    > **Nota**: \*El recurso gratuito de Servicios de Azure AI para Búsqueda de Azure AI se puede usar para indexar un máximo de 20 documentos. En una solución real, debe crear un recurso de Servicios de Azure AI en la suscripción a fin de habilitar el enriquecimiento con IA para un mayor número de documentos.

1. En la sección **Agregar enriquecimientos**, haga lo siguiente:
    - Cambia el **Nombre del conjunto de aptitudes** por `margies-skillset`.
    - Seleccione la opción **Habilitar OCR y combinar todo el texto en el campo merged_content**.
    - Asegúrese de que el **Campo de datos de origen** está establecido en **merged_content**.
    - Deje el **Nivel de granularidad de enriquecimiento** como **Campo de origen**, que establece todo el contenido del documento que se va a indexar; pero tenga en cuenta que puede cambiar esto para extraer información en niveles más detallados, como páginas u oraciones.
    - Seleccione los siguientes campos enriquecidos:

        | Conocimiento cognitivo | Parámetro | Nombre del campo |
        | --------------- | ---------- | ---------- |
        | **Aptitudes cognitivas de texto** | |  |
        | Extraer nombres de personas | | people |
        | Extracción de nombres de ubicaciones | | locations |
        | Extracción de frases clave | | keyphrases |
        | **Aptitudes cognitivas de imagen** | |  |
        | Generación de etiquetas a partir de imágenes | | imageTags |
        | Generación de descripciones a partir de imágenes | | imageCaption |

        Compruebe las selecciones (puede ser difícil cambiarlas más adelante).

1. En la sección **Guardar enriquecimientos en un almacén de conocimientos**:
    - Active solo las siguientes casillas (se mostrará un <font color="red">error</font>, que resolverá en breve):
        - **Proyecciones de archivos de Azure**:
            - Image projections (Proyecciones de la imagen)
        - **Proyecciones de tablas de Azure**:
            - Documentos
                - Frases clave
        - **Proyecciones de blobs de Azure**:
            - Document
    - En **Cadena de conexión de la cuenta de almacenamiento** (debajo de los <font color="red">mensajes de error</font>):
        - Seleccione **Elegir una conexión existente**
        - Selección de la cuenta de almacenamiento
        - Seleccione el contenedor **documents** (*esto solo es necesario para seleccionar la cuenta de almacenamiento en la interfaz de exploración; especificará otro nombre de contenedor para los recursos de conocimiento extraídos*).
    - Cambie el **Nombre del contenedor** a `knowledge-store`.
1. Continúe con el paso siguiente (**Personalizar el índice de destino**), donde especificará los campos para el índice. 
1. Cambie el **Nombre del índice** a `margies-index`.
1. Asegúrese de que el valor de **Clave** está establecido en **metadata_storage_path**, deje en blanco el campo **Nombre del proveedor de sugerencias** y asegúrese de que el campo **Modo de búsqueda** sea **analyzingInfixMatching**.
1. Realice los siguientes cambios en los campos de índice y deje todos los demás campos con su configuración predeterminada (**IMPORTANTE**: es posible que tenga que desplazarse a la derecha para ver toda la tabla):

    | Nombre del campo | Retrievable | Filtrable | Ordenable | Clasificable | Buscable |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | people | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    Compruebe las selecciones y preste especial atención a que las opciones **Recuperable**, **Filtrable**, **Ordenable**, **Clasificable** y **Buscable** estén seleccionadas correctamente para cada campo (puede ser difícil cambiarlas más adelante).

1. Continúe con el paso siguiente (**Crear un indexador**), donde creará y programará el indexador.
1. Cambie el **Nombre del indexador** a `margies-indexer`.
1. Deje la **Programación** establecida en **Una vez**.
1. Selecciona **Enviar** para crear un origen de datos, un conjunto de aptitudes, un índice y un indexador. El indexador se ejecuta automáticamente y ejecuta la canalización de indexación, que hace lo siguiente:
    - Extrae los campos de metadatos del documento y el contenido del origen de datos.
    - Ejecuta el conjunto de aptitudes cognitivas para generar campos enriquecidos adicionales.
    - Asigna los campos extraídos al índice.
    - Guarda los recursos de datos extraídos en el almacén de conocimiento.
1. En el panel de navegación de la izquierda, en **Administración de búsqueda**, vea la página **Indexadores**, en la que se debe mostrar la instancia de **margies-indexer** recién creada. Espere unos minutos y haga clic en **&orarr; Actualizar** hasta que el campo **Estado** indique **Correcto**.

## Búsqueda en el índice

Ahora que tienes un índice, puedes realizar búsquedas en él.

1. Vuelva a la página **Información general** del recurso de Búsqueda de Azure AI y, en la barra de herramientas, seleccione **Explorador de búsqueda**.
1. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escribe `*` (un solo asterisco) y, a continuación, selecciona **Buscar**.

    Esta consulta recupera todos los documentos del índice en formato JSON. Examina los resultados y fíjate en los campos de cada documento, que incluyen el contenido, los metadatos y los datos enriquecidos del documento extraídos mediante las aptitudes cognitivas seleccionadas.

1. En el menú **Ver**, selecciona **Vista JSON** y observa que se muestra la solicitud JSON para la búsqueda, de la siguiente manera:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Los resultados incluyen un campo **@odata.count** en la parte superior que indica el número de documentos devueltos por la búsqueda.

1. Modifique la solicitud JSON para incluir el parámetro **select** como se muestra aquí:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    Esta vez, los resultados incluyen solo el nombre de archivo y las ubicaciones mencionadas en el contenido del documento. El nombre de archivo está en el campo **metadata_storage_name**, que se ha extraído del documento de origen. El campo **locations** se ha generado mediante una aptitud de IA.

1. Ahora prueba la siguiente cadena de consulta:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Con esta búsqueda se buscan documentos que mencionen "New York" en cualquiera de los campos en los que se pueden realizar búsquedas y devuelve el nombre de archivo y las frases clave del documento.

1. Vamos a probar una consulta más:

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    Esta consulta devuelve el nombre de archivo y las frases clave de los documentos que mencionan "Nueva York" que tienen un tamaño inferior a 380 000 bytes.

## Creación de una aplicación cliente de búsqueda

Ahora que tiene un índice útil, puede usarlo desde una aplicación cliente. Para ello, puede consumir la interfaz de REST, enviar solicitudes y recibir respuestas en formato JSON mediante HTTP. También puede usar el kit de desarrollo de software (SDK) para su lenguaje de programación preferido. En este ejercicio, usaremos el SDK.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

### Obtención del punto de conexión y las claves del recurso de búsqueda

1. En Azure Portal. cierre la página del explorador de búsqueda y vuelva a la página **Información general** del servicio recurso de Búsqueda de Azure AI.

    Anote el valor **Url**, que debe ser similar a **https://*nombre_del_recurso*.search.windows.net**. Este es el punto de conexión del recurso de búsqueda.

1. En el panel de navegación de la izquierda, expanda **Configuración** y vea la **página Claves**.

    Observe que hay dos claves de **administración** y una sola de **consulta**. Una clave de *administración* se usa para crear y administrar los recursos de búsqueda. En cambio, las aplicaciones cliente que solo necesitan realizar consultas de búsqueda utilizan una clave de *consulta*.

    *Necesitará la clave de **consulta** y el punto de conexión para la aplicación cliente.*

### Preparación para el uso del SDK de Búsqueda de Azure AI

1. Use el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de Azure Portal para crear una instancia de Cloud Shell en Azure Portal; para ello, seleccione un entorno de ***PowerShell*** sin almacenamiento en la suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo. Inicialmente, deberá ver Cloud Shell y Azure Portal (para que pueda buscar y copiar el punto de conexión y la clave que necesitará).

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. Ejecute los siguientes comandos para instalar el SDK de Búsqueda de Azure AI y los paquetes de identidad de Azure:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. Ejecute el comando siguiente para editar el archivo de configuración de la aplicación:

    ```
   code .env
    ```

    El archivo de configuración se abre en un editor de código.

1. Edite el archivo de configuración para reemplazar los valores de marcador de posición siguientes:

    - **your_search_endpoint** (*debe reemplazarlo por el recurso de Búsqueda de Azure AI*)
    - **your_query_key** *(debe reemplazarlo por la clave de consulta del recurso de Búsqueda de Azure AI*)
    - **your_index_name** (*debe reemplazarlo por el nombre del índice que debe ser `margies-index`*)

1. Cuando haya actualizado los marcadores de posición, use el comando **CTRL+S** para guardar el archivo y después el comando **CTRL+Q** para cerrarlo.

    > **Sugerencia**: Ahora que ha copiado el punto de conexión y la clave de Azure Portal, es posible que quiera maximizar el panel de Cloud Shell para que sea más fácil trabajar.

1. Ejecute el comando siguiente para abrir el archivo de código de la aplicación:

    ```
   code search-app.py
    ```

    El archivo de código se abre en un editor de código.

1. Revise el código y observe que realiza las siguientes acciones:

    - Recupera los valores de configuración del recurso y el índice de Búsqueda de Azure AI del archivo de configuración que ha editado.
    - Crea una instancia de **SearchClient** con el punto de conexión, la clave y el nombre del índice para conectarse al servicio de búsqueda.
    - Solicita al usuario una consulta de búsqueda (hasta que escriba "salir")
    - Busca en el índice mediante la consulta y devuelve los siguientes campos (ordenados por metadata_storage_name):
        - metadata_storage_name
        - locations
        - people
        - keyphrases
    - Analiza los resultados de la búsqueda que se devuelven para mostrar los campos devueltos para cada documento del conjunto de resultados.

1. Cierre el panel del editor de código (*CTRL+Q*) y mantenga abierto el panel de la consola de línea de comandos de Cloud Shell
1. Escriba el siguiente comando para ejecutar la aplicación:

    ```
   python search-app.py
    ```

1. Cuando se le solicite, escriba una consulta como `London` y vea los resultados.
1. Pruebe otra consulta, como `flights`.
1. Cuando haya terminado de probar la aplicación, escriba `quit` para cerrarla.
1. Cierre Cloud Shell para volver a Azure Portal.

## Visualización del almacén de conocimiento

Después de ejecutar un indexador que usa un conjunto de aptitudes para crear un almacén de conocimiento, los datos enriquecidos extraídos mediante el proceso de indexación se conservan en las proyecciones del almacén de conocimiento.

### Visualización de las proyecciones de objetos

Las proyecciones de *objetos* definidas en el conjunto de aptitudes de Margie's Travel contienen un archivo JSON para cada documento indexado. Estos archivos se almacenan en un contenedor de blobs en la cuenta de Azure Storage especificada en la definición del conjunto de aptitudes.

1. En la Azure Portal, visualice la cuenta de Azure Storage que creó anteriormente.
1. Seleccione la pestaña **Explorador de Storage** (en el panel de la izquierda) para ver la cuenta de almacenamiento en la interfaz del explorador de almacenamiento en Azure Portal.
1. Expanda **Contenedores de blobs** para ver los contenedores de la cuenta de almacenamiento. Además del contenedor **documents** en el que se almacenan los datos de origen, debería haber dos nuevos contenedores: **knowledge-store** y **margies-skillset-image-projection**. Estos contenedores se han creado mediante el proceso de indexación.
1. Seleccione el contenedor **knowledge-store**. Debe contener una carpeta para cada documento indexado.
1. Abra cualquiera de las carpetas y, después, seleccione el archivo **objectprojection.json** que contiene y use el botón **Descargar** de la barra de herramientas para descargarlo y abrirlo. Cada archivo JSON contiene una representación de un documento indexado, incluidos los datos enriquecidos extraídos por el conjunto de aptitudes como se muestran aquí (con formato para facilitar la lectura).

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

La capacidad de crear proyecciones de *objetos* como esta le permite generar objetos de datos enriquecidos que se pueden incorporar en una solución de análisis de datos empresariales.

### Visualización de las proyecciones de archivos

Las proyecciones de *archivos* definidas en el conjunto de aptitudes crean archivos JPEG para cada imagen extraída de los documentos durante el proceso de indexación.

1. En la interfaz del *explorador de almacenamiento* en Azure Portal, seleccione el contenedor de blobs **margies-skillset-image-projection**. Este contenedor contiene una carpeta para cada documento que incluye imágenes.
2. Abra cualquiera de las carpetas y vea su contenido (cada una contiene al menos un archivo \*.jpg).
3. Abra cualquiera de los archivos de imagen y descárguelo y ábralo para ver la imagen.

Esta capacidad de generar proyecciones de *archivos* hace que la indexación sea una manera eficaz para extraer imágenes insertadas de un gran volumen de documentos.

### Visualización de las proyecciones de tablas

Las proyecciones de *tablas* definidas en el conjunto de aptitudes forman un esquema relacional de datos enriquecidos.

1. En la interfaz del *explorador de almacenamiento* en Azure Portal, expanda **Tablas**.
2. Seleccione la tabla **margiesSkillsetDocument** para ver los datos. Esta tabla contiene una fila para cada documento que se ha indexado:
3. Vea la tabla **margiesSkillsetKeyPhrases**, que contiene una fila para cada frase clave extraída de los documentos.

La capacidad de crear proyecciones de *tablas* le permite compilar soluciones de análisis y generación de informes que consultan un esquema relacional. Las columnas de clave generadas automáticamente se pueden usar para combinar las tablas en las consultas; por ejemplo, para devolver todas las frases clave extraídas de un documento específico.

## Limpieza

Ahora que ha completado el ejercicio, elimine todos los recursos que ya no necesita. Eliminación de los recursos de Azure:

1. En **Azure Portal**, seleccione Grupos de recursos.
1. Seleccione el grupo de recursos que no necesite y luego **Eliminar grupo de recursos**.

## Más información

Para obtener más información sobre Búsqueda de Azure AI, consulte la [documentación de Búsqueda de Azure AI](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
