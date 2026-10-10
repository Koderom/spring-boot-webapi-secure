# Lab 6 — Resumen Policy as Code con Conftest

## Identificación

| Campo | Valor |
|---|---|
| Herramienta | Conftest |
| Imagen | `openpolicyagent/conftest:latest` |
| Política | `policy/compose.rego` (Rego v1, `package main`) |
| Configuración evaluada | `compose-lab.yaml` |
| Fecha | 2026-10-09 |

## Política

```rego
package main

import rego.v1

deny contains msg if {
    some name, service in input.services
    service.privileged == true
    msg := sprintf("Servicio %s: privileged=true no permitido", [name])
}
```

La regla `deny` genera un mensaje por cada servicio cuyo atributo
`privileged` sea `true`. No despliega Compose: solo evalúa el archivo.

## Resultados

| Ejecución | Salida | Exit code |
|---|---|---|
| Antes (contenedores `privileged: true`) | `FAIL - compose-lab.yaml - main - Servicio app: privileged=true no permitido` — 1 test, 0 passed, 1 failure | 1 |
| Después (`privileged: false`) | `1 test, 1 passed, 0 warnings, 0 failures` | 0 |

Reportes: `reports/conftest-before.json` y `reports/conftest-after.json`.

## Riesgo controlado

Ejecutar contenedores con `privileged: true` les otorga privilegios ampliados
sobre el host (acceso a dispositivos, capacidades del kernel y otros recursos),
lo que puede ampliar el impacto de una aplicación comprometida. La política
bloquea esa configuración al integrarse al pipeline como calidad de Pull
Request.

## Corrección aplicada

`compose-lab.yaml` cambió `privileged: true` por `privileged: false`. Tras el
cambio, Conftest no reporta violaciones (`successes: 1`).

## Alcance y limitaciones

- La política únicamente verifica `privileged`. No evalúa usuarios, puertos
  publicados, capabilities, read-only filesystems ni origen de las imágenes.
- Pasar esta regla **no** demuestra que la configuración sea segura en general:
  es una verificación acotada que debe combinarse con otras políticas y
  controles.

## Entregables

- `compose-lab.yaml`
- `policy/compose.rego`
- `reports/conftest-before.json`
- `reports/conftest-after.json`
- `reports/lab6-resumen.md`