# wms — reglas locales

Contrato completo: `docs/components/wms.md`.

- WMS 1.3.0 y solo 1.3.0. En EPSG:4326 el orden de ejes del BBOX es **lat,lon**: se normaliza en el
  parseo, antes de nada más, y hay un test que lo fija.
- `WIDTH`/`HEIGHT` tienen techo. Una petición gigante es 400, no un OOM.
- Orden de dibujo: la primera capa de `LAYERS` al fondo, la última arriba.
- Este paquete **no dibuja**: llama a `MapRenderer.render()` con geometrías ya proyectadas.
- Se filtra con el bbox transformado al SRID de la tabla y se proyecta la salida en el `SELECT`.
- `image/jpeg` ignora `TRANSPARENT`: se compone sobre `BGCOLOR`.
- Permiso `view` para GetMap, `query` para GetFeatureInfo.
