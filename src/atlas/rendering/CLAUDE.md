# rendering — reglas locales

Contrato completo: `docs/components/rendering.md`. Decisión: `docs/adr/0002-renderer-pillow-sobre-mapnik.md`.

- **No importa `postgis` ni `layers`.** Recibe geometrías shapely ya consultadas, filtradas y
  proyectadas al CRS de salida. Esta regla es lo único que hace barato cambiar de motor gráfico.
- No reproyecta: solo aplica la transformación afín mundo → píxel. Y el eje Y se invierte.
- Pillow no antialiasa polígonos: se dibuja a `supersample`× y se reescala con LANCZOS.
- Una geometría inválida o vacía se salta; no rompe el render completo.
- Stateless y reentrante: la misma instancia sirve peticiones concurrentes.
