# Manual Técnico — Automatización de Noticias IA con Make

## 1. Arquitectura General

```
+-------------------+       +-------------------+       +-------------------+
|   Capa de entrada |  -->  | Capa de proceso   |  -->  | Capa de salida    |
|                   |       |                   |       |                   |
|  RSS TechCrunch   |       |  Filtro IF/Else   |       |  Telegram Bot     |
|  (disparador)     |       |  MyMemory API     |       |                   |
|                   |       |  Google Sheets    |       |                   |
+-------------------+       +-------------------+       +-------------------+
```

El flujo comienza con la lectura de una fuente RSS, pasa por un filtro de contenido, luego se traduce automáticamente mediante una API externa, se almacena en una hoja de cálculo para revisión humana y finalmente se envía a un grupo de Telegram si es aprobado.

## 2. Descripción de Módulos y Componentes

### 2.1. Disparador: RSS TechCrunch
- **Responsabilidad:** Obtener el artículo más reciente sobre Inteligencia Artificial desde el feed RSS de TechCrunch.
- **Frecuencia:** Ejecución diaria programada en Make.
- **Salida:** Título, resumen, enlace del artículo.

### 2.2. Filtro IF/Else
- **Responsabilidad:** Excluir artículos que contengan las palabras "review" o "smart" en el título, reduciendo ruido no relacionado con IA.
- **Comportamiento:** Si el título contiene alguna de esas palabras, el flujo se detiene para ese artículo. Si pasa, continúa a traducción.

### 2.3. Módulo de Traducción (MyMemory API)
- **Responsabilidad:** Traducir automáticamente el título y el resumen del inglés al español usando la API gratuita MyMemory.
- **Módulos:** Dos llamadas HTTP independientes (una para título, otra para resumen).
- **Límite gratuito:** 1.000.000 caracteres/mes.

### 2.4. Módulo de Revisión Humana (Google Sheets)
- **Responsabilidad:** Almacenar cada artículo traducido en una hoja de cálculo para que un revisor humano decida si se publica.
- **Formato:** Una fila por artículo con columnas: título original, título traducido, resumen original, resumen traducido, enlace, estado (SI/vacío).
- **Flujo:** El escenario de Make espera a que la columna de estado sea "SI" (puede ser mediante una verificación periódica o un webhook, según configuración).

### 2.5. Módulo de Salida (Telegram Bot)
- **Responsabilidad:** Enviar el artículo aprobado al grupo de Telegram mediante un bot.
- **Formato del mensaje:** Markdown con título traducido, resumen traducido y enlace original.
- **Requisitos:** Token de bot y ID del grupo configurados en Make.

## 3. APIs y Endpoints Documentados

### 3.1. MyMemory API (Traducción)
| Método | Ruta | Descripción | Parámetros |
|--------|------|-------------|------------|
| GET | `https://api.mymemory.translated.net/get` | Traduce texto de un idioma a otro | `q` (texto a traducir), `langpair` (ej: `en\|es`), `de` (dominio opcional) |

### 3.2. Telegram Bot API
| Método | Ruta | Descripción | Parámetros |
|--------|------|-------------|------------|
| POST | `https://api.telegram.org/bot<TOKEN>/sendMessage` | Envía un mensaje a un chat específico | `chat_id` (ID del grupo), `text` (mensaje), `parse_mode` (Markdown) |

### 3.3. TechCrunch RSS
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `https://techcrunch.com/feed/` | Feed RSS principal (se puede filtrar por categoría) |

### 3.4. Google Sheets API (v4)
- **Uso indirecto:** Make utiliza conectores integrados para leer y escribir en hojas de cálculo. No se exponen endpoints directamente, pero se requiere autenticación OAuth2.

## 4. Variables de Entorno

No existe un archivo de variables de entorno tradicional, ya que la configuración se realiza directamente en la interfaz de Make. Sin embargo, los valores sensibles o parametrizables son:

| Variable | Valor de ejemplo | Obligatoria | Descripción |
|----------|------------------|-------------|-------------|
| **Telegram Bot Token** | `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11` | Sí | Token del bot de Telegram para enviar mensajes. |
| **Telegram Chat ID** | `-1001234567890` | Sí | ID del grupo o canal donde se publicarán las noticias. |
| **Google Sheets URL** | `https://docs.google.com/spreadsheets/d/...` | Sí | Enlace a la hoja de cálculo utilizada para revisión humana. |
| **TechCrunch RSS URL** | `https://techcrunch.com/feed/` | No (por defecto) | Fuente RSS alternativa si se desea cambiar de medio. |

