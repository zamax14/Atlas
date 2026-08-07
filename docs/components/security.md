# security

> Transversal. Código en `src/atlas/wfs/filters.py`, `src/atlas/postgis/queries.py`,
> `src/atlas/layers/permissions.py`.

## Propósito

Reunir en un solo sitio las dos superficies de riesgo de Atlas: **inyección SQL** (porque acepta
nombres de columna y filtros desde el query string) y **control de acceso** (porque publica datos
que pueden no ser públicos). Cualquier PR que toque estas rutas se revisa contra este documento.

## Superficie 1 — Inyección SQL

Atlas construye SQL dinámico a partir de entrada del usuario en tres puntos:

| Entrada | De dónde viene | Cómo se protege |
|---|---|---|
| Nombre de tabla y esquema | Configuración del desarrollador, no del usuario final | `sql.Identifier(schema, table)` |
| Nombres de columna (`propertyName`, filtros, CSV) | Query string | Whitelist contra `layer.attribute_names` → `sql.Identifier` |
| Operadores de filtro | Query string | Whitelist `wfs.filters.OPERATORS`; nada fuera del dict existe |
| Valores de filtro | Query string | Placeholder `%s`, casteado al tipo de la columna |
| BBOX, CRS, limit, offset | Query string | Parseados a `float`/`int` por Pydantic; el CRS contra `SUPPORTED_CRS` |

Reglas no negociables:

1. Nunca f-string, `%`, `+` ni `.format()` sobre una cadena SQL.
2. Un identificador que no supere la whitelist es `InvalidFilter` (400), no un `Identifier` de todas
   formas. La whitelist es la validación, el `Identifier` es la defensa en profundidad — hacen falta
   las dos.
3. La whitelist se deriva de la introspección real de la tabla, no de una lista escrita a mano.
4. `LIKE` recibe el patrón como parámetro; no se construye el patrón en SQL.
5. Los tests de filtros incluyen casos ofensivos explícitos: `?nombre'; DROP TABLE--=x`,
   `?propertyName=pg_sleep(10)`, `?poblacion__union=...`. Si el test no está, el PR no pasa.

## Superficie 2 — Autorización

Atlas **no autentica** (§17). Recibe un `principal` de la app host y pregunta a un
`PermissionChecker` que la app host provee.

Cuatro operaciones independientes (§18), comprobadas por capa:

| Operación | Rutas afectadas | Permiso convencional |
|---|---|---|
| `view` | `GET /layers`, `WMS GetMap`, `GetCapabilities` | `atlas.<layer>.view` |
| `query` | `WFS GetFeature`, `WMS GetFeatureInfo`, `GET /layers/{layer}` | `atlas.<layer>.query` |
| `download` | `GET /layers/{layer}/download` | `atlas.<layer>.download` |
| `admin` | endpoints de administración (v0.2) | `atlas.<layer>.admin` |

Reglas:

1. Sin checker configurado, todo se permite. Es un default explícito y documentado, no un descuido:
   Atlas puede usarse en un portal 100 % público.
2. Con checker configurado, **el fallo es cerrado**: si el checker lanza una excepción, se deniega.
3. Los capabilities y `GET /layers` filtran por permiso. Una capa que el usuario no puede ver
   tampoco debe aparecer en el catálogo — un 403 sobre una capa listada ya filtra información.
4. Una capa que no pasa el permiso responde igual que una capa inexistente en catálogos, y con 403
   en acceso directo.

## Superficie 3 — Agotamiento de recursos

Menos glamurosa, igual de real: una petición puede tumbar el proceso sin ninguna intención maliciosa.

- `default_limit` / `max_limit` acotan toda consulta paginada (§23).
- `WIDTH`/`HEIGHT` de `GetMap` tienen techo.
- `statement_timeout` se aplica en cada conexión; un timeout es 504, no una conexión colgada.
- Las descargas van en streaming con cursor server-side (§24).

## Fuera de alcance

Atlas **no** protege contra esto, y es responsabilidad de la aplicación host:

- **Autenticación.** Atlas recibe un `principal` ya validado. No verifica tokens ni sesiones.
- **Rate limiting y protección DoS.** Los límites de arriba acotan una petición individual, no el
  número de peticiones. Va en el reverse proxy o en un middleware de la app.
- **HTTPS, CORS y cabeceras de seguridad.** Son de la app host.
- **Permisos a nivel de fila o de columna.** El grano mínimo es la capa. Si una capa tiene filas
  que no todos pueden ver, se publica una vista con el filtro aplicado y se registra esa vista.
- **Cifrado en reposo y gestión de secretos.** La `database_url` llega ya resuelta.
- **Auditoría.** El `LogHook` da la información (§25), pero almacenarla y retenerla es de la app.

## Referencias

`requirements.md` §13.1 (seguridad de filtros), §13.2 (operadores), §17 (auth), §18 (autorización),
§23 (límites), §26 (errores).
