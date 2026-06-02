# 🤖 Automatización de flujos con Make
## Noticias de IA filtradas, traducidas, revisadas por humanos y enviadas a Telegram

> Flujo de automatización completo construido con Make (antiguo Integromat) que obtiene noticias de IA desde RSS, las filtra, traduce y publica en un grupo de Telegram tras revisión humana.

---

## 📌 Justificación del flujo

Este flujo representa un caso realista y completo de automatización porque integra **todos los elementos clave**:

- **Disparador claro:** RSS de TechCrunch, ejecutado diariamente.
- **Intervención de IA:** Traducción automática EN→ES mediante MyMemory API.
- **Condición IF/Else:** Filtro que descarta artículos irrelevantes (títulos con "review" o "smart").
- **Salida verificable:** Envío final al grupo de Telegram.
- **Revisión humana:** Validación manual mediante Google Sheets antes de publicar.

Resuelve un problema real: obtener noticias relevantes de IA, filtrarlas, traducirlas y publicarlas sin ruido ni contenido irrelevante.

---

## 🔄 Diagrama del flujo

```
RSS TechCrunch → Filtro IF/Else → HTTP MyMemory (título) 
                                → HTTP MyMemory (resumen)
                                → Google Sheets (revisión humana)
                                → Router → Telegram Bot
```

---

## 🧩 Elementos del flujo

### Entrada
- **Fuente RSS:** TechCrunch
- Recupera el artículo más reciente sobre IA.

### Filtro (IF/Else)
- Excluye artículos cuyo título contenga `"review"` o `"smart"`.
- Evita contenido no relacionado con IA.

### Procesamiento con IA
- Traducción automática del título (EN→ES).
- Traducción automática del resumen (EN→ES).
- API gratuita MyMemory, sin registro.

### Revisión humana
- Google Sheets recibe cada noticia traducida.
- Un revisor marca manualmente si debe enviarse (`SI` / vacío).
- El flujo solo continúa si la columna F está marcada como `"SI"`.

### Salida
- Envío al grupo de Telegram mediante bot.
- Mensaje formateado en Markdown con título, resumen y enlace.

---

## 🛠️ Herramientas utilizadas (todo gratuito)

| Herramienta | Uso | Coste | Límite gratuito |
|---|---|---|---|
| Make | Motor de automatización | Gratuito | 1.000 ops/mes |
| TechCrunch RSS | Fuente de noticias en inglés | Gratuito | Sin límite |
| MyMemory API | Traducción EN → ES | Gratuito | 1.000.000 chars/mes |
| Google Sheets | Revisión humana | Gratuito | Sin límite práctico |
| Telegram Bot API | Envío al grupo | Gratuito | Sin límite |

---

## 📊 Consumo de recursos

### Make — Operaciones mensuales

| Módulo | Ops por ítem | Ítems/día | Ops/día |
|---|---|---|---|
| RSS | 1 | — | 1 |
| HTTP traducción título | 1 | 2 | 2 |
| HTTP traducción resumen | 1 | 2 | 2 |
| Telegram | 1 | 2 | 2 |
| **TOTAL** | — | — | **7 ops/día** |

> **Consumo mensual Make:** 7 × 30 = **210 operaciones/mes** → 21% del límite gratuito ✅

### MyMemory — Caracteres mensuales

| Elemento | Chars estimados | Artículos/día | Total/día |
|---|---|---|---|
| Título (EN) | ~80 chars | 2 | ~160 |
| Resumen (EN) | ~400 chars | 2 | ~800 |
| **TOTAL** | — | — | **~960 chars/día** |

> **Consumo mensual MyMemory:** ~28.800 chars/mes → 2,9% del límite ✅

---

## ⚙️ Configuración paso a paso

### Paso 1: Crear el bot de Telegram con BotFather

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

### Paso 2: Crear el grupo de Telegram y añadir el bot

1. Crea un nuevo grupo en Telegram (nombre libre).
2. Ve a **Editar grupo → Añadir miembros** y busca tu `@NombreBot`.

### Paso 3: Obtener el Chat ID del grupo

1. Con el bot dentro del grupo, escribe cualquier mensaje.
2. Abre esta URL en el navegador (reemplaza `TOKEN` por el tuyo):
   ```
   https://api.telegram.org/botTOKEN/getUpdates
   ```
3. En el JSON de respuesta, busca el campo `"id"` dentro de `"chat"`:
   ```json
   {
     "chat": {
       "id": -1009999999999,
       "title": "Nombre de tu grupo"
     }
   }
   ```
   > ⚠️ El Chat ID de un grupo **siempre empieza por signo negativo (–)**. Cópialo completo.

   > Si el JSON aparece vacío, escribe otro mensaje en el grupo y recarga. Si sigue vacío, repite el `/setprivacy`, expulsa el bot y vuelve a añadirlo.

### Paso 4: Configurar el flujo en Make

#### Módulo 1 — RSS TechCrunch
- **URL del feed:** `https://techcrunch.com/feed/`
- **Frecuencia:** Diaria (a la hora que prefieras)
- **Máx. resultados:** 2 artículos

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
| Variable resultado | `Data > responseData > translatedText` |

#### Módulo 4 — HTTP (traducción del resumen)

| Parámetro | Valor |
|---|---|
| URL | `https://api.mymemory.translated.net/get` |
| Method | GET |
| Parámetro `q` | `{{1.summary}}` |
| Parámetro `langpair` | `en\|es` |
| Variable resultado | `Data > responseData > translatedText` |

#### Módulo 5 — Google Sheets: Add a Row
Columnas recomendadas: `Título traducido | Resumen traducido | URL | Fecha | Revisado (SI/vacío)`

#### Módulo 6 — Google Sheets: Search Rows
- Busca filas donde la columna `Revisado = "SI"`.

#### Módulo 7 — Router (IF/Else)
- **Sin filas aprobadas:** el flujo termina.
- **Con filas aprobadas:** continúa a Telegram.

#### Módulo 8 — Telegram: Send a Message

| Campo | Valor |
|---|---|
| Chat ID | Tu Chat ID (con signo negativo) |
| Parse Mode | Markdown |
| Texto | Ver plantilla abajo |

**Plantilla de mensaje Markdown:**
```
📰 *{{titulo_traducido}}*

{{resumen_traducido}}

🔗 [Leer artículo completo]({{url}})
```

---

## ✅ Verificación final

- [ ] El bot aparece en el grupo de Telegram
- [ ] Make puede conectar con Google Sheets
- [ ] La primera ejecución de prueba muestra una fila en Sheets
- [ ] Al marcar `SI` en la columna, el mensaje llega a Telegram

---

## 📈 Escalabilidad

- Añadir más fuentes RSS (The Verge AI, MIT Tech Review…)
- Añadir más filtros temáticos
- Reemplazar MyMemory por DeepL API (mayor calidad) si se supera el límite
- Generar resúmenes con IA (Claude API, GPT) en lugar de solo traducir

---

## 📄 Licencia

MIT — libre para usar, modificar y distribuir.
