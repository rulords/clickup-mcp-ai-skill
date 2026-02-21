# Integración de ClickUp MCP

Este repositorio proporciona una integración fluida con ClickUp utilizando el Model Context Protocol (MCP). Despliega un servidor MCP local que permite a tus asistentes de IA interactuar de manera segura con tu entorno de trabajo de ClickUp.

## Instrucciones de Configuración

1. **Clonar el repositorio**:
   ```bash
   git clone <repository_url>
   cd clickup-mcp-integration
   ```

2. **Uso con Clientes MCP (ej. Claude Desktop, Cursor, Antigravity)**:
   Añade la siguiente configuración al archivo de ajustes MCP de tu cliente (`mcp_config.json` o equivalente):
   
   ```json
   {
     "mcpServers": {
       "clickup": {
         "command": "npx.cmd",
         "args": [
           "-y",
           "mcp-remote",
           "https://mcp.clickup.com/mcp"
         ]
       }
     }
   }
   ```
   **Importante:** El servidor oficial no utiliza un Token API local en un archivo `.env`. En su lugar, cuando tu IA intente usar el servidor por primera vez, te guiará mediante un enlace para completar la autenticación OAuth directamente en la web de ClickUp.
## Capacidades y Herramientas (Servidor Oficial)

El servidor oficial cuenta con 32 herramientas directas organizadas para interactuar de forma segura con los Workspaces. Todas están prefijadas con `clickup_`:

### 🔍 Jerarquía, Búsqueda y Resolución
*   Búsqueda global con `clickup_search` (ideal para hallar tareas perdidas o chats).
*   Obtención de la estructura mediante `clickup_get_workspace_hierarchy`.
*   Búsqueda de elementos exactos con `clickup_get_list`, `clickup_get_folder`.
*   Manejo de equipo: `clickup_resolve_assignees`, `clickup_get_workspace_members`.

### 📝 Gestión de Tareas y Etiquetas
*   Soporte completo CRUD: `clickup_create_task`, `clickup_get_task` (formato inteligente reducido para optimizar contextos de IA), `clickup_update_task`.
*   Gestión de Tags: `clickup_add_tag_to_task`, `clickup_remove_tag_from_task`.

### 📂 Carpetas y Listas
*   Creación y edición con: `clickup_create_folder`, `clickup_create_list`, `clickup_update_list`.

### 💬 Comentarios y Chat Interno
*   Añadir progreso textual: `clickup_create_task_comment`.
*   Integración con chats: `clickup_get_chat_channels`, `clickup_send_chat_message`.

### ⏱️ Time Tracking
*   Soporte para cronómetros vivos (`clickup_start_time_tracking`, `clickup_stop_time_tracking`).
*   Registro manual (`clickup_add_time_entry`).

### 📄 Wiki y Documentos
*   Crear y editar Docs en el Workspace con `clickup_create_document`, `clickup_create_document_page` y actualizaciones directas a las páginas.
## Optimización para Agentes (Skill de Inteligencia Artificial)

En este repositorio se incluye el archivo `clickup-skill.md`. Contiene instrucciones específicas en español para enseñar a los agentes de IA cómo utilizar estas herramientas asegurando el **rendimiento técnico y cuidando drásticamente el uso de tokens** mediante identificadores directos y previniendo búsquedas innecesarias. 

### ⚠️ Importante: Límite de Herramientas en Gemini (60 Tools Limit)
El uso del MCP oficial de ClickUp inyecta de golpe **32 herramientas** al asistente de IA. 
Si utilizas clientes MCP como *Claude Desktop* o *Antigravity* integrando **Modelos de Google Gemini (1.5 Flash, 3.1 Pro, etc.)**, llegarás a un límite técnico estricto: **Gemini no puede procesar un request que contenga más de ~60 herramientas combinadas**. 

Si enciendes ClickUp junto a otros servidores pesados (como NotebookLM o GitKraken), el cliente colapsará y la IA no podrá responder.

**La Solución:**
Debes agregar la exclusión de herramientas de ClickUp que no vayas a usar en tu cliente MCP para achicar el conteo total.
Ejemplo en el `mcp_config.json`:
```json
"clickup": {
  "command": "npx.cmd",
  "args": ["-y", "mcp-remote", "https://mcp.clickup.com/mcp"],
  "disabledTools": [
    "clickup_get_task_comments",
    "clickup_start_time_tracking",
    "clickup_stop_time_tracking",
    "clickup_add_time_entry",
    "clickup_get_workspace_hierarchy"
    // ... agrega más aquí para despejar contexto
  ]
}
```

¡Asegúrate de proporcionarle además el archivo de Instrucciones `clickup-skill.md` a tu agente como parte de su Prompt de Sistema!