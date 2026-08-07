# styles

> Módulo: `src/atlas/styles/`

## Propósito

Modelo de estilos propio y deliberadamente pequeño (§11). Describe cómo se pinta una capa sin
depender de SLD ni de ningún motor de renderizado concreto.

## Contrato público

```python
# styles/models.py
class PolygonStyle(BaseModel):
    fill: str = "#3178c6"            # hex
    fill_opacity: float = 0.7        # 0..1
    stroke: str = "#ffffff"
    stroke_width: float = 1.0
    stroke_opacity: float = 1.0

class LineStyle(BaseModel):
    stroke: str = "#3178c6"
    stroke_width: float = 2.0
    stroke_opacity: float = 1.0

class PointStyle(BaseModel):
    radius: float = 5.0
    fill: str = "#3178c6"
    fill_opacity: float = 1.0
    stroke: str = "#ffffff"
    stroke_width: float = 1.0

Style = PolygonStyle | LineStyle | PointStyle

def default_style(geometry_type: GeometryType) -> Style
def parse_color(value: str) -> tuple[int, int, int]     # "#3178c6" -> (49, 120, 198)
def rgba(color: str, opacity: float) -> tuple[int, int, int, int]
```

## Invariantes

- Los colores son hex de 3 o 6 dígitos. Un valor inválido lanza `ValidationError` al construir el
  estilo, no al dibujar.
- Las opacidades están en `0..1`, validadas por Pydantic. El renderer las convierte a `0..255`.
- El estilo se elige por el tipo de geometría de la capa; `MultiPolygon` usa `PolygonStyle`, etc.
- `styles/` es datos puros: no importa Pillow, no conoce píxeles, no dibuja.

## Dependencias

Importa: `core`.
Lo importan: `layers`, `rendering`, `wms`, `api`.

## Fuera de alcance

- SLD, SE, symbolizers, reglas por escala (§32).
- Etiquetas (`field`, `font`, `size`, `halo`) — §11.4 las difiere explícitamente. Cuando lleguen,
  serán un `LabelStyle` opcional en cada estilo, no un rediseño.
- Estilos temáticos / clasificación por atributo (coropletas). Fuera del MVP.
- Rampas de color y leyendas.

## Referencias

`requirements.md` §11 (sistema de estilos), §11.4 (etiquetas, diferido), §32 (fuera de alcance).
