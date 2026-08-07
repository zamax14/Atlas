# postgis — reglas locales

Contrato completo: `docs/components/postgis.md`. Lee también `docs/components/security.md`.

- **Este es el único paquete que escribe SQL.** Si otro módulo necesita una query, se añade aquí
  una función que la componga.
- Nada de f-strings, `%`, `+` ni `.format()` sobre SQL. Identificadores con `psycopg.sql.Identifier`,
  valores con `%s`.
- Cada nombre de columna se valida contra `TableInfo.columns` antes de convertirse en `Identifier`.
  La whitelist es la validación; el `Identifier` es defensa en profundidad. Hacen falta las dos.
- Los predicados espaciales dejan la columna de geometría desnuda (`geom && ST_MakeEnvelope(...)`).
  Se transforma el bbox al SRID de la tabla, nunca la tabla al del bbox: rompe el índice GIST.
- Las consultas de descarga usan cursor server-side (`stream`). Nada de `fetchall()` sobre datasets.
- No conoce `Layer`: trabaja con `TableInfo`.
