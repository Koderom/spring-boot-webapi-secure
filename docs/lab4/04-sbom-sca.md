# Lab 4 — SBOM Maven con CycloneDX y SCA con Trivy

**Objetivo:** inventariar dependencias de Spring Boot y buscar vulnerabilidades conocidas a partir del SBOM.
**Duración:** 20–25 min.
**Requisitos:** proyecto Maven, Java compatible y Docker; no se necesita iniciar Spring Boot.

## Pasos (detalle)

1. Abrir la raíz del proyecto, crear `reports` si no existe y comprobar las dependencias:

   ```bash
   mvn dependency:tree
   ```

2. Compilar y ejecutar pruebas:

   ```bash
   mvn -B clean verify
   ```

3. Generar el SBOM sin necesidad de modificar el POM:

   ```bash
   mvn -B org.cyclonedx:cyclonedx-maven-plugin:2.9.3:makeAggregateBom -DoutputFormat=json -DoutputName=bom -DschemaVersion=1.6 -DincludeTestScope=false
   ```

4. Abrir `target/bom.json`. Identificar la aplicación, una dependencia directa y una transitiva; revisar nombre, versión y `purl`.
5. Analizar el SBOM y guardar el reporte:

   ```bash
   docker run --rm -v "${PWD}/target:/work" -v "${PWD}/reports:/reports" -v trivy-cache:/root/.cache/trivy aquasec/trivy:0.74.0 sbom --scanners vuln --format json --output /reports/sca-before.json /work/bom.json
   ```

6. Seleccionar un hallazgo y revisar la versión corregida. Actualizar la dependencia directa o el parent/BOM de Spring Boot que administra la transitiva, comprobando compatibilidad. Si no hay hallazgos, documentar el alcance y evaluar una actualización disponible sin inventar CVEs.
7. Repetir `mvn -B clean verify`, el comando del paso 3 y el escaneo del paso 5 cambiando el nombre de salida a `/reports/sca-after.json`. Conservar el SBOM anterior antes de regenerarlo.
8. Completar `reports/lab4-resumen.md` con componente, CVE, versión anterior/posterior, pruebas ejecutadas y comparación. Conservar ambos SBOMs identificados por commit.

## Entregable

SBOM anterior y posterior, `sca-before.json`, `sca-after.json` y `lab4-resumen.md`.

## Consejos / troubleshooting

- El SBOM es un inventario; el reporte de vulnerabilidades es un archivo distinto.
- Regenerar el SBOM después de cada cambio de dependencias.
- Para el caso didáctico del taller, `commons-text:1.9` permite estudiar CVE-2022-42889. `1.10.0` es su corrección histórica mínima, no una recomendación de versión vigente. No es necesario ejecutar la vulnerabilidad.
- La presencia de una versión afectada no prueba explotabilidad del endpoint; revisar cómo se usa la biblioteca.
- CLI local: `trivy sbom --scanners vuln --format json --output reports/sca-before.json target/bom.json`.

**Documentación:** https://cyclonedx.github.io/cyclonedx-maven-plugin/ y https://trivy.dev/docs/latest/guide/target/sbom/
