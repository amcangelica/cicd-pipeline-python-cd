# cicd-pipeline-python

La aplicación requiere `FLASK_SECRET_KEY` para firmar los tokens CSRF. Configura esta variable en el entorno de ejecución; no guardes su valor en el repositorio. Puedes generar una clave con `python -c "import secrets; print(secrets.token_hex(32))"`.

Manuela Jimenez Arredondo manujimeneza | David Moreno Moran davidgmoreno (Propietario repositorio) | Angélica Martínez Carmona amcangelica

## Evidencias de la sección 9

- Pull request: [#1](https://github.com/davidgmoreno/cicd-pipeline-python/pull/1)
- Ejecución fallida del pipeline por el error intencional: [ver ejecución](https://github.com/davidgmoreno/cicd-pipeline-python/actions/runs/43618026686/job/132517267855?pr=1)

## Respuestas

### 4. Problemas o dificultades al implementar el taller

**Manuela Jiménez Arredondo:** Mi primer problema fue el entorno de Python. Al instalar las dependencias, pip no encontraba `flake8==7.5.0`. Revisando el error vi que esa versión pide Python 3.11 o superior y mi entorno virtual se había creado con el Python 3.10 que tenía en pyenv. Borré la carpeta `venv`, la volví a crear con Python 3.13 y ahí sí se instaló todo.

El segundo fue el puerto 5000: al correr la app en mi Mac salía "Address already in use" porque AirPlay Receiver usa ese puerto. Lo resolví cambiando temporalmente el puerto de la app y de las pruebas de aceptación a 5001, y entendí que el 5000 solo importa para la ejecución local, porque el contenedor y el pipeline usan el 8000.

El tercero fue en GitHub Actions: el paso de `docker build` falló con `invalid tag "/cicd-pipeline-python"`. Al ver el log, `IMAGE_NAME` empezaba con una barra, o sea que la variable `DOCKERHUB_USERNAME` estaba vacía porque no se había guardado en el repositorio. La creé de nuevo y aprendí la diferencia entre secretos y variables, y que conviene comprobar con `gh variable list` que de verdad existen.

### 6. Pull Request de la sección 9

**Pruebas agregadas.** Angélica agregó dos pruebas unitarias en `tests/test_calculadora.py`:

- `test_sumar_decimales` comprueba que `sumar(0.1, 0.2)` sea aproximadamente 0.3 (con `pytest.approx`). Detectaría un error en `sumar` que trabaje solo con enteros o que redondee los decimales.
- `test_multiplicar_dos_negativos` comprueba que `multiplicar(-2, -3)` dé 6. Detectaría un error en el manejo de signos, por ejemplo que la función devolviera -6.

Ninguna de las dos pruebas nuevas detecta el error que metimos en `restar`. Ese lo detectan `test_restar` (en `tests/test_calculadora.py`) y `test_index_post_restar` (en `tests/test_app.py`), que ya existían: con `return a + b`, `restar(5, 2)` da 7 y no 3.

**Qué pasó con el error intencional.** Cambiamos `restar` para que devolviera `a + b`. En la ejecución del PR el pipeline falló en el paso **Run Black (Formatter)**, no en las pruebas: el commit dejó el comentario `# Error intencional` con un solo espacio antes del `#` y líneas en blanco sobrantes al final, y Black pidió reformatear `app/calculadora.py`. El commit de corrección siguiente también falló en Black por lo mismo, y recién el commit `400c6c6` dejó todo en verde.

**Pasos que no se ejecutaron.** Como Black falló, todos los pasos siguientes quedaron omitidos (*skipped*): Pylint, Flake8, las pruebas unitarias y de componente, el build de la imagen Docker, el arranque del contenedor, las pruebas de aceptación, SonarCloud, el login a Docker Hub y el push de la imagen. Sí corrieron los de diagnóstico y reportes, por sus condiciones `failure()` y `always()`.

**Cómo se evitó que el error llegara a `main` y a Docker Hub.** La regla `proteger-main` exige un Pull Request con una aprobación y que el check `build-and-test` pase, y además bloquea los pushes directos, los force pushes y el borrado de la rama. Mientras el check estuvo en rojo no se podía integrar el PR; solo lo aprobé e integré cuando la ejecución quedó en verde. Además, el login y el push a Docker Hub solo corren en un push a `main`, así que durante el PR nunca se publicó nada; la imagen se publicó después del merge, en la ejecución sobre `main`, donde esos dos pasos sí se ejecutaron. Aun así, aunque Black no hubiera fallado, las pruebas de `restar` habrían detenido el pipeline en el paso de pruebas antes de construir la imagen.
