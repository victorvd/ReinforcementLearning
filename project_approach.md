# NTN Beam Hopping y Asignación de Recursos mediante Aprendizaje Profundo por Refuerzo (DRL)

## El Problema Central

-----

### **Formulación Matemática**

Nos enfrentamos a un **problema de optimización estocástica de alta dimensión**:

$$\maximize  \mathbb{E}\left[\sum_{t} \gamma^{t} (\alpha\cdot\text{Rendimiento}(t) - \beta\cdot\text{Caídas}(t) + \delta\cdot\text{Equidad}(t) - \eta\cdot\text{Energía}(t))\right]$$
sujeto a:

$$
\sum_{s=1}^{10} \mathbb{1}_{\{\text{haz}_s \text{ activo}\}} \le 4 \quad \quad \quad \quad \quad (\text{Restricción de haz}) \\
\sum_{i=1}^{300} \text{PRB}_i \le 51 \quad \quad \quad \quad \quad \quad \quad (\text{Restricción de recurso}) \\
E_{\text{batería}}(t) \ge 0 \quad \forall t \quad \quad \quad \quad \quad \quad (\text{Restricción de energía}) \\
Q_i(t) \le Q_{\text{max}} \quad \forall i,t \quad \quad \quad \quad \quad \quad (\text{Estabilidad de la cola})
$$

### **Desglose de la Complejidad del Sistema**

| Dimensión | Complejidad | Por qué es Difícil |
|-----------|------------|---------------|
| **300 Usuarios** | Selección combinatoria de usuarios | $2^{300}$ posibles subconjuntos |
| **10 Haces** | Selección de haz | $C(10,4) = 210$ patrones |
| **2 Tipos de Tráfico** | Programación eMBB vs mMTC | Diferentes requisitos de QoS |
| **Canales Variables en el Tiempo** | Dinámica de la SNR | Entorno no estacionario |
| **Restricciones de Energía** | Batería + solar | Compromisos intertemporales |
| **Tráfico en Ráfagas** | 5% de probabilidad de ráfaga | Eventos raros pero críticos |

## Por Qué Fallan los Métodos Tradicionales

-----

### **Enfoques Clásicos de Optimización**

| Método | Limitación Fundamental |
|--------|------------------------|
| **Optimización Convexa** | Requiere un conocimiento futuro perfecto de canales y tráfico |
| **Optimización de Lyapunov** | Miope, no puede anticipar eventos de ráfaga |
| **Reglas Heurísticas** | Las políticas estáticas no se pueden adaptar a condiciones dinámicas |
| **Teoría de Juegos** | Asume agentes racionales independientes |

### **La "Maldición de la Dimensionalidad"**

$$\text{Tamaño del Espacio de Estados} = (\text{Estados de Usuario})^{300} \times (\text{Estados de Haz})^{10} \times (\text{Estados de Energía}) \approx 10^{100}$$
**La programación dinámica tradicional es computacionalmente imposible.**

## Por Qué Triunfa el Aprendizaje Profundo por Refuerzo (DRL)

-----

### **El DRL Coincide con las Características del Problema**

| Característica del Problema | Ventaja del DRL |
|-----------------|---------------|
| **Estado de alta dimensión** | Redes neuronales como aproximadores de funciones |
| **Observabilidad parcial** | Aprende de observaciones comprimidas de 40D |
| **Decisiones secuenciales** | Asignación de crédito temporal mediante aprendizaje TD |
| **Objetivos múltiples** | Aprende automáticamente complejos compromisos de recompensa |
| **Dinámica del sistema desconocida** | Aprendizaje sin modelo a partir de la experiencia |

### **La Formulación del Aprendizaje**

$$\text{MDP} = \langle S, A, P, R, \gamma \rangle$$
Donde:
$$S \in \mathbb{R}^{40} \quad (\text{Estado del sistema comprimido})$$
$$A \in [0,10]^{12} \quad (\text{Pesos continuos del haz y del programador})$$
$$P \quad \quad \quad \quad \quad (\text{Desconocido - aprendido de la experiencia})$$
$$R \quad \quad \quad \quad \quad (\text{Función de recompensa multi-objetivo})$$
$$\gamma = 0.99 \quad \quad (\text{Factor de descuento para la planificación a largo plazo})$$

## Desafíos Técnicos Clave que Resuelve el DRL

-----

### **1. Selección Adaptativa de Haces bajo Incertidumbre**

$$\text{Pesos}_{\text{Haz}}(t) = f_{\theta}(\text{Clústeres}_{\text{SNR}}(t), \text{Colas}_{\text{Sector}}(t), \text{Energía}(t))$$

  - **Aprende** qué sectores atender basándose en las condiciones actuales
  - **Anticipa** las ráfagas de tráfico observando los patrones de cola
  - **Equilibra** el rendimiento inmediato frente a las oportunidades futuras

