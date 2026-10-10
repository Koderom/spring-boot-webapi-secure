# Lab 3 — Resumen escaneo de imágenes con Trivy

## Identificación

| Campo | Valor |
|---|---|
| Herramienta | Trivy 0.74.0 (imagen `aquasec/trivy:0.74.0`) |
| Objetivo | `nginx:1.24.0` (antes) → `nginx:stable` (después) |
| Escaneo | `image --image-src remote --scanners vuln` |
| Fecha | 2026-10-10 |
| Digest analizado (ImageID) | Antes `sha256:6c0218f1...904e5` · Después `sha256:3a8edbda...57e0e60` |

## Resultados generales

| Imagen | Base | Vulnerabilidades | HIGH/CRITICAL |
|---|---|---|---|
| `nginx:1.24.0` | Debian 11.9 (bullseye) | 747 | 192 |
| `nginx:stable` | Debian 13.7 (trixie) | 330 | 70 |

Gate HIGH/CRITICAL sobre `nginx:1.24.0` → **fail (exit 1, 192 hallazgos)**,
registrado en `reports/trivy-gate.txt`.

## Hallazgos seleccionados (antes)

| CVE | Severidad | Paquete | Versión instalada | Versión corregida |
|---|---|---|---|---|
| CVE-2024-45491 | CRITICAL | libexpat1 | 2.2.10-2+deb11u5 | 2.2.10-2+deb11u6 |
| CVE-2024-2961 | HIGH | libc6 | 2.31-13+deb11u8 | 2.31-13+deb11u9 |

- **CVE-2024-45491 (libexpat)**: desreferencia de puntero en el analizador XML de
  Expat en entornos específicos; riesgo de DoS.
- **CVE-2024-2961 (glibc/libc6)**: escritura fuera de límites dentro del rango
  del búfer de salida de `iconv` al convertir a ISO-2022-CN-EXT; puede provocar
  lectura/escritura en memoria del proceso y, en condiciones, ejecución.

## Comparación antes/después

- Las dos CVEs seleccionadas **no aparecen** en `nginx:stable`.
- El número total de vulnerabilidades se reduce 747 → 330 (-56 %) y las
  HIGH/CRITICAL 192 → 70 (-64 %).
- La diferencia principal es la base: Debian 11 (EOL, sin soporte de seguridad)
  frente a Debian 13 (soporte vigente). No asumir que `stable` está limpio:
  todavía reporta 70 HIGH/CRITICAL.

## Cambio propuesto

Migrar la imagen de referencia a una base soportada (`nginx:stable` o una
versión fija reciente con base con soporte), reconstruir la aplicación y
repetir `mvn clean verify` además del escaneo. Reducir paquetes innecesarios y
actualizar la base son acciones complementarias.

## Hallazgos pendientes

- `nginx:stable` mantiene 70 hallazgos HIGH/CRITICAL (paquetes como `libgd3`,
  `libgnutls30`, `libxml2`, `openssl`). Requiere seguimiento con actualización
  de paquetes o una imagen más reducida.

## Notas

- Escaneo remoto: no fue necesario el socket Docker ni levantar `nginx`.
- Se usó `--timeout 20m` para evitar el error `context deadline exceeded` del
  análisis de capas en este entorno.
- Reportes: `trivy-before.json`, `trivy-after.json`, `trivy-gate.txt`.