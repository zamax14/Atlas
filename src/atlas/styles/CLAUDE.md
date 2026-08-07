# styles — reglas locales

Contrato completo: `docs/components/styles.md`.

- Datos puros: no importa Pillow, no conoce píxeles, no dibuja nada.
- Colores hex de 3 o 6 dígitos, opacidades `0..1`, validados por Pydantic al construir el estilo,
  no al dibujar.
- El modelo es deliberadamente pequeño (§11). Etiquetas, clasificación temática y SLD están fuera:
  no los añadas "porque son fáciles".