### **2. Asignación de Recursos de Servicios Múltiples**

$$\text{Pesos}_{\text{Programador}}(t) = [w_{\text{mmtc}}(t), w_{\text{embb}}(t)]$$

  - **eMBB**: Alta tasa, tolerante a la demora $\to$ pondera la calidad del canal
  - **mMTC**: Baja tasa, sensible a la demora $\to$ pondera las colas pendientes
  - **El DRL aprende** la priorización óptima específica del servicio

### **3. Gestión del Compromiso Energía-Demora**

$$\text{Costo}_{\text{Energía}} = \text{Activación}_{\text{Haz}} + \text{Potencia}_{\text{Transmisión}}$$

  - **Aprende** cuándo conservar energía frente a cuándo gastarla
  - **Se adapta** a los patrones de recolección de energía solar
  - **Equilibra** el servicio inmediato frente a la sostenibilidad a largo plazo

## Por Qué Importa Este Problema

-----

### **Impacto en el Mundo Real**

  - **Despliegues NTN 5G/6G**: Starlink, OneWeb, Amazon Kuiper
  - **Comunicaciones de Emergencia**: Respuesta a desastres cuando fallan las redes terrestres
  - **Conectividad Rural**: Reducir la brecha digital a través de satélite
  - **Conectividad Masiva IoT**: Millones de sensores en áreas remotas

### **Importancia Económica**

  - **Costos operativos de satélite**: $\sim\$1\text{M/año}$ por satélite
  - **Eficiencia del espectro**: Asignación limitada de espectro licenciado
  - **Restricciones de energía**: Necesidad de operación con energía solar

## Brecha de Investigación Abordada

-----

### **Limitaciones Actuales en la Literatura**

1.  **Modelos simplificados**: Asumen tráfico de búfer completo o canales estáticos
2.  **Enfoque de objetivo único**: Optimizan rendimiento O equidad O energía
3.  **Optimización centralizada**: Requieren conocimiento global instantáneo
4.  **Sin adaptación**: Políticas estáticas para entornos dinámicos

### **Nuestra Contribución de DRL**

1.  **Modelado realista**: Simulación basada en trazas con tráfico en ráfagas
2.  **Aprendizaje multi-objetivo**: Optimización de extremo a extremo de objetivos en competencia
3.  **Inteligencia distribuida**: Observaciones locales $\to$ optimización global
4.  **Adaptación en línea**: Aprendizaje continuo a partir de la retroalimentación del entorno

## Ventajas Esperadas de la Solución DRL

-----

### **Mejoras Cuantitativas**

$$\text{Esperado: DRL vs Mejor Referencia}$$
$$\text{Rendimiento: } \quad +15-20\%$$
$$\text{Caídas de Paquetes: } \quad -70-85\%$$
$$\text{Eficiencia Energética: } +10-15\%$$
$$\text{Equidad: } \quad \quad +20-30\%$$

### **Ventajas Cualitativas**

  - **Robustez**: Maneja patrones de tráfico inesperados
  - **Escalabilidad**: Política única para densidades de usuario variables
  - **Generalidad**: Se transfiere a diferentes constelaciones de satélites
  - **Autonomía**: No requiere ajuste manual de parámetros

## Resumen: ¿Por Qué DRL para la Asignación de Recursos NTN?

| Aspecto | Métodos Tradicionales | Solución DRL |
|--------|---------------------|--------------|
| **Complejidad del Estado** | Maldición de la dimensionalidad | Aproximación por red neuronal |
| **Compromisos Objetivos** | Ajuste manual de pesos | Aprendizaje automático de equilibrio |
| **Incertidumbre Ambiental** | Asume estacionariedad | Aprende de datos no estacionarios |
| **Horizonte de Decisión** | Miope o requiere pronóstico | Aprendizaje del valor a largo plazo |
| **Capacidad de Adaptación** | Políticas estáticas | Mejora continua de la política |

**El DRL transforma un problema de optimización irresoluble en una política de control que se puede aprender y que descubre estrategias complejas que los diseñadores humanos pasarían por alto.**

-----

# Perspectiva de la Informática: Formulación del Problema y Algoritmos

## Clasificación del Problema

-----

### **Tipo de Problema Central**

```python
Problema: Proceso de Decisión de Markov Parcialmente Observable de Alta Dimensión (POMDP)

Formalmente: ⟨S, A, O, P, R, Ω, γ⟩ donde:
- S: Espacio de estado del sistema (oculto, alta dimensión)
- A: Espacio de acción continuo ℝ¹²
- O: Espacio de observación ℝ⁴⁰ (comprimido, parcial)
- P: Dinámica de transición de estado desconocida
- R: Función de recompensa multi-objetivo
- Ω: Función de observación (mapeo parcial S→O)
- γ: Factor de descuento 0.99
```

