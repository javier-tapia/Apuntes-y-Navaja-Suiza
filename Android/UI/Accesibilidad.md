<h1>Accesibilidad (<i>a11y</i>)</h1>

***Index***:
<!-- TOC -->
  * [Fundamentos: WCAG *mobile*](#fundamentos-wcag-mobile)
  * [Árbol de accesibilidad](#árbol-de-accesibilidad)
    * [Nodos](#nodos)
    * [Delegados](#delegados)
  * [Tecnologías asistivas](#tecnologías-asistivas)
    * [Servicios de accesibilidad de Android](#servicios-de-accesibilidad-de-android)
  * [Buenas prácticas y errores comunes](#buenas-prácticas-y-errores-comunes)
    * [Etiquetado de encabezados](#etiquetado-de-encabezados)
    * [Uso de precios/montos](#uso-de-preciosmontos)
    * [Uso de textos alternativos](#uso-de-textos-alternativos)
    * [Orden de navegación](#orden-de-navegación)
    * [Uso de roles](#uso-de-roles)
    * [Uso de `labelFor`](#uso-de-labelfor)
    * [Ocultar/Mostrar vistas](#ocultarmostrar-vistas)
    * [Regiones o elementos *clickeables* o accionables](#regiones-o-elementos-clickeables-o-accionables)
    * [Anuncio de mensajes dinámicos](#anuncio-de-mensajes-dinámicos)
    * [Vistas semánticas](#vistas-semánticas)
    * [*Custom Actions*](#custom-actions)
    * [Animaciones y *reduce motion*](#animaciones-y-reduce-motion)
<!-- TOC -->

---

## Fundamentos: WCAG *mobile*
Las Pautas de Accesibilidad para el Contenido Web, más conocidas por sus siglas en inglés como WCAG (*Web Content Accessibility Guidelines*), son un conjunto de reglas que ayudan a crear aplicaciones inclusivas.

Estas pautas se dividen en cuatro principios fundamentales que forman la base de la accesibilidad mobile:  
- **Perceptible**: Todo en la aplicación debe ser accesible a los sentidos. Esto incluye proporcionar descripciones para las imágenes y garantizar que el texto sea legible.  
- **Operable**: La interacción con la aplicación debe ser fluida y sencilla. Las personas usuarias deben poder navegar sin dificultad, ya sea tocando la pantalla, utilizando un teclado o incluso mediante comandos de voz.  
- **Comprensible**: El contenido debe ser legible y predecible para que sea accesible a la mayor cantidad de personas posible, incluidas aquellas que usan tecnologías de asistencia. Esto significa utilizar un lenguaje claro y simple, definir términos inusuales y mantener la coherencia en la navegación y el diseño. Además, es importante ayudar a los usuarios a evitar y corregir errores, proporcionando instrucciones claras, mensajes de error y la oportunidad de revisar su trabajo.  
- **Robusto**: La aplicación debe ser robusta y capaz de funcionar correctamente en diversas plataformas y con diferentes tecnologías. Al seguir buenas prácticas de codificación, se garantiza que la aplicación sea compatible con una variedad de navegadores y dispositivos, y se permite que todas las personas usuarias tengan una experiencia gratificante.

Entre otras características, las WCAG también establecen **tres niveles de conformidad:**

- **Nivel A**: Este nivel es el más básico. Si no se cumple, los usuarios con discapacidades encontrarán barreras significativas.  
- **Nivel AA**: Este es el nivel objetivo para la mayoría de los sitios web, ya que aborda los problemas más comunes y significativos. En este nivel, existen 55 criterios a tener en cuenta. Ejemplo: contraste de color.  
- **Nivel AAA**: Este es el nivel más alto de conformidad, que no todos los sitios pueden cumplir para todo el contenido, pero es ideal para alcanzar en la mayor medida posible.

## Árbol de accesibilidad
Es una representación estructural que organiza los elementos de la UI jerárquicamente para servicios de accesibilidad (por ejemplo, *TalkBack*), proporcionando información clave sobre estos como descripciones, estados, roles y acciones disponibles, y permitiendo que las personas con discapacidades puedan interactuar con la aplicación exitosamente.  
Este árbol es generado por Android de manera automática cuando se renderizan las vistas en la pantalla, creando un objeto nuevo para cada una de esas vistas llamados `AccessibilityNodeInfo`.

### Nodos
Las propiedades del objeto `AccessibilityNodeInfo` le indican al servicio de accesibilidad cuál es su rol, si es un botón, si está activo o no, qué acciones permite y hasta su descripción accesible. Cada vista de la *app*, desde un simple botón hasta una lista completa, se convierte en un nodo de información.  
La principal ventaja es que trabajar sobre esos nodos del árbol de accesibilidad permite abstraer de qué servicios seran consumidos, sin preocuparse por desarrollar para un servicio en particular.

### Delegados
Los `AccessibilityDelegates` o delegados son una herramienta poderosa en Android que actúan como intermediarios entre las vistas y el sistema de accesibilidad. Mientras que el nodo proporciona información sobre la vista para que los lectores de pantalla puedan interpretarla, los delegados permiten personalizar y enriquecer esta información.  Esto es crucial en situaciones donde la información predeterminada no es suficiente o no describe adecuadamente la función de una vista.

Los delegados permiten:

- **Personalizar descripciones**: Proporcionar descripciones más precisas o contextuales que mejoren la experiencia del usuario.  
- **Modificar comportamiento de interacción**: Controlar cómo se manejan eventos de accesibilidad, permitiendo que ciertos gestos o interacciones se interpreten de manera diferente según el contexto de la vista.  
- **Agrupar elementos**: Agrupar múltiples vistas y proporcionar una descripción conjunta que facilite la navegación, en lugar de que el usuario escuche descripciones individuales para cada vista.

## Tecnologías asistivas
Las tecnologías asistivas son **_herramientas y dispositivos diseñados para ayudar a las personas con discapacidad a interactuar de manera efectiva con sitios webs y aplicaciones_**, ofreciendo una amplia gama de alternativas y asegurando que todos los usuarios puedan navegar de manera eficiente y autónoma.

- **Lectores de pantalla**
- **Magnificadores de pantalla**
- **Ajustes de contraste**
- **Dispositivos adaptados (teclados alternativos)**
- **Switches**
- **Licornios o punteros**
- **Comandos de voz**

### Servicios de accesibilidad de Android
Android tiene tecnologías asistivas propias para asegurar la accesibilidad. A la hora de diseñar una *app*, es crucial asegurarse que sea compatible con todas las herramientas.  
*TalkBack*, el lector de pantalla nativo de Android, es una de las más conocidas. Pero también existen otras como *Select to Speak*, *Voice Access* o *Switch Access*.

## Buenas prácticas y errores comunes

### Etiquetado de encabezados
Los encabezados o *headings* son elementos que estructuran el contenido de una pantalla, proporcionando un orden jerárquico que facilita la navegación y comprensión del contenido.  
Permiten a *TalkBack* identificar la jerarquía del contenido, generando que los usuarios recorran rápidamente secciones como títulos y subtítulos. Un texto debe marcarse como encabezado (utilizando la propiedad `isAccessibilityHeading` o `android:accessibilityHeading`) cuando actúa como título o subtítulo de una sección. Esto incluye elementos que introducen nuevas secciones de contenido, o que destacan la información relevante dentro de una pantalla.

### Uso de precios/montos
Uno de los mayores desafíos en cuanto a la accesibilidad es que los montos y monedas varían según el país, y una mala implementación puede llevar a que el lector de pantalla los interprete y pronuncie mal.  
Un error frecuente es crear componentes personalizados sin considerar la accesibilidad, lo que lleva a que los montos se lean incorrectamente (por ejemplo, en dólares en lugar de la moneda local). Para evitarlo, se debe implementar la lógica de accesibilidad adecuada mediante el `AccessibilityDelegate` de la *custom view*.

### Uso de textos alternativos
Los textos alternativos o *content descriptions* en Android son descripciones que asignamos a elementos visuales como imágenes,íconos o botones gráficos. Estos textos permiten a los lectores de pantalla comunicar a los usuarios con discapacidad visual el propósito o contenido de esos elementos.  
Siempre que una imagen o icono cumpla una función o transmita información importante, debe tener un texto alternativo (utilizando la propiedad `android:contentDescription` o `contentDescription`).  
Sin embargo, si una imagen es puramente decorativa y no aporta valor adicional, es recomendable dejar el `contentDescription` vacío o marcado como nulo (`null`), para no distraer al usuario con descripciones innecesarias.

### Orden de navegación
En Android, hay dos atributos (`accessibilityTraversalBefore` | `android:accessibilityTraversalBefore="@id/x"` y `accessibilityTraversalAfter` | `android:accessibilityTraversalAfter="@id/x"`) que permiten controlar el orden en que los elementos de la interfaz son navegados por los usuarios que utilizan tecnologías asistivas como *TalkBack*.  
Por defecto, la navegación sigue el orden en que los elementos fueron agregados en el *layout* de una forma secuencial: arriba - abajo e izquierda - derecha. Sin embargo, cuando este orden no refleja el flujo de interacción deseado, puede personalizarse utilizando estas herramientas.

¿Cuándo usarlos?

1. **Diseños complejos o no lineales**: Diseños de tipo *grid* o interfaces con múltiples columnas.  
2. **Formularios**: El usuario debe ingresar información en un orden específico, pero el *layout* tiene elementos dispuestos de forma que no reflejan ese flujo.  
3. **Contenidos dinámicos**: Como listas expansibles o menús que se muestran y ocultan, lo que puede alterar el orden natural de navegación.

### Uso de roles
El uso de roles en componentes de la UI es fundamental para asegurar una experiencia accesible y fluida. Los lectores de pantalla dependen de ellos para proporcionar información clara y contextualizada sobre cada elemento interactivo.  
Siempre que sea posible, es importante utilizar componentes nativos de Android con roles predefinidos Por ejemplo, un `TabLayout` está diseñado para anunciar correctamente la cantidad de pestañas y su selección actual, lo cual no ocurre si se utilizan botones personalizados para replicar este comportamiento.

### Uso de `labelFor`
Permite que los componentes de texto actúen como etiquetas descriptivas para otras vistas en la misma pantalla relacionándolas entre sí. De esta forma, se proporciona contexto adicional sobre el propósito de un campo sin la necesidad de alterar el `contentDescription`.  
Siempre que haya un campo de texto o una vista que requiera una descripción adicional, como formularios, menús o configuraciones, es recomendable agregar un *Text* que sirva de etiqueta y marcarlo con `labelFor`.

### Ocultar/Mostrar vistas
Cuando diseñamos una aplicación accesible debemos controlar la visibilidad de ciertos elementos de la interfaz. El atributo `importantForAccessibility` | `android:importantForAccessibility=|yes|no|noHideDescendants|auto` juega un papel crucial en determinar si una vista es reconocida o ignorada por los lectores de pantalla.

¿Cuándo usarlo?

- **Ocultar elementos irrelevantes**: si tienes íconos decorativos o elementos visuales que no aportan información útil al usuario, puedes marcarlos con `importantForAccessibility="IMPORTANT_FOR_ACCESSIBILITY_NO"`.  
- **Mostrar información relevante:** para componentes clave, como botones o etiquetas de campos, asegúrate de que tengan `importantForAccessibility="IMPORTANT_FOR_ACCESSIBILITY_YES"`, garantizando que sean accesibles para todos los usuarios.  
- **Controlar grupos de vistas:** en contenedores o *layouts* complejos a veces es útil ocultar no solo una vista, sino todos sus elementos hijos. Utilizar `IMPORTANT_FOR_ACCESSIBILITY_NO_HIDE_DESCENDANTS` es una forma eficaz de lograr este resultado.

### Regiones o elementos *clickeables* o accionables
Es crucial que las áreas de interacción o acción dentro de la interfaz se detecten y describan correctamente. Los lectores de pantalla deben poder identificar los elementos *clickeables* y proporcionar al usuario una descripción clara de lo que sucede cuando interactúan con estos elementos.  
Para ajustar y definir qué vistas son interactuables se utiliza el `AccessibilityNodeInfo`**.** Además, usar también el `contentDescription` proporcionará una descripción concisa pero completa de lo que sucede al seleccionar la interacción.

¿Cuándo usarlo?

- ***Widget* interactivo**: para asegurarse de que un botón, imagen, o *checkbox* tenga una descripción clara de su funcionalidad.  
- **Vistas contenedoras**: cuando tienes vistas que también deben manejar acciones porque contienen un botón o *checkbox*, por ejemplo, para mejorar la usabilidad y permitir que los clics se gestionen en áreas más amplias.  
- **Acciones adicionales**: la interfaz permite que la persona usuaria realice acciones adicionales. Es necesario asegurarse de que el lector de pantalla anuncie correctamente el propósito de la acción.

### Anuncio de mensajes dinámicos
Los anuncios dinámicos permiten notificar a los usuarios de cambios importantes en la interfaz sin que tengan que navegar manualmente para descubrirlos. En Android, los anuncios tienen un propósito y dependen de la acción previa, como actualizar estados, informar errores o simplemente anunciar algo.

¿Cúando usarlos?

- **Cambios no visuales**: para informar sobre actualizaciones de estado, texto o acciones que no alteren la interfaz visual.  
- **Confirmación de acciones**: notificar al usuario cuando una acción importante, como enviar un formulario, se completa.  
- **Errores o advertencias**: alertar sobre errores o problemas que requieren atención inmediata sin que el usuario tenga que buscarlos.

Para gestionar manualmente cuándo realizar un anuncio, es importante considerar las siguientes situaciones:

- **Cambios de estados**: Utiliza la propiedad `stateDescription` ya sea por medio de *ViewCompat* o *View*.  
- **Anuncio de errores**: Usa la propiedad `setError` en `Nodeinfo`.  
- **Información general**: Usa los *Live regions* para notificar cambios relevantes al usuario

Debemos tener en cuenta que, si bien el método `announceForAccesibility` sigue siendo válido y ampliamente utilizado de forma nativa, Android lo marcará como obsoleto a partir de Android 16. Sin embargo, su código interno es público, lo que permite replicar su funcionalidad.

### Vistas semánticas
Proporcionan significado y contexto a los elementos de la interfaz de usuario, facilitando la comprensión de la información presentada. A veces, varias vistas transmiten un significado juntas pero no tienen sentido por separado. Por ejemplo, si pensamos en una casa, se puede señalar que las distintas habitaciones le dan sentido la una a la otra. Una casa con un living pero sin cocina o sin baño no sería habitable. Sin embargo, cuando el living está acompañado de una cocina, baños, y habitaciones, obtiene su utilidad.  
En estos casos, es útil agrupar las vistas para permitir que *TalkBack* navegue rápidamente toda la vista en lugar de tener que enfocar varios elementos individuales. Para eso, en el contenedor del grupo de vistas se debe setear el atributo `android:focusable` | `isFocusable` en `true`, como también proporcionar un `contentDescription` acorde a lo que representa el conjunto de vistas.

¿Cuándo usarlas?

- **Múltiples vistas relacionadas**: cuando hay varios elementos que trabajan juntos para transmitir un mensaje o realizar una acción, como un conjunto de opciones en un formulario.  
- **Mejorar la navegación**: para que el recorrido de la persona usuaria sea más intuitivo se puede agrupar las vistas relacionadas.  
- **Ofrecer información contextual**: cuando se desee proporcionar una descripción más rica y coherente que la que los servicios de accesibilidad dan automáticamente.

### *Custom Actions*
Las *Custom Actions* o como son anunciadas por *TalkBack* "acciones disponibles”, permiten definir acciones específicas que los usuarios pueden realizar en una *view*. Estas acciones se integran con los servicios de accesibilidad, brindando a las personas usuarias la posibilidad de interactuar con los elementos de una manera más significativa y eficiente.  
Para agregar una acción, se utiliza el método `ViewCompat.addAccessibilityAction()`. Esta función recibe como argumentos la vista a la cual se aplicará la acción, el *label* que mostrará al abrirse el menú de acciones (debe soportar los diferentes idiomas) y, finalmente, un *lambda* donde se define la acción a ejecutar.

¿Cuándo usarlas?

- **Cuando un elemento tiene múltiples acciones**: las *Custom Actions* permiten elegir qué acción se desea ejecutar.  
- **Para simplificar la navegación**: cuando se desea reducir el número de interacciones necesarias para completar una tarea.  
- **Cuando la funcionalidad estándar no es suficiente**: si las acciones predeterminadas de un componente no cubren las necesidades de la persona usuaria, las *Custom Actions* pueden proporcionar soluciones específicas.

### Animaciones y *reduce motion*
En cuanto a las animaciones y los movimientos dentro de la interfaz, es fundamental respetar las configuraciones de accesibilidad que las personas usuarias establecen en sus dispositivos. Siempre que sea posible, minimiza las animaciones de la pantalla cuando detectes que la persona usuaria ha configurado su dispositivo para reducir el movimiento (*reduce motion* o quitar animaciones). Esto ayuda a evitar la desorientación y la incomodidad para aquellos que son sensibles a las alteraciones visuales.  
En Android, los usuarios pueden desactivar las animaciones del dispositivo accediendo a **_Configuración_** ▶️ **_Accesibilidad_** ▶️ **_Quitar animaciones_**. De esta forma, se modifican varios tipos de animaciones:

- **Animaciones de transición de pantallas**: las transiciones de entrada/salida de actividades y fragmentos se realizan instantáneamente.  
- **Animaciones basadas en objetos `Animator`**: todas las animaciones basadas en objetos `Animator` se ejecutarán instantáneamente, lo que significa que tanto el valor de *delay* de inicio de la animación como la duración de la misma se reducirán a 0.  
- **Animaciones de ventana**: todas las animaciones a nivel de ventana, como las animaciones de entrada/salida de objetos `PopupWindow`, también se ejecutarán instantáneamente.
