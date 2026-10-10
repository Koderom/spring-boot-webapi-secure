# Guías prácticas de DevSecOps — ejecución local y Docker

Proyecto de referencia: Spring Boot con Maven. Cada laboratorio tiene ocho pasos y conserva el formato de los documentos proporcionados: objetivo, duración, pasos, entregable y consejos.

## Preparación común

Abrir la terminal en la raíz del proyecto. Usar Docker Desktop con contenedores Linux en Windows/macOS, o Docker Engine en Linux. Los comandos Docker están en una sola línea y utilizan "${PWD}", compatible con Bash y PowerShell. Para Git Bash, si hay conversión automática de rutas, usar WSL o PowerShell.

Crear una carpeta `reports` desde el editor o con `mkdir reports`. No repetir el comando si ya existe. Para Maven Wrapper, sustituir `mvn` por `./mvnw` en Linux/WSL o `.\mvnw.cmd` en PowerShell. Ejecutar los laboratorios sobre una copia del proyecto y conservar los reportes anteriores antes de volver a analizar.

Las imágenes `latest`/`stable` simplifican la práctica, pero pueden cambiar. Registrar la versión o digest utilizado; para repetir exactamente el laboratorio, fijar un tag o digest comprobado. Los tiempos no incluyen descargas iniciales. Un reporte vacío requiere comprobar alcance y ejecución, y no demuestra ausencia de riesgos.

## Laboratorios

| Archivo | Área | Duración |
|---|---|---|
| 01-semgrep.md | SAST de código Java | 15–20 min |
| 02-zap.md | DAST baseline de Juice Shop | 20–25 min |
| 03-trivy-imagenes.md | Vulnerabilidades de imágenes | 15–20 min |
| 04-sbom-sca.md | Inventario Maven y análisis SCA | 20–25 min |
| 05-gitleaks.md | Detección de secretos | 15–20 min |
| 06-conftest.md | Policy as Code | 15–20 min |
| 07-snyk.md | SCA con autenticación, opcional | 15–20 min |

Dependabot no se incluye como laboratorio local: es un servicio de GitHub que propone PRs. Estas guías complementan su configuración en el repositorio.

