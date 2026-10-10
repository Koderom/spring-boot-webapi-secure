# Lab 5 — Detección de secretos con Gitleaks

**Objetivo:** detectar credenciales expuestas en archivos y distinguirlas de secretos que permanecen en el historial Git.
**Duración:** 15–20 min.
**Requisitos:** copia del repositorio y Docker.

## Pasos (detalle)

1. Crear `reports` y una carpeta `secret-demo` desde el editor. Utilizar únicamente un valor ficticio para este ejercicio.
2. Crear `.gitleaks-demo.toml` en la raíz con esta regla didáctica:

   ```toml
   title = "Regla didactica"
   [extend]
   useDefault = true

   [[rules]]
   id = "demo-token"
   description = "Token ficticio del laboratorio"
   regex = '''DEMO_TOKEN_[A-Z0-9]{16}'''
   keywords = ["DEMO_TOKEN_"]
   ```

3. Crear `secret-demo/demo.env` con este contenido ficticio:

   ```text
   API_TOKEN=DEMO_TOKEN_ABCD1234EFGH5678
   ```

4. Escanear solo la carpeta de demostración y redactar los valores en el reporte:

   ```bash
   docker run --rm -v "${PWD}:/repo" ghcr.io/gitleaks/gitleaks:latest dir /repo/secret-demo --config /repo/.gitleaks-demo.toml --redact --report-format json --report-path /repo/reports/gitleaks-before.json
   ```

5. Abrir el reporte y localizar la regla `demo-token`, el archivo y la línea. El código de salida 1 es esperado si se detecta el secreto ficticio.
6. Reemplazar el valor del archivo por `API_TOKEN=REPLACE_AT_RUNTIME` y repetir el comando guardando `gitleaks-after.json`.
7. Examinar además el historial del repositorio con las reglas predeterminadas:

   ```bash
   docker run --rm -v "${PWD}:/repo" ghcr.io/gitleaks/gitleaks:latest git /repo --redact --report-format json --report-path /repo/reports/gitleaks-history.json
   ```

8. Completar `reports/lab5-resumen.md` y retirar los archivos de demostración. Si se detecta una credencial real, revocarla o rotarla antes de resolver su exposición en Git; no incluir el valor en capturas.

## Entregable

Reportes redactados `gitleaks-before.json`, `gitleaks-after.json`, `gitleaks-history.json` y resumen de mitigación.

## Consejos / troubleshooting

- Borrar una credencial del archivo actual no la elimina de commits anteriores.
- La regla personalizada garantiza un hallazgo didáctico sin usar claves reales.
- El escaneo `dir` se limita a `secret-demo` para evitar que la propia definición de la regla u otros materiales del curso contaminen la demostración.
- CLI local: `gitleaks dir secret-demo --config .gitleaks-demo.toml --redact --report-format json --report-path reports/gitleaks-before.json`.
- Si `git /repo` falla por propiedad de directorio, ejecutar la CLI local o una imagen/configuración de Git que confíe específicamente en `/repo`; no desactivar globalmente los controles de Git.

**Documentación:** https://github.com/gitleaks/gitleaks
