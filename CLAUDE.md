# La porra de Catalina

Web estática para una porra familiar sobre el día de nacimiento de Catalina, la hija de Víctor y Adriana. Adriana sale de cuentas el **30 de diciembre de 2026**. La web se publica en **GitHub Pages** y se comparte por el grupo de WhatsApp de la familia, así que la mayoría la abrirá en el **móvil**. Entre quienes la van a usar hay abuelos, por lo que tiene que ser muy legible y sin fricción.

Idioma: todo en **español de España**, tanto el contenido como los comentarios del código. El trato con el usuario (Víctor) también es en español.

## Reglas de la porra (lo que la web debe reflejar)

- Cada participante apuesta un día.
- **Solo gana quien acierta el día exacto.** Si varios eligieron ese día, ganan todos.
- **No hay dinero.** Se decidió explícitamente no pedir dinero a nadie: no hay bote, ni repartos, ni sección de bases. No volver a añadirlos.
- El premio es simbólico: quien gane se hace la foto con Catalina con un cartelito o un body que lleva su nombre. Por eso la web genera un **cartel imprimible**.
- Si nadie acierta, «la porra la gana Catalina» y el body dice «La gané yo».

## Estructura

```
index.html   ← toda la web: HTML + CSS + JS en un único archivo, sin build ni dependencias
assets/      ← ecografia.mp4 (vídeo en bucle, sin audio, ~370 KB), ecografia.jpg (póster), video.mp4 (original, ignorado en `.gitignore`)
             y fotos/ para las fotos de los participantes
README.md    ← instrucciones para publicar y mantener (orientado a personas)
CLAUDE.md    ← este archivo
```

No hay build, framework ni npm. Hay que mantenerlo así: un solo archivo que se pueda editar desde la interfaz web de GitHub. La única carga externa son las fuentes de Google Fonts. Si fallan, la web funciona igual con la fuente del sistema.

## Datos: el objeto `CONFIG`

Está al principio del `<script>` en `index.html`. Es lo único que debería hacer falta tocar para el día a día:

| Campo | Uso |
|---|---|
| `bebe` | `"Catalina"`. Se usa en todos los textos. |
| `fpp` | Fecha en la que sale de cuentas (`"2026-12-30"`). |
| `nacimiento` | `null` hasta que nazca. Después, `"AAAA-MM-DD"`. Este campo cambia todo el estado de la página. |
| `participantes` | `{ nombre, fecha, foto? }` con fechas ISO. El orden no importa porque se ordenan por fecha. `foto` es opcional (`"assets/fotos/x.jpg"`, cuadrada, ~120 px); sin ella se ve un círculo rosa con la inicial. |
| `fiestas` | Días señalados, con una etiqueta de texto en la celda. |

Participantes actuales (14):

| Fecha | Nombres |
|---|---|
| 21 dic | Rafael |
| 24 dic | Sandra, Jesús |
| 27 dic | Adriana |
| 28 dic | Raquel (abuela) |
| 30 dic | Ángela |
| 31 dic | Marga |
| 1 ene | Víctor |
| 2 ene | José, Javito, Sheila |
| 3 ene | Paloma |
| 5 ene | Fer |
| 6 ene | Raquel (tía) |

Hay dos Raqueles, y se distinguen como «(abuela)» y «(tía)». Los nombres deben escribirse con tilde: José, Jesús, Ángela, Víctor.

## Comportamiento

### Estados
- **Antes de nacer:** el body muestra «?». La cabecera dice cuántos días faltan para la fecha prevista o, si ya ha pasado, «N días de propina». Las apuestas cuyo día ya ha pasado aparecen tachadas (estado `out`) y las demás siguen en juego (`live`). Debajo del calendario aparece el recuento «Siguen en juego X de las 14 apuestas».
- **Después de nacer** (`nacimiento` definido): el título cambia a «Catalina ya está aquí». El body lleva los nombres de quienes han ganado, uno por línea, o «La gané yo» si no hay ganador. En el calendario los ganadores aparecen subrayados en amarillo (`win`) y el resto tachados. Aparece también el botón «Imprimir el cartel».

### Parámetros de URL para probar (no tocan el código)
- `?hoy=2026-12-29` simula la fecha de hoy.
- `?nacimiento=2027-01-02` simula el nacimiento ese día.
- Se pueden combinar.

