# Modelo de Optimización: Balanceo de Carga de Trabajo TI

## Contexto del Problema
El objetivo de este modelo es optimizar la distribución de horas y tareas entre el equipo de ingeniería de infraestructura. Se busca evitar tanto la sobreutilización (riesgo de burnout y errores) como la subutilización (ineficiencia de recursos), asegurando que la carga de trabajo asignada se acerque lo más posible a un "Target Ideal" de operación.El modelo considera no solo las horas nominales, sino un factor de peso ($\alpha$) que representa la complejidad cognitiva o criticidad del servicio.

## Definición del Modelo Matemático

### Conjuntos e Indices
- $I$: Conjunto de Ingenieros disponibles ($i = 1, \dots, m$).
- $J$: Conjunto de Servicios o Tareas a asignar ($j = 1, \dots, n$).

### Parametros (inputs)
- $H_j$: Horas promedio estimadas requeridas para atender el servicio $j$.
- $\alpha_j$: Factor de ponderación (peso/complejidad) del servicio $j$.
    - $\alpha > 1$: Alta complejidad/criticidad.
    - $\alpha = 1$: Complejidad estándar.
- $C_i$: Capacidad máxima de horas disponibles del ingeniero $i$ (ej. 40 horas semanales).
- $T$: Carga objetivo ideal (Target Workload). Representa el punto óptimo de productividad (ej. 80% de la capacidad total).

### Variables de Decision
Definimos una variable binaria que determina la asignación:

$$x_{ij} = \begin{cases} 
1 & \text{si el ingeniero } i \text{ es asignado al servicio } j \\
0 & \text{en caso contrario}
\end{cases}$$

## Planteamiento del Problema
### Función Objetivo
*Minimizar la desviación de la carga respecto al ideal.* Buscamos que la suma de las diferencias (en valor absoluto) entre la carga real asignada y la carga objetivo ($T$) sea mínima para todo el equipo.

$$\text{Min } Z = \sum_{i \in I} \left| \left( \sum_{j \in J} x_{ij} \cdot H_j \cdot \alpha_j \right) - T \right|$$

Donde el término $\sum_{j \in J} x_{ij} \cdot H_j \cdot \alpha_j$ representa la Carga Ponderada Real del ingeniero $i$.

### Restricciones

1. *Restricción de Cobertura Total* Todos los servicios deben ser asignados a un ingeniero. No pueden quedar tareas sin responsable.

$$\sum_{i \in I} x_{ij} = 1 \quad \forall j \in J$$

2. *Restricción de Capacidad Máxima (Hard Limit)* La carga asignada a un ingeniero no puede exceder su capacidad física disponible.

$$\sum_{j \in J} x_{ij} \cdot H_j \cdot \alpha_j \leq C_i \quad \forall i \in I$$

3. *Restricción de Dominio* Las variables de decisión son binarias.

$$x_{ij} \in \{0, 1\}$$

## Interpretación de Resultados
Al resolver este modelo, obtendremos una matriz de asignación $X$ que nos permitirá clasificar a los ingenieros en tres estados:

1. *Subutilizado:* Carga Ponderada $< (T - \delta)$
2. *Zona Óptima:* Carga Ponderada $\approx T$
3. *Sobreutilizado:* Carga Ponderada $> (T + \delta)$ (pero siempre $\leq C_i$)

Este enfoque asegura un balanceo equitativo considerando la dificultad real de las tareas y no solo el tiempo nominal.