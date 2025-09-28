<h1>Comandos útiles - Git & GitHub</h1>

***Index***:
<!-- TOC -->
  * [Cambiar nombre rama local](#cambiar-nombre-rama-local)
  * [Borrar rama local](#borrar-rama-local)
  * [Borrar rama remota](#borrar-rama-remota)
  * [Configurar rama local para que rastree automáticamente una rama remota](#configurar-rama-local-para-que-rastree-automáticamente-una-rama-remota)
  * [Obtener URL’s (o SSH *keys*) de los repositorios remotos configurados](#obtener-urls-o-ssh-keys-de-los-repositorios-remotos-configurados)
  * [Cambiar URL’s (o SSH *keys*) de los repositorios remotos configurados](#cambiar-urls-o-ssh-keys-de-los-repositorios-remotos-configurados)
  * [Agregar repositorio *upstream* (cuando el *fork* es *origin*)](#agregar-repositorio-upstream-cuando-el-fork-es-origin)
  * [Mostrar lista de ramas locales con info adicional](#mostrar-lista-de-ramas-locales-con-info-adicional)
  * [*Stashes*](#stashes)
    * [Ver lista de los *stashes*](#ver-lista-de-los-stashes)
    * [Obtener *stash* de la lista con el número](#obtener-stash-de-la-lista-con-el-número)
    * [Hacer un *stash* con un mensaje *custom* para identificarlo](#hacer-un-stash-con-un-mensaje-custom-para-identificarlo)
    * [Borrar *stash* de la lista con el número](#borrar-stash-de-la-lista-con-el-número)
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

## Borrar rama remota
```bash
  git push origin -d remote-branch-name
```

## Configurar rama local para que rastree automáticamente una rama remota
```bash
  # `-u` equivale a `--set-upstream`
  git push -u origin fix/test
```

## Obtener URL’s (o SSH *keys*) de los repositorios remotos configurados
```bash
    git remote -v
    
    # Muestra por ejemplo:
    origin  git@github.com:Ulises-Jota/JetpackComposeCatalog.git (fetch)
    origin  git@github.com:Ulises-Jota/JetpackComposeCatalog.git (push)
```

## Cambiar URL’s (o SSH *keys*) de los repositorios remotos configurados
```bash
    git remote set-url origin {URL-REPO}
    
    # Por ejemplo:
    git remote set-url origin git@github.com:javier-tapia/JetpackComposeCatalog.git
    
    # Volviendo a validar con 'git remote -v', muestra:
    origin  git@github.com:javier-tapia/JetpackComposeCatalog.git (fetch)
    origin  git@github.com:javier-tapia/JetpackComposeCatalog.git (push)
```

## Agregar repositorio *upstream* (cuando el *fork* es *origin*)
```bash
    git remote add upstream {URL-REPO-ORIGINAL}
    
    # Por ejemplo, suponiendo que el usuario "john-doe" haya hecho un fork, luego haría:
    git remote add upstream git@github.com:javier-tapia/JetpackComposeCatalog.git
    
    # Si vuelve a validar con 'git remote -v', le mostraría:
    origin  git@github.com:john-doe/JetpackComposeCatalog.git (fetch)
    origin  git@github.com:john-doe/JetpackComposeCatalog.git (push)
    upstream        git@github.com-emu:javier-tapia/JetpackComposeCatalog.git (fetch)
    upstream        git@github.com-emu:javier-tapia/JetpackComposeCatalog.git (push)
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
