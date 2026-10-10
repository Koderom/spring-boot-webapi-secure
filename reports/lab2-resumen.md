# Lab 2 — Resumen DAST baseline con OWASP ZAP

## Identificación

| Campo | Valor |
|---|---|
| Herramienta | OWASP ZAP (baseline) |
| Imagen | `ghcr.io/zaproxy/zaproxy:stable` |
| Objetivo | `http://juice-shop:3000` (Juice Shop, `bkimminich/juice-shop:latest`) |
| Red de laboratorio | `devsecops-lab` |
| Fecha | 2026-10-09 |
| Código de salida | 2 (WARN sin FAIL; revisar reporte) |
| Resumen | 0 FAIL / 8 WARN / 59 PASS |

## Alcance explorado

El spider del baseline solo descubrió rutas de primer nivel y archivos estáticos:

`/`, `/robots.txt`, `/sitemap.xml`, `/assets/public/favicon_js.ico`, `/styles.css`,
`/main.js`, `/scripts.js`, `/polyfills.js`, `/ftp`, y archivos protegidos de `/ftp/`
(`coupons_2013.md.bak`, `package.json.bak`, `encrypt.pyc`, `suspicious_errors.yml`).

Juice Shop es una SPA con rutas gestionadas por el cliente (alerta *Modern Web
Application*); por eso el spider estándar no explora las rutas de negocio
(login, carrito, perfil, etc.). Para cubrirlas se necesitaría el Client/Spider o
un escaneo activo autenticado, fuera del alcance de este baseline.

## Alertas seleccionadas (hasta 3)

### 1. Content Security Policy (CSP) Header Not Set — Medium (High)

- **URL:** `http://juice-shop:3000` (y 4 más)
- **Evidencia:** ausencia de cabecera `Content-Security-Policy` en la respuesta.
- **Detalle:** sin CSP el navegador no puede restringir las fuentes de scripts,
  estilos u otros recursos cargados por la página, lo que facilita XSS e
  inyección de contenido.
- **Mitigación:** configurar el servidor/aplicación para enviar la cabecera
  `Content-Security-Policy` con las fuentes permitidas.

### 2. Cross-Domain Misconfiguration (CORS) — Medium (Medium)

- **URL:** `http://juice-shop:3000` (y 4 más)
- **Evidencia:** `Access-Control-Allow-Origin: *`
- **Detalle:** la política CORS abierta permite que cualquier origen cargue y
  lea recursos del sitio, pudiendo exponer datos en orígenes no confiables.
- **Mitigación:** restringir `Access-Control-Allow-Origin` a un conjunto de
  dominios conocidos (o eliminar las cabeceras CORS) y no servir datos
  sensibles sin autenticación.

### 3. Cross-Origin-Embedder-Policy Header Missing or Invalid — Low (Medium)

- **URL:** `http://juice-shop:3000` (y 4 más)
- **Evidencia:** ausencia de cabecera `Cross-Origin-Embedder-Policy`.
- **Detalle:** más que una vulnerabilidad, es una deficiencia de
  *hardening*: sin esta cabecera la aplicación no aísla documentos de
  recursos cross-origin y pierde mitigaciones de COOP/COEP.
- **Mitigación:** enviar `Cross-Origin-Embedder-Policy: require-corp` en las
  respuestas de documentos, si la aplicación es compatible.

## Distinción de tipos de alerta

- Las tres alertas son **de configuración** (cabeceras HTTP ausentes o
  incorrectas). No representan una explotación de lógica de negocio.
- No se encontraron fallos de nivel High/Medium explotables en el alcance
  explorado, pero esto **no** implica que Juice Shop sea segura: el baseline no
  descubre rutas de una SPA, no ejecuta escaneo activo ni autenticación, y el
  laboratorio está diseñado para entrenar DAST sobre una app intencionalmente
  vulnerable.

## Rutas que podrían faltar

Login y registro, carrito, perfil de usuario, foros y reseñas, búsqueda de
productos, retos/directorio del scoreboard y archivos de `/ftp` permitidos.

## Notas

- Mitigaciones propuestas **no** se aplican al contenedor para no simular una
  remediación persistente en el laboratorio (objetivo fue solo Juice Shop).
- Reportes completos: `zap-report.html` y `zap-report.json` en esta carpeta.