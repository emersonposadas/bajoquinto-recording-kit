# Plantilla de grabación de bajo quinto en Ardour

Contexto actualizado el 22 de noviembre de 2026. Proyecto de referencia: `La Cosecha.ardour`, Ardour 9, sesión a 48 kHz. Este archivo documenta el flujo reutilizable; la sesión actual contiene una grabación y no es una plantilla vacía.

## Equipo y conexiones

- Bajo quinto conectado por cable desde su pastilla a una Focusrite Scarlett Solo 4th Gen, entrada de instrumento Inst/Line.
- Entrada guardada: `Split Scarlett Solo 4th Gen Input 1 Inst/Line:monitor_AUX0`. Los nombres pueden cambiar en otro equipo o al reconectar la interfaz; confirmar el medidor antes de grabar.
- Salida actual: audio integrado de la computadora, no auriculares en la Scarlett.
- Todos los canales deben salir exclusivamente al Master. Master L/R conecta a `ALC293 Analog:playback_FL` y `ALC293 Analog:playback_FR` respectivamente.
- No conectar los canales directamente a la salida física: eso elude el Master y puede producir diferencias entre escucha y exportación.

## Canales y balance de partida

| Canal | Función | Fader | Estado guardado |
|---|---|---|---|
| Audio 1 | Canción original de referencia | −12 dB | Silenciado |
| La Cosecha - editada | Copia del acompañamiento original con MDA Degrade | −12 dB | Activo |
| BajoQuinto | Grabación mono de la Scarlett, centrada | −2 dB | Activo |
| Master | Mezcla estéreo hacia la computadora | −3 dB | Activo |

Usar solo uno de los dos acompañamientos a la vez. Armar para grabación únicamente BajoQuinto. Los valores de fader son un punto de partida, no una normalización válida para cualquier canción o pastilla.

## Efectos

### BajoQuinto: ACE EQ

Efecto activo antes del fader. Ajustes suaves para dar presencia al instrumento:

- Estante grave: 70 Hz, −2 dB.
- Banda 1: 300 Hz, −1,5 dB, ancho 1,2.
- Banda 2: 1000 Hz, 0 dB.
- Banda 3: 1800 Hz, +1,5 dB, ancho 1,2.
- Banda 4: 6000 Hz, 0 dB.
- Estante agudo: 7500 Hz, −1 dB.
- Ganancia interna del EQ: −1 dB.

No aplicar MDA Degrade al instrumento. Ajustar EQ a oído con una toma real; no afirmar que estos ajustes son óptimos universalmente.

### Acompañamiento: MDA Degrade

Procesamiento en tiempo real dentro de Ardour sobre el audio original; no requiere un WAV degradado como fuente. Los carriles de automatización pueden ocultarse. Para un preset fijo, mantener la automatización en Off.

Presets instalados en `~/.lv2/LaCosecha-MDA-Degrade.lv2/`:

| Nivel | Nombre | headroom | quant | rate | post_filt | non_lin | output |
|---|---|---:|---:|---:|---:|---:|---:|
| 0 | Minimo | .98 | .95 | .51 | .95 | .51 | .40 |
| 1 | Suave | .94 | .80 | .555 | .80 | .53 | .40 |
| 2 | Actual | .90 | .65 | .60 | .65 | .55 | .40 |
| 3 | Medio | .866666667 | .566666667 | .64 | .616666667 | .583333333 | .383333333 |
| 4 | Marcado | .833333333 | .483333333 | .68 | .583333333 | .616666667 | .366666667 |
| 5 | Intenso | .80 | .40 | .72 | .55 | .65 | .35 |

Valores normalizados del plugin, no porcentajes de velocidad. El nivel 5 equivale al antiguo preset Agresivo. El nivel 0 tiene degradación mínima, no es bypass. No se garantiza que ninguna degradación evite reconocimiento o bloqueos de plataformas.

## Flujo para otra canción

