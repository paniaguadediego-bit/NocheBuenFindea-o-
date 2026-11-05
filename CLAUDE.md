# App NocheBuenFindeaño

App web para organizar la fiesta NocheBuenFindeaño: un grupo de amigos (11 de momento) celebra Navidad y fin de año por adelantado. Hay cena, juegos de mesa, las uvas y mini charlas de 3 minutos sobre hobbies. Todos la usan desde el móvil y el "modo fiesta" se proyecta en la tele.

El dueño del repo (Pablo) la mantiene desde Claude Code en el móvil. Mantén todo simple, comentado en español y fácil de tocar en cambios pequeños.

## Arquitectura (no cambiar sin consultar)

- Un único `index.html` con HTML, CSS y JS inline, sin frameworks, sin build y sin dependencias externas, salvo Google Fonts (Shrikhand y Work Sans, con fuentes de respaldo). Se sirve con GitHub Pages desde este repo. Única excepción: `logo.jpg` (el cartel de la fiesta), que sale en grande en la pantalla de elegir nombre, como franja al final de cada pestaña y en miniatura arriba a la izquierda (botón para volver a Fechas).
- Los datos viven en otro repo privado, `nochebuen-datos`, en el archivo `datos.json`.
- Lectura y escritura con la API de contenidos de GitHub:
  - GET https://api.github.com/repos/paniaguadediego-bit/nochebuen-datos/contents/datos.json
  - PUT en la misma URL, con message, content (base64) y sha.
  - Cabeceras: Authorization: Bearer <clave>, Accept: application/vnd.github+json. Usar cache: "no-store".
  - Base64 con UTF-8 (hay tildes y eñes): codificar con TextEncoder y decodificar con TextDecoder, nunca con btoa/atob a pelo.
- Constantes al principio del script: USUARIO = "paniaguadediego-bit", REPO_DATOS = "nochebuen-datos", ARCHIVO = "datos.json".

## La clave del grupo

- Es un token fine-grained de GitHub con permiso Contents: Read and write solo sobre `nochebuen-datos`.
- NUNCA va en el código ni en este repo.
- Llega en un enlace: https://paniaguadediego-bit.github.io/{REPO_APP}/#k=CLAVE. Al cargar, la app la guarda en localStorage, la quita de la URL con history.replaceState y sigue.
- Sin clave guardada, la app muestra una pantalla amable: "Abre el enlace que se ha compartido en el grupo de WhatsApp".
- Si la API responde 401 o 403, avisar de que la clave ha caducado o no es válida y pedir el enlace nuevo.

## Identidad (sin cuentas)

- La primera vez, la app muestra un desplegable con los nombres de `participantes` de datos.json.
- Al final del desplegable aparece la opción "Añadirme". Al elegirla, sale un campo para escribir el nombre y un botón "Añadirme a la lista". El nombre nuevo se guarda en `participantes` de datos.json, así que lo ven todos.
- Antes de añadir, comprobar si ya existe un nombre igual, sin distinguir mayúsculas ni tildes ("edu" = "Edu", "Sofia" = "Sofía"). Si existe, preguntar "¿Eres tú?" y usar ese nombre en vez de duplicarlo.
- No admitir nombres vacíos ni de más de 20 caracteres. Quitar espacios al principio y al final.
- El nombre elegido se recuerda en localStorage. En la cabecera hay un botón "Cambiar" para elegir otro.

## Concurrencia (que nadie pise a nadie)

- Cada cambio es una función pequeña que modifica el objeto de datos (por ejemplo, votar(fechaId, nombre, respuesta) o anadirParticipante(nombre)).
- Guardar = leer la versión más reciente → aplicar ese cambio → PUT con su sha.
- Si GitHub responde 409 o 422 (alguien guardó a la vez), se relee, se aplica de nuevo solo ese cambio y se reintenta, hasta 3 veces. Si falla, se avisa: "No se pudo guardar, inténtalo otra vez".
- La app relee los datos al abrirse, al volver a la pestaña (visibilitychange) y cada 30 segundos mientras está visible.
- Mostrar "Guardando…" y después "Guardado" en cada cambio.

