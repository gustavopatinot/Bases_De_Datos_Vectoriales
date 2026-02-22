# Bases_De_Datos_Vectoriales
Comprensión de la importancia de la Bases de Datos Vectoriales

- Desglosar este complejo tema desde sus cimientos atómicos hasta las arquitecturas algorïtmicas más avanzadas.
---
1. **La Unidad Fundamental: ¿Qué es un "Nodo"?**

En el contexto de las bases de datos vectoriales, un **nodo (o vértice en la terminología de grafos)** no es simplemente un punto de datos; es una representación matemática comprimida del significado.
Un nodo es un **vector**, denotado comúnmente como $x$. Es una lista ordenada de números reales en un espacio multidimensional. Formalmente,
```math
x=[x_0,x_1,…,x_{d−1}],
```
donde **$d$** es la **dimensión.**

Los **vectores** no se crean manualmente. Son el resultado de modelos de **aprendizaje profundo** (Deep Learning) o **redes neuronales** que procesan datos no estructurados (imágenes, texto, audio). Este proceso se llama **embedding (incrustación)**. Por ejemplo, un modelo como **VGG** para imágenes o **BERT** para texto toma el objeto original y lo transforma en este vector $x$, donde la posición en el espacio captura su **semántica.**

---
2. **La Fórmula Básica de las Bases de Datos Vectoriales**

La razón de ser de una base de datos vectorial es resolver el problema de la Búsqueda del Vecino Más Cercano  **(NNS - Nearest Neighbor Search)**.
**La Fórmula Fundamental:** Dada una base de datos finita $S$ y un vector de consulta (query) $q$, se busca un vector $p$ $\in$ $S$ tal que:
```math
p=arg\min_{x \in S}δ(x,q)
```

**Desglose de Variables:**

* $S$: El conjunto de todos los vectores (nodos) almacenados en la base de datos.
* $x$: Un vector cualquiera dentro de la base de datos.
* $q$: El vector de consulta (lo que el usuario está buscando, convertido a vector).
* $δ(x,q)$: La función de distancia que mide la disimilitud entre $x$ y $q$.
* $argmin$: Operación que busca el argumento (el vector $x$) que minimiza la distancia resultante

**Fórmula de Distancia $\delta$**
- La distancia más común es la **Distancia Euclidiana $\left( L_2 \right)$**:
```math
δ(x,y)=\sqrt{\sum_{i=0}^{d-1}\left( x_i - y_i \right)^{2}}
```

Donde:
* $d$: Es la dimensionalidad del vector (pueden ser cientos o miles).
* $x_{i},y_{i}$ : Son los valores escalares en la posición $i$ de los vectores $x$ e $y$.
Otra fórmula crítica, especialmente en IA moderna, es el **Producto Interno** (Inner Product) o **Similitud del Coseno** (para vectores normalizados), usada a menudo en sistemas de recomendación:
```math
Sim(x,q)= \sum_{i=0}^{d-1}x_i\cdot q_i
```
En este caso, buscamos maximizar (argmax) el valor en lugar de minimizarlo.

---
3. **El Fenómeno del "Mundo Pequeño" (Small World)**

