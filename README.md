# Modelo de Optimización: Balanceo de Carga de Trabajo TI

## Contexto del Problema
El objetivo de este modelo es optimizar la distribución de horas y tareas entre el equipo de ingeniería de infraestructura. Se busca evitar tanto la sobreutilización (riesgo de burnout y errores) como la subutilización (ineficiencia de recursos), asegurando que la carga de trabajo asignada se acerque lo más posible a un "Target Ideal" de operación.El modelo considera no solo las horas nominales, sino un factor de peso ($\alpha$) que representa la complejidad cognitiva o criticidad del servicio.

Para ello, el planteamiento se divide en 2 problemas de optimizacion (fase 1 y 2):

# Fase 1: Calibración de Complejidad de Servicios (Optimization Model)

## 1. Descripción del Problema

En operaciones de infraestructura, las horas estándar teóricas ($H_j$) raramente coinciden con la realidad operativa debido a factores ocultos.

El objetivo de este modelo es encontrar el **Factor de Complejidad Real ($\alpha_j$)** para cada servicio, utilizando datos históricos. Se busca minimizar la discrepancia entre la carga teórica calculada y la carga real reportada por los ingenieros.

---

## 2. Definicion Matematica

### 2.1 Conjuntos e Índices
* **$J$**: Conjunto de Servicios o Tipos de Tarea ($j = 1, \dots, n$).
* **$K$**: Conjunto de Observaciones Históricas ($k = 1, \dots, m$).
    * *PD: Cada observación $k$ representa un registro consolidado (ej. "Ingeniero $i$ en el Mes $t$").*

### 2.2 Parámetros (Inputs)
Son los datos pre-procesados provenientes del historial operativo.

* **$A_{kj}$ (Carga Teórica Nominal):**
  Total de horas estándar que la observación $k$ debió tomar para el servicio $j$.
  $$A_{kj} = (\text{Conteo Tickets}_{kj}) \times (\text{Tiempo Estándar Esperado} H_j)$$

* **$B_k$ (Carga Real Reportada):**
  Total de horas que el ingeniero registró realmente en la observación $k$ (Timesheet).

### 2.3 Variables de Decisión
* **$\alpha_j$**: Factor de peso/complejidad del servicio $j$ (Variable Continua).
* **$\epsilon_k$**: Variable auxiliar de error para la observación $k$ (Variable Continua, $\epsilon_k \ge 0$).

---

## 3. Planteamiento del Modelo (Linear Programming)

### Función Objetivo
**Minimizar la Suma del Error Absoluto (Mean Absolute Error - MAE).**
Buscamos que la curva de carga teórica se ajuste lo mejor posible a la realidad histórica.

$$
\text{Min } Z = \sum_{k \in K} \epsilon_k
$$

### Restricciones

#### A. Linealización del Error Absoluto
1.  **Cota Superior:**
    $$\sum_{j \in J} (A_{kj} \cdot \alpha_j) - B_k \leq \epsilon_k \quad \forall k \in K$$

2.  **Cota Inferior:**
    $$B_k - \sum_{j \in J} (A_{kj} \cdot \alpha_j) \leq \epsilon_k \quad \forall k \in K$$

#### B. Restricciones de Negocio:

3.  **Límite de Consistencia Operativa:**
    Se evita que el modelo asigne pesos irreales (0 o infinitos). Un servicio no puede costar menos de la mitad ni más del triple de su estándar.
    $$0.5 \leq \alpha_j \leq 5.0 \quad \forall j \in J$$

- $\alpha = 0.5$: Tareas triviales o altamente automatizadas (tardan la mitad de lo estimado).
- $\alpha = 1.0$: Estimación precisa.
- $\alpha = 5.0$: Tareas críticas o mal dimensionadas (tardan hasta 5 veces más de lo estimado).

---

## 4. Interpretación de Resultados
Al resolver este modelo, obtenemos un vector de **Alphas ($\alpha$)**:

* **$\alpha_j \approx 1.0$**: El servicio está bien dimensionado.
* **$\alpha_j > 1.0$**: El servicio es más complejo de lo documentado (posible deuda técnica).
* **$\alpha_j < 1.0$**: El servicio está automatizado o sobreestimado en papel.

*Estos valores se utilizarán como constantes fijas en la **Fase 2 (Asignación de Carga)**.*

# Fase 2: Asignacion de Cargas

---

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

1. *Restricción de Cobertura Total* -> Todos los servicios deben ser asignados a un ingeniero. No pueden quedar tareas sin responsable.

$$\sum_{i \in I} x_{ij} = 1 \quad \forall j \in J$$

2. *Restricción de Capacidad Máxima (Hard Limit)* -> La carga asignada a un ingeniero no puede exceder su capacidad física disponible.

$$\sum_{j \in J} x_{ij} \cdot H_j \cdot \alpha_j \leq C_i \quad \forall i \in I$$

3. *Restricción de Dominio* -> Las variables de decisión son binarias.

$$x_{ij} \in \{0, 1\}$$

## Interpretación de Resultados
Al resolver este modelo, obtendremos una matriz de asignación $X$ que nos permitirá clasificar a los ingenieros en tres estados:

1. *Subutilizado:* Carga Ponderada $< (T - \delta)$
2. *Zona Óptima:* Carga Ponderada $\approx T$
3. *Sobreutilizado:* Carga Ponderada $> (T + \delta)$ (pero siempre $\leq C_i$)

Este enfoque asegura un balanceo equitativo considerando la dificultad real de las tareas y no solo el tiempo nominal.