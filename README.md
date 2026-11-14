# cicd-pipeline-python

La aplicación requiere `FLASK_SECRET_KEY` para firmar los tokens CSRF. Configura esta variable en el entorno de ejecución; no guardes su valor en el repositorio. Puedes generar una clave con `python -c "import secrets; print(secrets.token_hex(32))"`.

## Integrantes

Manuela Jimenez Arredondo manujimeneza | David Moreno Moran davidgmoreno (Propietario repositorio) | Angélica Martínez Carmona amcangelica

## Evidencias de la sección 9

- Pull request: [#1](https://github.com/davidgmoreno/cicd-pipeline-python/pull/1)
- Ejecución fallida del pipeline por el error intencional: [ver ejecución](https://github.com/davidgmoreno/cicd-pipeline-python/actions/runs/43618026686/job/132517267855?pr=1)

## Respuestas

### 3. Steps principales del workflow de GitHub Actions

1. **Checkout:** descarga el código del repositorio en el runner para que los pasos siguientes puedan acceder a los archivos del proyecto. Trae el historial completo para que SonarCloud identifique el código nuevo.
2. **Set up Python:** instala Python 3.13, la misma versión del `Dockerfile`, para que las herramientas y pruebas corran en un entorno controlado y consistente.
3. **Install dependencies:** instala las dependencias de desarrollo de `requirements-dev.txt` (que incluye `requirements.txt`) para poder ejecutar la aplicación, los linters y las pruebas.
4. **Run Black (Formatter):** verifica con `--check` que el código tenga el formato estándar, sin modificarlo. Si hay archivos sin formatear, el pipeline se detiene. Se hace para mantener un estilo uniforme y fácil de leer.
5. **Run Pylint (Linter):** analiza el código en busca de errores y malas prácticas y genera `pylint-report.txt`. Con `--exit-zero` los hallazgos no detienen el pipeline; el reporte lo consume SonarCloud.
6. **Run Flake8 (Linter):** revisa el estilo según PEP 8 y genera `flake8-report.txt` para SonarCloud, también sin detener el pipeline por los hallazgos.
7. **Run Unit and Component Tests:** ejecuta las pruebas unitarias y de componente con una cobertura mínima de 80 %. Se hace para comprobar que cada funcionalidad sigue funcionando después de los cambios y para generar `coverage.xml`.
8. **Build Docker image:** construye la imagen con la aplicación y sus dependencias, etiquetada con el SHA del commit. Así se genera un artefacto portable listo para desplegar.
9. **Start container:** inicia un contenedor con la imagen recién construida y espera a que la aplicación responda. Se hace para validar que la aplicación arranca dentro de su entorno de despliegue.
10. **Run Acceptance Tests:** ejecuta las pruebas de Selenium contra el contenedor en ejecución, para verificar que el sistema se comporta como espera el usuario.
11. **Show container logs on failure:** si algún paso falló, muestra los logs del contenedor para facilitar el diagnóstico.
12. **Upload Test Reports Artifacts:** guarda los reportes generados (cobertura, pruebas, linters), incluso si el pipeline falló, para poder revisarlos después.
13. **SonarCloud Scan:** envía el código y los reportes a SonarCloud, que analiza calidad, seguridad y cobertura. El paso falla si el Quality Gate no está en `Passed`.
14. **Login to Docker Hub:** autentica el workflow en Docker Hub con el usuario (variable) y el token (secreto), solo en un push a `main`.
15. **Push Docker image:** agrega la etiqueta `latest` a la imagen ya probada y publica ambas etiquetas (SHA y `latest`) en Docker Hub, solo en un push a `main` y solo si todo lo anterior pasó.

En resumen, el workflow descarga el código, valida formato y calidad, ejecuta las pruebas, construye una imagen Docker, comprueba que funcione, la analiza con SonarCloud y, solo cuando el cambio ya está en `main`, publica esa misma imagen en Docker Hub.

### 4. Problemas o dificultades al implementar el taller

**Manuela Jiménez Arredondo:** Mi primer problema fue el entorno de Python. Al instalar las dependencias, pip no encontraba `flake8==7.5.0`. Revisando el error vi que esa versión pide Python 3.11 o superior y mi entorno virtual se había creado con el Python 3.10 que tenía en pyenv. Borré la carpeta `venv`, la volví a crear con Python 3.13 y ahí sí se instaló todo.

El segundo fue el puerto 5000: al correr la app en mi Mac salía "Address already in use" porque AirPlay Receiver usa ese puerto. Lo resolví cambiando temporalmente el puerto de la app y de las pruebas de aceptación a 5001, y entendí que el 5000 solo importa para la ejecución local, porque el contenedor y el pipeline usan el 8000.

El tercero fue en GitHub Actions: el paso de `docker build` falló con `invalid tag "/cicd-pipeline-python"`. Al ver el log, `IMAGE_NAME` empezaba con una barra, o sea que la variable `DOCKERHUB_USERNAME` estaba vacía porque no se había guardado en el repositorio. La creé de nuevo y aprendí la diferencia entre secretos y variables, y que conviene comprobar con `gh variable list` que de verdad existen.

**Angélica Martínez Carmona:** Al principio algunos comandos no se ejecutaban. Lo solucioné guardando los archivos del repositorio y, en otro caso, instalando el comando en el computador. Otro problema fue que había ejecutado el `.env` en una carpeta externa al repositorio.

### 5. Ventajas de construir la imagen, probarla y publicar esa misma imagen

Construir la imagen Docker, ejecutar las pruebas sobre ella y publicar exactamente la misma imagen garantiza que lo que se probó es lo que se va a desplegar.

1. **Evita diferencias entre pruebas y producción.** Si solo se valida el código fuente, las pruebas corren en un entorno que puede ser distinto al real. Con Docker, la imagen incluye código, dependencias y configuración, y esa misma imagen validada es la que se publica en Docker Hub.
2. **Valida también la configuración del contenedor.** Las pruebas verifican el `Dockerfile`, las dependencias instaladas, el puerto expuesto y el comando de inicio. El código puede estar bien y la imagen tener un error de configuración; probando la imagen completa se detecta antes de desplegar.
3. **Garantiza reproducibilidad.** La imagen es un artefacto inmutable: cualquier servidor que descargue la imagen ejecutará exactamente lo que se probó en el pipeline.
4. **Reduce riesgos.** Si se probara el código y luego se construyera otra imagen para publicar, podrían aparecer diferencias por dependencias actualizadas, cambios accidentales o errores en la construcción.
5. **Facilita la trazabilidad.** Cada imagen publicada queda asociada a un commit y a una ejecución del pipeline que la aprobó, así se sabe qué versión se desplegó y qué validaciones superó.

En conclusión, se valida el artefacto final de despliegue y no solo el código, siguiendo la práctica de construir una vez, probar esa imagen y desplegar exactamente la misma.

### 6. Pull Request de la sección 9

**Pruebas agregadas.** Angélica agregó dos pruebas unitarias en `tests/test_calculadora.py`:

- `test_sumar_decimales` comprueba que `sumar(0.1, 0.2)` sea aproximadamente 0.3 (con `pytest.approx`). Detectaría un error en `sumar` que trabaje solo con enteros o que redondee los decimales.
- `test_multiplicar_dos_negativos` comprueba que `multiplicar(-2, -3)` dé 6. Detectaría un error en el manejo de signos, por ejemplo que la función devolviera -6.

Ninguna de las dos pruebas nuevas detecta el error que metimos en `restar`. Ese lo detectan `test_restar` (en `tests/test_calculadora.py`) y `test_index_post_restar` (en `tests/test_app.py`), que ya existían: con `return a + b`, `restar(5, 2)` da 7 y no 3.

**Qué pasó con el error intencional.** Cambiamos `restar` para que devolviera `a + b`. En la ejecución del PR el pipeline falló en el paso **Run Black (Formatter)**, no en las pruebas: el commit dejó el comentario `# Error intencional` con un solo espacio antes del `#` y líneas en blanco sobrantes al final, y Black pidió reformatear `app/calculadora.py`. El commit de corrección siguiente también falló en Black por lo mismo, y recién el commit `400c6c6` dejó todo en verde.

**Pasos que no se ejecutaron.** Como Black falló, todos los pasos siguientes quedaron omitidos (*skipped*): Pylint, Flake8, las pruebas unitarias y de componente, el build de la imagen Docker, el arranque del contenedor, las pruebas de aceptación, SonarCloud, el login a Docker Hub y el push de la imagen. Sí corrieron los de diagnóstico y reportes, por sus condiciones `failure()` y `always()`.

**Cómo se evitó que el error llegara a `main` y a Docker Hub.** La regla `proteger-main` exige un Pull Request con una aprobación y que el check `build-and-test` pase, y además bloquea los pushes directos, los force pushes y el borrado de la rama. Mientras el check estuvo en rojo no se podía integrar el PR; solo lo aprobé e integré cuando la ejecución quedó en verde. Además, el login y el push a Docker Hub solo corren en un push a `main`, así que durante el PR nunca se publicó nada; la imagen se publicó después del merge, en la ejecución sobre `main`, donde esos dos pasos sí se ejecutaron. Aun así, aunque Black no hubiera fallado, las pruebas de `restar` habrían detenido el pipeline en el paso de pruebas antes de construir la imagen.
