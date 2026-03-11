# Prueba Técnica - Desarrollo RPA con  PIX RPA 

Este proyecto es una solución RPA para el proceso de **análisis de productos disponibles en una tienda online** desarrollado en **PIX Studio**. 

Desarrollado en la plantilla universal propuesta por PIX para un manejo corporativo del proceso, el diseño de cada módulo cuenta con flujos robustos en donde se tiene en cuenta reintentos y manejo de errores.  

El video de ejecución se puede ver en la ruta https://drive.google.com/file/d/1-3FvFG0ExvIt25knplIKpHphFkfp8HIz/view?usp=sharing , además, en la carpeta de Evidencias se puede encontrar material visual para la validación de la automatización.

## Funcionalidades Generales

El proyecto cuenta con diferentes archivos .pix los cuales permiten un funcionamiento modular de la automatización, a continuación se presentan cada uno de estos con su respectiva descripción:

**Main.pix (Plantilla Universal):** 
- Es el formato genérico que permite una automatización basada en consumo de colas de trabajo y puede ser adaptada a la necesidad del negocio.
- Cuenta con los estados principales para cualquier flujo de trabajo: INIT, GET, PROCESS, END.
- Se modificó la forma de trabajo a un modelo **No Transaccional**, evitando el uso de colas para este caso.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/Main.png" width="60%" title="Diseño Main" alt="Diseño Main"/>

**InitApplications.pix:**
- Abre el navegador Chrome con la URL del formulario desarrollado en **Google Form**.
- Se hace la conexión a la base de datos **productos** alojada en **PostgreSQL**.
- Si la base de datos no existe, se conecta a la base de datos default **postgres** y genera la query de creación desde ahí.
- Flujos de reintento para ambos procesos, y manejo de excepciones de sistema **SYE001** para errores en Chrome y **SYE002** para errores en la base de datos.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/InitApplications.png" width="60%" title="Diseño InitApplications" alt="Diseño InitApplications"/>

**GetData.pix:** 
- Hace el llamado a la API de la tienda virtual mediante un **HTTP Request** con verbo GET.
- Si el código de status es 200, genera un archivo .json el cual sirve como backup de la información consultada.
- Llama al módulo **UploadOnedDiveFiles** para subir el archivo a una ruta en específico.
- Flujo de reintento para el request, y manejo de excepciones de sistema **SYE003** para errores en la respuesta.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/GetData.png" width="60%" title="Diseño GetData" alt="Diseño GetData"/>

**UpdateDatabase.pix:**
- Inserta la información del archivo .json mediante una query que lo convierte en tabla, cuenta con validación para evitar duplicados en la base de datos.
- Mediante otra query, genera una tabla resumen agrupando por categorías de productos y obteniendo cálculos necesarios para generar el reporte.
- Flujos de reintento y manejo de excepciones de sistema **SYE004** para errores en las query's.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/UpdateDatabase.png" width="60%" title="Diseño UpdateDatabase" alt="Diseño UpdateDatabase"/>

**WriteExcelFile.pix:**
- Se crea un archivo Excel en donde se pega la información extraida de la base de datos.
- Se valida si se creó correctamente y se le agregan estilos para una mejor visualización de la información.
- Llama al módulo **UploadOnedDiveFiles** para subir el archivo a una ruta en específico.
- Manejo de excepciones de sistema **SYE005** si el archivo no se generó correctamente.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/WriteExcelFile.png" width="60%" title="Diseño WriteExcelFile" alt="Diseño WriteExcelFile"/>

**FillGoogleForm.pix:** 
- Hace el registro del formulario con información de la ejecución.
- Carga el reporte de Excel generado en el módulo anterior, envía el formulario y toma pantallazo a la respuesta.
- Manejo de excepciones de sistema **SYE006** para errores en todo el flujo de diligenciamiento del formulario.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/FillGoogleForm.png" width="60%" title="Diseño FillGoogleForm" alt="Diseño FillGoogleForm"/>

**UploadToOneDrive.pix:**
- Hace el llamado a Microsoft Graph API mediante un **HTTP Request** con verbo GET para obtener el access token necesario para la subida de archivos.
- Mediante otro **HTTP Request** con verbo PUT, se pasa la ruta de OneDrive y el archivo a subir.
- Flujos de reintento y manejo de excepciones de sistema **SYE007** para errores en el GET del access token y **SYE008** para errores en el PUT de los archivos.

<img src="https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX/blob/main/Images/UploadToOneDrive.png" width="60%" title="Diseño UploadToOneDrive" alt="Diseño UploadToOneDrive"/>
---

## Estructura del Proyecto

```bash
Prueba_Técnica_PIX/
│
├── Data/
│   │
│   ├── input/
│   ├── Output/
│   ├── Temp/
│   └── Config.xlsx
│
├── Exceptions_Screenshots/
├── Framework/
│   │
│   ├── CloseApplications.pix
│   ├── GetTransactionItem.pix
│   ├── InitApplications.pix
│   ├── KillApplications.pix
│   ├── ReadConfig.pix
│   ├── SetTransactionStatus.pix
│   └── TakeScreenshot
│
├── Modules/
│   │
│   ├── INIT/
│   ├── GET/
│   ├── END/
│   └── PROCESS/
│       │
│       ├── GetData.pix
│       ├── UpdateDatabase.pix
│       ├── WriteExcelFile.pix
│       ├── FillGoogleForm.pix
│       └── UploadOneDriveFile.pix
│
├── Main.pix
├── ProcessTransactionItem.pix
├── Prueba_Técnica_PIX.pixproj
└── README.md
```

---

## Ejecución del Proyecto
### 1. Validar acceso al formulario
Antes de ejecutar el RPA, validar el correcto funcionamiento del formulario https://forms.gle/moDKQxCdQGSjEqzp6
Cualquier situación anormal comunicarse con el desarrollador.

### 2. PostgreSQL como base de datos
El RPA utiliza los parámetros de PostgreSQL para todo tipo de manejo de información, por lo que es mandatorio contar con el sistema de gestión indicado.
Se debe editar en el config los registros str_ConnectionConfig y str_GenericConnectionConfig con los datos de conexión propios (Server, Port, Database, Uid, Pwd).

### 3. Clonar repositorio
El proyecto está almacenado en **GitHub**; para clonarlo ejecuta en el CMD:
git clone https://github.com/nicolasmejiab97-dotcom/Prueba_Tecnica_PIX.git

### 4. Ejecución y revisión
Si desea revisar el código del proyecto puede ejecuta en el terminal el comando con la ruta en donde se clonó el repositorio:
cd C:\Prueba\Prueba_Técnica_PIX

Para abrir el código desde el CMD se debe ejecutar el comando:
start Main.pix

Desde ahí ya podrá revisar y ejecutar el proyecto.

---

## Tecnologías Usadas

Diseño:
- https://app.diagrams.net/

Desarrollo:
- Pix Studio v2.27.4 (x64)
- PostgreSQL v18.3 (x64)
- pgAdmim 4
- Microsoft Entra
- Google Chrome v145.0.7632.160 (x64)
- Postman  v12.0.8 (x64)

---

## Autor
**Desarrollado por: Alvaro Nicolás Mejía Botero**

**Contacto: nicolasmejiab97@gmail.com**
