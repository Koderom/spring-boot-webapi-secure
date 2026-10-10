# Lab 6 — Policy as Code con Conftest

**Objetivo:** bloquear una configuración Docker Compose que permite privilegios elevados.
**Duración:** 15–20 min.
**Requisitos:** Docker y editor; no requiere desplegar Compose.

## Pasos (detalle)

1. Crear `reports` y una carpeta `policy` en la raíz del laboratorio.
2. Crear `compose-lab.yaml` como archivo de configuración independiente:

   ```yaml
   services:
     app:
       image: nginx:stable
       privileged: true
   ```

3. Crear `policy/compose.rego` con sintaxis Rego moderna:

   ```rego
   package main

   import rego.v1

   deny contains msg if {
       some name, service in input.services
       service.privileged == true
       msg := sprintf("Servicio %s: privileged=true no permitido", [name])
   }
   ```

4. Ejecutar la política sobre el archivo:

   ```bash
   docker run --rm -v "${PWD}:/project" -w /project openpolicyagent/conftest:latest test compose-lab.yaml --policy policy
   ```

5. Guardar el resultado inicial en JSON:

   ```bash
   docker run --rm -v "${PWD}:/project" -w /project openpolicyagent/conftest:latest test compose-lab.yaml --policy policy --output json > reports/conftest-before.json
   ```

6. Editar `compose-lab.yaml` y cambiar `privileged: true` por `privileged: false`. No iniciar el servicio: estamos evaluando el archivo.
7. Repetir el comando del paso 4 y luego el del paso 5 cambiando la salida a `reports/conftest-after.json`. Comprobar que desaparece el fallo de la regla.
8. Crear `reports/lab6-resumen.md` explicando política, riesgo, corrección y diferencia entre pasar esta regla y demostrar que toda la configuración es segura.

## Entregable

`compose-lab.yaml`, `policy/compose.rego`, ambos reportes y `lab6-resumen.md`.

## Consejos / troubleshooting

- Conftest evalúa archivos; no requiere Kubernetes ni iniciar contenedores de la aplicación.
- La política prueba únicamente `privileged: true`; no revisa usuarios, puertos, capacidades ni imágenes.
- Esta sintaxis usa Rego v1. No mezclarla con ejemplos antiguos que omiten `if` o `contains`.
- CLI local: `conftest test compose-lab.yaml --policy policy --output json`.
- El comando falla ante violaciones aunque genere un JSON válido. Un error de sintaxis debe resolverse antes de interpretar el resultado como una decisión de política.

**Documentación:** https://www.conftest.dev/ y https://www.conftest.dev/install/
