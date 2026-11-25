# Algoritmos de clustering

**Beste Ozaydin · Factoría F5**

## 1. ¿Qué son el aprendizaje no supervisado y el clustering?

En el aprendizaje supervisado, cada ejemplo de entrenamiento tiene una respuesta conocida. Por ejemplo, si sabemos qué clientes dejaron de comprar, podemos entrenar un modelo para predecir el abandono. En el aprendizaje no supervisado, el modelo recibe las características de los ejemplos y busca patrones sin una variable objetivo.

El *clustering* es una tarea de aprendizaje no supervisado que agrupa observaciones según un criterio de similitud. En el ejemplo de mi píldora, cada punto representa un cliente. El eje horizontal muestra cuántos pedidos hizo en los últimos 90 días y el vertical, cuánto gastó de media por pedido. Podemos usar estas variables para descubrir perfiles de compra y luego describirlos en unidades fáciles de interpretar, como pedidos y euros.

![Clientes representados por número de pedidos y gasto medio](figuras/clientes.png)

La elección de variables afecta a los grupos. Si usamos distancias euclídeas y el gasto varía mucho más que el número de pedidos, el gasto puede dominar el cálculo. Estandarizar ambas variables ayuda a evitar que la escala de una sola determine el resultado.

## 2. Clustering basado en centroides y clustering jerárquico

**K-Means** representa el enfoque basado en centroides. Divide los datos en un número fijo de grupos, *K*, y asigna cada punto al centroide más cercano. Cada centroide es la media de los puntos asignados a su grupo. El resultado es una partición con un único nivel de detalle.

El **clustering jerárquico aglomerativo** empieza con un grupo por observación y va uniendo los dos grupos más cercanos. La regla de enlace define qué significa «más cercanos». Por ejemplo, Ward escoge la unión que menos aumenta la dispersión interna. El proceso genera un árbol llamado *dendrograma*. Al cortarlo a distintas alturas podemos ver agrupaciones más generales o más detalladas.

## 3. Clustering basado en densidad

**DBSCAN** busca regiones donde hay suficientes puntos cerca unos de otros. Usa un radio de vecindad (`eps`) y un mínimo de observaciones (`min_samples`), contando el propio punto. Un punto que alcanza el mínimo es un punto núcleo. Los núcleos conectados forman un grupo y los puntos de borde pueden pertenecer a él. Las observaciones que quedan fuera se etiquetan como ruido.

En K-Means, el criterio es la cercanía a un centroide. En el método aglomerativo, los grupos se forman mediante fusiones sucesivas según la regla de enlace. En DBSCAN, el criterio es la conexión entre vecindades densas. Por eso puede seguir un grupo curvo sin fijar de antemano el número de grupos.

| K-Means | Aglomerativo con Ward | DBSCAN |
| --- | --- | --- |
| ![K-Means divide las medias lunas](figuras/lunas-kmeans.png) | ![Ward también corta las medias lunas](figuras/lunas-ward.png) | ![DBSCAN recupera las dos medias lunas](figuras/lunas-dbscan.png) |

En este ejemplo de la píldora, los tres métodos reciben los mismos puntos. K-Means y Ward cortan las medias lunas, mientras que DBSCAN sigue las dos curvas.

## 4. Elección de parámetros y evaluación

Los parámetros cambian qué consideramos un grupo. En K-Means, un *K* demasiado pequeño puede mezclar perfiles distintos y uno demasiado grande puede fragmentar un perfil en varios grupos. En DBSCAN, un `eps` muy pequeño puede fragmentar grupos o clasificar muchos puntos como ruido. Uno demasiado grande puede unir grupos separados.

**Método del codo.** Entrenamos K-Means con varios valores de *K* sobre los mismos datos y representamos la inercia, que es la suma de las distancias cuadradas de cada punto a su centroide. Buscamos dónde la curva empieza a aplanarse. A partir de ese punto, añadir grupos aporta poca mejora en inercia. Puede haber varios valores razonables o ningún codo claro.

![Curva de inercia según el número de clusters](figuras/codo.png)

**Coeficiente de silueta.** Para cada observación, calculamos *a*, su distancia media a los demás puntos de su grupo, y *b*, la menor distancia media a otro grupo. La fórmula es `(b − a) / max(a, b)`. Si *a* vale 2 y *b* vale 6, la silueta es aproximadamente 0,67. Los valores cercanos a 1 indican buena separación, los cercanos a 0 indican que el punto está cerca de la frontera y los negativos indican que, en promedio, está más cerca de otro grupo. Podemos comparar la media entre configuraciones y revisar los valores por grupo.

![Gráfico de silueta de la segmentación de clientes con K igual a 3](figuras/silueta.png)

Antes de usar una segmentación de clientes, también comprobaría si aparecen perfiles parecidos al repetir el análisis con otra muestra o en otro periodo.

## 5. Comparación de los cuatro algoritmos

| Algoritmo | Cómo funciona | Ventajas | Desventajas | Usos comunes |
| --- | --- | --- | --- | --- |
| **K-Means** | Alterna entre asignar cada punto al centroide más cercano y recalcular los centroides como medias. | Rápido y fácil de interpretar cuando los grupos son compactos. | Hay que fijar *K*. Es sensible a la escala, a los valores atípicos y a la inicialización. Puede cortar grupos alargados o curvos. | Segmentación de clientes por comportamiento y reducción de la paleta de colores de una imagen. |
| **Jerárquico aglomerativo** | Une grupos según una regla de enlace y guarda las fusiones en un dendrograma. | Permite explorar varios niveles de agrupación sin fijar *K* antes de construir el árbol. | Puede consumir mucha memoria con conjuntos grandes. La métrica y el enlace cambian el resultado, y una fusión temprana no se deshace. | Agrupación de genes por perfiles de expresión y exploración de familias de documentos. |
| **DBSCAN** | Encuentra núcleos en regiones densas, los conecta y añade los puntos de borde alcanzables. | Detecta formas irregulares e identifica ruido sin fijar el número de clusters. | Es sensible a `eps`, `min_samples` y a la escala. Un mismo radio puede funcionar mal cuando los grupos tienen densidades distintas. | Identificación de zonas de alta concentración de accidentes y detección de observaciones aisladas. |
| **Mezclas gaussianas (GMM)** | Modela los datos como una mezcla de distribuciones gaussianas. Habitualmente usa EM para alternar entre estimar las probabilidades de pertenencia y actualizar los parámetros de cada componente. | Permite grupos elípticos y solapados. Da una probabilidad de pertenencia a cada componente. | Hay que seleccionar el número de componentes. El ajuste depende de la inicialización y puede ser inestable con pocos datos o demasiados parámetros. | Segmentación de imágenes y perfiles de clientes cuando la pertenencia a un segmento es incierta. |

![Componentes gaussianos elípticos con zonas de solapamiento](figuras/gmm.png)

Cuando dos componentes gaussianos se solapan, un punto puede tener probabilidades parecidas de pertenecer a ambos. Esto ayuda a expresar la incertidumbre de la asignación.

*Figuras de la píldora «Unsupervised Learning and Clustering», elaboradas con datos sintéticos.*
