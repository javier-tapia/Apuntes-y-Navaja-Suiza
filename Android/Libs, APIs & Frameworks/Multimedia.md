<h1>Multimedia</h1>

***Index***:
<!-- TOC -->
  * [Conceptos de Multimedia en Android](#conceptos-de-multimedia-en-android)
    * [Modelo Mental del flujo](#modelo-mental-del-flujo)
    * [DRM (*Digital Rights Management*)](#drm-digital-rights-management)
      * [CDM - SoC - TEE - SVP](#cdm---soc---tee---svp)
      * [Niveles de seguridad de *Widevine*: L1, L2 y L3](#niveles-de-seguridad-de-widevine-l1-l2-y-l3)
    * [*Buffer*](#buffer)
    * [*Decoder*](#decoder)
    * [*Renderer*](#renderer)
    * [*Surface*](#surface)
    * [*Stall* vs *Freeze*](#stall-vs-freeze)
      * [Perspectiva amplia (académica/general)](#perspectiva-amplia-académicageneral)
      * [Perspectiva en el contexto de *Bitmovin*](#perspectiva-en-el-contexto-de-bitmovin)
      * [Reconciliando ambas visiones](#reconciliando-ambas-visiones)
      * [Resumen práctico](#resumen-práctico)
    * [*Reload Source*](#reload-source)
    * [*Tweaks (``TweaksConfig``)*](#tweaks-tweaksconfig)
  * [*Video Players: ExoPlayer*, *JW Player* y *Bitmovin*](#video-players-exoplayer-jw-player-y-bitmovin)
    * [*ExoPlayer*/*Media3*](#exoplayermedia3)
      * [``MediaDrm``](#mediadrm)
    * [*JW Player*](#jw-player)
    * [*Bitmovin Player*](#bitmovin-player)
    * [Comparativa general](#comparativa-general)
    * [📚 Recursos recomendados](#-recursos-recomendados)
  * [Grabación y captura multimedia: *CameraX* y *MediaRecorder*](#grabación-y-captura-multimedia-camerax-y-mediarecorder)
    * [*CameraX*](#camerax)
    * [*MediaRecorder*](#mediarecorder)
    * [Comparativa general](#comparativa-general-1)
    * [📚 Recursos recomendados](#-recursos-recomendados-1)
  * [Audio: reproducción y grabación (*MediaPlayer*, *AudioRecord*, *AudioTrack*)](#audio-reproducción-y-grabación-mediaplayer-audiorecord-audiotrack)
    * [*MediaPlayer*](#mediaplayer)
    * [*AudioRecord*](#audiorecord)
    * [*AudioTrack*](#audiotrack)
    * [Comparativa general](#comparativa-general-2)
    * [📚 Recursos recomendados](#-recursos-recomendados-2)
  * [Edición y procesamiento multimedia (*FFmpeg*, *ML Kit*, *OpenCV*)](#edición-y-procesamiento-multimedia-ffmpeg-ml-kit-opencv)
    * [*FFmpeg*](#ffmpeg)
    * [*ML Kit*](#ml-kit)
    * [*OpenCV (Open Source Computer Vision Library)*](#opencv-open-source-computer-vision-library)
    * [Comparativa general](#comparativa-general-3)
    * [📚 Recursos recomendados](#-recursos-recomendados-3)
  * [Transmisión y *streaming* en tiempo real (*WebRTC*, *RTMP*, *HLS*, *DASH*)](#transmisión-y-streaming-en-tiempo-real-webrtc-rtmp-hls-dash)
    * [*WebRTC*](#webrtc)
    * [*RTMP (Real-Time Messaging Protocol)*](#rtmp-real-time-messaging-protocol)
    * [*HLS (HTTP Live Streaming)* y *DASH (Dynamic Adaptive Streaming over HTTP)*](#hls-http-live-streaming-y-dash-dynamic-adaptive-streaming-over-http)
    * [Comparativa general](#comparativa-general-4)
    * [📚 Recursos recomendados](#-recursos-recomendados-4)
  * [Almacenamiento y *caching* multimedia](#almacenamiento-y-caching-multimedia)
    * [*Cache* de medios en Android](#cache-de-medios-en-android)
    * [Librerías y herramientas comunes](#librerías-y-herramientas-comunes)
    * [Comparativa de *caching*](#comparativa-de-caching)
    * [📚 Recursos recomendados](#-recursos-recomendados-5)
  * [Procesamiento y efectos en tiempo real](#procesamiento-y-efectos-en-tiempo-real)
    * [Filtros y efectos de video/audio](#filtros-y-efectos-de-videoaudio)
    * [Librerías y herramientas comunes](#librerías-y-herramientas-comunes-1)
    * [Comparativa general](#comparativa-general-5)
    * [📚 Recursos recomendados](#-recursos-recomendados-6)
  * [Consideraciones clave: formatos, optimización y multiplataforma](#consideraciones-clave-formatos-optimización-y-multiplataforma)
    * [:one: Formatos](#one-formatos)
    * [:two: Optimización](#two-optimización)
    * [:three: Interoperabilidad multiplataforma](#three-interoperabilidad-multiplataforma)
<!-- TOC -->

---

## Conceptos de Multimedia en Android
### Modelo Mental del flujo

 ```text
                    ┌─────────────────┐
                    │     BUFFER      │
                    │  (datos compri- │
                    │    midos A/V)   │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │  VIDEO DECODER  │           │  AUDIO DECODER  │
    │  (frames raw)   │           │  (PCM samples)  │
    └────────┬────────┘           └────────┬────────┘
             │                             │
             ▼                             │
    ┌─────────────────┐                    │
    │ VIDEO RENDERER  │                    │
    │ (dibuja frames) │                    │
    └────────┬────────┘                    │
             │                             │
             ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │    SURFACE      │           │   AUDIO TRACK   │
    │   (pantalla)    │           │   (AudioTrack)  │
    └─────────────────┘           └─────────────────┘
             │                             │
             ▼                             ▼
          👁️ Ojos                      🔊 Oídos 
 ```

### DRM (*Digital Rights Management*)
Es un conjunto de tecnologías utilizadas para **proteger contenido audiovisual y controlar su uso** dentro de una aplicación.  
Cuando un recurso está protegido por DRM, **el video se distribuye cifrado**, y solo puede ser descifrado por un módulo autorizado en el dispositivo (un [**CDM**](#cdm---soc---tee---svp)). Esto **dificulta/mitiga la copia, extracción o reproducción no autorizada del contenido**.

En entornos móviles y de _streaming_, los sistemas DRM más utilizados son:

- **_Widevine_** (Google)
- **_FairPlay_** (Apple)
- **_PlayReady_** (Microsoft)

A nivel conceptual, todos cumplen el mismo propósito :arrow_right: Asegurar que el contenido solo pueda ser reproducido por usuarios y dispositivos que cumplan la política (ventana de licencia, _offline_, _output restrictions_, HDCP, etc.).

En reproductores como _ExoPlayer_, _JW Player_ o _Bitmovin_, la compatibilidad con estos sistemas permite la **reproducción segura de contenidos premium o licenciados**.

#### CDM - SoC - TEE - SVP
- **CDM (_Content Decryption Module_)** :arrow_right: “El orquestador”  
  Es el “DRM engine” que orquesta licencia, políticas y el uso de claves. En Widevine L1, el CDM suele estar dividido entre una parte en el sistema “normal” y un **núcleo seguro** en el **TEE** (_hardware-backed_), que realiza las operaciones sensibles (claves/cripto) y configura/autoriza la reproducción protegida.
- **SoC (_System on a Chip_)** :arrow_right: “Dónde vive el hardware”  
  Es el **_chip_ principal** del dispositivo; integra CPU, GPU, bloques multimedia (p. ej. _decoder_), controladores de memoria/pantalla y, típicamente, bloques de seguridad que habilitan el TEE y rutas protegidas.
- **TEE (_Trusted Execution Environment_)** :arrow_right: “Dónde se protegen claves y se hace la cripto”  
  Es un entorno aislado soportado por _hardware_ (p. ej. _ARM TrustZone_), separado del OS normal (Android/Linux y apps). En Widevine L1, aloja el núcleo seguro del DRM para **manejo de claves/licencias y criptografía**, evitando que el OS y las apps accedan a claves o material sensible.
- **SVP (_Secure Video Path_)** :arrow_right: “Cómo se mantiene protegido el video ya descifrado”  
  Es el **_pipeline_ protegido** (_decrypt_/_decode_/_render_) que usa **_secure/protected buffers_** y termina en una **_secure/protected surface_**. Así, una vez autorizado el uso de claves por el TEE, el contenido se **descifra** hacia _buffers_ protegidos, se **decodifica** en un *secure decoder* y los frames se muestran en **una *secure/protected surface***, sin pasar por memoria/superficies accesibles al entorno normal.

#### Niveles de seguridad de *Widevine*: L1, L2 y L3
> ℹ️ **Nota:**  
> En la práctica, hoy la conversación común es L1 vs L3 porque L2 es relativamente raro

| **Nivel** | **Descripción**             | **Seguridad** | **Calidad permitida** |
|-----------|-----------------------------|---------------|-----------------------|
| **L1**    | _Hardware_ (TEE)            | Alta          | HD, 4K                |
| **L2**    | _Crypto_ en HW, video en SW | Media         | HD                    |
| **L3**    | Todo en _software_          | Baja          | SD (480p)             |

- **L1 (_Level_ 1)**
    - Descifrado y reproducción a través de una **cadena segura** (TEE/_hardware_ + **_secure video path_**, típicamente con **_secure decoder/surface_**).
    - Es el nivel normalmente requerido para **contenido premium** y para habilitar **HD/Full HD/4K/HDR** *según la política del proveedor*.
    - **Dónde se usa** :arrow_right: Teléfonos/TV boxes/Smart TVs **certificados** (producción).
- **L2 (_Level_ 2)**
    - Descifrado en **TEE**, pero la **decodificación** ocurre fuera del entorno seguro (no hay cadena completa protegida).
    - Menos aceptado; puede implicar restricciones de calidad según políticas.
    - **Dónde se usa** :arrow_right: **Poco común** hoy; algunos dispositivos/intermedios.
- **L3 (_Level_ 3)**
    - Descifrado **fuera del TEE** (entorno no seguro); no hay _secure video path_ completo.
    - Frecuentemente se aplica **limitación de calidad** (muchos servicios restringen L3) y es menos robusto contra extracción en dispositivos comprometidos.
    - **Dónde se usa** :arrow_right: **Emuladores**, muchos dispositivos **no certificados** o con _Widevine_ degradado.

### *Buffer*
**¿Qué es?**  
Es una **memoria temporal** que almacena datos de video/audio antes de que se procesen. Funciona como un "tanque de agua" que se llena desde la red y se vacía hacia el [_decoder_](#decoder).

**¿Por qué importa?**  
- Si el _buffer_ está vacío :arrow_right: El video se detiene ([**_stall_**](#stall-vs-freeze))
- Si el _buffer_ es muy pequeño :arrow_right: Más probabilidad de interrupciones

**Analogía**  
Es como el tanque de agua de una casa. Si el tanque está vacío, no sale agua de la canilla aunque la bomba funcione.

### *Decoder*
**¿Qué es?**  
Es el componente que **descomprime** los datos de video/audio. Los videos vienen comprimidos (H.264, H.265, VP9, etc.) y el _decoder_ los convierte en _frames_ de imagen "crudos" que se pueden mostrar.

**Tipos de _decoders_**  
- **_Hardware_ (HW)**: Usa chips especializados del dispositivo. **Más rápido, menos batería**.
- **_Software_ (SW)**: Usa el CPU. **Más lento, más consumo, pero más compatible**.

**¿Por qué importa?**  
- Cada dispositivo tiene diferentes _decoders_ de _hardware_
- Si el _decoder_ falla, el video se congela pero el audio puede seguir (tienen _decoders_ separados)

### *Renderer*
**¿Qué es?**  
Es el componente que **dibuja** los _frames_ decodificados en la pantalla. Toma los _frames_ "crudos" del [_decoder_](#decoder) y los presenta en el [_Surface_](#surface).

**En _Bitmovin_/_ExoPlayer_**  
Hay _renderers_ separados para video y audio. Por eso pueden desincronizarse: el _audio renderer_ puede seguir funcionando aunque el _video renderer_ tenga problemas.

### *Surface*
**¿Qué es?**  
Es la **"ventana" o lienzo** donde se dibuja el video. Es una abstracción de Android que representa un área de la pantalla donde el [_renderer_](#renderer) puede pintar _frames_.

**Tipos**  
- `SurfaceView` :arrow_right: Ventana separada, **mejor rendimiento**
- `TextureView` :arrow_right: Integrado con la jerarquía de vistas, **más flexible pero más lento**

**¿Por qué importa?**  
- Si el _Surface_ se destruye (por ejemplo, cuando la app va a _background_), el video se congela
- El audio NO depende del _Surface_, por eso puede seguir sonando

### *Stall* vs *Freeze*
#### Perspectiva amplia (académica/general)
- **_Freeze_** :arrow_right: **Descripción fenomenológica (síntoma)**: la imagen se queda congelada. Términos relacionados: "_video freeze_", "_image freeze_", "_frozen frame_".
- **_Stall_** :arrow_right: **Descripción técnica/causal**: una parte del _pipeline_ se quedó sin avanzar, y por eso se produce el _freeze_. Términos relacionados: "_decoder stall_", "_render stall_", "_buffering stall_".

#### Perspectiva en el contexto de *Bitmovin*

| **Concepto**          | **_Stall_**                        | **_Freeze_**                                           |
|-----------------------|------------------------------------|--------------------------------------------------------|
| **Causa**             | _Buffer_ vacío (falta de datos)    | Problema de _decoder_/_surface_                        |
| **Comportamiento**    | Video y audio se detienen          | Video se detiene, audio puede continuar                |
| **Recuperación**      | Automática cuando llegan más datos | Puede requerir [_reload_ del _source_](#reload-source) |
| **Evento _Bitmovin_** | `StallStarted` / `StallEnded`      | `Error` (a veces)                                      |

- **_Freeze_** :arrow_right: "Tengo datos pero no puedo mostrarlos" (problema de [_decoder_](#decoder)/[_surface_](#surface))
- **_Stall_** :arrow_right: "No tengo datos para mostrar" (problema de red/_buffer_)

#### Reconciliando ambas visiones
En **_Bitmovin/ExoPlayer_**, "_Stall_" tiene un significado **específico y más estrecho**: eventos `StallStarted`/`StallEnded` que el _player_ emite cuando el **_buffer_ se vacía**.

Pero en sentido amplio, "_stall_" puede referirse a cualquier parte del _pipeline_ que se detiene:

```text
STALL (sentido amplio) ──────────────────────────────────────────┐
│                                                                │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────┐ │
│   │  Buffer Stall   │   │  Decoder Stall  │   │ Render Stall│ │
│   │ (Bitmovin event)│   │ (error decoder) │   │(surface lost)│ │
│   └────────┬────────┘   └────────┬────────┘   └──────┬──────┘ │
│            │                     │                   │        │
└────────────┴─────────────────────┴───────────────────┴────────┘
                                   │
                                   ▼
                          ┌─────────────────┐
                          │     FREEZE      │
                          │  (lo que el     │
                          │  usuario ve)    │
                          └─────────────────┘
```

#### Resumen práctico

| **Perspectiva**             | **_Stall_**                            | **_Freeze_**                          |
|-----------------------------|----------------------------------------|---------------------------------------|
| **_Bitmovin_ (específica)** | Evento de _buffer_ vacío               | Problema de _decoder_/_surface_       |
| **General (amplia)**        | Cualquier causa técnica del _pipeline_ | El síntoma visible (imagen congelada) |

**Para el trabajo diario con _Bitmovin_** :arrow_right: Cuando aparece `StallStarted` en los _logs_, significa específicamente "_buffer_ vacío". Si el video se congela sin ese evento, probablemente sea un problema de _decoder_ o _surface_ (lo que en sentido amplio también sería un "_stall_" del _pipeline_, pero _Bitmovin_ no lo reporta así).

### *Reload Source*
**¿Qué es?**  
Es la acción de **volver a cargar** el contenido de video desde cero.

📌 **Ejemplo**:  

```kotlin
player.load(source)
```

**¿Cuándo se usa?**  
- Después de un error de [_decoder_](#decoder) irrecuperable
- Para cambiar de calidad/[DRM](#drm-digital-rights-management) _level_
- Para "resetear" el estado del _player_

**Problemas**  
- Pierde la posición actual de reproducción (hay que guardarla y restaurarla)
- Puede causar _desync_ si no se hace correctamente
- El _buffer_ se vacía y hay que volver a llenarlo

### *Tweaks (``TweaksConfig``)*
**¿Qué es?**  
Son **configuraciones avanzadas** del _player_ que modifican comportamientos internos. Son "ajustes finos" que no están en la configuración principal.

📌 **Ejemplos en Bitmovin**:  

```kotlin
TweaksConfig(
    limitQualityOnDrmError = true,        // Bajar calidad si hay error DRM
    isNativeHlsParsingEnabled = false,    // Usar parser propio vs nativo
    preferSoftwareDecodingForAds = true   // SW decoder para ads
)

```

**¿Por qué importa?**  
Estos _tweaks_ pueden afectar la estabilidad del [_decoder_](#decoder) y la sincronización A/V.

## *Video Players: ExoPlayer*, *JW Player* y *Bitmovin*
> 💡 **Recomendación:** Para la mayoría de los proyectos modernos en Kotlin/Android, **ExoPlayer** es la opción ideal, salvo que se requiera un sistema comercial de _streaming_ con publicidad y analíticas integradas (caso en el que **JW Player** o **Bitmoving** puede ser más conveniente).

### *ExoPlayer*/*Media3*
Originalmente, fue una librería independiente de Google para reproducción de medios en Android. A partir de 2022, Google migró _ExoPlayer_ a la librería **_AndroidX Media3_** (``androidx.media3``).  
Es el reproductor recomendado oficialmente para Android y Android TV, y reemplaza gradualmente al reproductor nativo `MediaPlayer`, ofreciendo mayor flexibilidad y soporte para formatos modernos.

**Estructura *AndroidX Media3***:  
androidx.media3  
├── media3-common          # Clases comunes  
├── media3-exoplayer       # ExoPlayer (motor de reproducción)  
├── media3-ui              # Componentes UI (PlayerView)  
├── media3-dash            # Soporte DASH  
├── media3-hls             # Soporte HLS  
└── media3-session         # Media session

**Características principales:**
- Soporte extendido para formatos: MP4, MP3, AAC, FLAC, DASH, HLS, _SmoothStreaming_ (existe como módulo aparte, ``media3-smoothstreaming``), etc.
- Integración con **DRM (_Widevine_ y, donde esté disponible, _PlayReady_)** y **subtítulos**.
- Personalización completa de componentes (_renderers_, _track selectors_, etc.).
- Compatible con **Jetpack Compose** y **Kotlin Coroutines**.
- Reproducción adaptativa y eficiente, con gestión avanzada de _buffers_.

**Ventajas:**
- Gratuito y _open source_.
- Altamente extensible y modular.
- Actualizado regularmente por Google.
- Se integra fácilmente con sistemas de *analytics*, *ads* y *caching*.

**Limitaciones:**
- Requiere mayor configuración inicial que `MediaPlayer`.
- No incluye de forma nativa funciones avanzadas de monetización o _analytics_ (se deben agregar aparte).
- La seguridad real ([L1/L3](#niveles-de-seguridad-de-widevine-l1-l2-y-l3), bloqueo de captura, HDCP) no depende del _player_ sino del dispositivo/OS/certificación.

📌 **Ejemplo**:  

```kotlin
val player = ExoPlayer.Builder(context).build()
binding.playerView.player = player

val mediaItem = MediaItem.fromUri("https://www.example.com/video.mp4")
player.setMediaItem(mediaItem)
player.prepare()
player.play()
```

#### ``MediaDrm``
Es la **API nativa de Android** (clase `android.media.MediaDrm`) que usan los reproductores para **hablar con el [DRM](#drm-digital-rights-management) del dispositivo** (por ejemplo _Widevine_) durante la reproducción de contenido protegido.

Dicho simple: es el componente con el que la app/_player_ hace el flujo de DRM:  
1. Crea una instancia para un DRM en particular (_Widevine_, _PlayReady_ si existiera, etc.)
2. Abre una sesión DRM
3. Genera el **_license challenge_**
4. La app lo envía al **_license server_**
5. Recibe la licencia y se la entrega a `MediaDrm`
6. `MediaDrm` entrega las **_keys_** al _pipeline_ de decodificación para que el video se pueda descifrar y reproducir

**Para qué se usa (qué cosas “prueba”)**  
- Verificar si el dispositivo soporta un DRM (p. ej. _Widevine_)
- Consultar propiedades como el **nivel de seguridad** (_Widevine_ `L1` vs `L3`)
- Detectar errores de licencia/_keys_ (en el flujo que implemente el _player_)

**Cómo se relaciona con _Widevine_ y _ExoPlayer_**  
- _Widevine_ es el “sistema DRM” (_key system_ `com.widevine.alpha`)
- `MediaDrm` es la **API** mediante la cual Android expone _Widevine_ al resto del sistema
- _ExoPlayer_/_Media3_ por debajo usa `MediaDrm` (a través de _wrappers_ como `FrameworkMediaDrm` o `DefaultDrmSessionManager`) para todo el manejo de licencias

### *JW Player*
Es un _framework_ multimedia comercial orientado a _streaming_ profesional, monetización y análisis avanzado.  
Ofrece SDK's para Android, iOS y Web, y se utiliza en entornos donde se necesita control sobre derechos, publicidad y métricas.

**Características principales:**
- Reproducción de video con **HLS, DASH** y otros formatos.
- Integración directa con **IMA SDK** para publicidad.
- Herramientas de *analytics* y medición de audiencia.
- DRM y soporte para subtítulos multilenguaje.
- Interfaz personalizable y APIs en Kotlin/Java.

**Ventajas:**
- Integración lista para usar con monetización y analíticas.
- Soporte técnico y actualizaciones empresariales.
- Experiencia multiplataforma coherente (Android, iOS, Web).

**Limitaciones:**
- Licencia comercial (no _open source_).
- Menor flexibilidad interna que ExoPlayer.
- Requiere clave de API o cuenta JW Player.

**Ejemplo:**

```kotlin
val playerConfig = PlayerConfig.Builder()
    .file("https://cdn.jwplayer.com/manifests/video.m3u8")
    .build()

val jwPlayerView = JWPlayerView(this, playerConfig)
setContentView(jwPlayerView)
```

### *Bitmovin Player*
Es un reproductor multimedia multiplataforma, altamente configurable y orientado a escenarios profesionales de _streaming_ donde se requieren métricas avanzadas, optimización para múltiples dispositivos y compatibilidad con los principales estándares de video adaptativo (DASH, HLS) y DRM.  
Se integra fácilmente en Android (nativo y Compose), iOS, Web y Smart TVs, y ofrece un SDK con API extensible, analíticas integradas y soporte de _low-latency_.

**Características principales:**
- Compatibilidad con **protocolos adaptativos**: MPEG-DASH, HLS, _Smooth Streaming_. 
- Soporte DRM avanzado: _Widevine_, _FairPlay_, _PlayReady_, CPIX. 
- Reproducción de **baja latencia**: _Low Latency_ HLS y _Low Latency_ DASH. 
- API modular: amplia capacidad de personalización, configuración y escucha de eventos. 
- Analíticas integradas: métricas de _startup time_, _rebuffering_, _bitrate switching_, calidad, errores, sesiones, entre otros. 
- **Publicidad**: Compatible con VAST, VMAP, VPAID, Google IMA. 
- Amplio soporte **multiplataforma**: Android, iOS, Web, Roku, Tizen, WebOS, tvOS, FireTV. 
- ABR avanzado: algoritmos optimizados para seleccionar la mejor calidad con mínima latencia y sin cortes.

**Ventajas:**
- Alta calidad profesional: ideal para servicios OTT, _broadcasters_ o apps con requisitos estrictos. 
- Gran flexibilidad: permite controlar casi todos los aspectos del _pipeline_ de reproducción. 
- SDK muy completo: _callbacks_ detallados, configuración granular y herramientas de depuración. 
- Integración nativa con **_Bitmovin Analytics_** sin necesidad de instrumentación adicional. 
- Excelente soporte de DRM y estándares de la industria.

**Limitaciones:**
- Es un **servicio pago**, por lo que puede no ajustarse a proyectos personales o presupuestos limitados. 
- Mayor **complejidad de configuración** en comparación con otros _players_ más _plug-and-play_. 
- **Documentación amplia pero dispersa**, requiere cierta familiaridad para aprovechar todo su potencial. 
- **Tamaño del SDK** superior al de soluciones más livianas como _ExoPlayer_ o _Media3_.

📌 **Ejemplo**:  

```kotlin
// build.gradle
dependencies {
    implementation("com.bitmovin.player:player:3.31.0")
}

// Activity o Fragment
class MainActivity : AppCompatActivity() {
    private lateinit var player: BitmovinPlayer
    private lateinit var playerView: PlayerView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Configuración básica del Player
        val config = PlayerConfig(
            licenseKey = "YOUR_BITMOVIN_LICENSE_KEY"
        )

        // 2. Inicialización
        player = BitmovinPlayerFactory.create(this, config)

        // 3. Obtención de la vista del Player
        playerView = findViewById(R.id.bitmovinPlayerView)
        playerView.player = player

        // 4. Fuente de contenido
        val sourceConfig = SourceConfig.fromUrl("https://example.com/stream.mpd")

        // 5. Carga del contenido
        player.load(sourceConfig)
    }

    override fun onStart() {
        super.onStart()
        player.play()
    }

    override fun onStop() {
        super.onStop()
        player.pause()
    }

    override fun onDestroy() {
        super.onDestroy()
        player.destroy()
    }
}
```

### Comparativa general

| Característica          | ExoPlayer                | JW Player      | Bitmovin Player                    |
|-------------------------|--------------------------|----------------|------------------------------------|
| Licencia                | Apache 2.0 (gratis)      | Comercial      | Comercial                          |
| Soporte oficial         | Google                   | JW Player Inc. | Bitmovin GmbH                      |
| Personalización         | Muy alta                 | Media          | Muy alta                           |
| DRM                     | Sí (Widevine, PlayReady) | Sí             | Sí (Widevine, FairPlay, PlayReady) |
| Monetización            | No nativa                | Incluida       | Incluida (Ads, IMA, VAST/VMAP)     |
| Analytics               | Personalizable           | Incluido       | Incluido (Bitmovin Analytics)      |
| Integración con Compose | Sí                       | No (usa Views) | Sí (vía AndroidView)               |
| Comunidad / Open Source | Amplia                   | Cerrada        | Cerrada                            |

### 📚 Recursos recomendados
- [Guía oficial de Media3 ExoPlayer](https://developer.android.com/media/media3/exoplayer)
- [Documentación JW Player](https://docs.jwplayer.com/platform/docs/platform-welcome)
- [Documentación SDK Bitmovin para Android](https://bitmovin.com/video-player/android-sdk/)
- [Repo de AndroidX Media](https://github.com/androidx/media)

## Grabación y captura multimedia: *CameraX* y *MediaRecorder*

> 💡 **Recomendación:** Si se necesita capturar fotos o videos de forma moderna y compatible con la mayoría de los dispositivos Android, **CameraX** es la opción ideal. Para un control a más bajo nivel sobre la grabación o codificación, se la puede combinar con **MediaRecorder**.

### *CameraX*
Es una librería de Jetpack que simplifica el uso de la cámara en Android, ofreciendo una API moderna, consistente y fácil de integrar.  
Está construida sobre `Camera2`, pero abstrae su complejidad mediante componentes listos para usar.

**Características principales:**
- Compatible desde Android 5.0 (API 21).
- Permite capturar **fotos, videos y flujos de vista previa** con pocas líneas.
- Se integra con **Jetpack Compose** y **Lifecycle**.
- Soporta **análisis de imágenes** (detección, reconocimiento, etc.).
- Se adapta automáticamente a distintos tamaños y orientaciones de cámara.

**Ventajas:**
- Fácil de implementar, sin necesidad de manejar `CameraDevice` o `CaptureSession`.
- Se actualiza constantemente desde AndroidX.
- Compatible con **View** y **Compose UI**.

**Limitaciones:**
- No provee directamente funciones de edición ni postprocesado.
- En proyectos muy personalizados, puede ser necesario combinarla con APIs de más bajo nivel.

**Ejemplo:**  

```kotlin
val imageCapture = ImageCapture.Builder().build()

val outputOptions = ImageCapture.OutputFileOptions
    .Builder(File(outputDirectory, "foto.jpg"))
    .build()

imageCapture.takePicture(
    outputOptions,
    ContextCompat.getMainExecutor(context),
    object : ImageCapture.OnImageSavedCallback {
        override fun onImageSaved(output: ImageCapture.OutputFileResults) {
            Log.d("CameraX", "Foto guardada: ${output.savedUri}")
        }
        override fun onError(exc: ImageCaptureException) {
            Log.e("CameraX", "Error al capturar foto: ${exc.message}", exc)
        }
    }
)
```

### *MediaRecorder*
Es una API nativa de Android que permite grabar **audio y video** directamente desde las fuentes del dispositivo (micrófono, cámara, pantalla).  
Aunque puede usarse de manera independiente, suele integrarse con CameraX o Camera2 para obtener control total del proceso de grabación.

**Características principales:**
- Permite grabar video en formatos comunes (MP4, 3GP).
- Permite capturar audio desde micrófono o fuentes internas.
- Configurable mediante *encoders* y *bit rates*.
- Soporta grabación en almacenamiento interno o externo.

**Ejemplo:**  

```kotlin
val recorder = Recorder.Builder()
    .setQualitySelector(QualitySelector.from(Quality.HD))
    .build()

val videoCapture = VideoCapture.withOutput(recorder)

val mediaStoreOutput = MediaStoreOutputOptions.Builder(
    context.contentResolver,
    MediaStore.Video.Media.EXTERNAL_CONTENT_URI
).setContentValues(ContentValues().apply {
    put(MediaStore.MediaColumns.DISPLAY_NAME, "video-${System.currentTimeMillis()}")
    put(MediaStore.MediaColumns.MIME_TYPE, "video/mp4")
}).build()

val recording = videoCapture.output
    .prepareRecording(context, mediaStoreOutput)
    .apply { withAudioEnabled() }
    .start(ContextCompat.getMainExecutor(context)) { event ->
        when (event) {
            is VideoRecordEvent.Start -> Log.d("MediaRecorder", "Grabación iniciada")
            is VideoRecordEvent.Finalize -> Log.d("MediaRecorder", "Archivo: ${event.outputResults.outputUri}")
        }
    }
```

### Comparativa general

| Característica          | CameraX              | MediaRecorder       |
|-------------------------|----------------------|---------------------|
| Tipo de API             | Alto nivel (Jetpack) | Bajo nivel (nativa) |
| Compatibilidad          | API 21+              | API 1+              |
| Captura de imagen       | Sí                   | No                  |
| Grabación de video      | Sí                   | Sí                  |
| Grabación de audio      | No directa           | Sí                  |
| Integración con Compose | Sí                   | No                  |
| Control sobre codecs    | Parcial              | Completo            |
| Complejidad de uso      | Baja                 | Media/Alta          |

### 📚 Recursos recomendados
- [Guía oficial de CameraX](https://developer.android.com/training/camerax)
- [Referencia de MediaRecorder](https://developer.android.com/reference/kotlin/android/media/MediaRecorder)

## Audio: reproducción y grabación (*MediaPlayer*, *AudioRecord*, *AudioTrack*)

> 💡 **Recomendación:** Para la mayoría de los casos de uso, **MediaPlayer** es suficiente para la reproducción básica de audio. Sin embargo, si se requiere procesamiento personalizado o grabación en tiempo real, **AudioRecord** y **AudioTrack** ofrecen un mayor nivel de control.

### *MediaPlayer*
Es una API de alto nivel que permite reproducir archivos o flujos de audio de forma sencilla.  
Puede trabajar con archivos locales o remotos, y soporta los formatos más comunes (MP3, AAC, OGG, WAV, entre otros).

**Características principales:**
- Soporte para audio local y _streaming_.
- Control integrado de reproducción: inicio, pausa, _seek_ y finalización.
- Permite reproducir audio desde recursos (`res/raw`), archivos o URL's.
- Integración simple con componentes del ciclo de vida (`LifecycleObserver`).

**Ventajas:**
- Implementación rápida.
- Adecuado para la mayoría de los casos de reproducción multimedia.
- Requiere poca configuración.

**Limitaciones:**
- Menor control sobre _buffers_ y estados internos.
- No apto para aplicaciones que necesiten baja latencia o manipulación de audio en tiempo real.

**Ejemplo:**  

```kotlin
val mediaPlayer = MediaPlayer.create(context, R.raw.sonido)
mediaPlayer.start()

mediaPlayer.setOnCompletionListener {
    it.release()
}
```

### *AudioRecord*
Permite capturar audio crudo desde el micrófono del dispositivo para procesarlo o almacenarlo manualmente.  
Se utiliza en escenarios donde se requiere control sobre el formato de muestreo, la frecuencia y el manejo de _buffers_.

**Características principales:**
- Permite especificar frecuencia de muestreo, canal y formato de audio.
- Adecuado para aplicaciones de reconocimiento de voz, análisis o grabación avanzada.
- Brinda acceso directo al flujo de datos PCM.

**Ventajas:**
- Control total sobre la captura de audio.
- Ideal para procesamiento en tiempo real o codificación personalizada.

**Limitaciones:**
- Mayor complejidad de implementación.
- Requiere gestión manual de _buffers_ y almacenamiento.

**Ejemplo:**  

```kotlin
val sampleRate = 44100
val bufferSize = AudioRecord.getMinBufferSize(
    sampleRate,
    AudioFormat.CHANNEL_IN_MONO,
    AudioFormat.ENCODING_PCM_16BIT
)

val audioRecord = AudioRecord(
    MediaRecorder.AudioSource.MIC,
    sampleRate,
    AudioFormat.CHANNEL_IN_MONO,
    AudioFormat.ENCODING_PCM_16BIT,
    bufferSize
)

val buffer = ByteArray(bufferSize)
audioRecord.startRecording()
audioRecord.read(buffer, 0, buffer.size)
audioRecord.stop()
audioRecord.release()
```

### *AudioTrack*
Complementario a *AudioRecord*, *AudioTrack* permite reproducir audio PCM (sin comprimir) o generado dinámicamente.  
Se usa para aplicaciones que necesitan reproducir audio procesado o sintetizado en tiempo real.

**Características principales:**
- Permite escribir directamente en un _buffer_ de salida.
- Compatible con modos de reproducción **_static_** (**memoria completa**) o **_stream_** (**flujo continuo**).
- Ofrece baja latencia en dispositivos compatibles.

**Ventajas:**
- Permite reproducir audio generado o modificado por la aplicación.
- Control preciso sobre volumen, posición y mezcla.

**Limitaciones:**
- No reproduce formatos comprimidos directamente.
- Requiere gestión manual del flujo de datos PCM.

**Ejemplo:**  

```kotlin
val sampleRate = 44100
val bufferSize = AudioTrack.getMinBufferSize(
    sampleRate,
    AudioFormat.CHANNEL_OUT_MONO,
    AudioFormat.ENCODING_PCM_16BIT
)

val audioTrack = AudioTrack(
    AudioManager.STREAM_MUSIC,
    sampleRate,
    AudioFormat.CHANNEL_OUT_MONO,
    AudioFormat.ENCODING_PCM_16BIT,
    bufferSize,
    AudioTrack.MODE_STREAM
)

audioTrack.play()
// Se escribirían datos PCM en audioTrack.write(buffer, 0, buffer.size)
audioTrack.stop()
audioTrack.release()
```

### Comparativa general

| Característica              | MediaPlayer                  | AudioRecord      | AudioTrack       |
|-----------------------------|------------------------------|------------------|------------------|
| Nivel de API                | Alto                         | Bajo             | Bajo             |
| Uso principal               | Reproducción estándar        | Captura de audio | Reproducción PCM |
| Latencia                    | Media                        | Baja             | Baja             |
| Soporte de formatos         | Comprimidos (MP3, AAC, etc.) | PCM crudo        | PCM crudo        |
| Procesamiento personalizado | No                           | Sí               | Sí               |
| Complejidad de uso          | Baja                         | Alta             | Alta             |

### 📚 Recursos recomendados
- [Referencia de MediaPlayer](https://developer.android.com/reference/kotlin/android/media/MediaPlayer)
- [Referencia de AudioRecord](https://developer.android.com/reference/kotlin/android/media/AudioRecord)
- [Referencia de AudioTrack](https://developer.android.com/reference/kotlin/android/media/AudioTrack)

## Edición y procesamiento multimedia (*FFmpeg*, *ML Kit*, *OpenCV*)

> 💡 **Recomendación:** Para tareas de edición o procesamiento multimedia en Android, se recomienda utilizar **FFmpeg** para manipulación de audio y video, **ML Kit** para análisis inteligente de imágenes y **OpenCV** para procesamiento avanzado y visión por computadora (_computer vision_).

### *FFmpeg*
Es una colección de herramientas y librerías de código abierto diseñadas para la **_conversión, compresión y procesamiento de audio y video_**.  
En Android, puede utilizarse ejecutando los **binarios nativos** mediante Kotlin o Java, invocando comandos directamente. También pueden utilizarse _wrappers_ o dependencias que exponen sus comandos nativos mediante Kotlin o Java, pero primero hay que asegurarse que dichos proyectos sigan activos y en mantenimiento.

**Características principales:**
- Conversión entre formatos de video y audio.
- Extracción de cuadros, metadatos y pistas.
- Recorte, concatenación y mezcla de archivos multimedia.
- Aplicación de filtros y efectos personalizados.
- Soporte de una amplia variedad de códecs y contenedores.

**Ventajas:**
- Altamente flexible y potente.
- Amplio soporte de formatos.
- Permite ejecutar comandos directamente desde la aplicación, o mediante comandos o APIs (si están activos).

**Limitaciones:**
- Requiere inclusión de binarios nativos (NDK).
- Uso más complejo en comparación con otras librerías de alto nivel.
- Manejo de procesos y errores requiere atención para evitar bloqueos o fallas.

**Ejemplo (sin depender de _wrappers_ externos):**  

```kotlin
val inputPath = "/sdcard/input.mp4"
val outputPath = "/sdcard/output.mp3"

// Comando FFmpeg
val command = arrayOf(
    "ffmpeg", "-i", inputPath,
    "-vn", "-ar", "44100", "-ac", "2", "-b:a", "192k",
    outputPath
)

try {
    val process = ProcessBuilder(*command)
        .redirectErrorStream(true)
        .start()

    val reader = process.inputStream.bufferedReader()
    reader.forEachLine { line -> Log.d("FFmpeg", line) }

    val exitCode = process.waitFor()
    Log.d("FFmpeg", "Proceso finalizado con código: $exitCode")
} catch (e: Exception) {
    Log.e("FFmpeg", "Error ejecutando FFmpeg: ${e.message}", e)
}
```

### *ML Kit*
Es un SDK de Google que ofrece capacidades de inteligencia artificial y aprendizaje automático (_machine learning_) optimizadas para dispositivos móviles.  
Permite realizar análisis multimedia mediante modelos integrados o personalizados, con o sin conexión.

**Características principales:**
- Detección facial, de texto y de códigos de barras.
- Clasificación de imágenes y objetos.
- Reconocimiento de texto en tiempo real.
- Compatibilidad con **_TensorFlow Lite_** para modelos personalizados.

**Ventajas:**
- No requiere conocimientos avanzados de _machine learning_.
- Procesamiento local eficiente y seguro.
- Integración directa con **CameraX** y **Compose**.

**Limitaciones:**
- Limitado a los modelos disponibles en el SDK.
- Dependencia de los servicios de Google Play para algunas funciones.

**Ejemplo:**  

```kotlin
val image = InputImage.fromFilePath(context, uri)
val recognizer = TextRecognition.getClient(TextRecognizerOptions.DEFAULT_OPTIONS)

recognizer.process(image)
    .addOnSuccessListener { visionText ->
        Log.d("MLKit", "Texto detectado: ${visionText.text}")
    }
    .addOnFailureListener { e ->
        Log.e("MLKit", "Error: ${e.message}")
    }
```

### *OpenCV (Open Source Computer Vision Library)*
Es una librería ampliamente utilizada para tareas de visión por computadora (_computer vision_) y análisis de imágenes.  
Proporciona cientos de funciones optimizadas para la manipulación de imágenes y videos en tiempo real.

**Características principales:**
- Detección de bordes, rostros y movimiento.
- Aplicación de filtros y transformaciones geométricas.
- Procesamiento de video cuadro a cuadro.
- Integración con NDK y compatibilidad con Kotlin/Java.

**Ventajas:**
- Ideal para procesamiento científico y visión avanzada.
- Gran comunidad y documentación.
- Compatible con Android, iOS y escritorio.

**Limitaciones:**
- Requiere configuración inicial del entorno NDK.
- La curva de aprendizaje puede ser elevada para casos complejos.

**Ejemplo:**  

```kotlin
val src = Imgcodecs.imread("/sdcard/input.jpg")
val dst = Mat()
Imgproc.GaussianBlur(src, dst, Size(15.0, 15.0), 0.0)
Imgcodecs.imwrite("/sdcard/output_blur.jpg", dst)
```

### Comparativa general

| Característica              | FFmpeg                          | ML Kit               | OpenCV                        |
|-----------------------------|---------------------------------|----------------------|-------------------------------|
| Propósito principal         | Edición y conversión multimedia | Análisis inteligente | Procesamiento de imagen/video |
| Licencia                    | LGPL / GPL                      | Propietaria (Google) | BSD                           |
| Nivel de API                | Bajo (nativo)                   | Medio / Alto         | Medio                         |
| Dependencia del dispositivo | Alta (NDK)                      | Media                | Alta (NDK)                    |
| Procesamiento local         | Sí                              | Sí                   | Sí                            |
| Capacidades de IA           | No                              | Sí                   | Parcial (modelos externos)    |
| Complejidad de uso          | Alta                            | Media                | Alta                          |

### 📚 Recursos recomendados
- [Repositorio oficial de FFmpeg](https://github.com/FFmpeg/FFmpeg)
- [Guía oficial de ML Kit](https://developers.google.com/ml-kit)
- [OpenCV para Android](https://opencv.org/android/)

## Transmisión y *streaming* en tiempo real (*WebRTC*, *RTMP*, *HLS*, *DASH*)

> 💡 **Recomendación:** Para la transmisión de audio y video en tiempo real, se deben evaluar las necesidades de latencia, compatibilidad de plataformas y formatos soportados.  
> **WebRTC** es ideal para comunicaciones en tiempo real con baja latencia. **RTMP** es apropiado para _streaming_ hacia servidores y plataformas de distribución. **HLS** y **DASH** permiten transmisión adaptativa en redes variables.

### *WebRTC*
Es un estándar abierto que permite comunicación de audio, video y datos en tiempo real entre navegadores y aplicaciones móviles, sin necesidad de _plugins_ adicionales.

**Características principales:**
- Transmisión en tiempo real con baja latencia.
- _Peer-to-peer_ o mediante servidores de señalización.
- Compatible con Android, iOS y web.
- Soporta audio, video y canales de datos.

**Ventajas:**
- Baja latencia y comunicación directa.
- Amplio soporte multiplataforma.
- Integración con _frameworks_ modernos de Android y Kotlin.

**Limitaciones:**
- Requiere servidor de señalización para negociación de _peers_.
- Complejidad mayor para escalabilidad de múltiples participantes.

**Ejemplo:**  

```kotlin
val iceServers = listOf(
    PeerConnection.IceServer.builder("stun:stun.l.google.com:19302").createIceServer()
)

val pcFactory = PeerConnectionFactory.builder().createPeerConnectionFactory()
val peerConnection = pcFactory.createPeerConnection(
    iceServers,
    object : PeerConnection.Observer {
        override fun onIceCandidate(candidate: IceCandidate) {}
        override fun onAddStream(stream: MediaStream) {}
        override fun onIceConnectionChange(newState: PeerConnection.IceConnectionState) {}
        override fun onDataChannel(dc: DataChannel) {}
        override fun onSignalingChange(state: PeerConnection.SignalingState) {}
        override fun onIceGatheringChange(state: PeerConnection.IceGatheringState) {}
        override fun onRemoveStream(stream: MediaStream) {}
        override fun onRenegotiationNeeded() {}
        override fun onAddTrack(receiver: RtpReceiver?, streams: Array<out MediaStream>?) {}
    }
)
```

### *RTMP (Real-Time Messaging Protocol)*
Es un protocolo de transmisión utilizado para enviar audio y video a servidores de _streaming_ o plataformas como YouTube o Twitch.

**Características principales:**
- Envío de flujos de audio/video a servidores.
- Compatibilidad con _software_ de _streaming_ y servidores RTMP.
- Soporte de codificación **_H.264_** y **_AAC_** en la mayoría de implementaciones.

**Ventajas:**
- Estándar probado para _streaming_ en vivo.
- Integración directa con servidores de media y plataformas de transmisión.

**Limitaciones:**
- Latencia mayor que _WebRTC_.
- Requiere servidor _RTMP_ o servicio externo.

**Ejemplo:**  

```kotlin
val rtmpCamera = RtmpCamera1(surfaceView, object : ConnectCheckerRtmp {
    override fun onConnectionSuccessRtmp() { Log.d("RTMP", "Conexión exitosa") }
    override fun onConnectionFailedRtmp(reason: String) { Log.e("RTMP", reason) }
    override fun onDisconnectRtmp() { Log.d("RTMP", "Desconectado") }
    override fun onAuthErrorRtmp() { Log.e("RTMP", "Error de autenticación") }
    override fun onAuthSuccessRtmp() { Log.d("RTMP", "Autenticado") }
})
rtmpCamera.startStream("rtmp://tu-servidor/live/streamkey")
```

### *HLS (HTTP Live Streaming)* y *DASH (Dynamic Adaptive Streaming over HTTP)*
Son protocolos de _streaming_ adaptativo, diseñados para transmitir contenido multimedia sobre [HTTP](../../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#http-hypertext-transfer-protocol) con control de calidad dinámico según la red.

**Características principales:**
- Adaptación automática de la calidad de video según ancho de banda.
- Compatible con la mayoría de plataformas móviles y navegadores.
- Basado en fragmentos segmentados de video (**_chunks_**).

**Ventajas:**
- Buena experiencia de usuario en redes inestables.
- Amplio soporte en Android, iOS y navegadores web.

**Limitaciones:**
- Latencia mayor que _WebRTC_ y _RTMP_.
- Dependencia de un servidor [HTTP](../../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#http-hypertext-transfer-protocol) o [CDN](../../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#cdn-content-delivery-network) para entregar los segmentos.

**Ejemplo:**  

```kotlin
val dataSourceFactory = DefaultHttpDataSource.Factory()
val hlsMediaSource = HlsMediaSource.Factory(dataSourceFactory)
    .createMediaSource(MediaItem.fromUri("https://example.com/live/playlist.m3u8"))

val player = ExoPlayer.Builder(context).build().apply {
    setMediaSource(hlsMediaSource)
    prepare()
    play()
}

binding.playerView.player = player
```

### Comparativa general

| Característica                 | WebRTC                       | RTMP                            | HLS                                | DASH                               |
|--------------------------------|------------------------------|---------------------------------|------------------------------------|------------------------------------|
| Latencia                       | Muy baja                     | Media                           | Alta                               | Alta                               |
| Tipo de transmisión            | Peer-to-peer / server        | Push a servidor                 | Pull desde HTTP/CDN                | Pull desde HTTP/CDN                |
| Adaptación de calidad          | Limitada                     | Limitada                        | Sí                                 | Sí                                 |
| Audio/Video                    | Sí                           | Sí                              | Sí                                 | Sí                                 |
| Multiplicidad de destinatarios | Limitada nativamente         | Alta mediante servidor          | Alta                               | Alta                               |
| Facilidad de integración       | Media                        | Media                           | Alta                               | Alta                               |
| Escenarios recomendados        | Video llamadas, conferencias | Streaming en vivo a plataformas | Streaming adaptativo en apps o web | Streaming adaptativo en apps o web |

### 📚 Recursos recomendados
- [Guía oficial de WebRTC para Android](https://webrtc.github.io/webrtc-org/native-code/android/)
- [Índice API WebRTC para Android](https://chromium.googlesource.com/external/webrtc/+/HEAD/sdk/android/api/org/webrtc)
- [RTMP Android Library: RootEncoder](https://github.com/pedroSG94/rtmp-rtsp-stream-client-java)
- [ExoPlayer HLS](https://developer.android.com/media/media3/exoplayer/hls) & [DASH](https://developer.android.com/media/media3/exoplayer/dash)
- [DASH.js (web reference)](https://github.com/Dash-Industry-Forum/dash.js)

## Almacenamiento y *caching* multimedia

> 💡 **Recomendación:** Para mejorar la experiencia de usuario y reducir el uso de red, se deben implementar estrategias de _caching_ de audio y video, especialmente en aplicaciones con _streaming_ o reproducción frecuente de contenido multimedia.

### *Cache* de medios en Android
El _caching_ permite almacenar temporalmente fragmentos o archivos multimedia en el dispositivo para:
- Reducir la carga de la red.
- Minimizar la latencia en reproducciones repetidas.
- Mejorar la fluidez en _streaming_ adaptativo.

**Características principales:**
- Almacenamiento en memoria RAM o en disco.
- Gestión de expiración y tamaño máximo del _cache_.
- Integración con reproductores multimedia como _ExoPlayer_.

**Ventajas:**
- Experiencia de usuario más rápida y estable.
- Disminuye consumo de datos.
- Permite reproducir contenido sin conexión temporal.

**Limitaciones:**
- Requiere control de tamaño y limpieza de _cache_.
- Consumo de espacio en el dispositivo.

### Librerías y herramientas comunes
- **_ExoPlayer Cache_:** Soporte nativo de _cache_ de segmentos HLS/DASH o archivos locales.
- **_DiskLruCache_:** _Cache_ en disco con política **LRU** (**_Least Recently Used_**), útil para recursos multimedia.
- **_Room_:** Base de datos local para almacenar metadatos o referencias a archivos cacheados.
- **_WorkManager / Coroutines_:** Para limpieza y mantenimiento de _cache_ en segundo plano.

**Ejemplos:**  

```kotlin
// Configuración de cache de ExoPlayer y SimpleCache
val cacheDir = File(context.cacheDir, "media")
val simpleCache = SimpleCache(cacheDir, LeastRecentlyUsedCacheEvictor(100L * 1024L * 1024L)) // 100 MB

val dataSourceFactory = DefaultDataSource.Factory(context)
val cacheDataSourceFactory = CacheDataSource.Factory()
    .setCache(simpleCache)
    .setUpstreamDataSourceFactory(dataSourceFactory)
    .setFlags(CacheDataSource.FLAG_IGNORE_CACHE_ON_ERROR)

val mediaItem = MediaItem.fromUri("https://example.com/video.m3u8")
val player = ExoPlayer.Builder(context).build().apply {
    setMediaSource(HlsMediaSource.Factory(cacheDataSourceFactory).createMediaSource(mediaItem))
    prepare()
    play()
}

binding.playerView.player = player

// Ejemplo de uso de Room para guardar referencias a archivos cacheados
@Entity(tableName = "media_cache")
data class MediaCache(
    @PrimaryKey val url: String,
    val localPath: String,
    val timestamp: Long
)

@Dao
interface MediaCacheDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(mediaCache: MediaCache)

    @Query("SELECT * FROM media_cache WHERE url = :url")
    suspend fun get(url: String): MediaCache?
}

@Database(entities = [MediaCache::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun mediaCacheDao(): MediaCacheDao
}
```

### Comparativa de *caching*

| Herramienta / Librería   | Tipo de caching                                          | Casos de uso recomendados                                                            |
|--------------------------|----------------------------------------------------------|--------------------------------------------------------------------------------------|
| ExoPlayer Cache          | Segmentos de audio/video en memoria o disco              | Streaming HLS/DASH, reproducción adaptativa con minimización de red                  |
| DiskLruCache             | Archivos en disco con política LRU (Least Recently Used) | Recursos multimedia locales o descargas temporales                                   |
| Room                     | Base de datos local (SQLite)                             | Almacenar metadatos, rutas de archivos cacheados, gestión de índices                 |
| WorkManager / Coroutines | Tareas en segundo plano                                  | Limpieza periódica de cache, mantenimiento de almacenamiento y control de expiración |

### 📚 Recursos recomendados
- [ExoPlayer Caching (Official Docs)](https://developer.android.com/media/media3/exoplayer/downloading-media) – Documentación oficial sobre `SimpleCache`, `CacheDataSource` y estrategias de _caching_ en Android.
- [DiskLruCache (archivado en 2023)](https://github.com/JakeWharton/DiskLruCache) – Implementación LRU de _cache_ en disco para Android. El repositorio está archivado y no recibe actualizaciones, pero sigue siendo útil como referencia.
- [Room Persistence Library](https://developer.android.com/training/data-storage/room) – Documentación oficial de **Room** para almacenamiento local y gestión de datos.
- [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager) – Para tareas en segundo plano, útil en limpieza y mantenimiento de _cache_.

## Procesamiento y efectos en tiempo real

> 💡 **Recomendación:** Para aplicaciones que necesiten aplicar filtros, efectos o análisis a audio y video en reproducción o _streaming_, se deben considerar la latencia, el consumo de CPU/GPU y la compatibilidad con frameworks multimedia existentes como _ExoPlayer_ o _ML Kit_.

### Filtros y efectos de video/audio
El procesamiento en tiempo real permite aplicar transformaciones sobre _frames_ de video o señales de audio mientras se reproducen o capturan.

**Características principales:**
- Aplicación de filtros de color, efectos visuales o máscaras en video.
- Procesamiento de audio: ecualización, normalización, efectos especiales.
- Integración con cámaras en vivo o flujos de media existentes.
- Compatible con Android, iOS y en algunos casos con web (OpenGL/WebGL).

**Ventajas:**
- Mejora la experiencia de usuario con efectos dinámicos.
- Posibilita análisis de video y audio en tiempo real (detección de objetos, reconocimiento de gestos o audio).
- Integración con librerías de aprendizaje automático (_ML Kit_, _TensorFlow Lite_).

**Limitaciones:**
- Requiere mayor uso de CPU/GPU, con posible impacto en batería y temperatura.
- Puede aumentar la latencia si no se optimiza correctamente.
- Curva de aprendizaje más elevada que reproducción simple de media.

### Librerías y herramientas comunes
- **OpenGL / OpenGL ES**: Renderizado de gráficos y procesamiento de frames en GPU.
- **RenderScript** (deprecado en Android 12, aún usable en proyectos existentes): procesamiento de imágenes y video usando CPU/GPU.
- **GPUImage / GPUImage2**: filtros y efectos en tiempo real con **_OpenGL_**.
- **ML Kit (Google)**: análisis de video y audio en tiempo real, detección de rostros, objetos, texto, etc.
- **ExoPlayer + VideoProcessor**: integración para procesar frames antes de mostrar en pantalla.

**Ejemplos:**
- Aplicación de filtro de video simple con **_GPUImage2_**

```kotlin
val gpuImageView = GPUImageView(context)
val filter = GPUImageSepiaFilter()
gpuImageView.setFilter(filter)
gpuImageView.setImage(BitmapFactory.decodeResource(resources, R.drawable.video_frame))
```

- Procesamiento de frames con **_OpenGL_** en **_ExoPlayer_**

```kotlin
val frameProcessor = VideoFrameProcessor.Builder(context, SimpleVideoProcessorListener())
    .setInputSurface(surface)
    .setOutputSurface(playerView.surface)
    .build()

player.setVideoFrameProcessor(frameProcessor)
player.prepare()
player.play()
```

- Análisis de cámara en tiempo real con **_ML Kit_** (detección de rostros)

```kotlin
val imageAnalyzer = ImageAnalysis.Builder().build()
imageAnalyzer.setAnalyzer(executor, FaceDetectionAnalyzer { faces ->
    faces.forEach { face ->
        Log.d("MLKit", "Rostro detectado con ID: ${face.trackingId}")
    }
})
cameraProvider.bindToLifecycle(this, cameraSelector, imageAnalyzer)
```

### Comparativa general

| Herramienta / Librería     | Tipo de procesamiento                   | Casos de uso recomendados                                                 |
|----------------------------|-----------------------------------------|---------------------------------------------------------------------------|
| OpenGL / OpenGL ES         | GPU, shaders y renderizado de frames    | Filtros de video en tiempo real, efectos visuales avanzados               |
| RenderScript               | CPU/GPU, procesamiento de imágenes      | Transformaciones de imágenes, efectos simples, pipelines de procesamiento |
| GPUImage / GPUImage2       | GPU, filtros predefinidos               | Aplicar filtros y efectos en tiempo real con mínima configuración         |
| ML Kit                     | CPU/GPU, visión por computadora / audio | Detección de rostros, texto, objetos, análisis de audio en tiempo real    |
| ExoPlayer + VideoProcessor | GPU/CPU, integración con reproductor    | Modificación de frames antes de renderizar, streaming con efectos         |

### 📚 Recursos recomendados
- [GPUImage for Android](https://github.com/cats-oss/android-gpuimage) – Librería para filtros y efectos de video en tiempo real en Android (OpenGL).
- [Face Detection con ML Kit](https://developers.google.com/ml-kit/vision/face-detection) – Detección de rostros, objetos y análisis de video/audio en tiempo real.
- [Media3 / ExoPlayer – VideoFrameProcessor (API Reference)](https://developer.android.com/reference/kotlin/androidx/media3/common/VideoFrameProcessor) – Integración para procesar frames en ExoPlayer/Media3.
- [OpenGL ES Android (Guía oficial)](https://developer.android.com/guide/topics/graphics/opengl) – Guía oficial de OpenGL ES en Android.

## Consideraciones clave: formatos, optimización y multiplataforma

> 💡 Tip: Este resumen sirve como _checklist_ para asegurar compatibilidad, eficiencia y consistencia en proyectos Android o multiplataforma que manejen audio, video y efectos en tiempo real.

### :one: Formatos
Guía rápida de compatibilidad y limitaciones de formatos de audio y video en Android:

| Tipo                 | Formatos recomendados      | Notas                                                                                                           |
|----------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------|
| Video                | MP4 (H.264/AVC), WebM, 3GP | ExoPlayer soporta más (DASH, HLS, SmoothStreaming). MediaPlayer limitado a los comunes.                         |
| Audio                | MP3, AAC, WAV, FLAC        | AudioTrack / AudioRecord usan PCM crudo; decodificación de formatos comprimidos requiere librerías adicionales. |
| Streaming adaptativo | HLS, DASH, SmoothStreaming | Requiere librerías especializadas; latencia y compatibilidad dependen del protocolo.                            |

### :two: Optimización
_Checklist_ de buenas prácticas para mejorar el rendimiento y eficiencia al procesar multimedia en Android:

- **GPU vs CPU:** Procesar efectos en GPU (**_OpenGL_** / **_GPUImage_**) suele ser más eficiente que CPU (**_RenderScript_**).
- **Threads:** Ejecutar análisis de _frames_ o audio en hilos separados para no bloquear la UI.
- **Batching:** Agrupar _frames_ o paquetes de audio para reducir _overhead_.
- **Buffering:** Ajustar tamaño de _buffer_ en **_AudioTrack_**, **_ExoPlayer_** o _streaming_ para balancear latencia y estabilidad.
- **Eficiencia energética:** Evitar procesamientos innecesarios; usar modos de _streaming_ en **_ML Kit_** o procesamiento por _frame_ parcial.

### :three: Interoperabilidad multiplataforma
La siguiente tabla indica, para cada funcionalidad, las librerías correspondientes en Android y iOS, como también si se puede usar mediante un _wrapper_ o si requiere implementación nativa por plataforma para el caso de KMP.  
A tener en cuenta:
1. **KMP wrapper** :arrow_right: Existe una librería o _wrapper_ que permite usar esa funcionalidad de forma multiplataforma dentro de un módulo común (``shared``). Por ejemplo, un módulo KMP puede exponer funciones de _ExoPlayer_ o _WebRTC_ a código común usando un _wrapper_ que internamente llama al código nativo de Android o iOS.
2. **Nativo** :arrow_right: No hay un _wrapper_ KMP disponible; para usar esa funcionalidad en KMP, hay que escribir código específico por plataforma (`expect/actual`), llamando directamente a la API nativa de Android o iOS.

| 🔧 Funcionalidad                      | 🤖 Android               | 🍏 iOS                          | 🌐 KMP      |
|---------------------------------------|--------------------------|---------------------------------|-------------|
| 🎬 Reproducción audio/video           | MediaPlayer, ExoPlayer   | AVPlayer / AVFoundation         | KMP wrapper |
| 📸 Captura fotos/video                | CameraX, MediaRecorder   | AVCaptureSession / AVFoundation | Nativo      |
| 🎤 Grabación audio PCM                | AudioRecord / AudioTrack | AVAudioRecorder / AVAudioPlayer | Nativo      |
| ✂️ Edición / Procesamiento multimedia | FFmpeg                   | FFmpeg                          | KMP wrapper |
| 🤖 Análisis inteligente (IA/ML)       | ML Kit                   | ML Kit                          | KMP wrapper |
| 🧠 Visión por computadora             | OpenCV                   | OpenCV                          | KMP wrapper |
| 📡 Streaming en tiempo real           | WebRTC                   | WebRTC                          | KMP wrapper |
| 🔴 Streaming a servidores             | RTMP / RootEncoder       | —                               | Nativo      |
| 📺 Streaming adaptativo               | HLS / DASH               | AVPlayer / AVFoundation         | KMP wrapper |