## 5. Guía de Despliegue Paso a Paso

### 5.1. Requisitos previos
- Cuenta gratuita en [Make](https://www.make.com/).
- Token de un bot de Telegram (crear con [@BotFather](https://t.me/BotFather)).
- Hoja de cálculo de Google Sheets con las columnas: `Título original`, `Título traducido`, `Resumen original`, `Resumen traducido`, `Enlace`, `Estado`. (Recomendado: usar la plantilla del README original).

### 5.2. Crear el escenario en Make
1. Inicia sesión en Make y crea un nuevo escenario.
2. **Añadir el disparador RSS:**
   - Busca el módulo "RSS" y selecciona "Watch RSS Items".
   - Configura la URL: `https://techcrunch.com/feed/`.
   - Establece la frecuencia de actualización (ej: cada 24 horas).
3. **Añadir el filtro:**
   - Inserta un módulo "Router" o "Filter".
   - Configura la condición: `{{title}}` no contiene "review" y no contiene "smart".
4. **Añadir llamadas HTTP a MyMemory:**
   - Módulo HTTP → "Make a request".
   - URL: `https://api.mymemory.translated.net/get?q={{title}}&langpair=en|es`.
   - Repite para el resumen con `{{summary}}`.
5. **Añadir módulo de Google Sheets:**
   - Conecta tu cuenta de Google.
   - Selecciona "Add a row" con la hoja de cálculo creada.
   - Mapea los campos: título original, título traducido, resumen original, resumen traducido, enlace, estado (vacío inicialmente).
6. **Añadir módulo de espera/verificación:**
   - Opcional: usa un módulo "Wait" o "Loop" para revisar periódicamente si la columna "Estado" cambió a "SI". (Alternativa: usar un webhook que se dispare al editar la hoja).
7. **Añadir módulo de Telegram:**
   - Módulo "Telegram" → "Send a Text Message".
   - Configura token y chat ID.
   - Mensaje en Markdown: `*{{title_translated}}* \n\n {{summary_translated}} \n\n [Leer más]({{link}})`
8. **Conectar los módulos en el orden correcto según el diagrama.**
9. **Guardar y activar el escenario.**

### 5.3. Prueba del flujo
- Ejecuta manualmente el escenario con un artículo de prueba.
- Verifica que la hoja de cálculo reciba los datos.
- Marca "SI" en la columna de estado para forzar el envío a Telegram.
- Confirma la recepción del mensaje en el grupo.

## 6. Limitaciones Conocidas y Posibles Mejoras Futuras

### 6.1. Limitaciones
- **Dependencia de API gratuita:** MyMemory tiene límite de 1.000.000 caracteres/mes; para proyectos grandes puede resultar insuficiente.
- **Filtro básico:** La exclusión por palabras clave ("review", "smart") es simple y puede descartar noticias relevantes o incluir ruido.
- **Revisión humana manual:** El proceso requiere que un humano marque el estado en Google Sheets, lo que introduce latencia y posible error humano.
- **Sin manejo de errores avanzado:** Si la API de traducción falla, el flujo puede detenerse sin reintentos configurados.
- **Escalabilidad:** El escenario está diseñado para un solo artículo por ejecución; no maneja lotes grandes.

### 6.2. Mejoras futuras
- **Traducción con IA más potente:** Integrar DeepL o Google Translate API (con coste) para mayor precisión.
- **Filtro semántico:** Usar un clasificador de texto (ej: con IA entrenada) para determinar relevancia en lugar de palabras clave.
- **Aprobación automática:** Implementar un umbral de confianza en la traducción que permita aprobar automáticamente algunos artículos.
- **Programación flexible:** Permitir al usuario definir horarios personalizados mediante una interfaz simple.
- **Notificaciones de error:** Añadir notificaciones a Telegram cuando ocurra un fallo en el flujo.
- **Registro de métricas:** Almacenar en Google Sheets el número de artículos procesados, traducidos y enviados para análisis.

---

*Este manual técnico documenta la versión actual del proyecto según el repositorio original. Para más detalles, consultar el archivo README.md.*

<p align="center">Creado por <a href="https://github.com/migueljerico">@migueljerico</a> y documentado por QwenCloud (DeepSeek V4 Flash (free)) desde la App Asistente de IA · 2026</p>