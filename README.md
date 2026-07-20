# Bitácora Serrana

Diorama vivo del Valle de Calamuchita. Un solo archivo HTML, sin build tools, que muestra el cielo real de Villa Ciudad Parque en tiempo real: sol, luna, estrellas de verdad, clima en vivo, y un calendario de lo que pasa en la sierra.

No es un juego ni un panel de control — es más un bicho ambiental para dejar abierto en una pestaña, como un acuario que en realidad es el cielo de tu pueblo.

---

## Qué hace

- **Cielo 360°**: arrastrás con el mouse o el dedo para girar y mirar alrededor. Hay una brújula arriba que te dice hacia dónde estás mirando. Si no tocás nada, la cámara gira sola despacio.
- **Sol y luna reales**: posición calculada en el momento, no animaciones de mentira. El color del cielo es un degradé continuo según la altura real del sol sobre el horizonte.
- **~900 estrellas reales**, con las más brillantes (Sirio, Canopus, Cruz del Sur, Achernar, etc.) etiquetadas por nombre.
- **Meteoros**: aparecen al azar de noche, más seguido durante los días reales de pico de las lluvias de meteoros del año.
- **Un satélite simulado** que cruza el cielo durante el crepúsculo (comportamiento típico, no es tracking real del ISS).
- **Clima real** de Villa Ciudad Parque (Open-Meteo, sin API key) que dispara lluvia, niebla, nubes o nieve.
- **Cartel "Hoy en la sierra"**: feriados nacionales, fiestas del valle, efemérides, cercanía a solsticio/equinoccio, y "visitas especiales" que cargás vos mismo con el botón **+**.

---

## Cómo usarlo

Es un solo archivo. Abrilo con doble clic o subilo a cualquier hosting (FTP/cPanel, como el resto de tus proyectos) — no necesita build, ni Node, ni dependencias instaladas. Los fonts y el clima se piden a internet (Google Fonts + Open-Meteo), así que necesita conexión para verse completo; sin conexión igual funciona, pero sin datos de clima y con la tipografía por defecto del sistema.

Las "visitas especiales" que cargás se guardan en `localStorage` del navegador — quedan ahí una vez que lo subís a tu dominio. En el preview de Claude no persisten entre sesiones, eso es esperable.

---

## Cómo está armado (para cuando lo edites)

Todo vive en un único `<script>` al final del archivo, organizado en bloques comentados, en este orden:

1. **Ubicación** — `LAT`/`LON` de Villa Ciudad Parque. Cambialo acá si algún día lo adaptás a otro pueblo.
2. **Catálogo estelar** — array `STAR_FIELD` (RA, Dec, magnitud) sacado del Yale Bright Star Catalog, y `NAMED_STARS` con las estrellas etiquetadas.
3. **Lluvias de meteoros** — array `METEOR_SHOWERS` con las fechas de pico del año.
4. **Astronomía** — funciones de posición solar, lunar y de cualquier estrella (RA/Dec → altura/azimut), fase lunar.
5. **Cielo** — el degradé de color según la altura del sol.
6. **Cámara 360°** — rumbo, inclinación, arrastre, proyección de coordenadas celestes a la pantalla.
7. **Clima** — fetch a Open-Meteo y los efectos visuales que dispara (lluvia, niebla, nubes, nieve, bichitos de noche despejada).
8. **Render** — todo lo que se dibuja en el canvas cada frame: estrellas, meteoros, satélite, sol/luna, sierra, brújula.
9. **Calendario local** — `FERIADOS_2026`, `FIESTAS_CALAMUCHITA`, `EFEMERIDES`. Son arrays de objetos simples, fáciles de tocar sin entender el resto.
10. **Panel de visitas especiales** — el botón **+** y su lógica de guardado.
11. **Loops principales** — actualización de datos (cada 1s) y el loop de dibujo (cada frame).

### Cosas fáciles de personalizar

- **Paleta**: variables CSS al principio del `<style>` (`--night`, `--dusk`, `--gold`, `--teal`, `--ember`, `--parchment`).
- **Calendario 2026**: son arrays planos, se agregan o sacan líneas sin tocar lógica.
- **Sensibilidad del arrastre / límites de inclinación**: en el bloque "Cámara 360°", variables `FOV` y los clamps de `pitch` dentro de `onMove`.
- **Frecuencia de meteoros**: `baseChance` dentro de `maybeSpawnMeteor`.

---

## Precisión de los datos — qué es real y qué es aproximado

| Elemento | Cómo se calcula | Precisión |
|---|---|---|
| Posición del sol | Fórmulas astronómicas estándar (altura/azimut reales) | Alta, apta para el uso ambiental |
| Fase lunar (nombre, % iluminado) | Ciclo sinódico real desde una luna nueva de referencia | Alta |
| Posición de la luna en el cielo | Fórmula lunar simplificada de baja precisión | ±0.3° aprox., de sobra para esto |
| Estrellas | Catálogo real Yale Bright Star, ~900 estrellas hasta magnitud 4.5 | Posiciones reales, sin corrección por movimiento propio (no se nota a simple vista) |
| Clima | Open-Meteo, datos en vivo | Real |
| Feriados 2026 / fiestas del valle | Cargados a mano, verificados al armar el archivo | Revisar si el calendario oficial cambia algo sobre la marcha |
| Satélite | Simulado, no rastrea el ISS real | Solo estético |
| Meteoros | Aparición al azar, con frecuencia real según calendario de lluvias | La aparición individual es al azar, las fechas de pico son reales |

---

## Ideas para una v3 (si en algún momento querés seguir)

- **Incendios/alertas en vivo**: existe NASA FIRMS (focos de calor por satélite), pero pide una API key gratuita y no es un fetch directo — queda pendiente.
- **Fechas hiperlocales** (fundación del pueblo, aniversarios puntuales de Villa Ciudad Parque): no las cargué porque no tenía una fuente confiable — se agregan a mano con el botón **+**
- Un modo "compartir vista" que guarde el rumbo actual en la URL.

---

## Créditos de datos

- Posiciones estelares: [Yale Bright Star Catalog](https://github.com/brettonw/YaleBrightStarCatalog)
- Clima: [Open-Meteo](https://open-meteo.com/) (API gratuita, sin key)
- Fórmulas astronómicas: algoritmos estándar de posición solar/lunar de baja precisión, de uso público
