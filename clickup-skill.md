---
name: clickup
description: >
  Instrucciones optimizadas para interactuar con el MCP de ClickUp.
  Enseña a la IA cómo listar espacios, carpetas, listas y gestionar tareas
  minimizando el consumo de tokens y evitando consultas innecesarias.
  Útil cuando se necesita crear, leer o actualizar datos en ClickUp a través de MCP.
---

# ClickUp MCP - Skill de Optimización

## Cuándo usar esta skill

- Cuando necesites interactuar con ClickUp a través de MCP.
- Para buscar, listar, crear o actualizar tareas, listas o espacios.
- Cuando debas entender la jerarquía de ClickUp (Workspace -> Space -> Folder -> List -> Task) para operar correctamente.

---

## Jerarquía y Estructura de ClickUp

Para operar en ClickUp, es mandatorio entender cómo se estructuran los datos:
1. **Workspace (Team)**: El nivel más alto.
2. **Space**: Espacios de trabajo (ej. Marketing, Dev).
3. **Folder**: Contenedores opcionales dentro de un Space.
4. **List**: Contenedores obligatorios (pueden colgar directo de un Space o de un Folder).
5. **Task**: Las tareas individuales.

> **Regla de oro**: Para crear una tarea siempre necesitás el **List ID**. Nunca intentes crear una tarea proporcionando únicamente el Space ID o el Folder ID.

---

## Estrategia de Búsqueda Eficiente (Ahorro Masivo de Tokens)

Las APIs de ClickUp a través de MCP pueden devolver JSONs enormes con campos vacíos o metadatos irrelevantes. Para proteger la ventana de contexto, cumplí este proceso estricto:

### 1. Navegación Incremental (Top-Down)
No listes todo el Workspace o todo un Space de una sola vez. Hacé consultas granulares:
1. Consultá los **Spaces** (Espacios) del Workspace.
2. Identificá el Space ID deseado o pedile confirmación al usuario.
3. Listá los **Folders y Lists** locales *sólo* dentro de ese Space ID.
4. Identificá el **List ID** en el que operarás.

### 2. Lectura y Búsqueda de Tareas
- Usá filtros precisos en la herramienta que liste las tareas, en lugar de traer la lista completa.
- Si necesitás ver el detalle de una tarea, leé exclusivamente el ID correspondiente, enfocándote en campos vitales: `id`, `name`, `status`, `description` y `assignees`.
- **Ignorar**: Ignorá los historiales de cambios o campos personalizados vacíos a menos que el usuario lo pida explícitamente.

### 3. Creación y Modificación
- Al usar la herramienta de creación/actualización de tareas, enviá estrictamente los campos mínimos requeridos (habitualmente `name` y `description`).
- Evitá suponer o alucinar IDs de usuarios (`assignees`) o custom fields. Si son necesarios, consultalos primero.
- **Dato Permanente**: Para asignar tareas por defecto al usuario principal (`rsalvucci@persat.com.ar`), utilizá el **ClickUp User ID: `78838270`** en el array de `assignees`. Al tener este ID anotado en esta skill, evitas tener que usar la herramienta de obtener miembros del equipo, ahorrando un paso y tokens de la consulta.

---

## Equipo y Asignaciones Frecuentes (IDs pre-mapeados)

Para cuidar los tokens y la velocidad, **NO utilices herramientas de búsqueda de usuarios**. Usa directamente los IDs de esta lista cuando necesites interactuar o asignar tareas a miembros del equipo de Persat/Coordina:

*   **Rodrigo Salvucci** (`rsalvucci@persat.com.ar`): `78838270`
*   **Eduardo Andino** (`eandino@persat.com.ar`): `82077720`
*   **Maximiliano Gonzalez** (`mgonzalez@persat.com.ar`): `82077721`
*   **Martin Arcos** (`marcos@persat.com.ar`): `114128268`
*   **Pablo Haimovici** (`phaimovici@persat.com.ar`): `138027265`
*   **Federico Giacomozzi** (`fgiacomozzi@coordina.com.ar`): `112002000`
*   **Matias Soto** (`msoto@coordina.com.ar`): `112001999`
*   **Diego Mierez** (`dmierez@coordina.com.ar`): `111981959`
*   **Leonardo Girotto** (`lgirotto@coordina.com.ar`): `111981958`
*   **Melina Krimker** (`mkrimker@coordina.com.ar`): `111975520`
*   **Nicolas Burquet** (`nburquet@coordina.com.ar`): `111975507`

---

## ⚠️ Solución al Error en Modelos de Google (Gemini 3 Flash / 3.1 Pro)

Si al conectar este servidor oficial a un cliente MCP notas que **modelos de Claude funcionan perfecto, pero los modelos de Gemini lanzan error o se rompen**, es porque Gemini tiene un **límite estricto de herramientas** que puede procesar simultáneamente (usualmente alrededor de 50 a 64 tools).

El MCP de ClickUp inyecta **32 herramientas** de golpe. Si usas otros MCPs en paralelo (como GitKraken o NotebookLM), superarás fácilmente las 90 herramientas, causando que la API de Google colapse por superar su contexto de functions (tools).

### ✅ Cómo repararlo (El "Fix" de Gemini):
Para usar Gemini sin apagar ClickUp por completo, **debes desactivar (apagar el toggle) de las herramientas menos usadas en la interfaz de configuración del Cliente MCP** (ej. en Antigravity > Manage MCP Servers).

