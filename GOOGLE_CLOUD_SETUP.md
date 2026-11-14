# Configuración de Google Cloud + Earth Engine

Guía para obtener el `PROJECT_ID` que usa el notebook. Tiempo estimado: 15–20 minutos.

## 1. Cuenta de Google
Usa una cuenta personal o institucional (p. ej. `@unal.edu.co`, si la institución permite Google Cloud).

## 2. Crear un proyecto de Google Cloud
1. Entra a <https://console.cloud.google.com/>.
2. Menú de proyectos → **Proyecto nuevo**.
3. Nombre sugerido: `gee-visualizacion`. Anota el **ID del proyecto** (no es el nombre; suele tener un sufijo numérico). Ese valor va en `PROJECT_ID`.

## 3. Habilitar la Earth Engine API
1. En la consola: **APIs y servicios → Biblioteca**.
2. Busca **Google Earth Engine API** y pulsa **Habilitar**.

## 4. Registrar el proyecto en Earth Engine
1. Ve a <https://code.earthengine.google.com/register>.
2. Elige el tipo de uso (**no comercial / investigación / educación** o comercial) y selecciona tu proyecto.
3. Completa el registro. Cuando termine, el proyecto ya puede usar Earth Engine.

## 5. Permisos (roles) mínimos
Si eres propietario (Owner) del proyecto, no necesitas nada más. Si no, pide que te asignen en **IAM y administración → IAM**:

- `roles/serviceusage.serviceUsageConsumer`
- `roles/earthengine.viewer` (o `roles/earthengine.writer` si vas a exportar a Drive/Assets)

## 6. Usar el proyecto en el notebook
Opción A, directamente en el notebook:

```python
PROJECT_ID = "mi-proyecto-123456"
```

Opción B, sin escribir el ID en el código:

- **Local:** `export EE_PROJECT_ID=mi-proyecto-123456` (Windows PowerShell: `$env:EE_PROJECT_ID="mi-proyecto-123456"`).
- **Colab:** panel izquierdo → icono de llave (**Secretos**) → agregar `EE_PROJECT_ID` y activar el acceso del notebook.

## 7. Autenticación
- **Colab:** al ejecutar la celda de conexión aparece una ventana para iniciar sesión y autorizar. Acepta los permisos.
- **Local:** una vez, en la terminal, ejecuta:
  ```bash
  earthengine authenticate
  ```
  Si ya tienes `gcloud`, también sirve `gcloud auth application-default login`.

## 8. Verificación
Ejecuta en el notebook:

```python
initialize_ee(PROJECT_ID)
verify_connection()   # debe imprimir "Conexión con Earth Engine OK."
```

## 9. Errores frecuentes

| Mensaje | Causa | Solución |
| --- | --- | --- |
| `Project is not registered to use Earth Engine` | Falta el paso 4 | Registra el proyecto |
| `Earth Engine API has not been used in project…` | Falta el paso 3 | Habilita la API |
| `Caller does not have required permission` / 403 | Faltan roles | Paso 5 |
| `Quota exceeded` / 429 | Demasiadas consultas | Espera o reduce el AOI |
| `Computation timed out` / `User memory limit exceeded` | AOI o período muy grandes | Reduce el área, el período o aumenta `scale` |

## 10. Facturación
Para uso no comercial, Earth Engine ofrece acceso gratuito con cuotas. Revisa las condiciones vigentes en <https://developers.google.com/earth-engine/guides/access>.

## 11. Exportar a Google Drive
Pon `EXPORT_TO_DRIVE = True` en la configuración. Las tareas se ven en <https://code.earthengine.google.com/tasks> y los archivos llegan a la carpeta `EXPORT_FOLDER` de tu Drive.

## 12. Buenas prácticas
- No compartas tokens ni claves. El ID del proyecto no es secreto, pero puedes ocultarlo con `EE_PROJECT_ID`.
- Usa AOI pequeños mientras pruebas.

## 13. Cuentas de servicio (automatización sin intervención)
Solo para ejecuciones desatendidas (servidores, CI):

1. **IAM y administración → Cuentas de servicio → Crear cuenta de servicio**.
2. Asígnale `roles/earthengine.viewer` y `roles/serviceusage.serviceUsageConsumer`.
3. Registra la cuenta de servicio en Earth Engine si tu tipo de proyecto lo requiere.
4. Preferiblemente usa *Workload Identity* o secretos del sistema de CI. Si generas una clave JSON, **nunca la subas al repositorio** (el `.gitignore` ya excluye patrones comunes).
5. Inicializa con:
   ```python
   credentials = ee.ServiceAccountCredentials(EMAIL_CUENTA, "ruta/a/clave.json")
   ee.Initialize(credentials, project=PROJECT_ID)
   ```
