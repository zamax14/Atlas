# Roadmap

Cada fase es un milestone en GitHub. Las fases 0–4 tienen issues abiertos; las versiones
posteriores existen como milestone sin issues hasta que haya necesidad real.

## v0.1.0 — MVP

El criterio de éxito es el §42 de `requirements.md`: registrar una tabla PostGIS, verla, consultarla,
filtrarla, descargarla y exportarla como PNG, sin levantar GeoServer.

| Milestone | Contenido | Resultado |
|---|---|---|
| **Fase 0 — Fundaciones** | pyproject, docker-compose con PostGIS, CI, fixtures, spike | El stack elegido está validado y hay dónde correr tests |
| **Fase 1 — Núcleo** | config, excepciones, pool, introspección, `Layer`, `LayerRegistry`, CRS, permisos, logging, `Atlas.router`, `GET /layers` | `geo.register_layer("geo.municipios")` funciona |
| **Fase 2 — WFS** | GetCapabilities, DescribeFeatureType, GetFeature, bbox, paginación, filtros, selección de propiedades | El geoportal puede consultar datos vectoriales |
| **Fase 3 — WMS** | GetCapabilities, GetMap, GetFeatureInfo, estilos de polígono/línea/punto, PNG/JPEG, `POST /maps/export` | El geoportal puede visualizar y exportar mapas |
| **Fase 4 — Exportación** | GeoJSON y CSV en streaming, `GET /layers/{layer}/download` con filtros, GeoPackage, Shapefile ZIP | El usuario puede bajarse los datos |

## v0.2 — Administración

Catálogo persistente en PostgreSQL (tabla `atlas_layers`, §8) y endpoints para registrar, editar,
activar/desactivar capas, cambiar estilo y configurar permisos. Habilita una GUI administrativa.

**Se implementa cuando** haya más de una aplicación consumiendo Atlas o cuando registrar capas en
código deje de ser práctico.

## v0.3 — Optimización

Caché (metadata, query, imagen), WMTS (`GetCapabilities`, `GetTile`), vector tiles MVT vía
`ST_AsMVT`, simplificación geométrica por zoom, rate limiting, métricas Prometheus (§27).

**Se implementa cuando** existan datos de uso real que lo justifiquen. Optimizar antes es adivinar.

## v0.4 — Raster

WCS (`GetCapabilities`, `DescribeCoverage`, `GetCoverage`) sobre GeoTIFF/COG/PostGIS Raster con
rasterio y rio-tiler.

**Se implementa cuando** exista una necesidad real de trabajar con raster (DEM, NDVI, ortofotos).

## Fuera de alcance permanente

`WFS-T`, `LockFeature`, stored queries, SLD completo, GML completo, WPS, CSW, Oracle/MySQL/ArcSDE,
Shapefile o GeoPackage **como datasource**, edición remota, motor propio de reproyección, motor
propio de rendering, procesamiento raster propio.

Antes de aceptar cualquiera de estos, se aplica el principio rector del §45: *¿esta capacidad es
necesaria para publicar, visualizar, consultar o descargar información de PostGIS desde una
aplicación web?*
