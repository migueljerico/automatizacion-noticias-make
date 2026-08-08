# 🤖 Automatización de Noticias IA — Flujo con Make

![Make](https://img.shields.io/badge/Make-Automatización-6C5CE7?style=for-the-badge&logo=make&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-Revisión%20Humana-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![Estado](https://img.shields.io/badge/Estado-Completado-4CAF50?style=for-the-badge)
![Licencia](https://img.shields.io/badge/Licencia-MIT-yellow?style=for-the-badge)

> **Ejercicio Práctico — Automatización de Flujos con Make (anteriormente conocido como Integromat)**  
> Noticias de IA filtradas, traducidas, revisadas por humanos y enviadas a **Telegram**

---

## 📋 Descripción

Este proyecto implementa un flujo de automatización completo utilizando Make, que integra múltiples servicios para la curación y distribución de noticias sobre inteligencia artificial. El flujo se encarga de obtener artículos recientes desde el feed RSS de TechCrunch, filtrar aquellos que no sean relevantes (como reseñas o contenido sobre dispositivos «smart»), traducir automáticamente el título y resumen del inglés al español mediante la API gratuita MyMemory, y almacenar los resultados en una hoja de Google Sheets para una revisión humana antes de su publicación final en un grupo de Telegram.

Resuelve el problema real de gestionar el exceso de información y el ruido en las noticias tecnológicas, permitiendo a un equipo o comunidad recibir solo contenido de IA de alta calidad, traducido y verificado por humanos. Está dirigido a profesionales del análisis de datos, entusiastas de la automatización no-code y cualquier persona que desee mantenerse al día con las novedades de IA sin tener que revisar decenas de fuentes manualmente.

---

## ✨ Funcionalidades

| Funcionalidad | Descripción |
|---------------|-------------|
| **Disparador RSS** | Obtiene automáticamente hasta 2 artículos diarios desde el feed de TechCrunch |
| **Filtro condicional** | Excluye artículos cuyo título contenga «review» o «smart» (insensible a mayúsculas) |
| **Traducción automática** | Traduce título y resumen de inglés a español usando la API MyMemory (sin necesidad de registro) |
| **Revisión humana** | Almacena cada noticia traducida en Google Sheets; un revisor marca la columna F con «SI» para aprobar su envío |
| **Envío a Telegram** | Publica las noticias aprobadas en un grupo de Telegram con formato Markdown (título, resumen y enlace) |
| **Router inteligente** | El flujo solo continúa si existen filas aprobadas en Sheets; de lo contrario, finaliza sin enviar nada |

---

## ⚙️ Instalación

Sigue estos pasos para configurar el flujo completo en Make:

### 1. Crear el bot de Telegram con BotFather

1. Abre Telegram y busca `@BotFather` (tick azul de verificación).
2. Envía `/start` y luego `/newbot`.
3. Asigna un nombre visible al bot (p.ej.: `Noticias IA`).
4. Asigna un username único terminado en `bot` (p.ej.: `@IA_NewsBot`).
5. BotFather te proporcionará un **Token de acceso** con este formato:
   ```
   123456789:AAFxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```
   > ⚠️ Guárdalo en un lugar seguro. Es la contraseña de tu bot.
6. Desactiva el modo privacidad para que el bot pueda leer mensajes en grupos:
   ```
   /setprivacy → selecciona tu bot → Disable
   ```

### 2. Crear el grupo de Telegram y añadir el bot

1. Crea un nuevo grupo en Telegram (nombre libre).
2. Ve a **Editar grupo → Añadir miembros** y busca tu `@NombreBot`.

### 3. Obtener el Chat ID del grupo

1. Con el bot dentro del grupo, escribe cualquier mensaje.
2. Abre esta URL en el navegador (reemplaza `TOKEN` por el tuyo):
   ```
   https://api.telegram.org/botTOKEN/getUpdates
   ```
3. En el JSON de respuesta, busca el campo `"id"` dentro de `"chat"`:
   ```json
   { "chat": { "id": -1009999999999, "title": "Nombre de tu grupo" } }
   ```
   > ⚠️ El Chat ID de un grupo **siempre empieza por signo negativo (–)**. Cópialo completo.

### 4. Configurar el flujo en Make

#### Módulo 1 — RSS TechCrunch
- **URL del feed:** `https://techcrunch.com/feed/`
- **Frecuencia:** Diaria · **Máx. resultados:** 2 artículos

#### Módulo 2 — Filtro IF/Else

| Condición | Tipo | Valor a excluir | Operador |
|---|---|---|---|
| Condición 1 | Title does not contain (case insensitive) | `review` | AND |
| Condición 2 | Title does not contain (case insensitive) | `smart` | AND |

#### Módulo 3 — HTTP (traducción del título)

| Parámetro | Valor |
|---|---|
| URL | `https://api.mymemory.translated.net/get` |
| Method | GET |
| Parámetro `q` | `{{1.title}}` |
| Parámetro `langpair` | `en\|es` |

#### Módulo 4 — HTTP (traducción del resumen)

Igual que el módulo 3, sustituyendo `{{1.title}}` por `{{1.summary}}`.

#### Módulo 5 — Google Sheets: Add a Row

Columnas: `Título traducido | Resumen traducido | URL | Fecha | Revisado (SI/vacío)`

#### Módulo 6 — Google Sheets: Search Rows

Busca filas donde la columna `Revisado = "SI"`.

#### Módulo 7 — Router (IF/Else)

- **Sin filas aprobadas:** el flujo termina.
- **Con filas aprobadas:** continúa a Telegram.

#### Módulo 8 — Telegram: Send a Message

| Campo | Valor |
|---|---|
| Chat ID | Tu Chat ID (con signo negativo) |
| Parse Mode | Markdown |

**Plantilla de mensaje:**
```
📰 *{{titulo_traducido}}*

{{resumen_traducido}}

🔗 [Leer artículo completo]({{url}})
```

---

## 🚀 Uso

Una vez configurado, el flujo se ejecutará automáticamente una vez al día. Para probarlo manualmente:

1. Haz clic en **Run once** en Make para forzar una ejecución.
2. Revisa la hoja de Google Sheets; debería aparecer una nueva fila con los datos traducidos.
3. En la columna `Revisado`, escribe `SI` en la celda correspondiente.
4. Espera unos segundos: el mensaje llegará al grupo de Telegram formateado en Markdown.

**Ejemplo de mensaje publicado en Telegram:**
```
📰 *Nuevo modelo de IA de OpenAI supera a GPT-4 en razonamiento*

El modelo, llamado o3, demostró un rendimiento excepcional en pruebas de lógica y matemáticas, marcando un hito en la investigación de inteligencia artificial.

🔗 [Leer artículo completo](https://techcrunch.com/2026/01/15/openai-o3-reasoning-model/)
```

---

## 📁 Estructura del proyecto

```
automatizacion-noticias-make/
├── README.md          # Documentación principal del flujo
└── LICENSE            # Licencia MIT
```

El repositorio contiene únicamente la documentación del flujo de automatización, ya que Make no requiere código fuente local. Todo el pipeline se configura y ejecuta en la plataforma Make.

---

## 🛠️ Tecnologías

| Herramienta | Versión/Detalle | Uso en el proyecto |
|---|---|---|
| **Make** | Plataforma de automatización (sin versión específica) | Motor principal que orquesta todos los módulos del flujo |
| **TechCrunch RSS** | Feed XML público | Fuente de entrada que proporciona los artículos más recientes sobre IA |
| **MyMemory API** | v2 (gratuita, sin autenticación) | Traducción automática de título y resumen de inglés a español |
| **Google Sheets** | API de Google Sheets | Almacenamiento intermedio para revisión humana antes de la publicación |
| **Telegram Bot API** | API REST (sin versión específica) | Envío de mensajes formateados al grupo de Telegram |

---

## 📚 Contexto formativo

Este ejercicio forma parte del programa de formación en **Análisis de Datos**, dentro del módulo de herramientas de automatización no-code. El objetivo es diseñar flujos de trabajo complejos con múltiples módulos, integrar fuentes externas (RSS, APIs, bots) y aplicar lógica condicional y revisión humana en un pipeline de datos real.

---

<p align="center">
  <sub>Desarrollado por <a href="https://github.com/migueljerico">@migueljerico</a> · 2026</sub>
</p>

<p align="center">Creado por <a href="https://github.com/migueljerico">@migueljerico</a> y documentado por QwenCloud (DeepSeek V4 Flash (free)) desde la App Asistente de IA · 2026</p>