### **Clase de Complejidad Computacional**

```python
# Este problema exhibe:
NP-Hard:       Selección combinatoria de haz (selección de subconjunto)
PSPACE:        Toma de decisiones secuencial bajo incertidumbre
#P-Hard:       Razonamiento probabilístico sobre estados ocultos
```

## Formulación Abstracta del Problema

-----

### **Como un Problema de Optimización en Línea**

```python
def optimizacion_en_linea(observacion_parcial, historial):
    # Dado: Instantánea comprimida del sistema y contexto histórico
    # Encontrar: Vector de control continuo que maximiza la recompensa esperada a largo plazo
    # Bajo: Observabilidad parcial, restricciones de recursos, incertidumbre
    
    return vector_accion # Control continuo de 12 dimensiones
```

### **Desafíos Computacionales Clave**

#### **1. Maldición de la Dimensionalidad**

```python
# Los enfoques ingenuos fallan debido a:
tamano_espacio_estado = O(10^40) # ¡Después de la compresión!
tamano_espacio_accion = O(10^12) # Espacio continuo de 12D
```

#### **2. Observabilidad Parcial**

```python
# El problema de "inferencia de estado oculto":
observacion_actual = comprimir(estado_completo_del_sistema) # Compresión con pérdida
# Debe aprender: estado_creencia = P(estado_completo | historial_observacion)
```

#### **3. Compromisos Multi-Objetivo**

```python
# Los objetivos en competencia crean una compleja frontera de Pareto:
objetivos = ['rendimiento', 'equidad', 'energia', 'confiabilidad']
# No hay un único punto óptimo - debe aprender las preferencias de compromiso
```

## Análisis del Espacio de Algoritmos

-----

### **Enfoques Tradicionales de la Informática y Por Qué Fallan**

| Clase de Algoritmo | Por Qué Falla |
|----------------|--------------|
| **Programación Dinámica** | Espacio de estados demasiado grande ($10^{40}$ estados) |
| **Programación Lineal** | Dinámica no lineal, restricciones enteras |
| **Algoritmos Genéticos** | Demasiado lentos para tiempo real, baja eficiencia de muestreo |
| **Búsqueda en Árbol Monte Carlo** | Espacio de acción continuo, observabilidad parcial |
| **Q-Learning (tabular)** | Explosión del espacio estado-acción |
| **Iteración de Políticas** | Irresoluble para espacios de alta dimensión |

### **Por Qué Triunfa el Aprendizaje Profundo por Refuerzo (DRL)**

#### **Avance en la Aproximación de Funciones**

```python
# El DRL utiliza redes neuronales para aproximar:
Red_Q(s, a) ≈ Recompensa acumulada esperada # Función de valor
Red_Política(s) → a # Política directa
```

#### **Eficiencia de Muestreo a través de la Reproducción de Experiencia**

```python
# En lugar de:
for episodio in range(millones): # RL sin modelo
    experiencia = ejecutar_episodio()
    actualizar_politica(experiencia)
    
# El DRL utiliza:
buffer_reproduccion.almacenar(experiencia) # Aprende de diversos datos históricos
lote = buffer_reproduccion.muestra(256) # Rompe las correlaciones temporales
actualizar_redes(lote) # Aprendizaje estable mediante el uso de lotes
```

## Opciones de Diseño del Algoritmo DRL

-----

### **Por Qué TD3 en Lugar de Alternativas**

| Algoritmo | Ventajas | Desventajas para Este Problema |
|-----------|------|---------------------|
| **DQN** | RL discreto estable | No puede manejar acciones continuas |
| **DDPG** | Acciones continuas | Sobreestima los valores Q, inestable |
| **PPO** | Estable, buena convergencia | Menos eficiente en el muestreo |
| **SAC** | Estado del arte | Ajuste de hiperparámetros complejo |
| **TD3** | **Robusto, aborda la sobreestimación** | **Mejor equilibrio para nuestras restricciones** |

### **Innovaciones Técnicas de TD3 Aplicadas**

```python
# 1. Críticos Gemelos - Reduce el sesgo de sobreestimación
Q1, Q2 = redes_criticas(estado, accion)
Q_objetivo = min(Q1, Q2) # Estimación conservadora del valor Q

# 2. Actualizaciones de Política Retrasadas - Estabiliza el entrenamiento
if paso % 2 == 0: # Actualiza los críticos el doble de veces
    actualizar_criticos()
else:
    actualizar_actor()

# 3. Suavizado de la Política Objetivo - Previene el sobreajuste
accion_objetivo = actor_objetivo(siguiente_estado) + ruido_recortado
```

