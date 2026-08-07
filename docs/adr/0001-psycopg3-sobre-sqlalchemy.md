# ADR 0001 — psycopg3 async como capa de acceso a PostgreSQL

**Estado:** aceptado
**Fecha:** 2026-08-07

## Contexto

Atlas necesita ejecutar SQL espacial (`ST_AsGeoJSON`, `ST_AsBinary`, `ST_Transform`, `ST_Intersects`,
`ST_Extent`) sobre tablas y vistas arbitrarias descubiertas en tiempo de ejecución, con nombres de
tabla y columna que llegan de configuración y de query strings. El §13.1 exige queries
parametrizadas y whitelist de columnas y operadores.

## Decisión

Usar `psycopg[binary,pool]` v3 en modo async directamente, componiendo SQL con `psycopg.sql`
(`sql.SQL`, `sql.Identifier`, `sql.Composed`).

## Alternativas descartadas

**SQLAlchemy Core + asyncpg.** Ventaja real: si la aplicación host ya usa SQLAlchemy, podría
compartir el engine. Pero Atlas no mapea entidades — construye SQL espacial dinámico sobre tablas
que no conoce en tiempo de compilación, que es justo donde un ORM deja de ayudar y empieza a
estorbar. Añadiría dos dependencias y una capa de traducción para no ganar nada.

**GeoAlchemy2.** Resuelve el mapeo de tipos geométricos a modelos declarativos. Atlas no tiene
modelos declarativos: sus "modelos" son metadatos descubiertos por introspección.

## Consecuencias

- Una sola dependencia de base de datos.
- `psycopg.sql.Identifier` escapa identificadores correctamente, que es exactamente la primitiva
  que necesita la whitelist de columnas.
- El pool (`psycopg_pool.AsyncConnectionPool`) lo gestiona Atlas, no la app host. Si en el futuro
  hay que aceptar un pool externo, se introduce un `Protocol` de conexión — no antes.
- Todo el SQL vive en `src/atlas/postgis/`. Ningún otro paquete escribe SQL.
