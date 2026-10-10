# Lab 2 — DAST baseline con OWASP ZAP y Juice Shop

**Objetivo:** explorar una aplicación vulnerable y obtener alertas mediante el análisis pasivo de ZAP baseline.
**Duración:** 20–25 min.
**Requisitos:** Docker; este laboratorio utiliza Juice Shop como objetivo dedicado.

## Pasos (detalle)

1. Crear `reports` en la carpeta de trabajo y una red compartida:

   ```bash
   docker network create devsecops-lab
   ```

2. Levantar Juice Shop. El puerto queda accesible solo desde el equipo local:

   ```bash
   docker run --rm -d --name juice-shop --network devsecops-lab -p 127.0.0.1:3000:3000 bkimminich/juice-shop
   ```

3. Abrir `http://localhost:3000` y confirmar que aparece la aplicación. Si no está lista, revisar `docker logs juice-shop` y volver a comprobar.
4. Ejecutar ZAP en la misma red y guardar HTML y JSON:

   ```bash
   docker run --rm --network devsecops-lab --user root -v "${PWD}/reports:/zap/wrk:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://juice-shop:3000 -m 1 -T 5 -r zap-report.html -J zap-report.json
   ```

5. Abrir `reports/zap-report.html`. Seleccionar hasta tres alertas y registrar URL, riesgo, evidencia y solución sugerida.
6. Distinguir alertas de configuración, como cabeceras ausentes, de vulnerabilidades de lógica de negocio. Explicar qué rutas fueron exploradas y cuáles podrían faltar.
7. Crear `reports/lab2-resumen.md` con los hallazgos y una propuesta de mitigación. Si el objetivo es Juice Shop, documentar las correcciones propuestas; no modificar el contenedor para simular una remediación persistente.
8. Detener la aplicación y retirar la red si ningún otro laboratorio la utiliza:

   ```bash
   docker stop juice-shop
   docker network rm devsecops-lab
   ```

## Entregable

`zap-report.html`, `zap-report.json` y `lab2-resumen.md` con hasta tres alertas, evidencia y mitigación.

## Consejos / troubleshooting

- `-r` y `-J` reciben nombres relativos a `/zap/wrk`, no rutas como `/zap/wrk/report.html`.
- Baseline realiza exploración y análisis pasivo; no es un escaneo activo completo ni garantiza descubrir toda una SPA.
- Códigos de salida: `0` sin FAIL/WARN, `1` al menos un FAIL, `2` WARN sin FAIL y `3` error técnico. Revisar el reporte aunque aparezca código 2.
- La red compartida evita `--network host` y `host.docker.internal`. Si la red o el contenedor ya existe, reutilizarlo o retirarlo antes de crear otro.
- `--user root` evita problemas comunes de escritura del directorio montado en esta práctica; en Linux los reportes pueden quedar propiedad de root. Puede sustituirse por permisos de carpeta apropiados y el usuario predeterminado de ZAP.

**Documentación:** https://www.zaproxy.org/docs/docker/baseline-scan/
