# cicd-pipeline-python

La aplicación requiere `FLASK_SECRET_KEY` para firmar los tokens CSRF. Configura esta variable en el entorno de ejecución; no guardes su valor en el repositorio. Puedes generar una clave con `python -c "import secrets; print(secrets.token_hex(32))"`.
