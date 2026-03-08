<h1>Comandos útiles - Git & GitHub</h1>

***Index***:
<!-- TOC -->
  * [Cambiar nombre rama local](#cambiar-nombre-rama-local)
  * [Borrar rama local](#borrar-rama-local)
  * [Mostrar lista de ramas locales con info adicional](#mostrar-lista-de-ramas-locales-con-info-adicional)
  * [Trabajar con múltiples repos remotos](#trabajar-con-múltiples-repos-remotos)
    * [Ver repos remotos configurados](#ver-repos-remotos-configurados)
    * [Cambiar URL (o SSH *key*) de un repo remoto configurado](#cambiar-url-o-ssh-key-de-un-repo-remoto-configurado)
    * [Agregar nuevos repos remotos](#agregar-nuevos-repos-remotos)
    * ["Elegir" desde qué remoto hacer ``fetch``](#elegir-desde-qué-remoto-hacer-fetch)
    * [Crear nueva rama local que sea copia de una rama de otro repo](#crear-nueva-rama-local-que-sea-copia-de-una-rama-de-otro-repo)
    * [*Push* de la rama local apuntando al repositorio remoto propio (``origin``)](#push-de-la-rama-local-apuntando-al-repositorio-remoto-propio-origin)
    * [Configurar rama local para que rastree automáticamente otra rama remota](#configurar-rama-local-para-que-rastree-automáticamente-otra-rama-remota)
    * [Borrar rama remota](#borrar-rama-remota)
  * [*Stashes*](#stashes)
    * [Ver lista de los *stashes*](#ver-lista-de-los-stashes)
    * [Obtener *stash* de la lista con el número](#obtener-stash-de-la-lista-con-el-número)
    * [Hacer un *stash* con un mensaje *custom* para identificarlo](#hacer-un-stash-con-un-mensaje-custom-para-identificarlo)
    * [Borrar *stash* de la lista con el número](#borrar-stash-de-la-lista-con-el-número)
  * [Config global para *line endings*](#config-global-para-line-endings)
  * [Limpiar cache de Git](#limpiar-cache-de-git)
  * [*Cherry-pick*](#cherry-pick)
    * [Con *Android Studio*](#con-android-studio)
    * [Desde Terminal](#desde-terminal)
  * [Log de los últimos X commits abreviados](#log-de-los-últimos-x-commits-abreviados)
  * [Ver detalles de un *commit* con el *hash*](#ver-detalles-de-un-commit-con-el-hash)
  * [Revertir todos los cambios de un *commit* anterior](#revertir-todos-los-cambios-de-un-commit-anterior)
  * [Restaurar archivos a una versión anterior](#restaurar-archivos-a-una-versión-anterior)
  * [Comparar cambios entre distintos estados](#comparar-cambios-entre-distintos-estados)
<!-- TOC -->

---

## Cambiar nombre rama local
```bash
git branch -m new-branch-name
```

## Borrar rama local
```bash
git branch -d local-branch-name
```

## Mostrar lista de ramas locales con info adicional
```bash
# Comando:
git branch -vv

# Muestra por ejemplo:
* master   123abc  [origin/master]              Update README
feature/x  456def  [origin/feature/x: ahead 2]  Add x feature...
fix/y      789ghi                               Fix bug...

# Donde:
* -> Rama actual
[origin/feature/x: ahead 2] -> Referencia rama remota: Estado sincronización (si no tiene rama remota configurada, no va a estar esto entre [])
123abc -> Hash abreviado del último commit
Add x feature... -> Mensaje del último commit
```

## Trabajar con múltiples repos remotos
### Ver repos remotos configurados
```bash
git remote -v

# Muestra por ejemplo:
origin  git@github.com:Ulises-Jota/JetpackComposeCatalog.git (fetch)
origin  git@github.com:Ulises-Jota/JetpackComposeCatalog.git (push)
```

### Cambiar URL (o SSH *key*) de un repo remoto configurado
Si la URl del repo remoto cambia, se puede modificar la referencia para que se rastree el nuevo. 

```bash
git remote set-url {REFERENCIA-REPO} {NUEVA-URL-REPO}

# Por ejemplo, suponiendo que se cambió el nombre de usuario de `Ulises-Jota` a `javier-tapia`:
git remote set-url origin git@github.com:javier-tapia/JetpackComposeCatalog.git

# Volviendo a validar con 'git remote -v', muestra:
origin  git@github.com:javier-tapia/JetpackComposeCatalog.git (fetch)
origin  git@github.com:javier-tapia/JetpackComposeCatalog.git (push)
```

### Agregar nuevos repos remotos
Para agregar un repositorio *upstream* (cuando el *fork* propio es *origin*) o cualquier otro repo remoto (por ejemplo,el *fork* de otro desarrollador), se utiliza el comando ``git remote add``.

```bash
git remote add {REFERENCIA-REPO} {URL-REPO-ORIGINAL}

# Por ejemplo, suponiendo que el usuario "john-doe" haya hecho un fork, luego haría:
git remote add upstream git@github.com:javier-tapia/JetpackComposeCatalog.git
git remote add fork-otro-dev git@github.com:otro-dev/JetpackComposeCatalog.git

# Si vuelve a validar con 'git remote -v', le mostraría:
fork-otro-dev   git@github.com-emu:otro-dev/JetpackComposeCatalog.git (fetch)
fork-otro-dev   git@github.com-emu:otro-dev/JetpackComposeCatalog.git (push)
origin  git@github.com:john-doe/JetpackComposeCatalog.git (fetch)
origin  git@github.com:john-doe/JetpackComposeCatalog.git (push)
upstream        git@github.com-emu:javier-tapia/JetpackComposeCatalog.git (fetch)
upstream        git@github.com-emu:javier-tapia/JetpackComposeCatalog.git (push)
```

### "Elegir" desde qué remoto hacer ``fetch``
Para **traer las ramas de un remoto específico**, simplemente se pasa el nombre de ese remoto al comando ``git fetch``.

```bash
git fetch {REFERENCIA-REPO}

# Por ejemplo:
git fetch fork-otro-dev
```

### Crear nueva rama local que sea copia de una rama de otro repo
Se debe ser explícito e indicarle a Git que la rama que se quiere usar como base **proviene de un remoto específico**.

```bash
git checkout -b {NOMBRE-RAMA-LOCAL} {REFERENCIA-REPO}/{NOMBRE-RAMA}

# Por ejemplo:
git checkout -b feature/sample fork-otro-dev/feature/sample
```

### *Push* de la rama local apuntando al repositorio remoto propio (``origin``)
```bash
git push --set-upstream origin {NOMBRE-RAMA-LOCAL}

# Por ejemplo:
git push --set-upstream origin feature/sample

# O simplemente:
git push -u origin HEAD

# Donde:
# `-u` es el atajo para `--set-upstream`
# `HEAD` es una referencia a la rama actual. Es una forma de no tener que escribir el nombre de la rama
```

### Configurar rama local para que rastree automáticamente otra rama remota
Permite "apuntar" la rama local a una rama remota específica.

```bash
# Por ejemplo, suponiendo que la rama `feature/sample` ahora deba apuntar a `origin/fix/test` en lugar de `origin/feature/sample` como se había seteado en el paso previo, se pasa el nombre de la nueva rama en lugar de `HEAD`:
git push -u origin fix/test
```

### Borrar rama remota
```bash
git push {REFERENCIA-REPO} -d {NOMBRE-RAMA}

# Por ejemplo:
git push origin -d remote-branch-name
```

## *Stashes*
### Ver lista de los *stashes*
```bash
git stash list
```

### Obtener *stash* de la lista con el número
```bash
git stash pop {NÚMERO-STASH}
```

### Hacer un *stash* con un mensaje *custom* para identificarlo
```bash
git stash push -m "Mensaje"
```

### Borrar *stash* de la lista con el número
```bash
git stash drop {NÚMERO-STASH}
```

## Config global para *line endings*
Sin el _flag_ (``true`` o ``false``) para consultar cuál es.

```bash
git config --global core.autocrlf true
```

## Limpiar cache de Git
```bash
git rm -r --cached .
```

- ``git rm``: Es el comando para eliminar archivos.
- ``-r``: Significa "recursivo", para que actúe sobre directorios enteros.
- ``--cached``: ¡Esta es la parte más importante! Le dice a Git que solo elimine los archivos del área de seguimiento (el "índice" o "_cache_" de Git). NO los borrará del disco duro.
- ``.``: Es una abreviatura para "el directorio actual y todo lo que contiene".

## *Cherry-pick*
### Con *Android Studio*
- _Switch_ a la rama en la cual se quiere aplicar el ``cherry-pick``
- **Show Git Log** (Alt+9 en Windows, Command+9 en Mac)
- Encontrar el _commit_ que se desea aplicar
- _Clic_ derecho sobre el _commit_
- Seleccionar **Cherry-pick**
- _Pushear_ cambios
- Si surgen conflictos:
    - Resolver conflictos en la *merge tool*
    - Aplicar, Guardar y Commit
    - _Pushear_ cambios

### Desde Terminal
- _Switch_ a la rama en la cual se quiere aplicar el ``cherry-pick``
- Correr el comando:

```bash
git cherry-pick <hash-del-commit>

# Con `-n`, se aplican los cambios sin crear un commit automáticamente
git cherry-pick -n <hash>
```

- _Pushear_ cambios
- Si surgen conflictos:
    1. Git pausará el ``cherry-pick`` y mostrará los archivos en conflicto
    2. Resolver los conflictos editando los archivos (Android Studio marcará las secciones conflictivas)
    3. Después de resolver, correr los comandos:
    ```bash
    git add <archivos-resueltos>
    git cherry-pick --continue  
    ```
    4. _Pushear_ cambios
    5. Si en algún momento se decide cancelar el proceso, correr:
    ```bash
    git cherry-pick --abort  
    ```

## Log de los últimos X commits abreviados
```bash
git log --oneline -{NUMERO}


# Ejemplo: obtener los últimos 3 commits en una sola línea
git log --oneline -3

# Ejemplo del output:
12a3b4c (HEAD -> feature/something) Merge with develop and resolve conflicts
34jh98y (origin/develop, origin/HEAD, develop) Added - Add some tests (#123)
daf986f Feature - Add some feature (#122)
```

## Ver detalles de un *commit* con el *hash*
Muestra el autor, la fecha, el comentario y los _diffs_.

```bash
git show {HASH}


# Ejemplo:
git show 12a3b4c
```

## Revertir todos los cambios de un *commit* anterior
Crea un nuevo _commit_ que deshace los cambios introducidos por un _commit_ anterior.  
Es la forma más segura de revertir cambios en ramas compartidas, ya que no reescribe el historial.

```bash
git revert {HASH-DEL-COMMIT-A-REVERTIR}


# Ejemplos:

# Se revierte el commit 'b1a2c3d', abriendo el editor para confirmar el mensaje del nuevo commit
git revert b1a2c3d

# O para referir al último commit
git revert HEAD

# Para evitar abrir el editor y usar el mensaje por defecto ("Revert 'mensaje original'")
git revert --no-edit b1a2c3d
```

## Restaurar archivos a una versión anterior
Restaura archivos en el directorio de trabajo a una versión anterior (de otro _commit_ o del _staging area_). Es ideal para descartar cambios locales o para restaurar una versión específica de un archivo.  
La ruta al archivo debe ser siempre la relativa a la **raíz (_root_) del repositorio**.

```bash
# Para descartar cambios locales en un archivo (volver al estado del último commit)
git restore {RUTA-AL-ARCHIVO}

# Para sacar un archivo del área de "staging" (index) y devolverlo al estado de "cambios sin seguimiento" (untracked changes)
git restore --staged {RUTA-AL-ARCHIVO}

# Para restaurar un archivo a la versión de un commit específico
git restore --source={HASH-DEL-COMMIT} {RUTA-AL-ARCHIVO}


# Ejemplos:

# Descarta los cambios locales no guardados en el staging area
git restore app/src/main/java/com/example/MainActivity.kt

# Si se quiere restaurar ese archivo a como estaba en un commit más antiguo (ej: 'a0f9b8e')
git restore --source=a0f9b8e app/src/main/java/com/example/MainActivity.kt

# O, para referir al commit inmediatamente anterior al último (usando '~1')
git restore --source=HEAD~1 app/src/main/java/com/example/MainActivity.kt
```

## Comparar cambios entre distintos estados
Muestra las diferencias en el código. Es la herramienta principal para revisar qué ha cambiado entre el directorio de trabajo, el _staging area_, _commits_ antiguos o distintas ramas.

```bash
# Para mostrar todos los cambios que aún no están en el staging area
git diff

# Para mostrar los cambios de un archivo específico que aún no están en el staging area
git diff {RUTA-AL-ARCHIVO}

# Para mostrar todos los cambios que ya están en el staging area (añadidos con git add)
git diff --staged

# Para mostrar las diferencias entre tu rama actual (HEAD) y otra rama
git diff {RAMA-ORIGEN}..{RAMA-DESTINO}

# Para mostrar los cambios que ocurrieron entre dos commits usando sus hashes
git diff {HASH-COMMIT-ANTIGUO} {HASH-COMMIT-NUEVO}

# Para comparar un commit contra el commit inmediatamente anterior a él (usando '~1')
git diff {HASH-DEL-COMMIT}~1 {HASH-DEL-COMMIT}


# Ejemplos:

# Comparar la rama actual (HEAD) contra 'develop'
# La referencia a la rama local con HEAD se puede omitir en este caso
git diff develop

# Comparar la rama 'main' contra 'develop'
# Muestra los cambios que tiene 'develop' que NO están en 'main'
git diff main..develop

# Comparar dos commits
git diff a1b2c3d e4f5g6h
```
