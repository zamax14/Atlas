# api — reglas locales

Contrato completo: `docs/components/api.md`.

- Los filtros de `/download` usan **el mismo parser que WFS** (`atlas.wfs.filters`). No dupliques
  la whitelist: acaba parcheada en un solo sitio de los dos.
- `POST /maps/export` reutiliza `wms.get_map`. No reimplementa el pipeline de render.
- `GET /layers` devuelve solo capas `enabled` sobre las que el principal tiene permiso `view`:
  filtrar el catálogo es control de acceso, no presentación.
- `/download` exige `layer.downloadable` y permiso `download`.
- Los errores salen con la forma de `core.exceptions.error_response`, igual que en WMS/WFS.
- Nada de endpoints de administración aquí todavía: son v0.2.
