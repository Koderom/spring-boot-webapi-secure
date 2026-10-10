# Lab 3 — Escaneo de imágenes con Trivy

**Objetivo:** identificar vulnerabilidades conocidas de una imagen Docker y priorizar su actualización.
**Duración:** 15–20 min.
**Requisitos:** Docker y conectividad al registro y a las bases de vulnerabilidades.

## Pasos (detalle)

1. Crear `reports` si no existe y preparar la caché:

   ```bash
   docker volume create trivy-cache
   ```

2. Descargar el escáner con la versión del taller:

   ```bash
   docker pull aquasec/trivy:0.74.0
   ```

3. Analizar una imagen antigua de referencia y guardar el reporte completo:

   ```bash
   docker run --rm -v "${PWD}/reports:/reports" -v trivy-cache:/root/.cache/trivy aquasec/trivy:0.74.0 image --image-src remote --scanners vuln --format json --output /reports/trivy-before.json nginx:1.24.0
   ```

4. Abrir `reports/trivy-before.json` y seleccionar dos CVEs HIGH/CRITICAL, si existen. Registrar paquete, versión instalada, versión corregida y severidad.
5. Ejecutar un control que falle ante HIGH/CRITICAL:

   ```bash
   docker run --rm -v "${PWD}/reports:/reports" -v trivy-cache:/root/.cache/trivy aquasec/trivy:0.74.0 image --image-src remote --scanners vuln --severity HIGH,CRITICAL --exit-code 1 --format table --output /reports/trivy-gate.txt nginx:1.24.0
   ```

6. Analizar una alternativa actual para comparar:

   ```bash
   docker run --rm -v "${PWD}/reports:/reports" -v trivy-cache:/root/.cache/trivy aquasec/trivy:0.74.0 image --image-src remote --scanners vuln --format json --output /reports/trivy-after.json nginx:stable
   ```

7. Comparar los reportes. Verificar si las CVEs seleccionadas desaparecieron; no asumir que `stable` carece de vulnerabilidades. Registrar el digest analizado que aparezca en los metadatos del reporte.
8. Completar `reports/lab3-resumen.md` con dos CVEs, cambio propuesto, resultado y hallazgos pendientes. Si se aplica al proyecto, seleccionar una base compatible y volver a construir y probar la aplicación.

## Entregable

`trivy-before.json`, `trivy-after.json`, `trivy-gate.txt` y `lab3-resumen.md`.

## Consejos / troubleshooting

- Se consulta la imagen remota: no hace falta montar el socket Docker ni levantar nginx.
- Los reportes se escriben en una carpeta del equipo, por lo que sobreviven a `--rm`.
- Una imagen actual puede seguir teniendo CVEs. Reducir paquetes innecesarios y actualizar la base son acciones diferentes.
- CLI local ya instalada: `trivy image --scanners vuln --format json --output reports/trivy-before.json nginx:1.24.0`.
- Para una imagen local de Spring Boot, el escáner necesita acceder a ella: exportarla con `docker save -o reports/app.tar NOMBRE_IMAGEN` y analizarla mediante `image --input /reports/app.tar`, con la misma carpeta montada.

**Documentación:** https://trivy.dev/docs/latest/guide/target/container_image/
