# rendering

> Módulo: `src/atlas/rendering/`

## Propósito

Convierte geometrías en imágenes. Es la frontera que permite cambiar de motor gráfico sin tocar
`wms/` (§10). La implementación de v0.1 es Pillow + shapely (ADR 0002).

## Contrato público

```python
# rendering/base.py
class RenderLayer(BaseModel):
    geometries: Sequence[BaseGeometry]   # shapely, YA en el CRS de salida
    style: Style

class MapRenderer(Protocol):
    def render(
        self, layers: Sequence[RenderLayer], *, bbox: BBox, width: int, height: int,
        image_format: Literal["png", "jpeg"] = "png",
        transparent: bool = False, background: str = "#FFFFFF",
    ) -> bytes: ...

def world_to_pixel(bbox: BBox, width: int, height: int) -> Callable[[float, float], tuple[float, float]]

# rendering/pillow.py
class PillowRenderer:
    def __init__(self, supersample: int = 2) -> None
    def render(...) -> bytes      # implementa MapRenderer
```

## Invariantes

- **`rendering/` no importa `postgis` ni `layers`.** Recibe geometrías shapely ya consultadas,
  filtradas y proyectadas. Esta regla es lo único que hace barato sustituir el motor.
- Las geometrías llegan en el CRS de salida; el renderer solo aplica la transformación afín
  mundo → píxel. No reproyecta.
- El eje Y se invierte: `miny` del bbox es la fila `height` de la imagen, no la `0`.
- `PillowRenderer` dibuja a `width*supersample` y reescala con `LANCZOS`. Pillow no antialiasa
  polígonos, y sin esto los bordes salen escalonados.
- `transparent=True` solo aplica a PNG; con JPEG se compone sobre `background`.
- Una geometría inválida o vacía se salta, no rompe el render de toda la petición.
- El renderer es stateless y reentrante: la misma instancia sirve peticiones concurrentes.

## Dependencias

Importa: `core`, `styles`. Externas: `Pillow`, `shapely`.
Lo importan: `wms`, `api` (`/maps/export`).

## Fuera de alcance

- Consultar la base de datos.
- Etiquetas y colisión de etiquetas (§11.4).
- Simplificación geométrica por zoom — es v0.3 y se hará en `postgis/` con `ST_Simplify`,
  no aquí.
- Mapnik: es una implementación futura de la misma `Protocol`, en `rendering/mapnik.py`.

## Referencias

`requirements.md` §10 (renderizado WMS), §11 (estilos). ADR 0002.
