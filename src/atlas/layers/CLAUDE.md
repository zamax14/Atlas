# layers — reglas locales

Contrato completo: `docs/components/layers.md`.

- `Layer.attributes` **nunca** contiene la columna de geometría: es la whitelist que protege el SQL.
- El registry es un `dict` en memoria y no persiste nada. La persistencia es v0.2 y se construirá
  encima de esta interfaz sin cambiarla — no la anticipes.
- Atlas no decide permisos: llama al `PermissionChecker` de la app host y respeta su respuesta.
  Sin checker configurado, todo se permite (default explícito).
- Las cuatro operaciones (`view`, `query`, `download`, `admin`) se comprueban por separado.
- Una capa con `enabled=False` no aparece en ningún capabilities ni responde a nada.
- No importa `wms`, `wfs`, `api` ni `exports`.
