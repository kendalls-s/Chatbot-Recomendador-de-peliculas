# 🎬 Recommender Bot Netflix

Un asistente conversacional inteligente construido en **n8n** que recomienda películas basándose en un catálogo alojado en Google Drive. Utiliza modelos de IA (Google Gemini) para procesar el lenguaje natural, interpretar el estado de ánimo del usuario y buscar la película perfecta mediante filtros avanzados, todo presentado con una interfaz inspirada en Netflix.

**Desarrollado por:** Kendall Solano y Roberto Coto

---

## 🚀 Características Principales

* **Comprensión de Lenguaje Natural:** El bot entiende consultas casuales como *"algo para pasar el rato"*, *"una película para llorar"* o *"la mejor de Spider-Man"*, convirtiéndolas en filtros de búsqueda exactos.
* **Interfaz Estilo Netflix:** El widget de chat cuenta con CSS personalizado que emula la paleta de colores y el estilo visual (Dark Mode y rojo clásico) de la plataforma.
* **Protección Anti-Inyección (Guardrails):** Incluye un Agente de IA clasificador que bloquea consultas no relacionadas con el cine y evita la manipulación del *system prompt*.
* **Traducción Dinámica:** Traduce automáticamente las descripciones de las películas al idioma en el que el usuario hace la consulta, manteniendo los metadatos (como país y géneros) en su formato original.
* **Optimización de Tokens:** Configuración de LLMs dividida en tareas frías (extracción estricta de JSON) y tareas creativas (formateo cálido) para maximizar la velocidad y reducir costos de API.

---

## 📂 Arquitectura del Proyecto

El sistema está dividido en dos flujos de n8n para mantener una arquitectura limpia y modular.

### 1. Flujo Principal: `Recommender Bot netflix.json`
Es el núcleo de la interacción con el usuario. Se encarga de:
* **Chat Trigger:** Recibe los mensajes del usuario.
* **Validación de Intención (AI Agent):** Verifica si el mensaje trata sobre películas o entretenimiento. Rechaza temas ajenos.
* **Extracción de Parámetros:** Usa **Gemini 2.5 Flash Lite** (con temperatura 0) para convertir la frase del usuario en un JSON estructurado (género, año, rating, popularidad, límite, etc.).
* **Formateador de Respuesta:** Un segundo modelo Gemini (con temperatura 0.2) recibe los datos del sub-flujo y redacta una respuesta cálida y amigable.

### 2. Sub-flujo de Datos: `Buscar Películas.json`
Actúa como el "motor de base de datos". Se encarga de:
* **Descarga de Datos:** Se conecta a Google Drive mediante credenciales de *Service Account* para descargar el CSV actualizado del catálogo de películas.
* **Filtrado Avanzado (JavaScript):** Aplica lógica de código para filtrar el CSV según los parámetros recibidos (rating, popularidad mínima de votos, idioma, director, etc.).
* **Sistema de Ordenamiento:** Ordena los resultados según el criterio solicitado (aleatorio, mejor valorada, más popular, etc.) devolviendo solo los campos necesarios.

---

## 🛠️ Requisitos Previos e Instalación

Para ejecutar este proyecto en tu propia instancia de n8n, necesitarás:

* **Instancia de n8n:** (Versión self-hosted o Cloud) con soporte para nodos de LangChain y Advanced IA.
* **API Key de Google Gemini:** Para los nodos de procesamiento de lenguaje.
* **Credenciales de Google Drive (Service Account):** Para que el sub-flujo pueda descargar el archivo CSV del catálogo.
* **Archivo CSV:** Un catálogo de películas alojado en Drive que contenga columnas como `title`, `release_year`, `genres`, `description`, `vote_average`, etc.

### Pasos de importación:
1.  Importa el archivo `Buscar Películas.json` en n8n.
2.  Configura el ID del archivo de Google Drive en el nodo **Download CSV**.
3.  Guarda el flujo y anota su ID.
4.  Importa el archivo `Recommender Bot netflix.json` en n8n.
5.  En el nodo **Buscar Películas** (tipo *Execute Workflow*), selecciona el sub-flujo que importaste en el Paso 1.
6.  Configura tus credenciales de Gemini en los nodos correspondientes.
7.  ¡Activa el flujo y prueba el chat!

---

## 🛡️ Seguridad y Sanitización

El bot incluye múltiples capas de seguridad:
* **Filtro de Contexto:** El agente inicial posee instrucciones estrictas para denegar cualquier tarea que no sea recomendar películas.
* **Sanitización de Salida:** Un nodo de código final limpia la respuesta generada por el LLM, eliminando posibles bloques de código residuales o etiquetas de sistema (`[INST]`, `<|...|>`) antes de enviarlo al usuario.
