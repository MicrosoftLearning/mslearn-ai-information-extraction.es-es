---
lab:
  title: Desarrollo de una aplicación cliente de comprensión de contenidos
  description: Use la API REST de Servicio de comprensión de contenido de IA de Azure a fin de desarrollar una aplicación cliente para un analizador.
---

# Desarrollo de una aplicación cliente de comprensión de contenidos

En este ejercicio, usará Servicio de comprensión de contenido de IA de Azure para crear un analizador que extraiga información de tarjetas de presentación. Después, desarrollará una aplicación cliente que use el analizador para extraer los detalles de contacto de las tarjetas de presentación escaneadas.

Este ejercicio dura aproximadamente **30** minutos.

## Creación de un centro y un proyecto de Fundición de IA de Azure

Las características de Fundición de IA de Azure que usaremos en este ejercicio requieren un proyecto basado en un recurso del *centro* de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./media/ai-foundry-home.png)

1. En el explorador, accede a `https://ai.azure.com/managementCenter/allResources` y selecciona **Crear nuevo**. A continuación, elige la opción para crear un nuevo **recurso del centro de IA**.
1. En el asistente para **crear un proyecto**, escribe un nombre válido para tu proyecto y selecciona la opción para crear un centro. A continuación, usa el vínculo **Cambiar nombre del centro** para especificar un nombre válido para el nuevo centro, expande **Opciones avanzadas** y especifica la siguiente configuración para el proyecto:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Nombre del centro**: un nombre válido para el centro
    - **Ubicación**: Elija una de las ubicaciones siguientes:\*
        - Este de Australia
        - Centro de Suecia
        - Oeste de EE. UU.

    > \*En el momento de escribir este documento, Servicio de comprensión de contenido de IA de Azure solo está disponible en estas regiones.

    > **Sugerencia**: si el botón **Crear** sigue deshabilitado, asegúrate de cambiar el nombre del centro a un valor alfanumérico único.

1. Espere a que se cree el proyecto y, después, vaya a su página de información general.

## Uso de la API REST para crear un analizador de Servicio de comprensión de contenido

Va a usar la API REST para crear un analizador que pueda extraer información de imágenes de tarjetas de presentación.

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Sugerencia**: Cambie el tamaño del panel para que pueda trabajar principalmente en Cloud Shell, pero seguir viendo las claves y el punto de conexión en la página de Azure Portal; deberá copiarlas en el código.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de la aplicación:

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    La carpeta contiene dos imágenes de tarjeta de presentación escaneadas, así como los archivos de código de Python que necesita para compilar la aplicación.

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace los marcadores de posición **YOUR_ENDPOINT** y **YOUR_KEY** por el punto de conexión de los servicios de Azure AI y cualquiera de sus claves (copiadas de Azure Portal) y asegúrese de que **ANALYZER_NAME** esté establecido en `business-card-analyzer`.
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, después, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

    > **Sugerencia**: Ya puede maximizar el panel de Cloud Shell.

1. En la línea de comandos de Cloud Shell, escriba el siguiente comando para ver el archivo JSON **biz-card.json** que se ha proporcionado:

    ```
   cat biz-card.json
    ```

    Desplácese por el panel de Cloud Shell para ver el código JSON del archivo, que define un esquema de analizador para una tarjeta de presentación.

1. Cuando haya visto el archivo JSON del analizador, escriba el siguiente comando para editar el archivo de código de Python **create-analyzer.py** que se ha proporcionado:

    ```
   code create-analyzer.py
    ```

    El archivo de código de Python se abre en un editor de código.

1. Revisa el código, que:
    - Carga el esquema del analizador desde el archivo **biz-card.json**.
    - Recupera el punto de conexión, la clave y el nombre del analizador del archivo de configuración del entorno.
    - Llama a una función denominada **create_analyzer**, que actualmente no está implementada