### Fotos y ecografía
- Cada nombre lleva su foto (o la inicial). Cuando ese día ya ha pasado (`out`), la foto se pone en gris; la de quien gana lleva un aro amarillo.
- El subrayado amarillo va en un `span` inline con `box-decoration-break: clone` para que marque todas las líneas de un nombre largo como «Raquel (abuela)».
- En la cabecera hay una ecografía (`assets/ecografia.mp4`, `autoplay muted loop playsinline`) pegada con celo, a la derecha en escritorio y bajo el texto en tablet y móvil.

### Calendario
- El rango se calcula solo: va desde el lunes de la semana de la primera fecha relevante hasta el domingo de la última. Las fechas relevantes son las apuestas, la `fpp` y el nacimiento, de modo que un nacimiento fuera de rango amplía el calendario.
- En escritorio es una rejilla de 7 columnas (lunes a domingo). Por debajo de 700 px se convierte en una lista vertical de días.
- Fechas: todo se maneja como cadenas `AAAA-MM-DD` y `Date` en hora local (`parse`/`key`). No usar `new Date("2026-12-30")`, porque lo interpreta en UTC y desplaza un día.

### El body (SVG)
- La silueta es el path `BODY` (viewBox `0 0 240 262`) y se dibuja con `drawOnesie(svg, lines)`. Lleva una costura discontinua recortada con `clipPath`, broches y dos pinzas de tender.
- `fitText` ajusta el tamaño del texto para que quepa dentro, midiendo con `getComputedTextLength`. Por eso se vuelve a ejecutar tras `document.fonts.ready`.
- **El cartel de impresión** (`#poster`, visible solo en `@media print`, A4 vertical) **clona el SVG ya ajustado** de la cabecera, renombrando los ids del `clipPath`. No se dibuja de cero porque un SVG oculto no se puede medir y el texto se saldría del body.

## Diseño

El concepto es un **calendario de nevera apuntado a boli** con **un body colgado de una cuerda de tender** en la cabecera. Ese body es el elemento memorable y es literalmente el premio.

Tokens en `:root`:
- `--paper #EDF1F7` (azul sábana, fondo)
- `--ink #1E2B4A` (tinta)
- `--muted #5D6883`
- `--rule #C9D2E1`
- `--pink #F4B6C8` (el body)
- `--pen #AD456A` (círculo a mano de la fecha prevista, «Salimos de cuentas», «Nació»)
- `--marker #FFD84D` (subrayador de ganadores)

Tipografías:
- **Bricolage Grotesque** para titulares, números de día y el texto del body.
- **Atkinson Hyperlegible** para el texto, elegida por legibilidad.

Animación: una única animación de balanceo del body al cargar, que respeta `prefers-reduced-motion`.

Víctor pidió explícitamente que **no parezca generado por IA**. Hay que evitar:
- Fondo crema con serif de titular y acento terracota.
- Todo metido en tarjetas redondeadas con sombra.
- Etiquetas en MAYÚSCULAS espaciadas.
- Metadatos separados por « · ».
- Emojis como decoración.
- Una sola palabra del titular resaltada con otro color o en cursiva.

Las celdas son planas, sin sombra. Las que tienen apuesta llevan fondo blanco y las vacías se quedan en el color del papel.

## Verificación

Después de cada cambio, renderizar con Playwright (Chromium) y revisar estos casos:
- Escritorio a 1100 px: sin parámetros, con `?hoy=2026-12-29` y con `?nacimiento=2027-01-02` (tres ganadores).
- Móvil a 390 px: con `?hoy=2026-12-26` y con `?nacimiento=2026-12-26` (nadie acierta).
- `page.pdf(format="A4")` con nacimiento definido, para comprobar que el nombre cabe dentro del body en el cartel.
- Que no haya errores de JS (`pageerror`).

## Publicación

GitHub Pages: Settings → Pages → Deploy from a branch → `main`, carpeta `/ (root)`. La web es pública, así que cualquiera con el enlace ve nombres de pila y fechas. Víctor lo sabe.

## Pendiente o posible

- El día del nacimiento: poner la fecha en `CONFIG.nacimiento`, hacer commit e imprimir el cartel.
- Las reglas pueden cambiar todavía («puede haber cambios»). Si cambian, actualizar este archivo y el README.
