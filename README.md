Rama fix-entrenamiento

Corrige el entrenamiento de la mascota original y mejora el rendimiento y la reproducibilidad del juego de salto de Neurito.

Cambios
main.py
La tasa de aprendizaje pasa de 0.001 a 0.5 y las épocas de 200 a 500. Con los valores anteriores la red casi no aprendía.
El registro de pérdida se imprime cada 50 épocas. Antes epoch % 200 solo imprimía la época 0.
Se corrige el comentario de LANGUAGE.

Resultado medido: la pérdida baja de 1.38 a 0.08 y la precisión de prueba frente a la política sintética es de 99.20 %.

saltar.py
La precisión de prueba se calcula solo al arrancar, al cambiar el número de épocas y al reiniciar el cerebro (R). Antes se recalculaba 30 veces por segundo sobre 1500 ejemplos.
salto_modelo.py
reiniciar() recrea el generador aleatorio con la semilla guardada (self.semilla). Así, al pulsar R los pesos vuelven exactamente a los iniciales.
Archivos sin cambios

salto_juego.py, GUIA_SALTO.md, pyproject.toml y uv.lock.