## Racional del Diseño de la Arquitectura Neuronal

-----

### **Red del Actor (Política)**

```python
Entrada(40) → FC(256) → ReLU → LayerNorm → FC(256) → ReLU → FC(12) → Tanh → Escala(10)

# Opciones de diseño:
# - LayerNorm: Maneja distribuciones de entrada no estacionarias
# - Tanh + Escala: Limita las acciones al rango continuo [0,10]
# - 256 unidades: Capacidad suficiente sin sobreajuste
```

### **Redes Críticas (Estimación de Valor)**

```python
Estado(40) → FC(256) → ReLU → LayerNorm → FC(256) → ReLU → 
            Concatenar(Acción(12)) → FC(256) → ReLU → FC(1)

# Por qué funciona esto:
# - Vías separadas: Comprensión del estado + evaluación de la acción
# - Fusión tardía: Acción evaluada en el contexto de la comprensión del estado
```

## Perspectiva de la Teoría del Aprendizaje Computacional

-----

### **Análisis de la Complejidad del Muestreo**

```python
# Límites teóricos frente a la realidad práctica:
muestras_teoricas = O(|S|×|A|) # Millones para nuestro problema
muestras_practicas = 5000 episodios × 200 pasos = 1M muestras

# Por qué funciona en la práctica:
1. Aproximación de funciones: Generalización a través de estados similares
2. Reproducción de experiencia: Eficiencia de datos a través de la reutilización
3. Aprendizaje por transferencia: Las políticas se generalizan a través de escenarios
```

### **Garantías de Convergencia**

```python
# Bajo condiciones ideales:
- TD3 converge a un óptimo local con probabilidad 1
- La mejora de la política es monotónica bajo ciertas suposiciones

# En la práctica:
- Se requiere un ajuste cuidadoso de los hiperparámetros
- La conformación de la recompensa es crítica para la convergencia
- El equilibrio exploración-explotación es esencial
```

## Familias de Algoritmos Alternativos

-----

### **Enfoque Jerárquico de RL**

```python
def solucion_jerarquica():
    controlador_alto_nivel() # Selección de haz (cada 10 pasos)
    programador_nivel_medio() # Priorización de usuarios (a cada paso)
    asignador_nivel_bajo() # Distribución de recursos (a cada paso)
```

### **Extensión de Meta-Aprendizaje**

```python
# Aprender a adaptarse rápidamente a nuevos escenarios:
politica_base = entrenar_en_diversos_entornos()
def adaptar_politica(nuevo_entorno, pocas_muestras):
    return ajuste_fino(politica_base, nuevo_entorno, pocas_muestras)
```

### **Formulación de RL Multi-Agente**

```python
# Solución distribuida:
agentes_haz = [Agente_RL() for _ in range(10)] # Cada haz autónomo
coordinador = Agente_RL() # Coordinador global de recursos
```

## Por Qué Este Es un Problema de Referencia para RL

-----

### **Propiedades Deseables de Referencia de RL**

```python
propiedades = {
    'no_trivial': True, # No puede ser resuelto por reglas simples
    'dificultad_ajustable': True, # Se puede ajustar la carga, las restricciones
    'metricas_claras': True, # Rendimiento, caídas, energía, equidad
    'observabilidad_parcial': True, # Estado oculto realista
    'acciones_continuas': True, # Control de alta dimensión
    'multi_objetivo': True, # Objetivos en competencia
    'relevancia_mundo_real': True, # Aplicaciones prácticas
    'reproducible': True # Entorno controlado
}
```

## Ideas Clave de la Informática

-----

### **La Contribución Central de la Informática**

Este problema demuestra cómo **la aproximación de funciones profunda + la reproducción de experiencia + el diseño cuidadoso de algoritmos** pueden resolver problemas que son:

  - **Demostrablemente irresolubles** para algoritmos tradicionales
  - **De alta dimensión** tanto en el espacio de estados como de acciones
  - **Parcialmente observables** con dinámicas complejas
  - **Multi-objetivo** con metas en competencia

### **Patrón de Innovación Algorítmica**

```python
# Receta exitosa para problemas de control difíciles:
1. Formalizar como MDP/POMDP
2. Elegir el algoritmo DRL apropiado (TD3 para control continuo)
3. Diseñar una arquitectura neuronal que coincida con la estructura del problema
4. Implementar la reproducción de experiencia para la eficiencia de muestreo
5. Conformar cuidadosamente la función de recompensa para el comportamiento deseado
6. Ajustar los hiperparámetros mediante experimentación sistemática
```

Esto representa la **vanguardia de lo que es computacionalmente factible** para problemas complejos de control del mundo real utilizando las metodologías actuales de aprendizaje profundo por refuerzo.
