# Comandos útiles - Git & GitHub

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
