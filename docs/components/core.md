# core

> Módulo: `src/atlas/core/`

## Propósito

Piezas transversales que todo el resto usa: configuración, jerarquía de excepciones, tipos
compartidos, registro de CRS y enganche de logging. No contiene lógica de negocio ni SQL.

## Contrato público

```python
# core/config.py
class AtlasConfig(BaseModel):
    database_url: str
    default_limit: int = 1000        # features devueltos si el cliente no pide límite
    max_limit: int = 10000           # techo absoluto para consultas paginadas
    max_download_features: int | None = None   # None = sin límite en descargas por streaming
    default_srid: int = 4326
    pool_min_size: int = 1
    pool_max_size: int = 10
    query_timeout: float = 30.0      # segundos

# core/exceptions.py
class AtlasError(Exception):
    status_code: int = 500
    code: str = "atlas_error"        # identificador estable, ej. "layer_not_found"
    def __init__(self, message: str, **details: Any) -> None
    message: str                     # texto para quien lee la respuesta
    details: dict[str, Any]          # contexto estructurado y público; {} si no hay

# jerarquía plana: un solo nivel bajo AtlasError
class LayerNotFound(AtlasError):        404, "layer_not_found"
class ServiceDisabled(AtlasError):      404, "service_disabled"   # capa existe pero wms/wfs off
class UnsupportedCRS(AtlasError):       400, "unsupported_crs"
class UnsupportedFormat(AtlasError):    400, "unsupported_format"
class InvalidParameter(AtlasError):     400, "invalid_parameter"  # OGC Missing/InvalidParameterValue
class InvalidFilter(AtlasError):        400, "invalid_filter"
class InvalidGeometry(AtlasError):      400, "invalid_geometry"
class QueryTooLarge(AtlasError):        413, "query_too_large"
class QueryTimeout(AtlasError):         504, "query_timeout"
class DatabaseError(AtlasError):        500, "database_error"
    PUBLIC_MESSAGE: str              # lo que se le dice al cliente, diga lo que diga el driver
    technical_message: str           # el mensaje real de psycopg, solo para el log
class PermissionDenied(AtlasError):     403, "permission_denied"

def error_response(exc: AtlasError) -> dict   # {"error": {"code", "message", "details"}}

# core/crs.py
SUPPORTED_CRS: dict[int, CRSInfo]    # 4326 y 3857 en v0.1
def parse_crs(value: str) -> int     # "EPSG:3857" | "urn:ogc:def:crs:EPSG::3857" | "3857" -> 3857
def require_crs(srid: int) -> CRSInfo  # lanza UnsupportedCRS

# core/types.py
BBox = tuple[float, float, float, float]     # minx, miny, maxx, maxy
GeometryType = Literal["Point", "LineString", "Polygon", "MultiPoint", ...]

# core/logging.py
class OperationLog(BaseModel):
    layer: str | None; operation: str; execution_time: float
    feature_count: int | None; user: str | None; error: str | None
LogHook = Callable[[OperationLog], None]
```

## Invariantes

- `core` no importa nada de otro paquete de Atlas. Es la hoja del grafo de dependencias.
- Toda excepción que pueda llegar al usuario hereda de `AtlasError` y tiene `status_code` y `code`.
  El `code` es parte del contrato público: no se renombra sin bump de versión.
- Ningún `code` se repite entre excepciones: es lo que el cliente usa para distinguir el fallo.
- `error_response` devuelve siempre la misma forma, con `details` como dict —vacío si no hay
  contexto—, para que el cliente nunca tenga que comprobar si la clave existe.
- `details` es payload **público**. Nada que el cliente no deba ver entra ahí: el mensaje de psycopg
  vive en `DatabaseError.technical_message`, fuera de la respuesta, porque nombra esquemas, tablas
  y a veces valores del query.
- Ningún límite se aplica fuera de `AtlasConfig`. Nada de constantes mágicas dispersas.
- El logging es un hook opcional que la app host provee. Atlas nunca configura `logging.basicConfig`
  ni escribe a stdout por su cuenta.
- Los CRS soportados viven en un único diccionario. Añadir uno es añadir una entrada, no un `if`.

## Dependencias

Importa: nada del proyecto. Solo `pydantic` y stdlib.
Lo importan: todos.

## Fuera de alcance

- Motor de reproyección propio — las transformaciones son `ST_Transform` en `postgis/`.
- Autenticación, sesiones o usuarios (§17).
- Caché (v0.3).

## Referencias

`requirements.md` §20 (CRS), §23 (límites), §25 (logging), §26 (errores).
