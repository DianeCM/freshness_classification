Para la solución del problema se propone un modelo de red neuronal, resultante de aplicar fine-tuning a la red neuronal convolucional pre-entrenada en el dataset Imagenet, VGG16.

# Preparación del Dataset:

No fue necesario realizar data-augmentation, pues el dataset proporcionado ya lo contemplaba.

**Elección de dataset de entrenamiento, validación y evaluación**

- La sección de Test no contenía datos representativos de las clases: *freshbittergroud*, *freshcapsicum*, *rottencapsicum* y *rottenbittergroud*. Por lo cual, se utilizó alrededor del 30% de los datos de cada una de estas clases, disponibles en la sección de Train, para la evaluación del modelo. 

- Luego, de las restantes imágenes se tomó el 10% por cada una de las clases para la validación del modelo y el resto se destinó al entrenamiento. 

- Tanto la selección de las imágenes para completar el dataset de entrenamiento como las imágenes de validación se realizaron de manera aleatoria.

- Se ajusta el tamaño de todas las imágenes a 128×128.


# Modelo utilizado:

**VGG16**

Una red pre-entrenada es una red guardada que fue previamente entrenada en un gran conjunto de datos, típicamente en una tarea de clasificación de imágenes a gran escala. Si este conjunto de datos original es lo suficientemente grande y general, entonces la jerarquía espacial de las características aprendidas por la red pre-entrenada puede actuar efectivamente como un modelo genérico del mundo visual, y por lo tanto sus características pueden resultar útiles para muchos problemas diferentes de visión por computadora, incluso si estos nuevos problemas involucran clases completamente diferentes a las de la tarea original.

La arquitectura VGG16 es bastante simple, que usa bloques compuestos por un número progresivo de capas convolucionales con filtros de tamaño 3×3. Además, para reducir el tamaño de los mapas de activación que se van obteniendo, se intercalan bloques maxpooling entre los convolucionales, reduciendo a la mitad el tamaño de estos mapas de activación. Finalmente, se utiliza un bloque de clasificación compuesto por dos capas densas de 4096 neuronas cada una, y una última capa, que es la de salida, de 1000 neuronas.

Para realizar fine-tuning solo utilizamos los bloques convolucionales de esta red y añadimos nuestro propio clasificador.


**Clasificador**

Las capas de clasificación del modelo utilizado consisten en:
- Una capa Flatten: Esta capa convierte la imagen de tres dimensiones a una sola.

- Una capa densa con 128 neuronas ocultas y función de activación *relu*.

- Se añade dropout: El término "dropout" se refiere a la eliminación de nodos (de entrada y ocultos) en una red neuronal. Todas las conexiones hacia adelante y hacia atrás con un nodo eliminado se eliminan temporalmente, creando así una nueva arquitectura de red a partir de la red original. Los nodos se eliminan mediante una probabilidad de dropout de p (el valor usado en este caso es igual a 0.5). Este proceso es repetido continuamente, siempre escogiendo un nuevo conjunto de neuronas a ocultar. De esta forma el modelo aprende a comportarse adecuadamente cuando existen pequeños cambios en los datos de entrenamiento, y por tanto generaliza mejor.

- Como se trata de un problema de clasificación binaria, se termina la red con una sola unidad (una capa densa de tamaño 1) y activación *sigmoide*. Esta unidad codificará la probabilidad de que la red prediga una clase u otra.


# Entrenamiento

Para realizar fine-tuning o ajuste fino a la red neuronal preentrenada VGG16 se siguen los siguientes pasos:
- Añadir capas de clasificación en el tope del modelo preentrenado (de manera que solo obtenemos del modelo la base convolucional).
- Congelar la red base (el modelo pre-entrenado).
- Entrenar el modelo (solo serán entrenables los parámetros correspondientes a las capas de clasificación añadidas).
- Descongelar algunas capas en la base de la red para hacer el *ajuste fino*:
La idea es descongelar solo algunas de las capas superiores de la red. Las primeras capas en la base convolucional codifican características más genéricas y reutilizables, mientras que las capas más altas codifican características más especializadas. Por tanto, es más útil ajustar las características más especializadas, ya que son las que necesitan ser reutilizadas en nuestro nuevo problema. Habría un rápido decrecimiento en los beneficios al ajustar las capas inferiores.
En este caso solo se hace ajuste fino a las últimas tres capas convolucionales del modelo.

- Entrenar las capas descongeladas y las de clasificación: En este paso es importante tener en cuenta el uso de un valor bajo de *learning rate*. Dado que se desea limitar la magnitud de las modificaciones que se realizan a las representaciones de las capas que se están ajustando. Las actualizaciones que son demasiado grandes pueden dañar estas representaciones.


**Hiperparámetros**

- batch_size: 8
- learning rate : 0.00001
- En la primera fase del entrenamiento se realizan 10 epochs, dado que solo se tienen 1,048,833 parámetros entrenables. En la segunda fase se realizan 50 epochs.

El optimizador utilizado es RMSprop.


# Análisis de resultados

Aunque se obtienen valores altos de las métricas en el conjunto de evaluación, las curvas de aprendizaje generadas durante el entrenamiento no son las ideales. Los valores de *validation loss* y *validation accuracy* oscilan, aunque se mantienen en rangos aceptables la mayor parte de las *epochs*.
Posibles explicaciones:
  - El uso de tamaño de lote (batch_size) muy pequeño. En general, no existe receta para elegir el valor indicado de este hiperparámetro. En una primera evaluación del modelo no se obtuvieron buenos resultados usando un tamaño de lote igual a 32.

  - El tamaño del conjunto de validación puede ser demasiado pequeño, de modo que pequeños cambios en la salida causen grandes fluctuaciones en el error de validación.

# Recomendaciones
- Ajustar algunos parámetros como batch_size y dropout rate.
- Probar técnicas de regularización como L2, que es una de las más usadas.