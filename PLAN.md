# Plan de montaje en 3 sesiones

## Sesión 0: preparación (desde Chrome)
- [x] Repo privado nochebuen-datos con datos.json
- [ ] CLAUDE.md, prompt.md, guia.md y PLAN.md en el repo de la app
- [ ] GitHub Pages activado: Settings → Pages → Branch: main → / (root) → Save
- [ ] Clave del grupo: token fine-grained, 90 días, solo nochebuen-datos, Contents: Read and write. Guardado en mis notas, nunca en el repo.

## Sesión 1: base y fechas

```
Lee CLAUDE.md. En esta sesión crea index.html con: la base (lectura y escritura de datos.json con la API de GitHub, enlace con #k=, pantalla sin clave, desplegable de nombre con la opción "Añadirme", reintentos por conflicto y relectura cada 30 s), la cabecera, las 5 pestañas (las que no son Fechas solo con "Próximamente") y el módulo de FECHAS completo. Este repo se llama [NOMBRE_REPO_APP]. Antes de programar, resúmeme el plan.
```

Prueba: abre https://paniaguadediego-bit.github.io/[NOMBRE_REPO_APP]/#k=TU_TOKEN en dos móviles. Añádete con "Añadirme" en uno, añade una fecha y vota. Comprueba que en el otro aparece en menos de 30 segundos y que en nochebuen-datos hay un commit nuevo.

## Sesión 2: comida, charlas y juegos

```
Lee CLAUDE.md. Fechas ya funciona. En esta sesión construye los módulos COMIDA, CHARLAS (con prompt.md y guia.md incrustados) y JUEGOS, reutilizando la misma lógica de guardado. Antes de programar, resúmeme el plan.
```

Prueba: apunta un plato, una charla y un juego desde dos móviles a la vez y comprueba que no se pierde ninguno.

## Sesión 3: modo fiesta y pulido

```
Lee CLAUDE.md. En esta sesión construye el modo FIESTA (sorteo, cronómetro, campanadas) y haz un repaso final: textos, contraste, zona segura del móvil, mensajes de error y que nada se rompa si datos.json tiene campos vacíos. Antes de programar, resúmeme el plan.
```

Prueba: abre la pestaña Fiesta en el portátil conectado a la tele, sortea el orden, deja correr el cronómetro y lanza las campanadas.

## Mensaje para el grupo (cuando funcione)

> 🎄 Ya tenemos app para la NocheBuenFindeaño. Abrid este enlace (es la llave, no lo compartáis fuera del grupo): [ENLACE CON #k=]
> Elegid vuestro nombre (o pulsad "Añadirme"), votad la fecha y apuntad qué traéis. Si os animáis con una charla de 3 minutos, en la pestaña Charlas tenéis el prompt y la guía. 🥂

## Si algo falla
- "Clave no válida": el token ha caducado o no tiene permiso Contents en nochebuen-datos. Crea uno nuevo y manda otro enlace.
- Pages no carga: espera 1-2 minutos tras cada cambio y revisa Settings → Pages.
- Datos raros: en nochebuen-datos → datos.json → History puedes recuperar cualquier versión.
- El enlace se ha filtrado: borra el token en GitHub (deja de funcionar al instante), crea otro y manda un enlace nuevo.