Recomendamos **DEJAR ENCENDIDAS ÚNICAMENTE** unas ~7 herramientas esenciales y apagar el resto:
1. `clickup_search`
2. `clickup_get_list` o `clickup_get_workspace_hierarchy`
3. `clickup_create_task`
4. `clickup_get_task`
5. `clickup_update_task`
6. `clickup_create_task_comment`
7. `clickup_resolve_assignees`

*Al apagar las demás (time tracking, chat, tags, docs, etc.), el cliente dejará de inyectarlas internamente en la API de Gemini, previniendo el error y dándote un rendimiento perfecto con bajo uso de contexto.*

---

## Catálogo de Herramientas MCP (Oficiales de ClickUp)

El servidor oficial cuenta con 32 herramientas diseñadas ex profeso para integrarse con asistentes. Todas comienzan con el prefijo `clickup_`. **Para ahorrar tokens**, sigue los tips marcados en la lista a continuación.

### 🔍 Jerarquía, Búsqueda y Resolución (Ahorro Masivo de Tokens)
*   `clickup_search`: Búsqueda universal difusa (tareas, documentos, items). Úsalo **solo** cuando desconozcas dónde está algo. Devuelve respuestas gigantescas; utiliza filtros estrictos si es posible.
*   `clickup_get_workspace_hierarchy`: Devuelve espacios, carpetas y listas. **Consume muchísimos tokens**. Úsalo estrictamente una sola vez de forma panorámica para entender la estructura.
*   `clickup_get_list` y `clickup_get_folder`: Te permiten obtener el respectivo ID pasando el nombre. Ideal para no tener que parsear toda la jerarquía de nuevo.
*   `clickup_resolve_assignees`: Permite pasar un nombre o un email genérico y ClickUp te devuelve el ID del Usuario. (*Aunque recuerda usar nuestro mapeo de IDs manual de esta Skill para Rodrigo, Maxi, etc. y evitar esta llamada extra*).

### 📝 Gestión de Tareas
*Para crear tareas siempre vas a necesitar el List ID, sácalo buscando la lista con `clickup_get_list`.*
*   `clickup_create_task`
*   `clickup_get_task`: **¡Token Tip Exclusivo!** Esta herramienta oficial cuenta con un modo de "optimizador de reducción de tamaño" que entrega resúmenes livianos (Summary format) por defecto previniendo colapsos del cliente.
*   `clickup_update_task`
*   `clickup_add_tag_to_task` y `clickup_remove_tag_from_task`

### 📂 Carpetas y Listas
*   `clickup_create_list`, `clickup_create_list_in_folder`, `clickup_update_list`
*   `clickup_create_folder`, `clickup_update_folder`

### 💬 Comentarios y Chat
*   `clickup_get_task_comments` y `clickup_create_task_comment` (Soporta `@mentions`).
*   `clickup_get_chat_channels` y `clickup_send_chat_message`

### ⏱️ Time Tracking (Carga de horas)
*   **Automático**: `clickup_start_time_tracking`, `clickup_stop_time_tracking`, `clickup_get_current_time_entry`
*   **Manual**: `clickup_add_time_entry`
*   **Historial**: `clickup_get_task_time_entries`

### 📄 Documentos de ClickUp (Wiki)
*   `clickup_create_document`, `clickup_list_document_pages`
*   `clickup_get_document_pages`, `clickup_create_document_page`, `clickup_update_document_page`
*   Para archivos adjuntos en tareas usa: `clickup_attach_task_file`

---

## Modificadores para el Sistema (Prompt Engineering)

Si el usuario pide actuar de forma iterativa ("Buscá tal lista, leé las tareas y si les falta descripción poneles X"), NO uses `clickup_search` al aire. Procede paso a paso con llamadas en cadena:
1. `clickup_get_list` -> Identificas el List ID.
2. `clickup_search` **filtrando** únicamente por ese List ID -> Obtienes los IDs de las tareas.
3. Llamadas iterativas a `clickup_update_task`.

## Control de Errores y Patrones Comunes

### ❌ Error: "List ID is required"
Ocurre al intentar crear una tarea en un Space o Folder mediante herramientas genéricas.
**Solución**: Las tareas *solo* viven en Listas. Encontrá las listas que existen dentro de ese Space/Folder y enviá el "list_id" correcto.

### ❌ Error de Contexto Inundado (Too Many Tokens)
Si la herramienta de obtener tarea devuelve demasiado texto (ej. historial o comentarios infinitos):
**Solución**: Filtrá tu análisis mental sólo a las claves de negocio esenciales. No devuelvas todo el JSON al usuario, elaborá un resumen.

---

## Ejemplo de Flujo de Trabajo Recomendado

**Objetivo del USER**: Crear una tarea llamada "Fix Login" en el entorno "Dev".
1. Acción: Tool para listar Spaces -> *El agente encuentra el Space "Dev" (ID: 101)*.
2. Acción: Tool para listar Lists en Space "101" -> *El agente encuentra la lista "Bugs" (ID: 202)*.
3. Acción: Tool para crear Tarea -> *Payload mínimo apuntando a la List "202" con nombre "Fix Login"*.

> **Nota Final**: Este flujo incremental garantiza cero alucinaciones de IDs y utiliza la longitud de respuesta mínima de parte del MCP, cuidando los tokens de la conversación.