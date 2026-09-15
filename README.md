# Arena 13 — Infografía interactiva 360° (A-Frame)

Escenario virtual 360° hecho con A-Frame que narra, paso a paso, el duelo
entre la Liebre (ciclo rápido) y la Tortuga (control). Combina una escena 3D
navegable con una capa de infografía en HTML (texto, barras de elixir, vida
de torres y mazos) sincronizada con la historia.

## Estructura del proyecto

```
arena13-infografia/
├── index.html                    → toda la experiencia (escena A-Frame + lógica + overlay)
├── aframe.min.js                 → librería A-Frame, YA INCLUIDA (no depende de internet)
├── README.md
└── assets/
    ├── liebre.glb                 (modelo 3D de la Liebre — tú lo agregas)
    ├── tortuga.glb                 (modelo 3D de la Tortuga — tú lo agregas)
    └── arena_360.png               (cielo 360° del paisaje, ya incluido)
```

**Importante:** al copiar el proyecto, lleva SIEMPRE los 4 elementos completos
(`index.html`, `aframe.min.js`, `README.md` y la carpeta `assets/` con todo su
contenido) y respeta esa misma estructura de carpetas. Si solo copias
`index.html` suelto, la página no tendrá cómo cargar A-Frame ni las
imágenes, y vas a ver la pantalla en blanco — en ese caso, la propia página
te lo va a advertir con un aviso rojo explicando qué falta.

`aframe.min.js` va incluido en el proyecto a propósito (en vez de cargarlo
desde `aframe.io` por internet) para que la experiencia funcione siempre,
sin depender de la conexión ni de que algún firewall/antivirus bloquee ese
dominio externo.

El cielo 360° usa por defecto `assets/arena_360.png`, un paisaje con árboles,
pradera y castillos al fondo. El piso y el camino central están coloreados
para combinar con ese paisaje (verde pasto, tierra café), y la escena no
aplica niebla/oscurecido, para que el fondo se vea siempre nítido. Cada
estación de la historia se marca con una corona dorada y azul (pedestal +
banda con gemas + puntas tipo almena), construida solo con geometría de
A-Frame, sin ningún archivo de modelo. Para cambiar la imagen del cielo por
otra tuya, sigue la sección "Cómo agregar tu imagen 360°" más abajo.

`index.html` ya referencia `assets/liebre.glb` y `assets/tortuga.glb`. Mientras
no existan esos archivos, la escena funciona igual (texto, barras, cámara,
botones) y en su lugar se ve un marcador de respaldo con la forma de cada
personaje: un cono rojo para la Liebre y un caparazón verde simple para la
Tortuga. En cuanto agregues el `.glb` correspondiente, ese marcador se
oculta automáticamente.

## Cómo abrirlo en VS Code

Los navegadores bloquean por seguridad la carga de modelos `.glb` y otros
archivos locales cuando abres `index.html` directamente con doble clic
(protocolo `file://`). Por eso hace falta un servidor local:

1. Abre la carpeta `arena13-infografia` en VS Code.
2. Instala la extensión **Live Server** (Ritwick Dey) desde el Marketplace.
3. Clic derecho sobre `index.html` → **Open with Live Server**.
4. Se abrirá en el navegador en algo como `http://127.0.0.1:5500`.

(También funciona con cualquier otro servidor estático, por ejemplo
`npx serve .` o `python -m http.server`, si lo prefieres.)

## Cómo agregar tus modelos 3D

1. Crea la carpeta `assets/` junto a `index.html` si no existe.
2. Copia ahí tus archivos `liebre.glb` y `tortuga.glb` (o cambia los nombres
   en el `<a-asset-item>` dentro de `<a-assets>` si prefieres otros nombres).
3. Ajusta si hace falta, dentro de `index.html`:
   - `scale="1 1 1"` en `#personajeLiebre` / `#personajeTortuga`: sube o baja
     la escala según el tamaño real de tu modelo.
   - `rotation="0 0 0"`: gira el modelo para que quede orientado hacia el
     centro de la arena.
   - Si tu `.glb` trae animaciones (idle, ataque, etc.), puedes activarlas
     agregando el componente `animation-mixer` a la entidad, por ejemplo:
     `animation-mixer="clip: Idle"` (cambia `Idle` por el nombre real del
     clip que traiga tu modelo).

