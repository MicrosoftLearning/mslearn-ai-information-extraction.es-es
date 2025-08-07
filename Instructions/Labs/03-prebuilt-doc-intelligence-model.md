---
lab:
  title: Análisis de formularios con modelos de Documento de inteligencia de Azure AI precompilados
  description: "Use modelos de Documento de inteligencia de Azure\_AI creados previamente para procesar campos de texto de documentos."
---

# Análisis de formularios con modelos de Documento de inteligencia de Azure AI precompilados

En este ejercicio, configurará un proyecto de Fundición de IA de Azure con todos los recursos necesarios para el análisis de documentos. Usará tanto el Portal de la Fundición de IA de Azure como el SDK de Python para enviar formularios a ese recurso para su análisis.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares mediante varios SDK específicos del lenguaje; incluidos los siguientes:

- [Biblioteca cliente de Documento de inteligencia de Azure AI para Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Biblioteca cliente de Documento de inteligencia de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Biblioteca cliente de Documento de inteligencia de Azure AI para JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Este ejercicio dura aproximadamente **30** minutos.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./media/ai-foundry-home.png)

1. En el explorador, accede a `https://ai.azure.com/managementCenter/allResources` y selecciona **Crear nuevo**. A continuación, elige la opción para crear un nuevo **recurso del centro de IA**.
1. En el asistente para **crear un proyecto**, escribe un nombre válido para tu proyecto y selecciona la opción para crear un centro. A continuación, usa el vínculo **Cambiar nombre del centro** para especificar un nombre válido para el nuevo centro, expande **Opciones avanzadas** y especifica la siguiente configuración para el proyecto:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**:  *Cualquier región disponible*

    > **Nota**: Si trabajas en una suscripción a Azure en la que se usan directivas para restringir los nombres de recursos permitidos, es posible que tengas que usar el vínculo situado en la parte inferior del cuadro de diálogo **Crear un nuevo proyecto** para crear el centro con Azure Portal.

    > **Sugerencia**: si el botón **Crear** sigue deshabilitado, asegúrate de cambiar el nombre del centro a un valor alfanumérico único.

1. Espera a que se cree el proyecto.
1. Cuando se cree el proyecto, cierra las sugerencias que se muestran y revisa la página del proyecto en el Portal de la Fundición de IA de Azure, que debe tener un aspecto similar a la siguiente imagen:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](./media/ai-foundry-project.png)

## Uso del modelo de lectura

Para empezar, se usará el portal de la **Fundición de IA de Azure** y el modelo de lectura para analizar un documento con varios idiomas:

1. En el panel de navegación de la izquierda, seleccione **Servicios de IA**.
1. En la página **Servicios de Azure AI**, seleccione el icono **Visión y documento**.
1. En la página **Visión y documento**, compruebe que la pestaña **Documento** está seleccionada y, después, seleccione el icono **OCR/Lectura**.

    En la página **Lectura**, el recurso de Servicios de Azure AI creado con el proyecto ya debe estar conectado.

1. En la lista de documentos de la izquierda, seleccione **read-german.pdf**.

    ![Captura de pantalla en la que se muestra la página Lectura de Documento de inteligencia de Azure AI Studio.](./media/read-german-sample.png#lightbox)

1. En la barra de herramientas superior, seleccione **Opciones de análisis**, después active la casilla **Idioma** (en **Detección opcional**) en el panel **Opciones de análisis** y seleccione **Guardar**. 
1. En la parte superior izquierda, seleccione **Ejecutar análisis**.
1. Una vez completado el análisis, el texto extraído de la imagen se muestra a la derecha en la pestaña **Contenido**. Revise este texto y compárelo con el texto de la imagen original para comprobar su exactitud.
1. Seleccione la pestaña **Resultado**. Esta pestaña muestra el código JSON extraído. 

## Preparación para desarrollar una aplicación en Cloud Shell

Ahora vamos a explorar la aplicación que usa el SDK del servicio Documento de inteligencia de Azure. Desarrollará la aplicación mediante Cloud Shell. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

Esta es la factura que analizará el código.

![Captura de pantalla que muestra un documento de factura de ejemplo.](./media/sample-invoice.png)

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Puntos de conexión y claves**, seleccione la pestaña **Servicios de Azure AI** y anote la **Clave de API** y el **Punto de conexión de Servicios de Azure AI**. Usará estas credenciales para conectarse a los servicios de Azure AI en una aplicación cliente.
1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.
1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

    ***Ahora sigue los pasos del lenguaje de programación que hayas elegido.***

1. Una vez que se haya clonado el repositorio, vaya a la carpeta que contiene los archivos de código:

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace los marcadores de posición **YOUR_ENDPOINT** y **YOUR_KEY** por el punto de conexión de Servicios de Azure AI y su clave de API (copiada del Portal de la Fundición de IA de Azure).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, después, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Agregar código para usar el servicio Documento de inteligencia de Azure.

Ahora está listo para usar el SDK para evaluar el archivo PDF.

1. Escriba el siguiente comando para editar el archivo de aplicación que se ha proporcionado:

    ```
   code document-analysis.py
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, busque el comentario **Importar las bibliotecas necesarias** y agregue el código siguiente:

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. Busque el comentario **Crear el cliente** y agregue el código siguiente (preste atención para mantener el nivel de sangría correcto):

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. Busque el comentario **Analizar la factura** y agregue el código siguiente:

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. Busque el comentario **Mostrar información de factura al usuario** y agregue el código siguiente:

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. En el editor de código, use el comando **CTRL+S** o **haga clic con el botón derecho > Guardar** para guardar los cambios. Mantenga abierto el editor de código en caso de que necesite corregir cualquier error en el código, pero cambie el tamaño de los paneles para que pueda ver el panel de la línea de comandos con claridad.

1. En el panel de línea de comandos, escriba el siguiente comando para ejecutar la aplicación.

    ```
    python document-analysis.py
    ```

El programa muestra el nombre del proveedor, el nombre del cliente y el total de la factura con niveles de confianza. Compare los valores que notifica con la factura de ejemplo que abrió al principio de esta sección.

## Limpieza

Si ha terminado de usar el recurso de Azure, recuerde eliminarlo en [Azure Portal](https://portal.azure.com) (`https://portal.azure.com`) para evitar cargos adicionales.
