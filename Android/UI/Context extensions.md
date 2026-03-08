<h1><i>Context extensions</i></h1>

***Index***:
<!-- TOC -->
  * [``Context.resources.getDimension…``](#contextresourcesgetdimension)
<!-- TOC -->

---

## ``Context.resources.getDimension…``
¿Qué diferencias hay entre `getDimension`, `getDimensionPixelSize` y `getDimensionPixelOffset`?

Los tres métodos leen un recurso de dimensión y **convierten automáticamente** la unidad definida (**`dp`**, **`sp`**, **`px`**, etc.) a **píxeles (px)** según la densidad del dispositivo.

Por ejemplo, si se define **`<dimen name="mi_dimen">14dp</dimen>`** y el dispositivo tiene densidad 1.33x (tvdpi), los métodos retornarán el valor convertido a px (~18.7px), no 14.

Donde difieren es en el tipo de retorno y en el redondeo:

| **Método**                      | **Retorna** | **Redondeo**                         |
|---------------------------------|-------------|--------------------------------------|
| **`getDimension()`**            | **`Float`** | Sin redondeo (valor exacto en px)    |
| **`getDimensionPixelSize()`**   | **`Int`**   | Redondea hacia arriba (al menos 1px) |
| **`getDimensionPixelOffset()`** | **`Int`**   | Trunca (redondea hacia abajo)        |

📌 **Ejemplo**:  

Si **`14dp`** se convierte a **`18.7px`**
- **`getDimension()`** :arrow_right: **`18.7f`**
- **`getDimensionPixelSize()`** :arrow_right: **`19`** (redondeado arriba)
- **`getDimensionPixelOffset()`** :arrow_right: **`18`** (truncado)

**Cuándo usar cada uno:**
- **`getDimension()`**: Para valores que necesitan precisión. Se puede usar por ejemplo con `textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, size)`, que acepta *Float* y mantiene la precisión
- **`getDimensionPixelSize()`**: Para tamaños de ``Views`` (**`height`**, **`width`**, **`padding`**) - garantiza al menos 1px
- **`getDimensionPixelOffset()`**: Para _offsets_/posiciones donde truncar es aceptable