## Cómo agregar tu imagen 360°

Por defecto el cielo ya usa una imagen equirectangular real
(`assets/arena_360.png`, un paisaje con árboles y castillos). Para
reemplazarla por otra:

1. Coloca tu imagen equirectangular (proporción 2:1, por ejemplo 4096×2048)
   en `assets/`, con el nombre que prefieras (por ejemplo `arena_360.jpg`).
2. En `index.html`, dentro de `<a-assets>`, cambia el `src` de `#cielo360`
   por la ruta de tu nueva imagen.

## Cómo editar la historia

Toda la narrativa vive en el arreglo `escenas` dentro del `<script>` al final
de `index.html`. Cada objeto representa un paso y controla:

- `titulo` / `texto`: lo que se muestra en el panel inferior.
- `elixirLiebre` / `elixirTortuga` (0–10): altura de la barra de elixir de
  cada jugadora.
- `vidaLiebre` / `vidaTortuga` (0–100): vida de cada torre.
- `resaltar`: `'liebre'`, `'tortuga'` o `null` — enciende el anillo de brillo
  bajo el personaje que está actuando.
- `victoria`: `true` solo en el paso donde debe aparecer el texto 3D
  "¡VICTORIA!" sobre la torre de la Tortuga.
- `posLiebre` / `posTortuga` (opcional): posición `"x y z"` a la que se
  desplaza el personaje en ese paso (por ejemplo, el push final de la Liebre
  o el contragolpe de la Tortuga). Si se omite, el personaje mantiene su
  última posición.

Puedes agregar, quitar o reordenar pasos libremente: los puntos de progreso,
los coronas-estación de la arena, los botones Anterior/Siguiente y las
barras se recalculan solos según la longitud del arreglo (cada paso ocupa
`360 / cantidad_de_pasos` grados alrededor del centro).

## Navegación por posición (estaciones)

Cada paso de la historia tiene asignado un ángulo alrededor del centro de
la arena (la función `anguloEscena(i)` en el script), y ahí se genera
automáticamente una corona numerada ("estación"), con su tarjeta de texto
flotando encima. El diseño evita a propósito animar la *rotación* de la
cámara (A-Frame la puede pelear con `look-controls` mientras el usuario
arrastra el mouse) y en su lugar mueve su *posición*, que nunca es
sobrescrita por los controles de A-Frame. Hay tres formas de avanzar, y las
tres quedan sincronizadas entre sí sin pelearse nunca:

1. **Caminar** con las teclas **W A S D**: al entrar en el radio de una
   corona-estación, la historia de esa escena aparece sola en el panel
   inferior. La cámara no puede salir caminando del piso circular de la
   arena (límite de mundo, igual que `limite-mundo` en el ejemplo de
   referencia, adaptado a un radio en vez de una caja).
2. **Clic directo** sobre una corona/estación, sobre uno de los puntos de
   progreso de la barra superior, o sobre los botones Anterior/Siguiente:
   estos TELETRANSPORTAN a la cámara justo frente a esa estación (cambia
   `position`, nunca `rotation`), así que el cambio siempre se queda
   puesto, sin importar qué esté haciendo `look-controls` en ese momento.
3. Si quieres cambiar cuántos grados separan cada estación, no hay que
   tocar nada: se recalcula solo a partir de `escenas.length`.

## Interacción disponible

- **Mirar alrededor**: arrastrar con el mouse, o el dedo en móvil (escena
  360° real, gracias a `look-controls`) — funciona en cualquier punto del
  recorrido, sin afectar en qué escena estás.
- **Recorrer las estaciones caminando**: teclas WASD (`wasd-controls`).
- **Avanzar/retroceder la historia**: botones del panel inferior, los
  puntos de progreso, o clic directo en una corona-estación (teletransporta).
- **Ver el mazo de cada jugadora**: clic sobre el círculo dorado flotante
  encima de la Liebre o de la Tortuga.
