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

## 3. Planteamiento del Modelo de Optimizacion

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

---

# Fase 2: Asignacion de Cargas

## 1. Objetivo
El modelo asigna servicios a ingenieros basándose en tres pilares:
1.  **Capacidad:** Respetar las horas de contrato.
2.  **Competencia:** Asignar tareas solo a quien tenga experiencia histórica demostrada.
3.  **Equidad:** Balancear la carga de trabajo entre el equipo.

---

## 2. Definición del Modelo Matemático

### 2.1 Conjuntos
* $I$: Conjunto de Ingenieros ($i = 1, \dots, m$).
* $J$: Conjunto de Servicios activos ($j = 1, \dots, n$).

### 2.1 Parámetros (Inputs)
* **$W_j$**: Carga Efectiva del servicio ($H_j \cdot \alpha_j^*$).
    * Calculada como: $W_j = H_j \times \alpha_j^*$ (Usando el $\alpha$ calibrado en Fase 1).
* **$C_i$**: Capacidad máxima del ingeniero $i$ (Horas Contrato).
* **$S_{ij}$**: **Matriz de Habilidades**.
    * $1$ si el ingeniero $i$ ha realizado el servicio $j$ anteriormente.
    * $0$ si no tiene experiencia registrada.
* **$T$**: Target Dinámico ($\sum W_j / |I|$).

### 2.2 Variables de Decisión
* **$x_{ij} \in \{0, 1\}$**: Asignación (1 = Asignado).
* **$d_i \ge 0$**: Desviación de carga respecto al promedio.
* **$HE_i \ge 0$**: Horas Extra virtuales necesarias para completar el trabajo.

### 2.3 Modelo (MILP)

**Función Objetivo:**
Prioridad 1: Minimizar Horas Extra (Costo $M$).
Prioridad 2: Minimizar Desbalance (Equidad).

$$
\text{Min } Z = \sum_{i \in I} d_i + (1000 \cdot \sum_{i \in I} HE_i)
$$

**Restricciones:**

1. **Restricción de Competencia (Skills):**

Un ingeniero solo puede tomar un servicio si tiene la habilidad para hacerlo.

$$x_{ij} \leq S_{ij} \quad \forall i \in I, \forall j \in J$$

Nota: Si $S_{ij}=0$, entonces $x_{ij}$ debe ser 0.

2. **Cobertura Total:**

Cada servicio debe tener un responsable.

$$\sum_{i \in I} x_{ij} = 1 \quad \forall j \in J$$

3. **Capacidad con Holgura (Overtime):**

La carga asignada no puede superar la capacidad, salvo emergencia (Horas Extra).

$$\sum_{j \in J} (x_{ij} \cdot W_j) \leq C_i + HE_i \quad \forall i \in I$$

4. **Balanceo (Cálculo de Desviación):**

* Caso A (Carga superior al promedio):

$$\sum_{j \in J} (x_{ij} \cdot W_j) - T \leq d_i$$

* Caso B (Carga inferior al promedio):

$$T - \sum_{j \in J} (x_{ij} \cdot W_j) \leq d_i$$

### Dominio de Variables

$$x_{ij} \in \{0, 1\}, \quad d_i \geq 0, \quad HE_i \geq 0$$

---

## Interpretación de Resultados
El modelo matemático entregará la mejor asignación posible dadas las restricciones. Para la toma de decisiones gerenciales, se clasificará a cada ingeniero según su **Carga Final ($L_i$)** y las **Horas Extra ($HE_i$)**:

| Estado | Indicador Matemático | Acción Recomendada |
| :--- | :--- | :--- |
| **🔴 CRÍTICO (Sobreocupado)** | $HE_i > 0$ **Ó** $L_i = C_i$ | **Riesgo de Burnout.** Revisar si es posible reasignar tareas o si se requiere contratar personal de apoyo. |
| **🟢 EQUILIBRADO (Óptimo)** | $L_i \approx T$ ($\pm 10\%$) | **Gestión Correcta.** El ingeniero tiene una carga justa comparada con el resto del equipo. |
| **🟡 SUBOCUPADO (Disponible)** | $L_i < T - 10\%$ | **Ineficiencia.** El ingeniero tiene horas libres pero no puede tomar tareas pendientes (posible falta de Skills). **Acción:** Plan de Capacitación (Upskilling). |

### Ejemplo de Salida del Sistema
> **Ingeniero A (Senior):** 45 horas asignadas ($C_i=40$). $HE_i=5$. -> **🔴 CRÍTICO**
> **Ingeniero B (Mid):** 38 horas asignadas ($C_i=40$). $HE_i=0$. -> **🟢 EQUILIBRADO**
> **Ingeniero C (Junior):** 20 horas asignadas ($C_i=40$). $HE_i=0$. -> **🟡 SUBOCUPADO**

*Conclusión del Ejemplo:* El Ingeniero A está saturado porque es el único que sabe hacer ciertas tareas críticas, mientras el Ingeniero C está libre porque le faltan habilidades. La solución no es matemática, es de capacitación.*

---
