#  On This Day API Workflow

Este archivo `README.md` explica línea por línea el workflow YAML `On This Day API Workflow`, indicando en qué líneas se realiza cada proceso.

##  Disparador manual (líneas 1-10)
- `on.workflow_dispatch.inputs`: Permite ejecutar el workflow manualmente desde GitHub con dos entradas: `month` y `day`.
- `description`, `required`, `default`: Definen la descripción, obligatoriedad y valor por defecto de cada entrada.

##  Variables de entorno (líneas 12-13)
- `env.LANGUAGE`: Usa una variable de entorno `LANG` definida en el repositorio para especificar el idioma de la consulta a la API.

##  Job: fetch_events (líneas 15-61)
- `jobs.fetch_events.runs-on`: Define que se ejecuta en `ubuntu-latest`.
- `outputs.matrix` y `outputs.has_data`: Salidas que se usarán en otros jobs.

###  Instalación de dependencias (líneas 19-21)
- Instala `curl` y `jq` necesarios para hacer peticiones HTTP y procesar JSON.

###  Restaurar caché (líneas 23-27)
- Usa `actions/cache@v3` para restaurar una respuesta previa de la API si existe.

### Llamada a la API (líneas 29-35)
- Si no hay caché (`if: steps.cache.outputs.cache-hit != 'true'`), se hace una llamada a la API de Wikimedia.
- La respuesta se guarda en `cache/api_response.json`.

###  Preparar matriz y salidas (líneas 37-61)
- Procesa la respuesta JSON para extraer eventos con páginas válidas.
- Crea una matriz con `title`, `url` y `filename`.
- Define las salidas `has_data` y `matrix` para usarlas en el siguiente job.

##  Job: generate_pdfs (líneas 63-84)
- `needs: fetch_events`: Este job depende del anterior.
- `if: needs.fetch_events.outputs.has_data == 'true'`: Solo se ejecuta si hay eventos.
- `strategy.matrix.include`: Ejecuta un job por cada evento en la matriz.

###  Convertir HTML a PDF (líneas 75-81)
- Usa `narthanaj/html-to-pdf-action@main` para convertir la URL HTML en un archivo PDF.

###  Subir PDF como artefacto (líneas 83-84)
- Usa `actions/upload-artifact@v4` para subir cada PDF como artefacto individual.

##  Job: zip_and_upload (líneas 86-104)
- `needs: generate_pdfs`: Este job depende de la generación de PDFs.

###  Descargar PDFs (líneas 88-90)
- Usa `actions/download-artifact@v4` para descargar todos los PDFs generados.

###  Generar ZIP y resumen (líneas 92-99)
- Crea un archivo ZIP con todos los PDFs.
- Muestra un resumen con nombre, tamaño y hash SHA256 de cada archivo.

###  Subir ZIP como artefacto (líneas 101-104)
- Usa `actions/upload-artifact@v4` para subir el archivo ZIP como artefacto final.

---
Este workflow automatiza la generación de PDFs de eventos históricos usando la API de Wikimedia y los empaqueta para su descarga.