## Estructura de datos.json

```json
{
  "version": 1,
  "evento": { "nombre": "NocheBuenFindeaño", "fechaOficial": null, "lugar": "" },
  "participantes": ["Ammar", "Pablo"],
  "fechas": [ { "id": "2026-12-12", "etiqueta": "Sáb 12 dic" } ],
  "votos": { "2026-12-12": { "Pablo": "si" } },
  "comida": [ { "id": "c1", "nombre": "Pablo", "plato": "Tortilla", "categoria": "entrante", "notas": "" } ],
  "charlas": [ { "id": "h1", "nombre": "Ammar", "tema": "…", "enlace": "", "remoto": true } ],
  "juegos": [ { "id": "j1", "nombre": "Pablo", "juego": "Dixit", "jugadores": "3-8", "votos": ["Pablo"] } ],
  "ajustes": { "minutosCharla": 3 }
}
```

- Respuestas de voto: "si", "quizas" (se muestra como "Si hace falta"), "no" y "veto". El veto solo informa (0 puntos, no cambia el ranking), pero se muestra quién veta.
- Categorías de comida: entrante, principal, postre, bebida, otros.
- Ids nuevos: Date.now().toString(36) más un sufijo aleatorio corto.
- Si falta algún campo (archivo antiguo o editado a mano), la app lo rellena con un valor vacío en vez de romperse.

## Módulos (pestañas inferiores)

1. **Fechas** (lo más importante): añadir fecha candidata, votar cada fecha con Sí / Si hace falta / No, ranking (sí = 2 puntos, si hace falta = 1), tarjeta "Va ganando", lista de quién falta por votar y botón para marcar la fecha oficial (solo Pablo), que se muestra en la cabecera de todas las pestañas. Calendario mensual: al tocar un día se añade como candidata, y al tocar una candidata se va a votarla. Cada fecha lleva puntos con la inicial de quien vota (o 2+ letras si se repite la inicial): verde = sí, amarillo = si hace falta, rojo = veto. No se pueden borrar fechas.
2. **Comida**: quién trae qué, agrupado por categoría, con aviso de categorías vacías y etiqueta de alergias/notas. Cada uno puede editar o borrar lo suyo.
3. **Charlas**: apuntarse con tema, enlace opcional y casilla "por videollamada". Botón "Copiar prompt" con el contenido de prompt.md incrustado en la app y botón "Ver la guía" con el contenido de guia.md.
4. **Juegos**: quién trae qué juego, número de jugadores y votos (un voto por persona y juego, se puede quitar).
5. **Fiesta** (para la tele, pantalla completa): sorteo del orden de las charlas, quién presenta y su tema, cronómetro de minutosCharla con pausar y siguiente, y botón "Empezar las campanadas" (12 toques, uno cada 3 segundos, con sonido generado con Web Audio y destello en pantalla).

## Diseño

- Estilo retro navideño. Colores: rojo #B3261E, verde bosque #1F4D3A, crema #F6EEDC, dorado #C9A227 (decoración) y #F2C94C (texto sobre verde), texto #2A1E17, tarjetas #FFFDF7 con borde #E2D3B5.
- Títulos en Shrikhand y texto en Work Sans.
- Cabecera roja con el nombre de la fiesta y borde punteado dorado. Pestañas inferiores verdes, con la activa en dorado.
- Primero para móvil: botones de al menos 44 px, texto mínimo 14 px y buen contraste.
- Confirmar antes de borrar cualquier cosa.
- Respetar la zona segura del móvil (env(safe-area-inset-*)).

## Forma de trabajar

- Al empezar cada sesión, leer este archivo y hacer solo lo que pida esa sesión.
- Antes de programar, resumir el plan en pocas líneas y preguntar si hay dudas.
- Al terminar, explicar cómo probarlo desde el móvil.
- No añadir funciones que no estén aquí sin preguntar.
- Nunca escribir la clave ni ningún token en ningún archivo.