1. Crear una nueva sesión o plantilla desde Ardour, conservando canales, conexiones y efectos. No sobrescribir esta grabación de referencia.
2. Sustituir las regiones de ambos acompañamientos por la nueva canción, alineadas en la misma posición. Mantener silenciado uno de ellos.
3. Crear una lista de reproducción vacía en BajoQuinto. Conservar las tomas anteriores para comparación.
4. Confirmar entrada, salida estéreo y nivel real de los rasgueos fuertes. Dejar margen de entrada; el fader no corrige una señal que satura en la Scarlett.
5. Grabar una prueba corta; comprobar el balance y después grabar completa.
6. Ajustar el rango de sesión/exportación al final real de la interpretación para no truncar el instrumento ni dejar silencio innecesario.
7. Guardar y exportar el Master con un único acompañamiento activo.

## Exportación confirmada compatible con Instagram Edits

**Usar el exportador de Ardour: WAV PCM firmado de 16 bits, estéreo, 48 kHz.**

El usuario confirmó que las cinco versiones exportadas por Ardour en este formato se importaron en Edits sin ruido. Las versiones anteriores de 24 bits procesadas externamente produjeron ruido blanco al importarlas. No se determinó la causa exacta; no asumir que todos los WAV de 24 bits fallan.

Herramientas disponibles: `ardour9-export` y `ardour9-lua`. Ejemplo de exportación:

```sh
ardour9-export -b 16 -s 48000 -o /ruta/salida.wav /ruta/sesion 'Nombre de snapshot'
```

El nombre de snapshot se pasa sin `.ardour`. El exportador usa el rango de sesión y las salidas del Master. No activar normalización automática a 0 dBFS sin revisar los picos. No convertir el archivo después de exportarlo salvo necesidad explícita y comprobación de compatibilidad.

## Sincronización con video

La duración total diferente no implica drift. Separar:

- Desfase fijo: desplazar la pista.
- Drift: ajustar duración/velocidad conservando el tono; comprobar un mismo rasgueo al principio y al final.

Para `20261122_120137.mp4`, la estimación inicial fue 100,31 % de velocidad y unos 8,46 s de desfase inicial. Es específica de esa captura, no una configuración para futuras canciones. Hubo pequeñas variaciones residuales; no se certificó sincronía visual perfecta.

Se generaron cinco pruebas: 99,69 %, 100 %, 100,15 %, 100,31 % y 100,47 %, sin relleno de silencio. Las versiones compatibles están en `export/Velocidades Ardour PCM16/`.

Para obtenerlas se usó el estiramiento nativo de Ardour mediante `ARDOUR.LuaAPI.Rubberband`, con ratio de duración `1 / factor_de_velocidad` y pitch `1`, y luego el exportador de Ardour. Se trabajó sobre sesiones temporales, sin modificar el proyecto principal. No igualar arbitrariamente la duración del audio a la del video.

## Precauciones al editar el proyecto

- Antes de modificar el XML desde fuera, el usuario debe guardar y cerrar Ardour. Guardar una sesión abierta después puede sobrescribir los cambios externos; ya ocurrió con ACE EQ y la entrada Scarlett.
- Hacer respaldo en `backup/` antes de cambiar la sesión.
- No volver a aplicar incrementos relativos de fader de forma acumulativa. Leer el estado actual primero.
- No borrar tomas ni medios originales para limpiar el repositorio.
- Los presets locales dentro de `presets/` pueden ser versiones antiguas: comparar con los seis presets instalados antes de distribuirlos.

## Repositorio GitHub

No subir WAV ni otros medios. El `.gitignore` excluye audio, video, exportaciones, cachés y respaldos. Mantener localmente los medios originales y respaldarlos por separado.

El repositorio sin medios permite compartir la configuración, pero no reconstruye la canción grabada por sí solo. Las sesiones pueden contener nombres/rutas locales: revisarlos antes de publicar. `.gitignore` no elimina archivos ya rastreados; comprobar el contenido preparado para commit. No se ha creado ni publicado ningún repositorio como parte de esta tarea.
