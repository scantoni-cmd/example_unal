# Guía de configuración: Google Cloud + Google Earth Engine

Guía paso a paso para personas que **nunca han usado Google Earth Engine (GEE)**. Al final tendrás un proyecto de Google Cloud listo para ejecutar `gee_data_visualization.ipynb` desde Google Colab o desde tu computador.

> **Sobre esta guía.** Los nombres de páginas y roles se tomaron de la documentación oficial (enlaces en cada paso). Las consolas de Google cambian de diseño con frecuencia, así que aquí se describe *qué* hacer y *dónde* (con enlace directo), no la posición exacta de cada botón. Si algo no coincide, la documentación enlazada es la fuente de verdad.

---

## Índice

0. [Conceptos clave en 1 minuto](#0-conceptos-clave-en-1-minuto)
1. [Crear o acceder a una cuenta de Google](#1-crear-o-acceder-a-una-cuenta-de-google)
2. [Obtener acceso a Earth Engine](#2-obtener-acceso-a-earth-engine)
3. [Crear un proyecto en Google Cloud Console](#3-crear-un-proyecto-en-google-cloud-console)
4. [Elegir un ID de proyecto adecuado](#4-elegir-un-id-de-proyecto-adecuado)
5. [Habilitar la Earth Engine API](#5-habilitar-la-earth-engine-api)
6. [Registrar el proyecto para Earth Engine](#6-registrar-el-proyecto-para-earth-engine)
7. [Configurar permisos IAM](#7-configurar-permisos-iam)
8. [Usuarios individuales vs. proyectos compartidos](#8-usuarios-individuales-vs-proyectos-compartidos)
9. [Autenticarse con `ee.Authenticate()`](#9-autenticarse-con-eeauthenticate)
10. [Inicializar con `ee.Initialize(project=...)`](#10-inicializar-con-eeinitializeproject)
11. [Verificar la conexión](#11-verificar-la-conexion)
12. [Solución de errores frecuentes](#12-solucion-de-errores-frecuentes)
13. [¿Necesito una cuenta de servicio?](#13-necesito-una-cuenta-de-servicio)
14. [Facturación, cuotas y uso comercial vs. no comercial](#14-facturacion-cuotas-y-uso-comercial-vs-no-comercial)
15. [Enlaces oficiales](#15-enlaces-oficiales)

---

## 0. Conceptos clave en 1 minuto

| Concepto | Qué es |
|---|---|
| **Cuenta de Google** | Tu identidad (Gmail o cuenta institucional de Google Workspace). |
| **Proyecto de Google Cloud** | Contenedor donde se habilitan APIs, se asignan permisos y se contabiliza el uso. Earth Engine **siempre** trabaja a través de un proyecto. |
| **ID del proyecto** | Identificador único global (p. ej. `gee-clima-cali`). Es lo que va en `ee.Initialize(project="...")`. No confundir con el *nombre* ni con el *número* del proyecto. |
| **Earth Engine API** | Servicio que hay que habilitar en el proyecto (`earthengine.googleapis.com`). |
| **Registro en Earth Engine** | Declarar el tipo de uso del proyecto (no comercial o comercial). Sin esto el proyecto no puede usar EE. |
| **IAM** | Sistema de roles y permisos de Google Cloud: decide quién puede hacer qué en el proyecto. |
| **EECU-hora** | Unidad con la que Earth Engine mide el cómputo consumido. |

---

## 1. Crear o acceder a una cuenta de Google

- Si ya tienes Gmail o una cuenta institucional de Google Workspace, puedes usarla.
- Si no, créala en <https://accounts.google.com/signup>.

**Cuenta institucional:** algunas organizaciones restringen la creación de proyectos o el uso de ciertas APIs. Si en los pasos siguientes ves errores de permisos a nivel de organización, contacta a tu administrador de Google Workspace / Google Cloud o usa una cuenta personal para aprender.

---

## 2. Obtener acceso a Earth Engine

Hoy el acceso a Earth Engine **está ligado a un proyecto de Google Cloud registrado** (pasos 3–6). En la práctica, “obtener acceso” significa crear (o elegir) un proyecto y registrarlo.

- Punto de entrada oficial: <https://earthengine.google.com/> → *Get started*.
- Página de Earth Engine en la Cloud Console (registro y configuración): <https://console.cloud.google.com/earth-engine>
- Elegibilidad para uso no comercial (academia, ONG, investigación, educación, gobierno, etc.): <https://earthengine.google.com/noncommercial/>

> Si tu uso es comercial (consultoría paga, producto de una empresa, etc.), revisa <https://earthengine.google.com/commercial/> y la sección 14.

---

## 3. Crear un proyecto en Google Cloud Console

1. Abre <https://console.cloud.google.com/projectcreate> (o el selector de proyectos en <https://console.cloud.google.com/>).
2. Escribe un **nombre de proyecto** (legible, puede cambiarse después).
3. Revisa y, si quieres, edita el **ID del proyecto** (ver paso 4: **no se puede cambiar después**).
4. Si perteneces a una organización, elige la organización/carpeta correcta.
5. Crea el proyecto y selecciónalo como proyecto activo.

Documentación: <https://cloud.google.com/resource-manager/docs/creating-managing-projects>

Alternativa con Google Cloud CLI:

```bash
gcloud projects create gee-clima-cali --name="GEE clima Cali"
gcloud config set project gee-clima-cali
```

**Permisos necesarios para crear el proyecto:** `resourcemanager.projects.create` (rol *Project Creator*, `roles/resourcemanager.projectCreator`). Las cuentas personales lo tienen por defecto; en organizaciones lo otorga el administrador.

---

## 4. Elegir un ID de proyecto adecuado

Reglas de Google Cloud para el ID:

- Entre **6 y 30 caracteres**.
- Solo **letras minúsculas, números y guiones**.
- Debe **empezar con una letra** y **no puede terminar en guion**.
- Es **único a nivel global** y **no se puede cambiar** una vez creado el proyecto.

Recomendaciones:

- Descriptivo y sin datos sensibles: `gee-curso-icesi`, `gee-clima-cali`, `ee-tu-usuario`.
- Un proyecto por propósito (curso, investigación, producción) facilita la gestión de cuotas y permisos.
- El ID **no es un secreto** (aparece en código y URLs), pero no pongas en él información personal.

---

## 5. Habilitar la Earth Engine API

Con el proyecto seleccionado:

- Consola: <https://console.cloud.google.com/apis/library/earthengine.googleapis.com> → *Habilitar*.
- O con la CLI:

```bash
gcloud services enable earthengine.googleapis.com --project=TU_PROJECT_ID
```

**Permiso necesario:** `serviceusage.services.enable` (rol *Service Usage Admin*, `roles/serviceusage.serviceUsageAdmin`, incluido en *Owner* y *Editor*).

Guía oficial de configuración de proyectos para EE: <https://developers.google.com/earth-engine/cloud/earthengine_cloud_project_setup>

---

## 6. Registrar el proyecto para Earth Engine

Todo proyecto que use Earth Engine debe estar **registrado** indicando su tipo de uso:

1. Abre <https://console.cloud.google.com/earth-engine> con tu proyecto seleccionado (o usa el flujo de registro de <https://code.earthengine.google.com/register>).
2. Indica si el uso es **no comercial** o **comercial**.
3. Para uso **no comercial**, completa el cuestionario de elegibilidad y **elige un nivel de cuota** (Community, Contributor o Partner; ver sección 14). Si no eliges, el proyecto queda en **Community**.
4. Para uso **comercial**, se asocia una cuenta de facturación y aplica la tarifa de pago por uso.

Notas importantes:

- El estado no comercial **debe re-verificarse cada año**.
- Cambiar un proyecto de **no comercial a comercial es permanente** (no se puede revertir).
- El nivel de cuota no comercial se puede cambiar en la página de **configuración de Earth Engine** de la Cloud Console.

---

## 7. Configurar permisos IAM

Hay que distinguir **dos tipos de permisos**:

### 7.1 Permisos para **crear y configurar** el proyecto (administrador)

| Tarea | Rol sugerido |
|---|---|
| Crear el proyecto | *Project Creator* (`roles/resourcemanager.projectCreator`) |
| Habilitar APIs | *Service Usage Admin* (`roles/serviceusage.serviceUsageAdmin`) |
| Conceder roles a otras personas | *Project IAM Admin* (`roles/resourcemanager.projectIamAdmin`) |
| Configuración inicial del proyecto para EE | *Editor* (`roles/editor`) **o** la combinación *Project Mover* + *Project IAM Admin* + *Service Usage Admin* |
| Administrar todo lo de Earth Engine (incluido el control de acceso a assets) | *Earth Engine Resource Admin* (`roles/earthengine.admin`) |

Quien crea el proyecto con una cuenta personal queda como **Owner** y ya tiene todo lo anterior.

### 7.2 Permisos **mínimos para ejecutar consultas** desde Python (usuario)

| Necesidad | Roles |
|---|---|
| Ejecutar consultas, visualizar y calcular estadísticas (este notebook) | **Service Usage Consumer** (`roles/serviceusage.serviceUsageConsumer`) **+** **Earth Engine Resource Viewer** (`roles/earthengine.viewer`) |
| Además crear/modificar assets y lanzar exportaciones a assets | Service Usage Consumer **+** **Earth Engine Resource Writer** (`roles/earthengine.writer`) |
| Autenticarse con el modo `notebook` (Jupyter fuera de Colab) | Además **OAuth Config Editor** (`roles/oauthconfig.editor`) en el proyecto de autenticación |

¿Por qué *Service Usage Consumer*? Google Cloud lo exige para usar un proyecto como **proyecto activo** al llamar APIs: sin él, `ee.Initialize(project="X")` falla aunque tengas roles de Earth Engine.

### 7.3 Cómo asignar un rol

- Consola: <https://console.cloud.google.com/iam-admin/iam> → *Conceder acceso* → correo de la persona o grupo → roles.
- CLI:

```bash
PROJECT_ID=tu-project-id
USER=correo@dominio.com

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="user:$USER" --role="roles/serviceusage.serviceUsageConsumer"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="user:$USER" --role="roles/earthengine.viewer"
```

Documentación: <https://developers.google.com/earth-engine/guides/access_control>

---

## 8. Usuarios individuales vs. proyectos compartidos

| Escenario | Configuración recomendada |
|---|---|
| **Individual** (tú creas y usas el proyecto) | Eres Owner: no necesitas asignarte roles adicionales. Solo habilita la API y registra el proyecto. |
| **Proyecto compartido** (curso, laboratorio, equipo) | Un administrador crea, habilita y registra el proyecto. A cada usuario se le da el mínimo: *Service Usage Consumer* + *Earth Engine Resource Viewer* (o *Writer* si crea assets). Usa **Grupos de Google** (`group:curso@dominio.com`) en lugar de usuarios uno por uno. |
| **Estudiantes de un curso** | Opción A: un proyecto compartido del curso (todos consumen la misma cuota mensual). Opción B: cada estudiante crea su propio proyecto no comercial (cuota independiente). Para cursos grandes, la opción B evita agotar una única cuota. |

Recuerda: la **cuota de cómputo es por proyecto**. En un proyecto compartido, todos consumen del mismo cupo mensual. Usar varios proyectos para simular uno solo y evadir límites (*quota farming*) viola los términos de servicio de Earth Engine.

---

## 9. Autenticarse con `ee.Authenticate()`

La autenticación prueba **quién eres** (tu cuenta de Google). `ee.Authenticate()` elige el modo automáticamente:

| Entorno | Modo | Qué ocurre |
|---|---|---|
| **Google Colab** | `colab` | Aparece una ventana de Google para elegir la cuenta y autorizar. Las credenciales viven en la sesión de Colab. |
| **Jupyter local** (con navegador, sin `gcloud`) | `localhost` | Se abre el navegador local para autorizar. |
| **Jupyter remoto / fuera de Colab** | `notebook` | Enlace al *Notebook Authenticator*; requiere elegir un proyecto de autenticación y el rol OAuth Config Editor. |
| **Con Google Cloud CLI** | `gcloud` | Equivale a `gcloud auth application-default login` con los alcances de EE. |

```python
import ee
ee.Authenticate()                 # modo automático
# ee.Authenticate(auth_mode="gcloud")   # si prefieres delegar en gcloud
# ee.Authenticate(force=True)           # para renovar credenciales o cambiar de cuenta
```

En local las credenciales se guardan en `~/.config/earthengine/credentials` (fuera del repositorio). Desde terminal: `earthengine authenticate`.

Documentación: <https://developers.google.com/earth-engine/guides/auth>

---

## 10. Inicializar con `ee.Initialize(project=...)`

La inicialización define **qué proyecto** se usa (permisos y cuota):

```python
ee.Initialize(project="TU_PROJECT_ID")
```

En el notebook, `PROJECT_ID` se configura en la sección 5 (también puede venir de la variable de entorno `EE_PROJECT_ID` o de un *Secret* de Colab con ese nombre). La función `initialize_ee()` combina autenticación + inicialización y traduce los errores comunes.

---

## 11. Verificar la conexión

```python
import ee
ee.Initialize(project="TU_PROJECT_ID")
print(ee.Number(1).add(1).getInfo())                          # → 2 (cómputo en el servidor)
print(ee.Image("USGS/SRTMGL1_003").bandNames().getInfo())     # → ['elevation'] (lectura de un dataset público)
```

Si ambas líneas responden, todo está listo. En el notebook esto lo hace `verify_connection()`.

---

## 12. Solución de errores frecuentes

| Mensaje (fragmento) | Causa | Solución |
|---|---|---|
| `Project ... is not registered to use Earth Engine` | Falta el registro (paso 6). | Regístralo en <https://console.cloud.google.com/earth-engine>. |
| `Earth Engine API has not been used in project ... or it is disabled` / `SERVICE_DISABLED` | API no habilitada (paso 5). | Habilítala y espera 1–2 minutos. |
| `Caller does not have required permission to use project ...` / `USER_PROJECT_DENIED` | Falta `serviceusage.services.use`. | Rol *Service Usage Consumer*. |
| `PERMISSION_DENIED` al leer datos o assets | Falta rol de EE o el asset es privado. | *Earth Engine Resource Viewer* o permisos sobre el asset. |
| `Please authorize access...` / `invalid_grant` / credenciales vencidas | Credenciales ausentes, expiradas o de otra cuenta. | `ee.Authenticate(force=True)` y reinicializa. |
| Error en el *Notebook Authenticator* | Falta *OAuth Config Editor* o el proyecto de autenticación no es válido. | Asigna el rol o usa el modo `colab`/`localhost`/`gcloud`. |
| `Too many concurrent aggregations` / HTTP 429 | Límite de concurrencia. | Espera y reintenta; reduce peticiones simultáneas. |
| Consultas lentas o limitadas tras mucho uso | Cuota mensual no comercial agotada (modo restringido). | Revisa el uso en Cloud Monitoring; sube de nivel (Community → Contributor) o espera el reinicio mensual. |
| `User memory limit exceeded` / `Computation timed out` | Consulta interactiva demasiado pesada. | Reduce el área o el período, aumenta `scale`, o usa exportación batch. |
| Errores de política de organización | Restricciones de tu institución. | Contacta al administrador de Google Cloud de tu organización. |

Guía de depuración: <https://developers.google.com/earth-engine/guides/debugging>

---

## 13. ¿Necesito una cuenta de servicio?

| Situación | ¿Cuenta de servicio? |
|---|---|
| Uso interactivo en Colab o Jupyter local (este notebook) | **No.** Usa tu cuenta personal con `ee.Authenticate()`. |
| Clases, talleres, exploración | **No.** |
| Scripts **desatendidos**: tareas programadas (cron), servidores, pipelines CI/CD, aplicaciones web con backend | **Sí.** No hay una persona para autorizar en el navegador. |
| Código en VMs de Google Cloud | Puede usarse la cuenta de servicio por defecto de la VM. |

Si usas una cuenta de servicio:

- Debe pertenecer a un proyecto registrado para Earth Engine y tener los roles de la sección 7.2.
- **Evita las claves JSON** cuando sea posible (prefiere credenciales del entorno o *Workload Identity Federation*). Si debes usar una clave, **nunca la subas al repositorio**: el `.gitignore` excluye patrones como `*service-account*.json` y `*-key.json`, pero la responsabilidad final es tuya. Guarda las claves en un gestor de secretos.

Documentación: <https://developers.google.com/earth-engine/guides/service_account>

---

## 14. Facturación, cuotas y uso comercial vs. no comercial

### Uso no comercial (academia, investigación, educación, ONG, gobierno)

Earth Engine es **gratuito** para uso no comercial elegible, pero desde el **27 de abril de 2026** cada proyecto no comercial tiene una **cuota mensual de cómputo** medida en EECU-hora, que se reinicia cada mes:

| Nivel | Cuota mensual | Requisitos |
|---|---|---|
| **Community** | 150 EECU-hora | Proyecto no comercial verificado. Sin requisitos adicionales. Nivel por defecto. |
| **Contributor** | 1000 EECU-hora | Requiere una **cuenta de facturación activa**, pero **no se cobra** el uso no comercial de Earth Engine. |
| **Partner** | Mayor (por solicitud) | Para grupos de universidades, ONG o gobierno con trabajo de alto impacto; revisión manual de varias semanas. |

- Si se agota la cuota, puedes seguir usando EE en **modo restringido** (menos concurrencia y menos recursos) hasta el siguiente mes.
- La cuota aplica a **todo** el cómputo del proyecto: Code Editor, API de Python, `xee`, apps de EE, etc.
- Tener una cuenta de facturación (nivel Contributor) **puede generar cobros por otros servicios de Google Cloud** que uses en ese proyecto (p. ej. Cloud Storage); el uso no comercial de EE en sí no se cobra.
- Este notebook, con el AOI y período por defecto, hace consultas pequeñas; su consumo es una fracción mínima de la cuota Community.

### Uso comercial

- Requiere un proyecto registrado como **comercial** con facturación; el cómputo y el almacenamiento se cobran según la tarifa vigente: <https://cloud.google.com/earth-engine/pricing>
- El cambio de no comercial a comercial es **permanente**.
- Usar un proyecto no comercial para trabajo comercial viola los términos de servicio.

Documentación: <https://developers.google.com/earth-engine/guides/noncommercial_tiers>

---

## 15. Enlaces oficiales

**Earth Engine**
- Inicio y registro: <https://earthengine.google.com/> · <https://code.earthengine.google.com/register>
- Configuración de proyectos Cloud: <https://developers.google.com/earth-engine/cloud/earthengine_cloud_project_setup>
- Autenticación e inicialización: <https://developers.google.com/earth-engine/guides/auth>
- Control de acceso y roles IAM: <https://developers.google.com/earth-engine/guides/access_control>
- Niveles de cuota no comercial: <https://developers.google.com/earth-engine/guides/noncommercial_tiers>
- Cuentas de servicio: <https://developers.google.com/earth-engine/guides/service_account>
- Instalación del cliente Python: <https://developers.google.com/earth-engine/guides/python_install>
- Uso no comercial / comercial: <https://earthengine.google.com/noncommercial/> · <https://earthengine.google.com/commercial/>
- Catálogo de datos: <https://developers.google.com/earth-engine/datasets>

**Google Cloud**
- Crear y administrar proyectos: <https://cloud.google.com/resource-manager/docs/creating-managing-projects>
- IAM — conceder y revocar acceso: <https://cloud.google.com/iam/docs/granting-changing-revoking-access>
- Habilitar APIs: <https://cloud.google.com/service-usage/docs/enable-disable>
- Google Cloud CLI: <https://cloud.google.com/sdk/docs/install>

**Google Colab**
- Colab: <https://colab.research.google.com/>
- Abrir notebooks de GitHub en Colab: <https://colab.research.google.com/github/googlecolab/colabtools/blob/main/notebooks/colab-github-demo.ipynb>
- Preguntas frecuentes: <https://research.google.com/colaboratory/faq.html>
