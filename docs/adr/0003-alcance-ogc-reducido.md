# ADR 0003 — Alcance OGC reducido y deliberado

**Estado:** aceptado
**Fecha:** 2026-08-07

## Contexto

WMS y WFS son estándares grandes. Implementarlos completos (todas las versiones, GML completo,
SLD, filtros OGC en XML, WFS-T) es un proyecto de años y es exactamente el criterio de no-éxito
del §43.

## Decisión

Atlas implementa **el subconjunto que un geoportal usa de verdad**:

| Servicio | Se implementa | No se implementa |
|---|---|---|
| WMS 1.3.0 | `GetCapabilities`, `GetMap`, `GetFeatureInfo` (JSON) | `GetLegendGraphic`, SLD, `DescribeLayer`, versiones 1.0/1.1 |
| WFS 2.0.0 | `GetCapabilities`, `DescribeFeatureType`, `GetFeature` (GeoJSON) | GML como salida principal, `Transaction`, `LockFeature`, stored queries, filtros OGC en XML |
| Filtros | operadores `= != > >= < <= IN LIKE` sobre columnas whitelisted, `bbox` espacial | `<fes:Filter>` XML, operadores espaciales avanzados, funciones |

La respuesta de `GetFeature` es GeoJSON (§RF-WFS-03), no GML. Los filtros se expresan como
parámetros de query estilo `poblacion__gt=100000` (§15.2), no como XML de Filter Encoding.

## Consecuencias

- Atlas **no es un servidor WFS conforme**. Un cliente de escritorio que exija GML o Filter
  Encoding XML puede no funcionar. Es un compromiso aceptado: el consumidor objetivo es un
  frontend web (OpenLayers, MapLibre, Leaflet), que habla GeoJSON de forma nativa.
- El `GetCapabilities` es válido y suficiente para que un cliente descubra capas, CRS y extents,
  pero declara solo las operaciones realmente soportadas.
- Cualquier issue que proponga ampliar esta tabla necesita pasar antes por el §45 y por un ADR
  nuevo. Que un estándar tenga una operación no es razón para implementarla.
