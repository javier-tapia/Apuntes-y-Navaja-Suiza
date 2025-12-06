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
