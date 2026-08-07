# core — reglas locales

Contrato completo: `docs/components/core.md`.

- `core` no importa nada de otro paquete de Atlas. Es la hoja del grafo. Si necesitas importar
  `postgis` o `layers` aquí, el código no va en `core`.
- Toda excepción hereda de `AtlasError` y define `status_code` y `code`. El `code` es contrato
  público: no se renombra.
- Los límites viven solo en `AtlasConfig`. Cero constantes mágicas fuera.
- Añadir un CRS es añadir una entrada a `SUPPORTED_CRS`, no un `if`.
- Sin `logging.basicConfig` ni prints: el logging es un hook que provee la app host.
