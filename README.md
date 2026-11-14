# cicd-pipeline-python

La aplicación requiere `FLASK_SECRET_KEY` para firmar los tokens CSRF. Configura esta variable en el entorno de ejecución; no guardes su valor en el repositorio. Puedes generar una clave con `python -c "import secrets; print(secrets.token_hex(32))"`.

Manuela Jimenez Arredondo manujimeneza | David Moreno Moran davidgmoreno (Propietario repositorio) | Angélica Martínez Carmona amcangelica

## Evidencias de la sección 9

- Pull request: [#1](https://github.com/davidgmoreno/cicd-pipeline-python/pull/1)
- Ejecución fallida del pipeline por el error intencional: [ver ejecución](https://github.com/davidgmoreno/cicd-pipeline-python/actions/runs/43618026686/job/132517267855?pr=1)
