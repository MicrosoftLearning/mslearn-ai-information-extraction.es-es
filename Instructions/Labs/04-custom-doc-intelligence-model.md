---
lab:
  title: Análisis de formularios con modelos de Documento de inteligencia de Azure AI personalizados
  description: Cree un modelo personalizado de Documento de inteligencia para extraer datos específicos de documentos.
---

# Análisis de formularios con modelos de Documento de inteligencia de Azure AI personalizados

Imagine que, en una empresa, los empleados actualmente deben comprar manualmente hojas de pedidos y escribir los datos en una base de datos. Les gustaría usar servicios de inteligencia artificial para mejorar el proceso de entrada de datos. Usted decide crear un modelo de aprendizaje automático que lea el formulario y genere datos estructurados que se puedan usar para actualizar automáticamente una base de datos.

**Documento de inteligencia de Azure AI** es un servicio de Azure AI que permite a los usuarios compilar un software de procesamiento de datos automatizado. Este software puede extraer texto, pares clave-valor y tablas de documentos de formulario mediante el reconocimiento óptico de caracteres (OCR). Documento de inteligencia de Azure AI tiene modelos precompilados para reconocer facturas, recibos y tarjetas de presentación. El servicio también proporciona la capacidad de entrenar modelos personalizados. En este ejercicio, nos centraremos en la creación de modelos personalizados.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares mediante varios SDK específicos del lenguaje; incluidos los siguientes:

