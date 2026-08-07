# ADR 0002 — Pillow + shapely como renderer inicial de WMS

**Estado:** aceptado
**Fecha:** 2026-08-07

## Contexto

WMS `GetMap` (§RF-WMS-02) debe devolver PNG y JPEG. El §10 es explícito: la librería **no** debe
implementar un motor gráfico desde cero, y el renderizado debe abstraerse tras una interfaz
`MapRenderer` para poder cambiar de implementación.

El sistema de estilos del §11 es deliberadamente pequeño: relleno, borde, grosor, opacidad, radio
de punto. No hay SLD, ni etiquetas en el MVP, ni cartografía avanzada (§32).

## Decisión

La primera implementación de `MapRenderer` es `PillowRenderer`: `shapely` parsea el WKB que
devuelve PostGIS, una transformación afín lleva coordenadas del mundo a píxeles, y
`PIL.ImageDraw` dibuja polígonos, líneas y puntos.

## Alternativas descartadas

**Mapnik.** Es el motor correcto para cartografía seria y sería la elección obvia con etiquetas,
symbolizers y reglas por escala. Pero requiere binarios de sistema: complica el CI, complica el
Dockerfile de quien use Atlas, y rompe la promesa de "es una librería, `pip install` y ya". Para
tres tipos de geometría y cuatro propiedades de estilo, el coste de instalación supera al beneficio.

**Delegar a PostGIS (`ST_AsPNG`) o al cliente (MVT).** No cubre `GetMap`: el §RF-WMS-02 pide una
imagen renderizada por el servidor con bbox, tamaño y estilos arbitrarios.

## Consecuencias

- `pip install atlas-geo` no requiere nada del sistema operativo.
- Rendimiento y calidad tipográfica limitados. Aceptable mientras no haya etiquetas ni volúmenes
  altos; ambos son señales de que toca implementar `MapnikRenderer` detrás de la misma interfaz.
- `rendering/` no conoce PostGIS: recibe geometrías shapely ya proyectadas al CRS de salida.
  Eso mantiene la sustitución barata.
- Antialiasing: se dibuja a 2× y se reescala (Pillow no antialiasa polígonos). Ceiling conocido, se
  revisa si el coste de CPU molesta.
