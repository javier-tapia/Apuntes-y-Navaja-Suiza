# Comandos útiles - ADB 

- [Ver la lista de dispositivos conectados](#ver-la-lista-de-dispositivos-conectados)
- [Ver la lista de procesos](#ver-la-lista-de-procesos)
- [Ver qué *activity* está en primer plano](#ver-qué-activity-está-en-primer-plano)
- [Listar todos los paquetes instalados](#listar-todos-los-paquetes-instalados)
- [Buscar un paquete en particular](#buscar-un-paquete-en-particular)
- [Matar un proceso](#matar-un-proceso)
- [Simular un *shake* en el emulador](#simular-un-shake-en-el-emulador)
- [Iniciar *activity* con un *deeplink*](#iniciar-activity-con-un-deeplink)
- [Hacer un *dump* del *stack* de *activities*](#hacer-un-dump-del-stack-de-activities)
- [Hacer un *dump* de las *windows*](#hacer-un-dump-de-las-windows)
- [Disparar el *callback* `onTrimMemory()`](#disparar-el-callback-ontrimmemory)
- [Iniciar *logcat* y guardarlo en un archivo local](#iniciar-logcat-y-guardarlo-en-un-archivo-local)

---

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

## Iniciar *activity* con un *deeplink*

```bash
    ~/Library/Android/sdk/platform-tools/adb shell am start -a android.intent.action.VIEW -d "<DEEPLINK>" com.example.something
    
    ~/Library/Android/sdk/platform-tools/adb shell am start -a android.intent.action.VIEW -d "<DEEPLINK>" com.example.something
    
    adb shell am start -a android.intent.action.VIEW -d "<DEEPLINK>" com.example.something
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
