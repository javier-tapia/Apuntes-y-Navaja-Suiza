<h1><i>Firebase</i></h1>

***Index***:
<!-- TOC -->
  * [¿Qué es *Firebase*?](#qué-es-firebase)
  * [Sobre la inicialización de *Firebase*](#sobre-la-inicialización-de-firebase)
  * [SDKs de *Firebase* más utilizados](#sdks-de-firebase-más-utilizados)
  * [*Firebase Authentication* (*Auth*)](#firebase-authentication-auth)
    * [Objetivos principales de *Firebase Auth*](#objetivos-principales-de-firebase-auth)
    * [Métodos de autenticación soportados](#métodos-de-autenticación-soportados)
    * [¿Qué maneja internamente *Firebase Auth*?](#qué-maneja-internamente-firebase-auth)
    * [Flujo general de funcionamiento](#flujo-general-de-funcionamiento)
    * [Casos de uso típicos](#casos-de-uso-típicos)
    * [Consideraciones importantes](#consideraciones-importantes)
  * [*Firebase Cloud Messaging* (FCM)](#firebase-cloud-messaging-fcm)
    * [Tipos de mensajes en FCM](#tipos-de-mensajes-en-fcm)
    * [*Token* de registro (*registration token*)](#token-de-registro-registration-token)
    * [Flujo general de funcionamiento](#flujo-general-de-funcionamiento-1)
    * [Casos de uso típicos](#casos-de-uso-típicos-1)
    * [Consideraciones importantes](#consideraciones-importantes-1)
  * [*Firebase Remote Config*](#firebase-remote-config)
    * [Características Clave de *Remote Config*](#características-clave-de-remote-config)
    * [Ejemplos de Uso](#ejemplos-de-uso)
<!-- TOC -->

---

## ¿Qué es *Firebase*?
> 🔍 Referencias:  
> https://firebase.google.com  
> https://firebase.google.com/docs  
> https://firebase.google.com/docs/android/setup  
> https://console.firebase.google.com

Es una **_plataforma de desarrollo de aplicaciones_** creada por Google que ofrece un conjunto integrado de **_servicios en la nube_** para facilitar la **_creación, operación y crecimiento_** de aplicaciones móviles y web.  
Proporciona herramientas listas para usar para **_autenticación_**, **_bases de datos en tiempo real_**, **_almacenamiento_**, **_análisis_**, **_mensajería push_**, **_funciones serverless_**, **_A/B testing_**, entre otros, eliminando la necesidad de gestionar infraestructura propia.

En esencia, _Firebase_ funciona como un **_Backend-as-a-Service_** (**_BaaS_**) que permite a los equipos enfocarse en la lógica de negocio y la experiencia de usuario, mientras delegan en Google el _backend_ escalable y seguro.

📌 Ejemplos de código: ver [acá](/Code%20Snippets%20with%20Kotlin/Firebase%20SDKs.md)

## Sobre la inicialización de *Firebase*
Normalmente, **_no es necesario inicializar Firebase manualmente_**, ya que **_se inicializa automáticamente_** cuando existe el archivo ``google-services.json`` y el _plugin_ ``com.google.gms.google-services``.

- ``google-services.json``:
    - Es un archivo que genera _Firebase_ al crear el proyecto y registrar la app.
    - Contiene la configuración necesaria para que los SDKs (_Auth_, _Firestore_, FCM, etc.) funcionen: IDs, claves API, URLs de _backend_ de _Firebase_, etc.
    - Se ubica en ``app/`` y es leído automáticamente al compilar.
- **_Plugin_** ``com.google.gms.google-services``:
    - Es un _plugin_ de Gradle que lee ``google-services.json`` y genera las configuraciones necesarias para que los SDKs se inicialicen automáticamente.
    - Se agrega normalmente en el ``build.gradle`` del módulo ``app``:
  ```groovy
  plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'
  }
  ```
    - **_No requiere dependencias extra_**, pero sí que esté aplicado en Gradle.
- Por lo tanto :arrow_right: Si se tiene ``google-services.json`` y se aplicó el _plugin_, **_Firebase se inicializa automáticamente_**.

**Casos donde sí se puede requerir inicialización manual:**
1. **Múltiples procesos**: la app tiene procesos separados (por ejemplo: _Service_, _Worker_) que no arrancan con la _Activity_ principal.
2. **Apps con varios proyectos _Firebase_**: se quiere inicializar un ``FirebaseApp`` secundario manualmente.
3. **Tests o configuración avanzada**: _mocks_, _fakes_, inicialización diferida para _Remote Config_, etc.

## SDKs de *Firebase* más utilizados
**1. Gestión de Identidad y Autenticación**
- **Firebase Authentication (_Auth_):** Gestiona de forma segura el registro de usuarios, el inicio de sesión (con correos, contraseñas, teléfonos y proveedores sociales como Google, Facebook, etc.) y la gestión de sesiones.

**2. Bases de Datos (Almacenamiento de Datos)**
- **_Cloud Firestore_:** Base de datos NoSQL moderna, escalable y flexible para almacenar y sincronizar datos de aplicaciones en tiempo real.
- **_Realtime Database_ (RTDB):** La base de datos original de _Firebase_, ideal para sincronización de datos con latencia ultra baja usando un árbol JSON gigante.

**3. Almacenamiento de Archivos**
- **_Cloud Storage for Firebase_:** Servicio robusto para almacenar archivos binarios de usuarios, como imágenes, audios y videos, con reglas de seguridad integradas.

**4. _Messaging_ y Notificaciones**
- **_Firebase Cloud Messaging_ (FCM):** Permite enviar notificaciones *push* y mensajes silenciosos a los usuarios de forma gratuita y a gran escala.

**5. Analítica y Monitoreo de Calidad**
- **_Google Analytics_:** Proporciona datos profundos sobre cómo interactúan los usuarios con la aplicación.
- **_Crashlytics_:** Ofrece informes detallados de *crashes* y errores en tiempo real para mantener la estabilidad de la aplicación.
- **_Performance Monitoring_:** Monitorea el rendimiento de la aplicación en condiciones reales, midiendo tiempos de carga y latencia de red.

**6. _Hosting_ y Despliegue**
- **_Firebase Hosting_:** Alojamiento seguro y rápido para aplicaciones web estáticas, SPAs y microservicios, con despliegue sencillo y CDN global.

**7. Configuración y Personalización Remota**
- **_Firebase Remote Config_:** Permite cambiar la UI y el comportamiento de la aplicación de forma remota sin requerir actualizaciones.

## *Firebase Authentication* (*Auth*)
Es un servicio que proporciona **_métodos de autenticación listos para usar_** sobre aplicaciones móviles y web, funcionando como un **_wrapper de alto nivel_** sobre los estándares OAuth 2.0 y OpenID Connect (OIDC).  
No reemplaza estos estándares: **_los encapsula y automatiza_** para evitar que el desarrollador implemente manualmente el flujo completo (validación de _tokens_, manejo de sesiones, intercambio de códigos, gestión de secretos, etc.).

Si no se utiliza _Firebase Auth_, el camino natural es implementar directamente el flujo de [Authorization Code + PKCE](Security.md#authorization-code-flow--pkce) + [OpenID Connect (OIDC)](Security.md#openid-connect-oidc) con los proveedores de identidad (Google, Apple, Facebook, correo/sms propios, etc.), lo que implica un mayor nivel de complejidad y responsabilidad operativa.

### Objetivos principales de *Firebase Auth*
- **_Simplificar la autenticación de usuarios_** sin desarrollar un _backend_ propio para manejar sesiones.
- **_Proveer métodos seguros_** de registro, inicio de sesión y persistencia de la identidad.
- **_Unificar múltiples proveedores_** de identidad bajo una API directa y coherente.
- **_Emitir y gestionar tokens_** (_ID Tokens_ y _Refresh Tokens_) basados en estándares OIDC.

### Métodos de autenticación soportados
_Firebase Auth_ ofrece múltiples mecanismos, todos accesibles mediante el mismo SDK:
1. **_Correo y contraseña_** (autenticación clásica con validación de contraseña _hash_).
2. **_Correo con enlace mágico_** (_Email Link_ / _Passwordless_).
3. **_Número de teléfono_** (SMS OTP).
4. **_Proveedores sociales_**: Google, Facebook, Apple, Microsoft, GitHub
5. **_Identidades personalizadas_** mediante **_Custom Tokens_** generados por un _backend_ propio.

### ¿Qué maneja internamente *Firebase Auth*?
El SDK y los servidores de _Firebase_ gestionan automáticamente:

- **_Almacenamiento seguro del estado de sesión_** en el dispositivo.
- **_Renovación automática del Refresh Token_** para mantener sesiones persistentes.
- **_Verificación criptográfica de ID Tokens vía certificados públicos de Google_**.
- **_Persistencia local_** (_SharedPreferences_ / _Keychain_) del usuario autenticado.
- **_Manejo de errores, intentos fallidos y throttling_** en proveedores sensibles (ej. SMS).
- **_Integración con reglas de seguridad_** en otros servicios como _Firestore_ y _Storage_.

De esta forma, la aplicación se enfoca únicamente en consumir los datos del usuario autenticado, sin necesidad de implementar infraestructura de seguridad.

### Flujo general de funcionamiento
Aunque el SDK lo abstrae, el flujo conceptual análogo a OAuth/OIDC es:

1. La aplicación inicia un flujo de autenticación (correo, Google, teléfono, etc.).
2. El proveedor de identidad valida las credenciales del usuario.
3. El servidor de _Firebase_ recibe y procesa la identidad verificada.
4. _Firebase_ emite un **_ID Token OIDC_** y un **_Refresh Token_** válidos para ese usuario.
5. El SDK almacena y renueva automáticamente los _tokens_ en el dispositivo.
6. La app utiliza estos _tokens_ para acceder a otros servicios (_Firestore_, _Storage_, etc.) cuyo acceso está condicionado por reglas de seguridad basadas en la identidad del usuario.

### Casos de uso típicos
- **_Aplicaciones que necesitan registro/login rápido_** sin desarrollar _backend_ propio.
- **_Flujos de onboarding sencillos_** con _Google Sign-In_ o _Email Link_.
- **_Aplicaciones con acceso protegido a Firestore o Storage_** mediante reglas de seguridad.
- **_Integración con identidades corporativas_**, usando **_custom tokens_** emitidos desde un _backend_.
- **_Sincronización multi-dispositivo_** usando la identidad única del usuario.

### Consideraciones importantes
- _Firebase Auth_ **_no almacena contraseñas en texto plano_**, sino que utiliza _hashing_ seguro administrado por Google.
- Los usuarios autenticados obtienen **_ID Tokens_** firmados que pueden verificarse localmente o en un _backend_ propio.
- En Android, la persistencia de sesión es automática, pero puede borrarse manualmente.
- La autenticación por SMS está sujeta a restricciones regionales y a límites antiabuso.

## *Firebase Cloud Messaging* (FCM)
Es un servicio multiplataforma que permite **_enviar notificaciones push y mensajes de datos_** a dispositivos Android, iOS y aplicaciones web **_de manera gratuita, segura y a escala masiva_**.  
Su propósito principal es facilitar la **_comunicación unidireccional desde el servidor hacia los dispositivos_**, permitiendo notificar eventos relevantes, sincronizar información en segundo plano o desencadenar acciones específicas dentro de la aplicación.

FCM opera como un **_canal de mensajería en tiempo real_**, donde cada dispositivo Android registrado obtiene un **_token de registro único_** que lo identifica ante la infraestructura de _Firebase_.  
Los servidores de la aplicación envían mensajes utilizando este _token_ como destino, delegando en Google la entrega eficiente, el enrutamiento y la gestión de conexiones persistentes.

### Tipos de mensajes en FCM
FCM permite enviar dos tipos principales de mensajes:

**1. Mensajes de “notificación”**
- Son procesados automáticamente por los servicios de _Google Play_.
- Se muestran directamente en la bandeja del sistema sin requerir que la aplicación esté en ejecución.
- Incluyen campos como ``title``, ``body``, íconos, acciones, etc.
- Propósito :arrow_right: **_Mostrar contenido al usuario_**.

**2. Mensajes de “data”**
- Son recibidos siempre por la aplicación, incluso en primer plano (_foreground_).
- Contienen un **_payload arbitrario_** definido por el servidor, **_enviado como un mapa de pares clave-valor_**.
- Permiten ejecutar lógica personalizada: navegación interna, sincronización en segundo plano, procesamiento de flags, etc.
- Propósito :arrow_right: **_Controlar el comportamiento de la aplicación_**.

### *Token* de registro (*registration token*)
Cada instalación de la aplicación obtiene un _registration token_, generado por el SDK de FCM. Este token funciona como un **_identificador único del dispositivo_** dentro del ecosistema de mensajería de _Firebase_.  
El _backend_ de la aplicación utiliza este _token_ para enviar mensajes dirigidos a ese dispositivo específico, a grupos de dispositivos o a temas (_topics_) suscritos.

El _token_ **_puede cambiar en cualquier momento_** (por reinstalación, limpieza de datos, actualización de la app, cambios internos del servicio), por lo que la aplicación debe:
- Obtenerlo al iniciar
- Escuchar su renovación mediante ``onNewToken``
- Opcionalmente, reportarlo al _backend_

### Flujo general de funcionamiento
1. La aplicación registra su servicio de mensajería mediante ``FirebaseMessagingService``.
2. El dispositivo obtiene su _token_ de FCM.
3. Ese _token_ se envía al _backend_ si se requiere integración servidor-a-dispositivo.
4. El _backend_ envía un mensaje dirigido a ese _token_ (o _topic_).
5. FCM enruta y entrega el mensaje al dispositivo.
6. La app procesa el mensaje:
    - Si es de notificación :arrow_right: Android muestra la notificación automáticamente;
    - Si es de data :arrow_right: El SDK entrega el _payload_ a ``onMessageReceived``.

### Casos de uso típicos
- **_Notificaciones push tradicionales_** (promociones, recordatorios, alertas).
- **_Mensajes silenciosos_** para sincronizar datos en segundo plano (por ejemplo, refrescar contenido).
- **_Acciones internas_** como navegación profunda (_deeplinks_) o activación de funcionalidades.
- **_Mensajería masiva_** por segmentos a través de suscripciones a _topics_.
- **_Notificaciones personalizadas_** enviadas solo a dispositivos específicos mediante su _token_.

### Consideraciones importantes
- FCM **_no garantiza entrega inmediata_**, pero optimiza y prioriza el envío según el tipo de mensaje y la disponibilidad de red.
- Los mensajes de “notificación” pueden ser descartados por el sistema en condiciones extremas.
- Los mensajes de “data” requieren que la app implemente su propio comportamiento, incluyendo la creación de notificaciones manualmente.
- El _token_ no es un secreto criptográfico, pero debe manejarse con cuidado porque identifica a una instalación específica de la aplicación y podría utilizarse para enviar mensajes no deseados si se expone públicamente.
- Los dispositivos Android dependen de _Google Play Services_ para recibir mensajes de manera confiable.

## *Firebase Remote Config*
Es un servicio en la nube que **_permite cambiar la apariencia y el comportamiento de la aplicación sin requerir que los usuarios descarguen una actualización_** desde la tienda. Para hacer eso, se definen parámetros (pares clave-valor) en la consola de *Firebase* que luego son descargados por el SDK de la aplicación en el dispositivo del usuario.  
Es una herramienta poderosa que proporciona flexibilidad y agilidad, permitiendo a los equipos de desarrollo y _marketing_ **_reaccionar rápidamente a las necesidades cambiantes del mercado y de los usuarios_**.

### Características Clave de *Remote Config*
- **Actualizaciones Dinámicas:** Se pueden modificar elementos como el color principal de la UI, el texto de un botón, o incluso activar/desactivar una función completa (como una nueva característica _beta_) de forma remota, en tiempo real, para todos los usuarios o para segmentos específicos.
- **Valores Predeterminados en la Aplicación:** El SDK siempre usa valores predeterminados integrados en la aplicación si no puede recuperar los valores más recientes del servidor (por ejemplo, si el dispositivo está sin conexión), garantizando que la aplicación funcione siempre.
- **Segmentación de Audiencia:** Se pueden definir condiciones complejas basadas en propiedades del usuario (como la versión de la aplicación, el sistema operativo, la ubicación o propiedades de usuario de _Google Analytics_) para entregar configuraciones diferentes a distintos grupos de usuarios.
- **Pruebas A/B Integradas:** Se integra con **_Firebase A/B Testing_**, permitiendo a los desarrolladores probar qué variaciones de configuración (por ejemplo, un botón verde frente a uno rojo) generan mejores resultados o métricas de participación del usuario.

### Ejemplos de Uso
- **Lanzamiento de Funciones Controlado:** Desplegar una nueva función a solo el 5% de los usuarios para monitorear su estabilidad antes de un lanzamiento masivo.
- **Personalización de la UI:** Cambiar el tema de la aplicación para una temporada festiva (Navidad, _Halloween_) sin lanzar una nueva versión.
- **Mantenimiento de Emergencia:** Desactivar rápidamente una función que está causando errores graves en producción hasta que se pueda solucionar el problema.
