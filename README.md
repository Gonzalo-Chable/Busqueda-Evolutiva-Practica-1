# AstroShield — Evolución de Constelación Inteligente

## Entregable 1: Definición del Problema y Modelo Computacional

Para evitar la trivialidad geométrica y física en la simulación, el problema se modeló utilizando distancias angulares ortodrómicas en lugar de proyecciones planas. 

**1. Representación del Cromosoma**
Un individuo se representa como un arreglo de variables reales (Float64Array) de 16 posiciones:
`[lat0, lon0, lat1, lon1, ..., lat7, lon7]`
Donde `lat` ∈ [-85°, 85°] y `lon` ∈ [-180°, 180°]. Se restringieron los polos extremos (-90, 90) para evitar singularidades en la mutación gaussiana.

**2. Cálculo de Distancia y Cobertura**
Toda distancia se calcula usando la Fórmula de Haversine. 
*   **Radio de las regiones:** 300 km equivale a $\approx 2.70^\circ$ sobre la superficie terrestre (Radio = 6371 km).
*   **Huella del satélite (Footprint):** Se estableció en $25^\circ$. Un satélite cubre una región si la distancia angular desde el satélite al centro de la región + $2.70^\circ \le 25^\circ$.

**3. Evaluación de Calidad (Función Fitness)**
La función premia la redundancia con rendimientos decrecientes y penaliza estrictamente las violaciones:
*   **Pesos Acumulados:** 1 satélite = 1.0x, 2 satélites = 1.6x, 3+ satélites = 1.7x. (Tener más de 2 satélites en la misma región aporta rendimientos marginales casi nulos).
*   **Bonus por Prioridad:** Las regiones "Alta" reciben un bonus extra de +0.4 si tienen redundancia (≥2 satélites).
*   **Restricciones Duras (Regla de Factibilidad de Deb):** Si un satélite invade una zona de desechos, un objeto ocupado, o viola la separación mínima inter-satelital ($8^\circ$), la solución se marca como `isValid = false` y recibe una penalización letal de $-50 \times \text{violaciones}$, garantizando que una solución inválida jamás supere a una válida.

**4. Estrategia del Algoritmo Evolutivo (AE)**
*   **Selección:** Torneo ($k=3$) para mantener presión selectiva sin convergencia prematura.
*   **Cruza:** BLX-$\alpha$ ($\alpha=0.3$) ideal para espacios continuos, ya que permite explorar valores ligeramente fuera del rango de los padres.
*   **Mutación:** Gaussiana con desviación estándar de $8^\circ$, aplicando corrección modular para la longitud (si cruza los $180^\circ$, aparece en $-180^\circ$).

---

## Entregables 3 y 4: Pruebas, Validación y Análisis de Resultados

El sistema incluye una Suite de 8 Pruebas Automatizadas (7 requeridas + 1 propia) que se ejecutan sobre 10 semillas independientes para extraer métricas estadísticas ($\bar{x}$, $\sigma$).

1.  **Escenario Inicial vs Restricciones (T1):** El AE demuestra capacidad de encontrar soluciones con $F > 0$ tanto en espacio libre como con las zonas A, B y los desechos activados, garantizando 0 violaciones al final de la corrida.
2.  **Evolución por Generaciones (T2):** La gráfica de convergencia demuestra que el elitismo (top 3) asegura una mejora monotónica. El fitness salta de $\approx 5.0$ (Gen 0) a $\ge 24.0$ (Gen Final).
3.  **Zonas Restringidas y Ocupadas (T3 y T4):** El validador espacial confirma que ningún satélite entra en los casquetes curvos de $12^\circ$, $10^\circ$, $8^\circ$ (desechos) ni $5^\circ$ (Objetos A/B).
4.  **Espacio Reducido (T5):** Al inyectar 4 zonas dinámicas extra, el AE logra reacomodar la constelación. El fitness máximo disminuye debido a la compresión del espacio operativo, pero se mantiene factible (`isValid = true`).
5.  **Condición Inviable (T6):** Se bloqueó explícitamente la Península de Yucatán (Alta) con una zona de exclusión de $30^\circ$. El AE intentó cubrirla en las primeras generaciones, fue penalizado masivamente, y optó evolutivamente por abandonar Yucatán ($0$ satélites) para maximizar el fitness con las 5 regiones restantes.
6.  **Variación Estadística (T7):** Tras 10 ejecuciones independientes de 80 generaciones, la desviación estándar se mantuvo por debajo del 25% de la media, demostrando robustez frente a la inicialización aleatoria.
7.  **Prueba Propia - Resiliencia N-1 y Cambio Dinámico (T8):** Se simuló el fallo de 1 satélite (quedando 7 activos) y se elevó la prioridad de "Australia" a "Alta" en tiempo real. El sistema detectó la invalidez parcial, re-evolucionó y logró restaurar el 90% del fitness original distribuyendo los 7 satélites restantes.

---

## Entregable 6: Declaración de uso de Inteligencia Artificial

Se utilizó asistencia de IA (Modelos LLM) como herramienta de *pair programming* y aceleración gráfica. Las verificaciones humanas realizadas sobre el código generado fueron:
1.  **Auditoría Trigonométrica:** Se verificó que la IA no mezclara kilómetros con grados angulares al calcular superposiciones. Se forzó la unificación matemática a grados utilizando $1^\circ \approx 111.195 \text{ km}$.
2.  **Corrección de Texturas 3D:** Se corrigió el mapeo UV de Three.js para asegurar que las coordenadas esféricas del modelo matemático coincidieran exactamente con los continentes de las texturas de la Tierra.
3.  **Depuración de la Función Fitness:** Se corrigió un error algorítmico común donde las IA aplican multiplicadores decrecientes de forma aislada (lo que penalizaba tener más satélites). Se reescribió la lógica para usar pesos estrictamente acumulativos.