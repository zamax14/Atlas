# exports — reglas locales

Contrato completo: `docs/components/exports.md`.

- GeoJSON y CSV se emiten **en streaming** desde un `AsyncIterator[dict]`. Nunca se acumula el
  dataset en memoria.
- Los exportadores no consultan la base de datos ni construyen SQL: reciben filas ya producidas.
- GeoPackage y Shapefile escriben a un temporal; el temporal se borra siempre, incluso si el
  cliente aborta la descarga.
- El módulo se llama `csv_export.py`, no `csv.py`: sombrearía la stdlib.
- `fiona`/GDAL son dependencias **opcionales** (`atlas-geo[export]`). El núcleo debe importar y
  funcionar sin ellas.
