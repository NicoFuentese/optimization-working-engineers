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