Este es el corazón teórico de los algoritmos más eficientes hoy en día.
**Origen:** El concepto proviene de la sociología (el experimento de **Stanley Milgram**  de los [seis grados de separación](https://en.wikipedia.org/wiki/Six_degrees_of_separation)) y fue formalizado matemáticamente por **Watts y Strogatz** en 1998. Descubrieron que ciertas redes tienen dos propiedades simultáneas que parecen contradictorias:
* **Alta agrupación local** (mis amigos son amigos entre sí).
* **Camino promedio corto** (puedo llegar a cualquier persona del mundo en pocos saltos).
**La Fórmula de la Navegabilidad (Kleinberg)**: **Jon Kleinberg** (2000) llevó esto más allá, preguntando no solo si el camino corto existe, sino si un algoritmo voraz (greedy) puede encontrarlo usando solo información local. Demostró que para que una red sea navegable (que puedas encontrar el destino rápidamente), la probabilidad $P(u,v)$ de que exista un enlace largo entre el nodo $u$ y el nodo $$ debe seguir una distribución específica basada en la distancia $r$ entre ellos:
```math
P(u,v)∝ \frac{1}{r^{\alpha}}
```
Si $α$ es igual a la dimensión de la red (ej. $d$), la búsqueda voraz puede encontrar el destino en tiempo polilogarítmico, 
```math
O\left( log^{2} N\right)
```
Esto es lo que permite que bases de datos con billones de vectores encuentren una respuesta en milisegundos.

---
4. **Los Algoritmos Más Usados y sus Estructuras**

En la práctica, usamos **Búsqueda Aproximada (ANNS)**, sacrificando un poco de precisión por velocidad.

**A. HNSW (Hierarchical Navigable Small World)**

Es el "estándar de oro" actual. Utiliza la teoría de **Small World** pero con una estructura jerárquica (multicapa).
* **Estructura:** Imagina un rascacielos.
* **Capa Superior (Capa $L$)**: Pocos nodos, conectados por enlaces muy largos. Actúa como una "autopista" para saltar grandes distancias en el espacio vectorial.
* **Capas Intermedias:** Más nodos, enlaces de distancia media.
* **Capa Base (Capa 0):** Todos los vectores están aquí, conectados a sus vecinos más cercanos locales,.
* **Fórmula de Construcción (Nivel del Nodo):** Al insertar un nuevo vector $q$, se decide aleatoriamente su altura máxima (nivel l) en la jerarquía usando una distribución exponencial para asegurar la propiedad de "mundo pequeño": 
```math
l=⌊−ln(unif(0,1))⋅m_L⌋
```

* **unif(0,1)**: Un número aleatorio uniforme entre 0 y 1.
* **$m_L$**: Un factor de normalización $(1/ln(M))$ donde $M$ es el número promedio de vecinos.
* **Significado**: Pocos nodos llegan a las capas altas (autopistas), todos están en la base.

* **Algoritmo de Búsqueda**: Comienza en la capa superior. Realiza una búsqueda voraz **(Greedy Search)**: muévete al vecino que esté más cerca de la consulta $q$. Cuando llegas a un mínimo local en la capa actual, "bajas" a la siguiente capa y continúas la búsqueda desde ese punto. Esto permite un efecto de "zoom-in" logarítmico hacia el objetivo.

**B. Algoritmos basados en Cuantización (Product Quantization - PQ)**

Usados para comprimir vectores masivos (billones de datos) para que quepan en memoria.
* Divide el vector original de dimensión $d$ en $m$ sub-vectores. Cada **sub-vector** se reemplaza por el **ID** de un centroide precalculado (quantizer).
  
**Fórmula de Aproximación de Distancia**
En lugar de calcular la distancia exacta, calculamos la distancia al código cuantizado $q(y)$: 
```math
d(x, y)≈d(x,q(y))=\sqrt{\sum_{j=1}^{m}d(x_j,c_{kj})^2}
```

* $x_j$: El sub-vector de la consulta.
* $c_kj$: El centroide que representa al sub-vector del dato almacenado.
* Estas distancias parciales se pueden pre-calcular en tablas (Lookup Tables), haciendo la búsqueda extremadamente rápida

**C. Algoritmos Híbridos (Grafos + Filtros)**

Un desarrollo reciente para resolver la "fractura" entre búsqueda vectorial y filtros de metadatos (SQL).
Los grafos **HNSW** no saben manejar filtros como `"precio < 50".`
Solución (ej. **ACORN, FUSEDANN** Herramientas investigadas para solucionar este problema):

**ACORN**: Modifica la navegación del grafo para que pueda atravesar un "subgrafo de predicado". No sigue solo la geometría del vector, sino que expande la búsqueda de vecinos basándose en si cumplen el filtro.

**FUSEDANN**: Fusiona matemáticamente el vector de contenido v con el atributo $f$ mediante una transformación $Ψ$: 
```math
Ψ(v,f,α,β)= \frac{v−αf}{β}
```
* $α$: Parámetro que empuja los vectores con diferentes atributos lejos unos de otros.
* $β$: Factor de escala.
* **Significado:** Esto crea un nuevo espacio donde los vectores que no cumplen el filtro están matemáticamente "lejos", permitiendo que el algoritmo de grafo estándar los ignore naturalmente.

## Importante
Una **base de datos vectorial** no es más que un sistema experto en calcular $δ(x,q)$ muy rápido. 
Para lograrlo, transforma datos en nodos **(vectores)**, organiza estos nodos en **grafos** que cumplen propiedades de mundo pequeño **(pocos saltos para llegar a cualquier lado)** o los comprime **(cuantización)** para sacrificar precisión por velocidad y memoria. 
La magia moderna radica en cómo estos sistemas ahora intentan entender no solo la distancia geométrica, sino también las restricciones lógicas (filtros) del mundo real.





