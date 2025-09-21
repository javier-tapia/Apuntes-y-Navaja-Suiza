# NoSQL

- [Esquema No-Relacional](#esquema-no-relacional)
- [Flexibilidad en el esquema](#flexibilidad-en-el-esquema)
- [Escalabilidad horizontal](#escalabilidad-horizontal)
- [*Queries* complejas](#queries-complejas)

---

## Esquema No-Relacional
- Las bases de datos relacionales requieren un esquema rígido con tablas y relaciones previamente definidas, utilizando un modelo de tablas, filas y columnas. En contraste, las bases de datos NoSQL son esquemáticas y ofrecen flexibilidad en la estructura de documentos almacenados con mayor eficiencia. Esta flexibilidad elimina la necesidad de establecer relaciones rígidas entre conjuntos de datos, permitiendo una adaptación más dinámica a las necesidades cambiantes de los datos.  
- **Ejemplo**: Consideremos una base de datos de productos de un catálogo. En una base de datos relacional, cada producto tendría que ajustarse a un esquema predefinido con campos como 'nombre', 'descripción', 'precio' y 'categoría'. Pero, ¿qué pasa si algunos productos tienen atributos adicionales como 'talla', 'color' o 'material'? Con NoSQL, cada documento puede tener su propia estructura, permitiendo agregar campos adicionales según sea necesario. Esto facilita la gestión de productos con atributos variables sin necesidad de alterar el esquema de toda la base de datos.

## Flexibilidad en el esquema
- Las bases de datos NoSQL ofrecen una notable flexibilidad en cuanto a cómo se estructuran y almacenan los datos. A diferencia de los sistemas relacionales, no es necesario definir un esquema rígido antes de almacenar la información. Esto permite que la estructura de los datos pueda variar de un documento a otro dentro de la misma colección, lo que es ideal para manejar datos no estructurados o semiestructurados. Gracias a ello, las bases de datos NoSQL son más ágiles y adecuadas para gestionar la variabilidad sin la necesidad de modificar constantemente la estructura de la base de datos.  
- **Ejemplo**: Pensemos en una red social donde cada usuario tiene un perfil. En una base de datos relacional, sería difícil manejar la diversidad de información que cada usuario puede agregar, como 'intereses', 'habilidades', 'ubicación' y 'amigos'. Con NoSQL, cada perfil de usuario puede ser un documento independiente con su propia estructura. Esto permite adaptarse a diferentes conjuntos de información para cada usuario, haciendo que la base de datos sea más flexible y fácil de mantener

## Escalabilidad horizontal
- Las bases de datos NoSQL están diseñadas para escalar horizontalmente, lo que permite manejar aumentos en el tráfico y el volumen de datos mediante la adición de más réplicas de lectura. Esto contrasta con las bases de datos relacionales, que suelen escalar verticalmente aumentando la capacidad de un solo servidor. Las réplicas de lectura comparten el mismo almacenamiento subyacente que la instancia principal, lo que permite una replicación de datos eficiente y de baja latencia. Este proceso es conocido como escalado horizontal de lectura.  
- **Ejemplo**: Imaginemos una aplicación con un alto volumen de datos y altas tasas de lectura y escritura. A medida que la aplicación crece, la base de datos necesita manejar más tráfico y más datos. Con NoSQL, podemos simplemente agregar más servidores a la infraestructura para distribuir la carga. Esta escalabilidad horizontal garantiza que la aplicación siga funcionando de manera eficiente, incluso con un aumento significativo en la demanda.

## *Queries* complejas
- NoSQL permite realizar *queries* complejas con varios campos y operaciones lógicas, aunque no es compatible con *full text search*.  
- **Ejemplo**: Supongamos que necesitamos implementar sistemas de paginación eficientes para manejar grandes volúmenes de datos. Con NoSQL, podemos realizar queries complejas que filtren, ordenen y limiten los resultados de manera eficiente. Esto permite dividir los datos en páginas y mostrar solo una parte a la vez, mejorando la experiencia del usuario y reduciendo la carga en la base de datos.
