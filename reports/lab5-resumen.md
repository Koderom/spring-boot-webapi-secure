# Lab 5 — Resumen detección de secretos con Gitleaks

## Identificación

| Campo | Valor |
|---|---|
| Herramienta | Gitleaks (`ghcr.io/gitleaks/gitleaks:latest`) |
| Regla didáctica | `.gitleaks-demo.toml` → `demo-token` |
| Alcance | `secret-demo/` (dir) y historial Git (git, reglas por defecto) |
| Fecha | 2026-10-10 |

## Regla didáctica

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

Valor de prueba (ficticio): `DEMO_TOKEN_ABCD1234EFGH5678`.

## Resultados

| Ejecución | Comando | Resultado | Exit |
|---|---|---|---|
| Antes | `dir secret-demo` con regla `demo-token` | **1 leak** (`demo-token`, `demo.env:1`, Match `REDACTED`) | 1 |
| Después | mismo, valor `REPLACE_AT_RUNTIME` | **no leaks** (`[]`) | 0 |
| Historial | `git /repo` con reglas por defecto | **1 leak**: `generic-api-key` en `reports/trivy-before.json:73` (commit `d52103a`) | 1 |

La redacción (`--redact`) funciona: el campo `Secret`/`Match` aparece como
`REDACTED` en los reportes.

## Hallazgo del historial (falso positivo)

Gitleaks marcó `reports/trivy-before.json` con la regla `generic-api-key` por la
cadena `NGINX_GPGKEY=...`. Es la **clave GPG pública** que la imagen oficial de
nginx expone como variable de entorno en su Dockerfile; quedó incrustada en el
reporte de Trivy del Lab 3 y, al commitearlo, entra al historial Git.

- **No es una credencial real** → no requiere rotación.
- **Lección:** una vez que un valor entra a un commit, borrar el archivo actual
  no lo elimina del historial. Los reportes de escáneres pueden contener cadenas
  que disparan reglas heurísticas; conviene redactarlos o excluirlos antes de
  versionarlos.

## Mitigación propuesta

- Añadir un `.gitleaks.toml` (o `.gitleaksignore`) que excluya `reports/**` o
  registre el fingerprint del falso positivo, en lugar de desactivar reglas.
- Evitar versionar reportes de escáner crudos; redactar o almacenar como
  artefactos de CI.
- Si se detectara una credencial real: **revocarla/rotarla primero** y luego
  limpiar el historial (p. ej. `git filter-repo`), nunca al revés.
- Integrar Gitleaks como pre-commit hook o job de CI para PRs.

## Entregables

- `reports/gitleaks-before.json`
- `reports/gitleaks-after.json`
- `reports/gitleaks-history.json`
- `reports/lab5-resumen.md`

Archivos de demostración (`secret-demo/`, `.gitleaks-demo.toml`) retirados tras
el ejercicio.