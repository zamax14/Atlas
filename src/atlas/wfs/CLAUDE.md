# wfs — reglas locales

Contrato completo: `docs/components/wfs.md`. Lee también `docs/components/security.md`.

- **Todo filtro pasa por `filters.parse_filters()` + `filters.compile_filters()`.** Jamás construyas
  un `WHERE` fuera de ahí, ni aquí ni en `api/`.
- Columna fuera de `layer.attribute_names` u operador fuera de `OPERATORS` → `InvalidFilter` (400).
- Los valores se castean al tipo de la columna antes de ejecutar. Un valor inválido es 400, nunca
  un 500 de PostgreSQL.
- El límite efectivo es `min(count, config.max_limit)`, por defecto `config.default_limit`.
- Salida GeoJSON en streaming, `application/geo+json`. Nada de GML ni de Filter Encoding XML.
- Solo lectura: nada de Transaction, LockFeature ni stored queries.