1. En la función **create_analyzer**, busque el comentario **Crear un analizador de comprensión de contenidos** y agregue el código siguiente (preste atención para mantener la sangría correcta):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. Revise el código que ha agregado, que:
    - Crea encabezados adecuados para las solicitudes REST
    - Envía una solicitud HTTP *DELETE* para eliminar el analizador si ya existe.
    - Envía una solicitud HTTP *PUT* para iniciar la creación del analizador.
    - Comprueba la respuesta para recuperar la dirección URL de devolución de llamada de *Operation-Location*.
    - Envía repetidamente una solicitud HTTP *GET* a la dirección URL de devolución de llamada para comprobar el estado de la operación hasta que ya no se ejecute.
    - Confirma al usuario si la operación ha sido correcta (o se ha producido un error).

    > **Nota**: El código incluye retrasos deliberados en el tiempo para evitar superar el límite de frecuencia de solicitudes para el servicio.

1. Use el comando **CTRL+S** para guardar los cambios en el código, pero mantenga abierto el panel del editor de código en caso de que tenga que corregir errores en el código. Cambie el tamaño de los paneles para que pueda ver claramente el panel de línea de comandos.
1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar el código Python:

    ```
   python create-analyzer.py
    ```

1. Revise la salida del programa, que debería indicar que se ha creado el analizador.

## Uso de la API REST para analizar contenido

Ahora que has creado un analizador, puedes consumirlo desde una aplicación cliente a través de la API de REST de comprensión de contenido.

1. En la línea de comandos de Cloud Shell, escriba el siguiente comando para editar el archivo de código de Python **read-card.py** que se ha proporcionado:

    ```
   code read-card.py
    ```

    El archivo de código de Python se abre en un editor de código:

1. Revisa el código, que:
    - Identifica el archivo de imagen que se va a analizar, con un valor predeterminado de **biz-card-1.png**.
    - Recupera el punto de conexión y la clave del recurso de Servicios de Azure AI del proyecto (mediante las credenciales de Azure de la sesión actual de Cloud Shell para autenticarse).
    - Llama a una función denominada **analyze_card**, que actualmente no está implementada

1. En la función **analyze_card**, busque el comentario **Usar Servicio de comprensión de contenido para analizar la imagen** y agregue el código siguiente (preste atención para mantener la sangría correcta):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. Revise el código que ha agregado, que:
    - Lee el contenido del archivo de imagen
    - Establece la versión de la API REST de comprensión de contenidos que se va a usar
    - Envía una solicitud HTTP *POST* al punto de conexión del Servicio de comprensión de contenido para indicarle que analice la imagen.
    - Comprueba la respuesta de la operación a fin de recuperar un id. para la operación de análisis.
    - Envía repetidamente una solicitud HTTP *GET* al Servicio de comprensión de contenido para comprobar el estado de la operación hasta que ya no se ejecute.
    - Si la operación se ha realizado correctamente, guarda la respuesta JSON y, después, analiza el código JSON y muestra los valores recuperados para cada tipo de campo concreto.

    > **Nota**: En este sencillo esquema de tarjeta de presentación, todos los campos son cadenas. Este código demuestra la necesidad de comprobar el tipo de cada campo para que pueda extraer valores de diferentes tipos de un esquema más complejo.

1. Use el comando **CTRL+S** para guardar los cambios en el código, pero mantenga abierto el panel del editor de código en caso de que tenga que corregir errores en el código. Cambie el tamaño de los paneles para que pueda ver claramente el panel de línea de comandos.
1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar el código Python:

    ```
   python read-card.py biz-card-1.png
    ```

1. Revise la salida del programa, que debe mostrar los valores de los campos de la siguiente tarjeta de presentación:

    ![Una tarjeta de presentación para Roberto Tamburello, un empleado de Adventure Works Cycles.](./media/biz-card-1.png)

1. Use el siguiente comando para ejecutar el programa con otra tarjeta de presentación:

    ```
   python read-card.py biz-card-2.png
    ```

1. Revise los resultados, que deben reflejar los valores de esta tarjeta de presentación:

    ![Una tarjeta de presentación para Mary Duartes, una empleada de Contoso.](./media/biz-card-2.png)

1. En el panel de línea de comandos de Cloud Shell, use el siguiente comando para ver la respuesta JSON completa que se ha devuelto:

    ```
   cat results.json
    ```

    Desplácese para ver el código JSON.

## Limpieza

Si has terminado de trabajar con el servicio de comprensión de contenidos, deberías eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. En Azure Portal, elimine los recursos que ha creado en este ejercicio.
