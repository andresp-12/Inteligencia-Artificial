# Neurito 3.0: aprende a encontrar comida en un laberinto

Neurito busca comida dentro de tres laberintos. Su cerebro debe elegir entre arriba, abajo, izquierda y derecha. La red se construye con NumPy, y Pygame dibuja el escenario, la mascota y una gráfica de la pérdida.

## Cómo aprende

1. BFS calcula rutas cortas y genera ejemplos etiquetados: mapa, posición, meta y movimiento recomendado.
2. Los ejemplos se dividen en entrenamiento (80 %) y prueba (20 %). Los pares posición-meta de prueba no aparecen en entrenamiento.
3. Cada entrada incluye un vector one-hot de tres mapas y dos vectores one-hot de 81 casillas: 165 características.
4. La arquitectura es 165 → 96 → 4. Usa ReLU en la capa oculta y Softmax en las cuatro salidas.
5. El entrenamiento minimiza entropía cruzada multiclase mediante retropropagación y descenso de gradiente con mini-lotes.
6. Durante el juego, el modelo decide sin consultar BFS. BFS es el experto que proporcionó las etiquetas.

Esto es aprendizaje supervisado por imitación, NO aprendizaje por refuerzo.

## Dos modos

- Red pura: toma la salida de mayor probabilidad. Puede chocar con paredes o quedarse en ciclos.
- Asistido: descarta movimientos contra paredes y penaliza casillas visitadas. Son reglas adicionales, no conocimientos aprendidos por la red.

La precisión de clasificación no equivale al porcentaje de laberintos completados.

## Estructura

- laberinto_mundo.py: mapas, movimiento, BFS y navegación asistida.
- laberinto_red.py: red neuronal y entrenamiento con NumPy.
- laberinto.py: juego Pygame, sprites y pérdida.
- test_laberinto.py: pruebas automáticas.
- GUIA_LABERINTO.md: esta guía.

Reutiliza assets/neutral_1.png, neutral_2.png y neutral_3.png. No modifica las versiones anteriores.

## Ejecutar

    uv sync
    uv run laberinto.py

## Controles

| Tecla | Acción |
|---|---|
| 1 | Entrenar una época |
| 2 | Entrenar cien épocas |
| 3 | Entrenamiento continuo |
| A | Alternar red pura y navegación asistida |
| P | Alternar navegación automática y paso a paso |
| Espacio | Avanzar un paso en modo manual |
| N | Cambiar mapa |
| G | Nuevas posiciones y meta |
| C | Reiniciar contadores |
| R | Reiniciar pesos y entrenamiento |
| Esc | Salir |

## Probar automáticamente

    uv run python -m unittest -v test_laberinto.py

## Experimentos

1. . Observa Neurito sin entrenar. ¿Por qLos pesos son aleatorios y las cuatro salidas Softmax quedan casi iguales (por ejemplo 27%, 29%, 24% y 20%). Como entradas parecidas dan salidas parecidas, la red elige casi siempre la misma dirección sin importar dónde esté la comida. Esa dirección suele chocar con una pared (82% de los retos), y como el movimiento fallido no cambia la posición, repite el error hasta agotar los intentos. Solo llega a la comida en el 3.1% de los retos y su exactitud (27.7%) es casi la del azar entre cuatro opciones.ué se atasca?


2. Entrena 100, 200 y 400 épocas. Compara pérdida, precisión y movimientos.

 | Épocas | Pérdida | Exactitud | Éxito red pura | Choques |
   |---|---|---|---|---|
   | 100 | 0.140 | 86.7% | 62.5% | 1.7% |
   | 200 | 0.040 | 89.9% | 81.1% | 0.7% |
   | 400 | 0.011 | 90.8% | 83.2% | 0.7% |

   La pérdida cae mucho, pero la exactitud de prueba mejora poco después de 200 épocas. Los choques casi desaparecen con 100 épocas; lo que falla después son sobre todo los ciclos.

3. Compara red pura frente a navegación asistida. ¿Qué parte corresponde al modelo y cuál a las reglas?
Con 400 épocas, la red pura completa el 83.2% de los retos y la asistida el 98.8%. Pero sin entrenar, las reglas solas (evitar paredes y penalizar casillas visitadas) ya resuelven el 57.5 %. El modelo aporta saber hacia dónde conviene ir. Las reglas aportan no chocar y romper ciclos. El éxito asistido no es mérito de la red.

4. Cambia las 96 neuronas ocultas por 16, 32 o 128.
Con 16 neuronas la red rinde algo peor (exactitud 88.5%, red pura 78.0%). Desde 32 neuronas se estabiliza (90.5 91% de exactitud y 83% de red pura), y 128 no mejora de forma apreciable. Con 96 hay más capacidad de la necesaria para estos tres mapas.

5. Cambia la tasa de aprendizaje 0.45 y observa si converge.
Con 0.01 casi no converge en 400 épocas (pérdida 0.695, exactitud 72.6%). Con 0.1 converge más despacio (85.9%). Entre 0.45 y 2.0 converge bien (90.8% a 92.6%) y no diverge. Las tasas más altas salen un poco mejor en este rango, pero eso no significa que "más grande" sea siempre mejor.

6. Cambia de mapa. Los mapas forman parte del entrenamiento, pero las combinaciones de inicio y meta usadas para probar no se entrenaron.
El rendimiento es parecido en los tres mapas (exactitud 90–91%, red pura 80–86%), porque la red conoce el identificador de cada mapa. En una prueba extra con un mapa totalmente desconocido (entrenando con los otros dos), la exactitud baja a 60–65% y la red pura a 19–25%. La red no ve las paredes, solo el número de mapa, la posición y la meta, así que no generaliza a laberintos nuevos.

7. Haz vibecoding para verificar como mejora o empeora Neurito al cambiar BFS, por DFS y A*
 | Experto | Exactitud | Éxito red pura | Éxito asistido |
   |---|---|---|---|
   | BFS | 90.8% | 83.2% | 98.8% |
   | A\* | 91.3% | 86.2% | 99.6% |
   | DFS | 99.9% | 7.5%  | 26.4% |

   A\* da etiquetas equivalentes a BFS (ambos encuentran rutas óptimas), y las diferencias son del orden del ruido. DFS empeora mucho a Neurito: la red aprende casi a la perfección sus etiquetas, pero esas etiquetas no forman una ruta coherente de una casilla a la siguiente. Una clasificación excelente no garantiza una buena conducta.


## Alcance y limitaciones

El modelo no interpreta capturas de pantalla: trabaja con características estructuradas. Conoce el identificador de cada mapa. Los tres mapas pertenecen al mismo universo de entrenamiento. La evaluación demuestra generalización a pares de posición-meta no entrenados, no a laberintos totalmente desconocidos. Para eso habría que representar también las paredes de cualquier mapa nuevo y entrenar y evaluar en mapas separados.
