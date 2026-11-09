# App NocheBuenFindeaño

App web para organizar la fiesta NocheBuenFindeaño: un grupo de amigos (11 de momento) celebra Navidad y fin de año por adelantado. Hay cena, juegos de mesa, las uvas y mini charlas de 3 minutos sobre hobbies. Todos la usan desde el móvil y el "modo fiesta" se proyecta en la tele.

El dueño del repo (Pablo) la mantiene desde Claude Code en el móvil. Mantén todo simple, comentado en español y fácil de tocar en cambios pequeños.

## Arquitectura (no cambiar sin consultar)

- Un único `index.html` con HTML, CSS y JS inline, sin frameworks, sin build y sin dependencias externas, salvo Google Fonts (Fraunces, Shrikhand y Work Sans, con fuentes de respaldo). Se sirve con GitHub Pages desde este repo. Excepciones: `compartir.jpg` (1200×630, la imagen de la vista previa al compartir el enlace en WhatsApp, puesta con las etiquetas og: del `<head>`) y `logo.jpg` (el cartel de la fiesta), que sale en grande en la pantalla de elegir nombre, como franja al final de cada pestaña y en miniatura arriba a la izquierda (botón para volver a Fechas).
- Otra excepción, aprobada por Pablo: el reproductor de YouTube (youtube-nocookie.com, solo se carga al pulsar un vídeo), las miniaturas de i.ytimg.com y el título automático con el oEmbed de youtube.com.
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
  "sugerencias": [ { "id": "s1", "nombre": "Pablo", "texto": "…", "fecha": "2026-11-05T16:00:00Z" } ],
  "fiesta": { "orden": ["h1"], "actual": 0 },
  "quienlo": { "preguntas": [ { "id": "q1", "texto": "¿Peor regalo?", "autor": "Pablo" } ], "respuestas": { "q1": { "Ammar": "Calcetines" } }, "ronda": { "pid": "q1", "de": "Ammar", "votos": { "Pablo": "Ammar" }, "revelada": false }, "hechas": [] },
  "quiz": { "preguntas": [ { "id": "z1", "texto": "…", "opciones": ["A", "B", "C", "D"], "correcta": 1, "autor": "Ammar" } ], "ronda": { "pid": "z1", "inicio": "ISO", "votos": { "Pablo": 1 }, "revelada": false }, "hechas": [] },
  "premios": { "categorias": [ { "id": "p1", "texto": "El más tardón", "autor": "Pablo" } ], "votos": { "p1": { "Ammar": "Pablo" } }, "reveladas": [] },
  "predicciones": { "lista": [ { "id": "r1", "nombre": "Ammar", "texto": "…", "fecha": "ISO" } ], "reveladas": false },
  "ajustes": { "minutosCharla": 3 }
}
```

- Respuestas de voto: "si", "quizas" (se muestra como "Si hace falta"), "no" y "veto". "No" y "Veto" valen 0 puntos y cada uno tiene su bola (gris y roja). Si un día suma 2 o más entre "No" y "Veto" (NOS_PARA_ANULAR), queda anulado: tachado en el calendario, la gráfica y el ranking (al final, sin posición, corona ni botón de hacer oficial) y fuera de "Faltan por votar". No se borran sus votos: si alguien cambia el suyo, vuelve a contar. Tocar otra vez el voto que ya tienes lo quita.
- Categorías de comida: entrante, principal, postre, bebida, otros.
- Ids nuevos: Date.now().toString(36) más un sufijo aleatorio corto.
- Si falta algún campo (archivo antiguo o editado a mano), la app lo rellena con un valor vacío en vez de romperse.

## Módulos (pestañas inferiores)

1. **Fechas** (lo más importante): añadir fecha candidata, votar cada fecha con Sí / Si hace falta / No, ranking ordenado por votos con su posición (sí = 2 puntos, si hace falta = 1; la que más puntos tiene lleva corona 👑, y si hay empate, todas las empatadas); un selector "🏆 Votos / 📅 Fecha" permite verlo en orden cronológico (se recuerda en cada móvil, `nb_orden`), lista de quién falta por votar (sin los nombres de NO_CUENTAN_EN_FALTAN: Ammar, que es una broma del grupo). Ammar está en OCULTOS: nunca sale en las bolas ni suma puntos, y no aparece en el desplegable de nombres. Edu se puso Ammar por error: PASAR_VOTOS hace que, al abrir la app, los votos de Ammar pasen a Edu (respetando el voto de Edu si ya tenía uno ese día) y que un móvil que entró como Ammar pase a ser Edu y botón para marcar la fecha oficial (solo Pablo, y pide confirmación), que se muestra en la cabecera de todas las pestañas. Calendario seguido del lunes 16 de noviembre al 31 de diciembre de 2026 (CAL_INICIO / CAL_FIN), sin flechas, con "DIC" en el día 1 de diciembre: al tocar un día se abre una ventana para votarlo (Sí / Si hace falta / No / Veto) o dejarlo en blanco; si no era candidata, se añade al votar. Cada fecha lleva puntos con la inicial de quien vota (o 2+ letras si se repite la inicial): verde = sí, amarillo = si hace falta, gris = no, rojo = veto. Si la fecha ya tiene algún voto, detrás van "en negativo" (bolas huecas con borde gris discontinuo) los que aún no la han votado (sin los de NO_CUENTAN_EN_FALTAN); quien vota "No" ya cuenta como votado; en los días anulados no salen. En el ranking y en la ventana del día, las bolas van en rejilla de parejas (PAREJAS): en horizontal, 2 filas × 6 columnas, una columna por pareja con el chico arriba y la chica abajo (Pablo/Cris, Fran/Sofi, Edu/Ari, Gabri/Estefi, Kike/Patri y Unai solo, con el hueco de abajo vacío; en PAREJAS, null = hueco vacío), emparejando por el principio del nombre; quien no esté en las parejas sale debajo; en el calendario siguen en fila. Iniciales fijas: Pablo = PP, Patri = PF. (Desactivado por ahora con USAR_AROS = false:) si varias iniciales empiezan por la misma letra (PP/PF, Ed/Es), cada una de esas bolas lleva un aro interior de un color distinto (por orden alfabético: dorado, azul, rosa, blanco, naranja, morado) por dentro, para que todas las bolas midan lo mismo. Las iniciales van como bolas de Navidad (degradado, reflejo y gancho dorado), todas del mismo tamaño y con la letra de los títulos. Las fechas se proponen tocando el calendario. Gráfica de barras por fecha (sí, si hace falta, no, veto). Una fecha sin votos no se muestra, y al quitar el último voto se borra de datos.json (salvo la oficial).
2. **Comida**: quién trae qué, agrupado por categoría, con aviso de categorías vacías y etiqueta de alergias/notas. Cada uno puede editar o borrar lo suyo.
3. **Charlas**: apuntarse con tema, enlace opcional y casilla "por videollamada". Botón "Copiar prompt" con el contenido de prompt.md incrustado en la app y botón "Ver la guía" con el contenido de guia.md.
4. **Juegos**: solo los juegos en vivo (ver abajo). La parte de juegos de mesa se quitó; el campo `juegos` de datos.json se conserva pero no se usa.
5. **Fiesta** (para la tele, pantalla completa): sorteo del orden de las charlas, quién presenta y su tema, cronómetro de minutosCharla con pausar y siguiente, y botón "Empezar las campanadas" (12 toques, uno cada 3 segundos, con sonido generado con Web Audio y destello en pantalla). El orden sorteado y la charla actual se guardan en `fiesta` de datos.json; solo Pablo sortea. Las charlas apuntadas después del sorteo van al final. El cronómetro es local: se pone rojo en los últimos 30 s y al llegar a 0 suena un gong y parpadea. Botón "Modo tele": pantalla completa, solo la pestaña Fiesta a lo grande, y la pantalla no se apaga. Tras la campanada 12: "¡Feliz 2027!" con confeti.

## Juegos en vivo, usuario Tele y presentaciones

- Pestaña Juegos: los juegos en vivo, donde cada uno participa desde el móvil.
- Juegos en vivo (cualquiera propone preguntas o categorías y cada uno borra lo suyo):
  - **¿Quién lo dijo?**: cada uno contesta en secreto; la tele saca una respuesta al azar, el resto adivina de quién es; 1 punto por acierto.
  - **Quiz familiar**: preguntas con 4 opciones; la tele lanza una al azar y hay 20 s para contestar; el autor no contesta la suya; 1 punto por acierto.
  - **Premios Adoce**: cada uno vota un nombre por categoría; la tele revela el ganador con redoble.
  - **Predicciones 2027**: se guardan en secreto (cada uno ve solo las suyas) hasta que se revelan en la tele.
- En Fiesta, un selector elige qué se enseña en la tele (Charlas, ¿Quién lo dijo?, Quiz, Premios, Predicciones). Revelar, siguiente, lanzar y sortear: solo Pablo o la Tele.
- Usuario "📺 Tele" (última opción del desplegable): para el PC de la fiesta. No es participante, no vota y entra en Fiesta.
- Mientras hay una ronda de juego abierta, se relee cada 10 s (GitHub permite ~5.000 lecturas por hora con la misma clave, entre todos).
- Presentaciones: en Charlas → "📎 Mi presentación" cada uno sube su archivo .html o pega el código (máx. 900 KB). Se guarda en `nochebuen-datos/presentaciones/<idCharla>.html` y la charla guarda la ruta en `presentacion`. En Fiesta, "▶ Lanzar presentación" la abre a pantalla completa en un iframe con sandbox="allow-scripts" (sin allow-same-origin, para que no pueda leer la clave).

## Música y vídeos (YouTube)

- Botón 📻 en la cabecera: lista compartida de vídeos de YouTube (los especiales de Manolo Vieira, villancicos…). Cualquiera pega un enlace (youtu.be, watch?v=, shorts, live) con título opcional (si no, se intenta sacar de YouTube); no se repiten; cada uno quita los suyos.
- `videos` = [ { id, yt, titulo, autor, fecha } ] (`yt` es el código de 11 letras).
- Al pulsar uno sale un mini-reproductor flotante abajo a la derecha (⤢ agrandar, ☰ lista, ✕ cerrar) que sigue sonando al cambiar de pestaña y en el modo tele; detrás suenan el resto de vídeos de la lista. Con el móvil bloqueado o la app cerrada no sigue (YouTube no lo permite fuera de su app).

## Calendario de Adviento

- Se abre desde la tarjeta grande de arriba en Fechas (pantalla entera). 24 puertas en orden desordenado, del 1 al 24 de diciembre de 2026.
- Lo rellena el grupo: cada uno puede rellenar hasta ceil(24 / participantes) puertas (mínimo 2) con un texto de hasta 300 caracteres (los enlaces http/https se vuelven clicables). Nadie más ve el contenido hasta su día; entonces se abre y se ve quién lo escribió.
- Cada puerta puede llevar una foto (opcional). Antes de subirla se reduce en el móvil a 1280 px como máximo y se pasa a JPEG (calidad 0,78, que baja hasta 0,4 si pesa más de 700 KB). Se guarda en `nochebuen-datos/adviento/puerta-<n>.jpg`.
- `adviento` = { puertas: { "5": { autor, texto, foto, fecha } } } (`foto` es la ruta o ""). Cada uno edita o vacía las suyas mientras siguen cerradas; al vaciar o quitar la foto, se borra el archivo.

## Cartas (el tablón) y Reyes Magos

- Pestaña "💌 Cartas" en la barra inferior: se escribe una carta en papel antiguo a los Reyes Magos o a Papá Noel (campo `para`: "reyes" o "noel" en cada elemento de `sugerencias`, máx. 400 caracteres). El buzón muestra todas como sobres con sello y lacre; cada uno borra las suyas.
- Tu Rey Mago: `reyes` = { nombre: "melchor" | "gaspar" | "baltasar" }. Su cara (🤴🏻 🤴🏼 🤴🏿) sale junto al nombre en la cabecera, las cartas y el carbón.
- Carbón dulce: `carbon` = [ { id, de, para, motivo, fecha } ]. Ranking de quién tiene más; cada uno quita el que puso. También se ve en la tele (Fiesta → ⚫ Carbón).
- Roscón de Reyes (Fiesta → 👑 Roscón): al partirlo se sortean la figurita (rey de la noche) y el haba (paga el roscón); se guarda en `roscon` = { figurita, haba, fecha }. Solo Pablo o la Tele lo parten.
- Decoración de Reyes: estrella de Oriente que cruza la cabecera, caravana de los Reyes en camello en la pantalla de entrada y corona 👑 en la fecha que va ganando.

## Diseño

- Cabecera: "NocheBuenFindeaño" con "2026-2027" al lado y el subtítulo "Familia de Adoce". El nombre de la persona va en el botón "Cambiar".
- Todo compacto: los formularios de Comida, Charlas y los juegos van plegados hasta que se pulsan.
- Icono de Fechas: hoja de calendario dibujada con "DIC 24" (en vez del emoji 📅).
- Modo tele: en el móvil se ve en una columna y siempre hay un botón flotante "✕ Salir del modo tele".
- Decoración navideña: guirnalda de bombillas de colores bajo la cabecera, nieve dorada suave de fondo, botones con relieve (filo dorado, se hunden al pulsar), tarjetas-botón grandes ("hero") para las acciones importantes (Modo tele, Campanadas con marco de bombillas, Copiar prompt, Ver la guía) y en la barra inferior la pestaña activa lleva el icono en una bola de Navidad dorada.
- Animaciones básicas (aparecer al cambiar de pestaña, barras que crecen, botones que se hunden al pulsar, guirnalda que parpadea) y se desactivan si el móvil pide menos movimiento.

- Estilo retro navideño. Colores: rojo #B3261E, verde bosque #1F4D3A, crema #F6EEDC, dorado #C9A227 (decoración) y #F2C94C (texto sobre verde), texto #2A1E17, tarjetas #FFFDF7 con borde #E2D3B5.
- Títulos en Fraunces (serif retro, legible), el nombre de la fiesta en la cabecera en Shrikhand y el texto en Work Sans.
- Cabecera roja con el nombre de la fiesta y borde punteado dorado. Pestañas inferiores verdes, con la activa en dorado.
- Primero para móvil y compacto: botones de unos 36-40 px, texto normal de 15 px (mínimo 12 px en etiquetas pequeñas) y buen contraste. Los campos de texto se quedan en 16 px para que el iPhone no haga zoom.
- Confirmar antes de borrar cualquier cosa.
- Respetar la zona segura del móvil (env(safe-area-inset-*)).

## Forma de trabajar

- Al empezar cada sesión, leer este archivo y hacer solo lo que pida esa sesión.
- Antes de programar, resumir el plan en pocas líneas y preguntar si hay dudas.
- Al terminar, explicar cómo probarlo desde el móvil.
- No añadir funciones que no estén aquí sin preguntar.
- Nunca escribir la clave ni ningún token en ningún archivo.
- El botón "atrás" del móvil cierra la ventana abierta (Adviento, puerta, día del calendario, guía, presentación) como la ✕: cada ventana se apunta en el historial con abrirCapa/cerrarCapa.
- En cada cambio, subir VERSION (y VERSION_FECHA) al principio del script: es la versión que se ve debajo del banner de Papá Noel (1.27 = cambio fusionado n.º 27).
- Al terminar cada cambio, acabar la respuesta a Pablo con una línea breve de la versión, por ejemplo: "📌 Versión 1.30 — bolas en horizontal por parejas". Si solo cambian documentos y no la app, decir que la versión sigue igual.
- Pablo autoriza a hacer commit, push y fusionar PRs en este repo sin pedir permiso, salvo que diga lo contrario.
