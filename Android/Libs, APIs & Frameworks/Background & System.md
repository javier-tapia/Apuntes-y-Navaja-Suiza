<h1><i>Background & System</i></h1>

***Index***:
<!-- TOC -->
  * [Fundamentos previos sobre el *Background* y la gestión de Energía](#fundamentos-previos-sobre-el-background-y-la-gestión-de-energía)
    * [*Doze Mode*](#doze-mode)
    * [*App Standby*](#app-standby)
    * [*Wakelocks*](#wakelocks)
  * [*WorkManager*: Programación de Tareas Persistentes](#workmanager-programación-de-tareas-persistentes)
  * [*Services*: Componentes de Servicio](#services-componentes-de-servicio)
  * [*AlarmManager*: Programación de Eventos Exactos](#alarmmanager-programación-de-eventos-exactos)
  * [*BroadcastReceiver*: Receptores de Eventos](#broadcastreceiver-receptores-de-eventos)
  * [Notificaciones](#notificaciones)
  * [Acceso Directo](#acceso-directo)
    * [*Deep Links*](#deep-links)
    * [*App Links*](#app-links)
    * [*Shortcuts*](#shortcuts)
    * [Comparativa](#comparativa)
  * [*Google Play, Widgets & System Core APIs*](#google-play-widgets--system-core-apis)
    * [*In-App Updates*](#in-app-updates)
    * [*In-App Reviews*](#in-app-reviews)
    * [*Widgets*](#widgets)
<!-- TOC -->

---

## Fundamentos previos sobre el *Background* y la gestión de Energía
> 🔍 Ver también [**_Android Vitals_**](../UI/Performance.md#android-vitals)

Antes de implementar cualquier API de ejecución, es necesario comprender cómo el sistema gestiona los recursos. Android **prioriza la autonomía de la batería**, lo que **condiciona cuándo y cómo se ejecutan las tareas de fondo**.

### *Doze Mode*
Es un **estado de ahorro agresivo** introducido en Android 6.0 (API 23) que se activa cuando el dispositivo está desenchufado, quieto y con la pantalla apagada.
- **Mecanismo**: El sistema restringe el acceso a la red, restringe severamente los *Wakelocks* y pospone la ejecución de tareas y alarmas hasta breves **ventanas de mantenimiento**.
- **Consecuencia**: Las tareas programadas (ej. en *WorkManager*) se agrupan y ejecutan en lote cuando el sistema "despierta" brevemente, evitando que cada app despierte al procesador por separado.

### *App Standby*
A diferencia de *Doze*, este mecanismo **afecta a aplicaciones individuales**. Internamente, el sistema clasifica la app en distintos niveles de uso (**_App Standby Buckets_**, introducidos en Android 9): **_Active_**, **_Working set_**, **_Frequent_**, **_Rare_**, **_Restricted_**. Si el usuario no interactúa con una app durante un tiempo, el sistema la mueve a un estado de restricción.
- **Mecanismo**: La app pierde prioridad para acceder a la red y ejecutar trabajos de fondo hasta que sea abierta nuevamente o el dispositivo se conecte a la corriente.

### *Wakelocks*
Son mecanismos que permiten a una aplicación "mantener despierto" al procesador (CPU) incluso si la pantalla se apaga.
- **Propósito**: Asegurar que una operación crítica no sea interrumpida por el modo de suspensión del _hardware_.
- **Riesgo**: Son un "arma de doble filo"; si no se liberan correctamente, impiden que el dispositivo entre en reposo profundo, agotando la batería rápidamente y afectando la salud del sistema.

## *WorkManager*: Programación de Tareas Persistentes
> 🔍 Referencia:  
> https://developer.android.com/develop/background-work/background-tasks/persistent

> ℹ️ **Nota:**  
> ``Application`` no es un buen lugar para encolar trabajos arbitrarios con ``WorkManager``, ya que:
> - **Se ejecuta siempre** (incluso en **_cold start_**).
> - **No representa una intención explícita del usuario**.  
> Se recomienda colocar esta lógica en una ``Activity``, ``ViewModel`` o capa de dominio, dependiendo del caso.

Es la API recomendada para tareas que **deben completarse obligatoriamente**, incluso si la aplicación se cierra o el dispositivo se reinicia.  
No está diseñada para garantizar ejecución inmediata, sino para una **ejecución confiable y diferible**, delegada al planificador del sistema según las condiciones del dispositivo. Dicho de otra forma: puede ejecutarse rápidamente si las condiciones lo permiten, pero **la prioridad final siempre la define el sistema**.

- **Persistencia**: Las tareas se guardan en una base de datos interna de _SQLite_, garantizando su ejecución tras un reinicio del sistema.
- **Restricciones (*Constraints*)**: Permite definir condiciones ideales (ej. solo con Wi-Fi, solo mientras carga).
- **Ejecución inteligente**: El sistema decide el mejor momento basándose en políticas del sistema como [*Doze Mode*](#doze-mode) y estado del dispositivo para optimizar recursos.

Responsabilidades:
- ``WorkManager`` :arrow_right: Decide **cuándo y cómo** se ejecuta algo. Define las condiciones.
- ``Worker`` :arrow_right: Define **qué** se ejecuta. Es quien realiza el trabajo.
- ``WorkRequest`` :arrow_right: Instrucción concreta que **encapsula el trabajo**, restricciones y políticas de reintento. Tiene implementaciones concretas como ``OneTimeWorkRequest``, ``PeriodicWorkRequest`` y variantes de *Unique Work* (trabajos únicos por nombre y política), que permiten modelar distintos patrones de ejecución.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#workmanager)

## *Services*: Componentes de Servicio
> 🔍 Referencia:  
> https://developer.android.com/develop/background-work/services

> ⚠️ **Importante**  
> Los servicios en segundo plano puro (*Background Services*) están fuertemente restringidos. Si la tarea no es perceptible por el usuario, se debe utilizar [***WorkManager***](#workmanager-programación-de-tareas-persistentes).

Componentes de la aplicación pensados para realizar **operaciones de larga duración en segundo plano** que no requieren necesariamente persistencia ante reinicios del dispositivo, pero que pueden necesitar interactuar con el usuario o con otros componentes.

- ***Foreground Services***: Tareas **activamente perceptibles** por el usuario mientras se ejecutan. Deben mostrar una **notificación persistente**.
    - Ejemplos :arrow_right: Navegación GPS; Reproducción de música.
- ***Bound Services***: Actúan como una interfaz cliente-servidor, **normalmente dentro del mismo proceso**, permitiendo que otros componentes (como una *Activity*) se vinculen a ellos para **interactuar o enviar comandos**. Este servicio **vive mientras haya clientes vinculados**: cuando el último cliente se desvincula, el sistema destruye el servicio. Es decir, no habrá ejecución “silenciosa” en _background_.
    - Ejemplos :arrow_right: Reproducción de audio con controles en UI; Descarga de archivos con progreso en tiempo real; Comunicación con hardware (Bluetooth, sensores, etc.).
- **Android 14+**: Es obligatorio declarar el `foregroundServiceType` en el archivo *Manifest* para indicar el propósito del servicio (ej. ``camera``, `location`, `mediaPlayback`, `dataSync`), que a su vez debe estar respaldado por el permiso correspondiente (``FOREGROUND_SERVICE_MEDIA_PLAYBACK``), o el sistema puede lanzar un ``IllegalArgumentException`` o un ``SecurityException`` en tiempo de ejecución.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#services)

## *AlarmManager*: Programación de Eventos Exactos
> 🔍 Referencia:  
> https://developer.android.com/develop/background-work/services/alarms

Se utiliza únicamente cuando se requiere **precisión horaria estricta para tareas sensibles al tiempo (_Time-sensitive tasks_)**, como un despertador o un recordatorio de medicación. A diferencia de *WorkManager*, ***AlarmManager* prioriza precisión sobre eficiencia energética** y no gestiona condiciones de red de forma inteligente; simplemente "despierta" al dispositivo en el momento indicado.

- **Precisión**: Permite ejecución **altamente precisa, incluso si el dispositivo está en [*Doze Mode*](#doze-mode)**.
- **Persistencia**: Las alarmas **se borran al reiniciar** el dispositivo. Es necesario reprogramarlas manualmente tras un reinicio escuchando el evento `BOOT_COMPLETED`.
- **Android 12+**: Requiere el permiso `SCHEDULE_EXACT_ALARM`, introducido como permiso especial para permitir alarmas exactas.
- **Android 13+**: Este permiso **ya no se concede automáticamente** a la mayoría de las apps recién instaladas que apunten a API 33+, quedando en estado *denied by default*.
- **Android 14+**: Una restauración desde _backup_ **no restituye** el permiso si estaba denegado.
    - Las apps que ya tenían el permiso concedido **lo conservan** al actualizar el dispositivo a Android 14.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#alarmmanager)

## *BroadcastReceiver*: Receptores de Eventos
> 🔍 Referencia:  
> https://developer.android.com/develop/background-work/background-tasks/broadcasts

Componentes que permiten a la aplicación **escuchar y reaccionar a mensajes (_intents_)** emitidos por el sistema (ej. batería baja) o por otras aplicaciones.

- **Receptores Estáticos**: Se declaran en el *Manifest*. Permiten que la aplicación reaccione a eventos **sin estar abierta**, ya que **el sistema operativo puede instanciar la clase automáticamente cuando ocurre el evento**.
    - **Uso**: Muy restringido desde **Android 8.0 (API 26)** para ahorrar batería.
    - **Ejemplo permitido**: El fin del encendido del dispositivo (`BOOT_COMPLETED`).
- **Receptores Dinámicos (_Context-registered_)**: Se registran mediante código y están vinculados al ciclo de vida del componente que los crea (ej. una *Activity*).
    - **Uso**: Es la forma obligatoria de escuchar la mayoría de los **Eventos del Sistema** (ej. cambios de conectividad o batería) mientras la app está en uso.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#broadcastreceiver)

## Notificaciones
> 🔍 Referencia:  
> https://developer.android.com/develop/ui/views/notifications/build-notification

> ℹ️ **Notas:**  
> Para la implementación del servidor y recepción de mensajes remotos, consultar la guía de [**FCM**](./Firebase.md#firebase-cloud-messaging-fcm) y los ejemplos de código en el archivo de [**_Code snippets_**](../../Code%20Snippets%20with%20Kotlin/Firebase%20SDKs.md#firebase-cloud-messaging-fcm).
> Desde **Android 7.0 (API 24)**, el sistema soporta respuestas enriquecidas desde la notificación (**_Rich Replies_**), permitiendo al usuario **interactuar y responder sin abrir la app**.  
> A partir de **Android 9 (API 28)**, se mejora la gestión de hilos, estados y conversaciones persistentes, tanto para notificaciones locales como para mensajes provenientes de FCM.

Es el canal principal de **comunicación visual con el usuario**.

- **_Notification Channels_**: Obligatorios desde Android 8.0. Permiten al usuario bloquear **categorías específicas** de notificaciones de una app sin silenciarla por completo.
- **Permiso de *Runtime***: Desde Android 13+, es obligatorio solicitar el permiso `POST_NOTIFICATIONS` para **mostrar notificaciones al usuario**.
- **Estilos y UI**: Soporta diversos formatos como `BigTextStyle` (para textos largos), `BigPictureStyle` (imágenes) o `MessagingStyle` (para hilos de conversación con respuestas rápidas).

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#notifications)

## Acceso Directo
> ⚠️ **Importante**  
> Para que la verificación automática de _App Links_ funcione, se utiliza **_Digital Asset Links_**, el **protocolo de seguridad** que valida la **relación de propiedad entre el sitio web y la aplicación**. Consiste en alojar un archivo JSON en la ruta `/.well-known/assetlinks.json` del dominio.  
> Sin este archivo, el sistema no podrá verificar la app y siempre mostrará el diálogo de selección de aplicaciones al usuario.

Define formas de acceso a la aplicación que permiten al usuario **saltar directamente a una sección específica**, optimizando la navegación y la retención.

📌 Ejemplos de código:
- [*Deep Links*](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#deep-links)
- [*App Links*](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#app-links)
- [*Shortcuts*](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#shortcuts)

### *Deep Links*
> 🔍 Referencia:  
> https://developer.android.com/training/app-links/create-deeplinks

Esquemas de URL personalizados (ej. `myapp://profile/123`) asociados a una _Activity_, la cual al abrirse puede acceder al ``Uri`` desde el ``Intent``.  
Se pueden invocar a través de diferentes formas:

1. Desde código (App :arrow_right: App):
    - Android busca una _Activity_ con un ``intent-filter`` compatible 
    - Si solo la app propia soporta ese esquema, se abre directamente
    - Si varias apps soportan el mismo esquema, el sistema **muestra un diálogo de selección (_chooser_)**
2. Desde otra app o navegador:
    - El sistema actúa igual que con un link web 
    - Se abre la app o se muestra el selector
3. Desde ``adb`` (testing / QA):
    - Verificar ``intent-filters`` 
    - Debuggear navegación 
    - Testing automatizado

### *App Links*
> 🔍 Referencia:  
> https://developer.android.com/training/app-links/about

> ⚠️ **Importante**  
> Para que Android verifique el dominio y el _link_ se abra directamente en la app (sin mostrar _chooser_), el dominio debe exponer el archivo https://www.example.com/.well-known/assetlinks.json correctamente configurado.  
> Sin esta verificación, el comportamiento será el de un _Deep Link_ común.

Son URLs estándar de HTTP/HTTPS verificadas (ej. `www.myapp.com`). Al estar vinculadas al dominio mediante ***Digital Asset Links***, abren la app directamente sin preguntar.  
La diferencia clave **no está en cómo se invoca**, sino en **cómo Android decide abrirlo** (verificación automática del dominio).

1. Desde código (App :arrow_right: App):
    - Si el dominio está **verificado** (``assetlinks.json`` correcto), se abre **directamente la app**, sin diálogo
    - Si no está verificado, Android muestra el _chooser_ (app / navegador)
2. Desde navegador (caso principal de uso)
    - Para el usuario es **solo un link web**
    - Para Android (**gran diferencial** frente a los _Deep Links_): 
        - Si el dominio está asociado :arrow_right: Abre la app 
        - Si no :arrow_right: Abre el navegador
3. Desde otra app (ej. email, _WhatsApp_, etc.)
    - No requiere ninguna integración especial 
    - La verificación del dominio sigue aplicando
4. Desde ``adb`` (testing / QA)
    - Verificar estado del dominio
    - Verificar si fue aprobado, denegado o nunca verificado

### *Shortcuts*
> 🔍 Referencia:  
> https://developer.android.com/develop/ui/views/launch/shortcuts

Son **puntos de entrada alternativos** a la app (no compite con _Deep/App Links_), ideales para acciones **frecuentes, atómicas y de alto valor** (crear, buscar, retomar). Permiten exponer acciones rápidas de la app al mantener presionado el ícono del _launcher_, aunque el sistema decide **cuándo y cuántos _shortcuts_ mostrar**, priorizando los más usados.

Pueden ser:
- **Estáticos** :arrow_right: Definidos en XML (los más comunes y estables)
- **Dinámicos** :arrow_right: Creados/modificados en tiempo de ejecución según el uso del usuario

Cada _shortcut_:
- Lanza un ``Intent`` 
- Normalmente **abre una _Activity_** 
- Suele reutilizar _Deep Links_ internos, pero **el usuario nunca ve la URL**

### Comparativa

| Característica          | Deep Link             | App Link                | Shortcut (App Shortcut)         |
|-------------------------|-----------------------|-------------------------|---------------------------------|
| Esquema                 | Custom (`example://`) | HTTPS                   | Intent / Deep Link interno      |
| Requiere dominio        | ❌                     | ✅                       | ❌                               |
| Verificación automática | ❌                     | ✅                       | ❌                               |
| Diálogo de selección    | Frecuente             | No (si está verificado) | ❌ (acción directa del launcher) |
| UX recomendada          | Interna / técnica     | Externa / pública       | Acceso rápido / recurrente      |

<br>

## *Google Play, Widgets & System Core APIs*
> 🔍 Referencias:  
> - [Documentación de *In-App Updates*](https://developer.android.com/guide/playcore/in-app-updates)
> - [Guía de *In-App Reviews*](https://developer.android.com/guide/playcore/in-app-review)
> - [Guía de *Widgets*](https://developer.android.com/develop/ui/views/appwidgets)

Este apartado agrupa interfaces y servicios proporcionados por el sistema operativo o por *Google Play Services*, los cuales **gestionan la interacción con el entorno y la tienda**. Su característica principal es que **lanzan diálogos del sistema que el desarrollador no controla visualmente** y, a menudo, requieren aprobación o configuración externa (ej. consola de _Google Play_).

### *In-App Updates*
Permite notificar y gestionar actualizaciones de la aplicación sin que el usuario deba buscar manualmente en la _Play Store_. Puede ser **Flexible** (descarga en segundo plano) o **Inmediata** (bloquea la app hasta actualizar).  
Google recomienda **no forzar el modo inmediato salvo casos críticos** (seguridad, bloqueo funcional).

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#in-app-updates)

### *In-App Reviews*
Facilita la solicitud de calificaciones y reseñas mediante un flujo nativo de _Google Play_, aumentando la tasa de _feedback_ sin interrumpir la experiencia de uso. Google decide si mostrar el diálogo o no basándose en cuotas de uso (máximo una vez cada varios meses) para evitar el *spam*.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#in-app-reviews)

### *Widgets*
Extensiones de la interfaz que viven en la pantalla de inicio. Usan ``AppWidgetProvider`` (que hereda de [*BroadcastReceiver*](#broadcastreceiver-receptores-de-eventos)) y utilizan `RemoteViews` para renderizar contenido fuera del proceso principal de la app.

📌 Ejemplos de código: ver [acá](../../Code%20Snippets%20with%20Kotlin/Background%20&%20System.md#widgets)
