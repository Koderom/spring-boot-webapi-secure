# Lab 1 — SAST con Semgrep

**Objetivo:** revisar el código Java del proyecto Spring Boot y analizar posibles patrones inseguros.
**Duración:** 15–20 min.
**Requisitos:** código fuente, Docker y acceso al registro de reglas.

## Pasos (detalle)

1. Abrir la raíz del proyecto y comprobar que existe `src/main/java`. Crear `reports` si no existe.
2. Descargar la herramienta:

   ```bash
   docker pull semgrep/semgrep:latest
   ```

3. Ejecutar el análisis de Java y guardar el reporte:

   ```bash
   docker run --rm -v "${PWD}:/src" -w /src semgrep/semgrep:latest semgrep scan --config p/java --json --output /src/reports/semgrep-report.json src/main/java
   ```

4. Abrir `reports/semgrep-report.json`. Revisar `results` y también `errors`; localizar archivo, línea, regla y mensaje. Si no hay resultados, registrarlo sin inventar vulnerabilidades.
5. Seleccionar hasta tres hallazgos y revisar las líneas afectadas en el editor. Explicar si son aplicables al comportamiento real del endpoint.
6. Corregir un hallazgo confirmado en una rama del laboratorio y ejecutar `mvn test`.
7. Repetir el análisis conservando el resultado en otro archivo:

   ```bash
   docker run --rm -v "${PWD}:/src" -w /src semgrep/semgrep:latest semgrep scan --config p/java --json --output /src/reports/semgrep-after.json src/main/java
   ```

8. Comparar ambos reportes y completar `reports/lab1-resumen.md` con regla, riesgo, corrección y resultado. Si no había hallazgos, documentar alcance y limitaciones.

## Entregable

`semgrep-report.json`, `semgrep-after.json` y `lab1-resumen.md` con hasta tres hallazgos o una explicación del resultado sin hallazgos.

## Consejos / troubleshooting

- No requiere ejecutar la aplicación ni una cuenta para este análisis comunitario.
- El registro de reglas requiere conectividad. Un error al descargar reglas no es un análisis exitoso.
- Para usar una CLI ya instalada: `semgrep scan --config p/java --json --output reports/semgrep-report.json src/main/java`.
- Para que el comando falle ante hallazgos, agregar `--error`; sin esa opción, encontrar resultados no implica por sí solo un código de salida de fallo.

**Documentación:** https://semgrep.dev/docs/getting-started/quickstart