- [Biblioteca cliente de Documento de inteligencia de Azure AI para Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Biblioteca cliente de Documento de inteligencia de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Biblioteca cliente de Documento de inteligencia de Azure AI para JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Este ejercicio dura aproximadamente **30** minutos.

## Crear un recurso de Documento de inteligencia de Azure AI

Para usar el servicio Documento de inteligencia de Azure AI, necesita un recurso de Documento de inteligencia de Azure AI o de Servicios de Azure AI en su suscripción de Azure. Usará Azure Portal para crear un recurso.

1. En una pestaña del explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En la página principal de Azure Portal, navegue al cuadro de búsqueda superior, escriba **Documento de inteligencia** y presione **Entrar**.
1. En la página **Documento de inteligencia** seleccione **Crear**.
1. En la página **Crear Documento de inteligencia**, cree un recurso con la siguiente configuración:
    - **Suscripción**: Su suscripción de Azure.
    - **Grupo de recursos**: crea o selecciona un grupo de recursos
    - **Región**: Cualquier región disponible
    - **Nombre**: Un nombre válido para el recurso de Documento de inteligencia
    - **Plan de tarifa**: Gratis F0 (*si no tiene un nivel Gratis disponible, seleccione * Estándar S0).
1. Cuando se complete la implementación, seleccione **Ir al recurso** para ver la página **Información general** del recurso.

## Preparación para desarrollar una aplicación en Cloud Shell

Desarrollará la aplicación de traducción de texto mediante Cloud Shell. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. Cambie el tamaño del panel de Cloud Shell para que pueda ver tanto la consola de la línea de comandos como Azure Portal. Deberá usar la barra dividida para cambiar a medida que cambie entre los dos paneles.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## Recopilación de documentos para el entrenamiento

Usará los formularios de ejemplo como este para entrenar un modelo de prueba: 

![Imagen de una factura usada en este proyecto.](./media/Form_1.jpg)

1. En la línea de comandos, ejecute `ls ./sample-forms` para enumerar el contenido de la carpeta **sample-forms**. Observe que hay archivos que terminan en **.json** y **.jpg** en la carpeta.

    Usará los archivos **.jpg** para entrenar el modelo.  

    Los archivos **.json** se han generado automáticamente y contienen información de etiqueta. Los archivos se cargarán en el contenedor de Blob Storage junto con los formularios.

1. En **Azure Portal**, vaya a la página **Información general** del recurso si aún no está allí. En la sección *Essentials*, anote el **grupo de recursos**, el **identificador de la suscripción** y la **ubicación**. Necesitará estos valores en los pasos siguientes.
1. Ejecute el comando `code setup.sh` para abrir **setup.sh** en un editor de código. Usará este script por para ejecutar los comandos de la interfaz de la línea de comandos (CLI) de Azure necesarios para crear los demás recursos de Azure que necesita.

1. En el script **setup.sh**, revise los comandos. El programa:
    - Creará una cuenta de almacenamiento en el grupo de recursos de Azure.
    - Cargará archivos de la carpeta local *sampleforms* a un contenedor denominado *sampleforms* en la cuenta de almacenamiento
    - Imprimirá un URI de firma de acceso compartido

1. Modificará las declaraciones de las variables **subscription_id**, **resource_group** y **location** con los valores adecuados para la suscripción, el grupo de recursos y el nombre de ubicación donde implementó el recurso de Documento de inteligencia.

    > **Importante**: En el caso de la cadena **location**, asegúrese de usar la versión de código de la ubicación. Por ejemplo, si la ubicación es "Este de EE. UU.", la cadena del script debe ser `eastus`. Puede ver que la versión es el botón **Vista JSON** del lado derecho de la pestaña **Essentials** del grupo de recursos en Azure Portal.

    Si la variable **expiry_date** está en el pasado, actualícela a una fecha futura. Esta variable se usa al generar el URI de firma de acceso compartido (SAS). En la práctica, establecerá una fecha de expiración adecuada para la SAS. Puede obtener más información sobre SAS [aquí](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

1. Escriba los siguientes comandos para que el script sea ejecutable y ejecútelo:

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. Cuando se complete el script, revise la salida mostrada.

1. En Azure Portal, actualice el grupo de recursos y compruebe que contiene la cuenta de Azure Storage que acaba de crear. Abra la cuenta de almacenamiento y, en el panel de la izquierda, seleccione **Explorador de almacenamiento** . Después, en el explorador de almacenamiento, expanda **Contenedores de blobs** y seleccione el contenedor **sampleforms** para comprobar que los archivos se hayan cargado desde la carpeta local **custom-doc-intelligence/sample-forms**.

## Entrene el modelo mediante Document Intelligence Studio

Ahora entrenará el modelo mediante los archivos cargados en la cuenta de almacenamiento.

1. Abra una nueva pestaña del explorador y vaya a Estudio de Documento de inteligencia en `https://documentintelligence.ai.azure.com/studio`. 
1. Desplácese hacia abajo hasta la sección **Modelos personalizados** y seleccione el icono **Modelo de extracción personalizado**.
1. Si se le solicita, inicie sesión con las credenciales de Azure.
1. Si se le pregunta qué recurso de Documento de inteligencia de Azure AI va a usar, seleccione la suscripción y el nombre del recurso que usó al crear el recurso de Documento de inteligencia de Azure AI.
1. En **Mis proyectos**, cree un proyecto con la siguiente configuración:

    - **Escriba los detalles del proyecto**:
        - **Project name** (Nombre del proyecto): Un nombre válido para el proyecto
    - **Configurar recurso de servicio**:
        - **Suscripción** : su suscripción a Azure.
        - **Grupo de recursos**: El grupo de recursos donde ha implementado el recurso de Documento de inteligencia
        - **Recurso de Documento de inteligencia** El recurso de Documento de inteligencia (seleccione la opción *Establecer como predeterminado* y use la versión predeterminada de la API)
    - **Conectar origen de datos de entrenamiento**:
        - **Suscripción** : su suscripción a Azure.
        - **Grupo de recursos**: El grupo de recursos donde ha implementado el recurso de Documento de inteligencia
        - **Cuenta de almacenamiento**: la cuenta de almacenamiento que ha creado el script de instalación (seleccione la opción *Establecer como predeterminado*, seleccione el contenedor de blobs `sampleforms` y deje en blanco la ruta de acceso de la carpeta)

1. Una vez que se cree el proyecto, en la parte superior derecha de la pantalla, seleccione **Entrenar** para entrenar el modelo. Use la siguiente configuración:
    - **Id. de modelo**: un nombre válido para el modelo (*necesitará el nombre del id. de modelo en el paso siguiente*)
    - **Compilar modo**: plantilla.
1. Selecciona **Go to Models**.
1. El entrenamiento puede tardar algún tiempo. Espere hasta que el estado sea **correcto**.

## Pruebe el modelo personalizado de Documento de inteligencia

1. Vuelva a la pestaña del explorador que contiene Azure Portal y Cloud Shell. En la línea de comandos, ejecute el siguiente comando para cambiar a la carpeta que contiene los archivos de código de la aplicación:

    ```
    cd Python
    ```

1. Ejecute el comando siguiente para instalar el paquete de Documento de inteligencia:

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

1. En el panel que contiene Azure Portal, en la página **Información general** del recurso de Documento de inteligencia, seleccione **Haga clic aquí para administrar las claves** a fin de ver el punto de conexión y las claves del recurso. Después, edite el archivo de configuración con los valores siguientes:
    - El punto de conexión de Documento de inteligencia
    - La clave de Documento de inteligencia
    - Id, de modelo que ha especificado al entrenar el modelo

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, después, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

1. Abra el archivo de código de la aplicación cliente (`code Program.cs` para C#, `code test-model.py` para Python) y revise el código que contiene, especialmente que la imagen de la dirección URL haga referencia al archivo de este repositorio de GitHub en la web. Cierre el archivo sin realizar ningún cambio.

1. Escriba el siguiente comando en la línea de comandos para ejecutar el programa:

    ```
   python test-model.py
    ```

1. Vea la salida y observe cómo la salida del modelo proporciona nombres de campo como `Merchant` y `CompanyPhoneNumber`.

## Limpiar

Si ha terminado de usar el recurso de Azure, recuerde eliminar el recurso en [Azure Portal](https://portal.azure.com/?azure-portal=true) para evitar cambios.

## Más información

Para obtener más información sobre el servicio Documento de inteligencia, consulte la [documentación de Documento de inteligencia](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).
