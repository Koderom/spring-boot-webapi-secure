# Lab 4 — Resumen SBOM Maven con CycloneDX y SCA con Trivy

## Identificación

| Campo | Valor |
|---|---|
| Herramientas | CycloneDX Maven Plugin 2.9.3 · Trivy 0.74.0 (`sbom --scanners vuln`) |
| Proyecto | `springboot-devsecops-lab` 1.0.0 (Spring Boot 3.5.14, Java 21) |
| SBOM | CycloneDX 1.6, JSON (`bom.json`) |
| Fecha | 2026-10-10 |

## Componentes identificados en el SBOM

| Componente | Versión | Tipo | purl |
|---|---|---|---|
| `springboot-devsecops-lab` | 1.0.0 | Aplicación | `pkg:maven/bo.edu.devsecops/springboot-devsecops-lab@1.0.0?type=jar` |
| `spring-boot-starter-web` | 3.5.14 | Directa (sin versión en POM, gestionada por el parent BOM) | `pkg:maven/org.springframework.boot/spring-boot-starter-web@3.5.14?type=jar` |
| `commons-text` | 1.9 | Directa (declarada en POM) | `pkg:maven/org.apache.commons/commons-text@1.9?type=jar` |
| `jackson-databind` | 2.21.2 | Transitiva (vía `spring-boot-starter-web`) | `pkg:maven/com.fasterxml.jackson.core/jackson-databind@2.21.2?type=jar` |

## Hallazgo seleccionado

| Campo | Valor |
|---|---|
| Componente | `org.apache.commons:commons-text` |
| Versión anterior | 1.9 |
| CVE | CVE-2022-42889 (Text4Shell) |
| Severidad | CRITICAL |
| Versión corregida | 1.10.0 |
| Tipo | Directa (declarada en `pom.xml`) |
| Uso | Sólo se usan utilidades de texto en el laboratorio; la API de interpolación de expresiones (el vector del CVE) no se invoca desde los endpoints del proyecto. **No se explotó.** |

## Comparación

| Métrica | Antes | Después |
|---|---|---|
| `commons-text` en POM | 1.9 | 1.10.0 |
| CVE-2022-42889 | presente (CRITICAL) | **ausente** |
| Total hallazgos SCA | 53 | 52 |
| HIGH/CRITICAL | 25 | 24 |
| Resultado de pruebas | — | `mvn -B clean verify` → BUILD SUCCESS (2 tests OK) |

SBOMs conservados identificados por fase: `reports/bom-before.json` (commons-text 1.9)
y `reports/bom-after.json` (commons-text 1.10.0).

## Pendientes

- 24 hallazgos HIGH/CRITICAL restantes, en su mayoría **transitivas gestionadas
  por el parent/BOM de Spring Boot** (`jackson-databind`, `jackson-core`,
  `micrometer-core`, etc.). Su corrección requiere actualizar el `parent`
  `spring-boot-starter-parent` o forzar versiones dentro de `dependencyManagement`,
  verificando compatibilidad.
- Reportes: `sca-before.json`, `sca-after.json`.

## Notas

- Se usó `JAVA_HOME=C:\Program Files\Java\jdk-21.0.12` (el JDK del sistema estaba
  en 17) y `--timeout 20m` en Trivy.
- Un hallazgo por versión no prueba explotabilidad; el alcance revisa el uso real
  de la biblioteca en la aplicación.