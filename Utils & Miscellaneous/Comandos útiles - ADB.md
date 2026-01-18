<h1>Comandos útiles - ADB (<i>Android Debug Bridge</i>)</h1>

***Index***:
<!-- TOC -->
  * [Obtener versión y/o ruta de ``adb``](#obtener-versión-yo-ruta-de-adb)
  * [Obtener ayuda de ``adb``](#obtener-ayuda-de-adb)
  * [Iniciar *activity* con un *deeplink*](#iniciar-activity-con-un-deeplink)
    * [Desglose del comando](#desglose-del-comando)
  * [Ver la lista de dispositivos conectados](#ver-la-lista-de-dispositivos-conectados)
  * [Ver la lista de procesos](#ver-la-lista-de-procesos)
  * [Ver qué *activity* está en primer plano](#ver-qué-activity-está-en-primer-plano)
  * [Listar todos los paquetes instalados](#listar-todos-los-paquetes-instalados)
  * [Buscar un paquete en particular](#buscar-un-paquete-en-particular)
  * [Matar un proceso](#matar-un-proceso)
  * [Simular un *shake* en el emulador](#simular-un-shake-en-el-emulador)
  * [Hacer un *dump* del *stack* de *activities*](#hacer-un-dump-del-stack-de-activities)
  * [Hacer un *dump* de las *windows*](#hacer-un-dump-de-las-windows)
  * [Disparar el *callback* `onTrimMemory()`](#disparar-el-callback-ontrimmemory)
  * [Iniciar *logcat* y guardarlo en un archivo local](#iniciar-logcat-y-guardarlo-en-un-archivo-local)
<!-- TOC -->

---

## Obtener versión y/o ruta de ``adb``
```bash
which adb
# Ejemplo de salida en Mac:
# /Users/<USER>/Library/Android/sdk/platform-tools/adb

adb version
# Ejemplo de salida en Mac:
# Android Debug Bridge version 1.0.41
# Version 35.0.2-12147458
# Installed as /Users/<USER>/Library/Android/sdk/platform-tools/adb
# Running on Darwin 25.2.0 (arm64)
```

## Obtener ayuda de ``adb``
```bash
adb shell am help
```

## Iniciar *activity* con un *deeplink*

```bash
# Versión corta (deja que Android resuelva la app o muestra *chooser* si corresponde)
adb shell am start -a android.intent.action.VIEW -d "<DEEPLINK>"

# Versión completa (no todo es requerido)
<PATH>/adb shell am start -W -a <ACTION> --ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> -d "<DEEPLINK>" <PACKAGE>

# Ejemplo en Mac
~/Library/Android/sdk/platform-tools/adb shell am start -W -a android.intent.action.VIEW --ez android.intent.extra.START_PLAYBACK true -d "<DEEPLINK>" com.example.something
```

### Desglose del comando

1. **`~/Library/Android/sdk/platform-tools/adb`**
    - **Qué es:** la ruta completa al binario `adb` (_Android Debug Bridge_).
    - **Para qué sirve:** ejecutar `adb` aunque no esté en el `PATH`.
    - **Impacto:** ninguno en el _deeplink_; solo afecta cómo se invoca la herramienta desde la máquina.
2. **`shell`**
    - **Qué es:** le dice a `adb` que ejecute lo que sigue **dentro del dispositivo/emulador**, en una _shell_.
    - **Impacto:** necesario para usar `am` (*Activity Manager*) del sistema Android.
3. **`am start`**
    - **Qué es:** comando del _Activity Manager_ para "iniciar" algo vía un ``Intent`` (normalmente una _Activity_).
    - **Impacto:** es el "disparador" que termina abriendo la pantalla correspondiente.
4. **`-W`**
    - **Qué es:** *wait*.
    - **Qué hace:** espera a que la _Activity_ se lance y devuelve métricas (tiempos tipo `ThisTime/TotalTime/WaitTime`).
    - **Impacto en el _deeplink_:** no cambia qué se abre; **solo cambia el comportamiento/salida del comando** (útil para medición o _scripts_).
5. **`-a android.intent.action.VIEW`**
    - **Qué es:** define la **acción** del ``Intent`` (`a` = *action*).
    - **Qué hace:** `VIEW` indica que se quiere "ver/abrir el recurso".
    - **Impacto:** es la acción típica para _deeplinks_/URLs; los `intent-filter` de la app suelen declarar `VIEW` para rutas de _deeplink_.
6. **`--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE>`**
    - **Qué es:** agrega un ***extra boolean*** al ``Intent`` (`--ez` = *extra boolean*).
    - **Qué hace:** equivale a `intent.putExtra(<EXTRA_KEY>, <true|false>)`.
    - **Impacto:** cambia parámetros que recibe la app y puede modificar el flujo (ej. en Android TV, pedir *autoplay*).
7. **`-d "<DEEPLINK>"`**
    - **Qué es:** define el **_data URI_** del ``Intent`` (**`d`** = *data*).
    - **Qué hace:** pasa el _deeplink_ (por ejemplo `myapp://...` o `https://...`) como URI.
    - **Impacto:** es el dato clave que Android compara contra los `intent-filter` (``scheme``/``host``/``path``, etc.) para decidir qué _Activity_ puede manejarlo.
8. **`com.example.something`**
    - **Qué es:** el **_package name_** que se pasa como destino (ver [acá](#buscar-un-paquete-en-particular) cómo buscar un paquete en particular o [acá](#listar-todos-los-paquetes-instalados) cómo listar todos los paquetes instalados).
    - **Qué hace:** **restringe** la resolución del ``Intent`` para que se intente abrir con esa app (en lugar de dejar que Android elija entre todas las que _matcheen_).
    - **Impacto:**
        - Evita _chooser_ / evita que otra app "intercepte" el _deeplink_ si también lo soporta.
        - Si esa app **NO tiene** una _Activity_ exportada que _matchee_ `VIEW + <DEEPLINK>`, puede fallar con _"unable to resolve Intent"_.

## Ver la lista de dispositivos conectados

```bash
adb devices
```

## Ver la lista de procesos

```bash
adb shell ps
```

## Ver qué *activity* está en primer plano

```bash
adb shell dumpsys window | findstr "mCurrentFocus"

# Ejemplo, podría mostrar: 
mCurrentFocus=Window{510bb9f u0 com.cursokotlin.jetpackcomponentscatalog/com.cursokotlin.jetpackcomponentscatalog.MainActivity}

# 'com.cursokotlin.jetpackcomponentscatalog' es el paquete
```

## Listar todos los paquetes instalados

```bash
adb shell pm list packages

# Ejemplo, podría mostrar:
package:com.android.chrome
package:com.cursokotlin.jetpackcomponentscatalog
package:com.whatsapp
```

## Buscar un paquete en particular

```bash
adb shell pm list packages | findstr "catalog"

# Ejemplo, podría mostrar:
package:com.cursokotlin.jetpackcomponentscatalog
```

## Matar un proceso

```bash
adb shell am kill <PACKAGE>
adb shell am force-stop <PACKAGE>

# Ejemplos:
adb shell am kill com.example.something
adb shell am force-stop com.example.something
```

## Simular un *shake* en el emulador

```bash
adb emu sensor set acceleration 100:100:100; sleep 1; adb emu sensor set acceleration 0:0:0
```

Ese mismo comando se puede agregar como un alias en el archivo `.zshrc`:

```bash
alias shake="adb emu sensor set acceleration 100:100:100; sleep 1; adb emu sensor set acceleration 0:0:0"
```

Pero mejor aún, se puede crear una función en el mismo archivo `.zshrc` para poder pasarle el emulador obtenido con el comando para [ver la lista de dispositivos](#ver-la-lista-de-dispositivos-conectados):

```bash
# En el archivo .zshrc:
shake() {
  local emulator_id=$1
  adb -s $emulator_id emu sensor set acceleration 100:100:100
  sleep 1
  adb -s $emulator_id emu sensor set acceleration 0:0:0
}

---------------------------------------------------------------

# En la terminal:
shake emulator-1234
```

## Hacer un *dump* del *stack* de *activities*
> 💻 **Sólo para Mac/Linux**  
> También se puede usar un *plugin* (ver [acá](https://github.com/Kardelio/easy-dumpsys)) y con el comando `easy-dumpsys`, se ve el *stack* de una forma más "amigable".

```bash
adb shell dumpsys activity activities | grep "{PACKAGE}"
```


## Hacer un *dump* de las *windows*

```bash
adb shell dumpsys window
```

## Disparar el *callback* `onTrimMemory()`

```bash
adb shell am send-trim-memory com.example.something RUNNING_CRITICAL
```

## Iniciar *logcat* y guardarlo en un archivo local

```bash
adb logcat -v time > logcat.txt
```
