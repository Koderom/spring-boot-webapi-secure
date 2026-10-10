# Lab 7 — SCA con Snyk CLI (opcional)

**Objetivo:** analizar las dependencias Maven y comparar recomendaciones con Trivy.
**Duración:** 15–20 min, con cuenta preparada.
**Requisitos:** cuenta Snyk con acceso al análisis, Node.js/npm, Java y Maven. Utiliza la CLI local; este laboratorio no requiere Docker.

## Pasos (detalle)

1. Abrir la raíz del proyecto y crear `reports` si no existe. Confirmar que `mvn -version` utiliza el Java requerido.
2. Instalar la CLI:

   ```bash
   npm install --global snyk
   ```

3. Autenticarse mediante el flujo que abra la herramienta:

   ```bash
   snyk auth
   ```

4. Registrar la versión y comprobar dependencias:

   ```bash
   snyk --version
   mvn -B clean verify
   ```

5. Analizar el POM y guardar el resultado:

   ```bash
   snyk test --file=pom.xml --json-file-output=reports/snyk-before.json
   ```

6. Revisar hasta dos hallazgos, sus rutas de dependencia y recomendaciones. Comparar con el reporte Trivy del Lab 4 si está disponible.
7. Aplicar una actualización compatible, ejecutar `mvn -B clean verify` y repetir el análisis con `--json-file-output=reports/snyk-after.json`.
8. Completar `reports/lab7-resumen.md` con versiones, identificadores, cambios, pruebas y diferencias entre escáneres. Si la cuenta no permite analizar, registrar el impedimento y utilizar el Lab 4 como alternativa.

## Entregable

`snyk-before.json`, `snyk-after.json` y `lab7-resumen.md` con comparación y remediación.

## Consejos / troubleshooting

- No guardar tokens en el POM, archivos de configuración del proyecto ni capturas.
- El acceso, los límites de análisis y el método de autenticación dependen de la cuenta.
- Códigos habituales: 0 sin vulnerabilidades, 1 con vulnerabilidades, 2 fallo técnico y 3 sin proyecto compatible.
- Para repetibilidad, fijar una versión comprobada de la CLI después de registrar la utilizada.
- Discrepancias con Trivy pueden deberse a cobertura, fuentes, severidad y fecha de análisis; comparar el mismo commit.

**Documentación:** https://docs.snyk.io/developer-tools/snyk-cli y https://docs.snyk.io/developer-tools/snyk-cli/authenticate-to-use-the-cli
