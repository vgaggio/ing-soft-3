# ing-soft-3
Practicos materia Ingenieria de Software 3

Paso 1
Se muestra a continuación que Git está instalado en el dispositivo.


Paso 2
Se muestra a continuación como se creó un nuevo repositorio en un directorio local, se agregó un nuevo archivo llamado "Readme.md" (con una línea de texto) y luego se hizo un commit de los cambios con un mensaje.

Comandos utilizados:

mkdir Repo01 : Crear directorio.
git init : Inicializa el repositorio local.
touch Readme.md : Crea el archivo.
git status : Muestra los archivos modificados.
git add . : Agrega los archivos modificados para incluirlos en el próximo commit.
git commit -m "Primer commit : Crea el commit con el mensaje especificado.


Paso 3
En primer lugar, se instaló TextMate.

Luego, se integra el editor con Git mediante los siguientes comandos.

Paso 4
En primer lugar, se crea un repositorio desde github.com con el archivo Readme.md por defecto.

Clonamos el repositorio en un directorio local.

Se modifica el archivo README, se agrega un archivo gitignore (que contiene *.bak). Con estos cambios, se hace un commit y luego se suben al repositorio remoto mediante git push origin main.

Estos cambios se pueden ver desde Github.

Paso 5
Para este paso, se utiliza el repositorio local creado anteriormente en el paso 2.

Creamos un nuevo repositorio vacío en Github.

Se asocia el repositorio local con el repositorio remoto y se suben los cambios previamente realizados a este último.

Se verifica que los cambios pueden verse desde Github.

Paso 6
Utilizando el repositorio creado en el paso 4, creamos en él una nueva rama llamada "Rama_A". Dentro de esta rama, se modificó el archivo README.md y se hizo un commit. Finalmente, se verificó la direrencia entre la rama main y esta nueva.


Paso 7
Merge Fast Forward
En primer lugar, se hace un merge Fast-Forward. Luego, se suben los cambios al repositorio remoto y se elimina la rama creada. Finalmente, se ve el log de commits.

Comandos utilizados:

git merge : fusiona ramas (mueve el puntero hacia adelante).
git branch -d : elimina la rama.
git push : sube los cambios al repositorio remoto.
git log --oneline : muestra el log de commits.


Merge No Fast Forward
Similar al paso anterior, creamos una nueva rama (Rama_B), pero antes de cambiarnos a ella, se hace un cambio en un archivo en la rama main (y un commit del mismo). Luego, nos movemos a esta nueva rama y realizamos un cambio en el archivo gitignore, del cual también hacemos commit. Finalmente, nos movemos nuevamente a la rama main y realizamos merge, pero esta vez de tipo No Fast Forward debido a que hubo cambios en ambas ramas, y eliminamos Rama_B.

Se muestra a continuación el log de lo descripto anteriormente.

Paso 8
Configuración de difftool y mergetool
Se instaló la herramienta "p4merge" y a continuación se muestran los comandos utilizados para configurar esta aplicación para ser usada por defecto desde git como difftool y mergetool.

Generación de conflicto
Para generar un conflicto, se creó una nueva rama "conflictBranch". Posteriormente, realizamos un cambio en la línea 7 del archivo README.md y se hizo un commit del mismo. Luego, nos trasladamos a la nueva rama creada y realizamos otro cambio sobre la línea 7 del mismo archivo, también realizando un commit sobre esta rama.

Uso de difftool
Habiendo configurado previamente p4merge como difftool por defecto de git, mediante el comando git difftool main conflictBranch podemos observar las diferencias entre las ramas especificadas mediante esta aplicación.

Intento de Merge
Habiendo realizado los cambios anteriormente mencionados, se intenta realizar un merge desde la rama main a conflictBranch, pero esto produce un conflicto.

Uso de mergetool
Se utiliza nuevamente la aplicación p4merge, en esta ocasión para resolver el conflicto previamente demostrado. Para esto, aplicamos el comando git mergetool.

Resolución del conflicto
Para resolver el conflicto, se decidió mantener la versión del archivo README proveniente de la rama main. Una vez guardados los cambios, se agrego "*.orig" al archivo .gitignore. Finalmente, se realizó un commit y se subieron los cambios al repositorio remoto.

Archivo .orig
A modo ilustrativo, se muestra el archivo README.md.orig que fue creado a partir de la resolución del conflicto, donde se muestra el origen del mismo.

Paso 9
¿Qué es Pull Request?
Un Pull Request es una funcionalidad de sistemas de control de versiones, que se utiliza para notificar a otros desarrolladores que se ha completado un conjunto de cambios en una rama de un proyecto y que se desea que esos cambios sean revisados y eventualmente fusionados (merged) en la rama principal o en otra rama de destino.

Creando un Pull Request
Para probar realizar un Pull Request dentro del repositorio de GitHub, en primer lugar creamos una nueva rama (newBranch) en la cual se realizaron modificaciones. Sobre estos cambios, se hizo un commit y el mismo fue subido al repositorio remoto.

Luego, desde GitHub, se creó un Pull Request para solicitar un merge a la rama main con los cambios sobre la rama newBranch.

Finalmente, el Pull Request fue aprobado, y de este modo se efectuó correctamente un merge entre newBranch y main. Además, la rama newBranch fue eliminada, ya que la misma dejó de ser necesaria al estar todos los cambios en la rama main.

Se muestra a continuación que el PR fue correctamente efectuado.

Paso 10
Niveles completados.

![Captura De Pantalla 2024 08 20 A La(S) 18.21.14](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.21.14.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.21.49](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.21.49.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.23.56](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.23.56.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.24.47](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.24.47.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.25.35](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.25.35.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.26.29](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.26.29.png)

![Captura De Pantalla 2024 08 20 A La(S) 18.26.39](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2018.26.39.png)

![Captura De Pantalla 2024 08 20 A La(S) 19.12.27](Captura%20de%20pantalla%202024-08-20%20a%20la(s)%2019.12.27.png)
