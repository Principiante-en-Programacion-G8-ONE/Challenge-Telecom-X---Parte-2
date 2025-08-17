# Challenge Telecom X

## 🧭 Resumen
Proyecto de ciencia de datos para **predecir la cancelación** en clientes de telecomunicaciones y **priorizar acciones de retención**.  
Se construyó un pipeline reproducible que incluye: preparación de datos, balanceo de clases, normalización selectiva, entrenamiento de modelos y evaluación con métricas enfocadas en la clase positiva (*Abandonaron = 1*).

**Modelos finalistas:**
- **Regresión Logística (normalizada) + RUS (undersampling)** → *Recall* alto (detección de abandono) con buen F1.  
- **Random Forest (sin normalizar) + RUS** → alternativa robusta con *recall* alto y F1 competitivo.

---

## 🎯 Objetivo
Anticipar qué clientes presentan **mayor probabilidad de cancelar** para activar **estrategias de retención** (precio, contrato, soporte proactivo).

---

## 🗂️ Datos y variables
- **Etiqueta:** `Abandono` (0 = permanecieron, 1 = abandonaron).
- **Continuas** *(se escalan para modelos sensibles)*:  
  `Meses_Contrato`, `CargoMensual`, `CargoTotal`, `GastoDiarioMensual`, `GastoDiarioAcumulado`
- **Dummies / 0-1** *(no se escalan)*: género, servicio de internet, contrato, método de pago, y servicios asociados (soporte, seguridad, streaming, etc.).

---

## 🔧 Pipeline (funcionalidad)
1. **Split estratificado** (80/20).  
2. **Balanceo** (solo en *train*):  
   - `RandomUnderSampler (RUS)`  
   - `RandomOverSampler (ROS)`  
   - `SMOTE`  
3. **Escalado selectivo** (solo continuas):  
   - `StandardScaler` → Regresión Logística  
   - *(Árboles / Random Forest no requieren escalado)*
4. **Modelado**  
   - **Regresión Logística** (`saga` + `elasticnet`)  
   - **Random Forest** (muchos árboles + control suave de complejidad)  
5. **Evaluación**  
   - Métricas: **Accuracy, Precision, Recall, F1**, matriz de confusión  
   - Énfasis de negocio: **Recall y F1** (minimizar **FN** = abandonos que no detectamos)

---

## 📈 Resultados (test)

| Modelo                 | Balanceo | Accuracy | Precision | **Recall** | **F1** |
|------------------------|----------|---------:|----------:|-----------:|-------:|
| Regresión Logística    | **RUS**  | 0.747 | 0.505 | **0.805** | **0.621** |
| Random Forest          | **RUS**  | 0.742 | 0.499 | **0.789** | **0.611** |
| Regresión Logística    | Base     | 0.743 | 0.501 | 0.802 | 0.617 |
| Random Forest          | Base     | **0.785** | **0.578** | 0.612 | 0.595 |
| Regresión Logística    | ROS      | 0.741 | 0.498 | 0.797 | 0.613 |
| Random Forest          | ROS      | 0.767 | 0.542 | 0.618 | 0.578 |
| Regresión Logística    | SMOTE    | 0.769 | 0.543 | 0.652 | 0.593 |
| Random Forest          | SMOTE    | 0.778 | 0.568 | 0.567 | 0.568 |

**Conclusión:** Para **retención**, priorizamos *Recall/F1*. La **Regresión Logística + RUS** y **Random Forest + RUS** fueron las mejores combinaciones (ambas con *recall* alto y F1 competitivo).

---

## 🔍 Factores que más influyen en el abandono

**Aumentan el riesgo** (coeficientes positivos / alta importancia en RF):
- **Contrato “Mes a mes”**
- **ServicioInternet “Fiber optic”**
- **Costo**: `CargoMensual`, `CargoTotal`, `GastoDiario*`

**Reducen el riesgo** (coeficientes negativos / baja propensión en RF):
- **Mayor antigüedad** (`Meses_Contrato`)
- **Contrato “Dos años”**
- **ServicioInternet “No”** *(menor exposición a incidencias de internet)*

**Interpretación:** El abandono se concentra en **clientes nuevos**, **mes-a-mes**, **con fibra** y **con cargos altos**.

---

## 🛠️ Decisiones de diseño (aspectos importantes)
- **Normalización selectiva:** solo en **continuas** para modelos basados en distancia/optimización (p. ej., Regresión Logística).  
- **Evitar fuga:** usar **solo `Abandono` (0/1)** como target; descartar variantes textuales del estado.  
- **Balanceo solo en *train***; evaluación en *test* intacto.  
- **Regularización** en LogReg (`elasticnet`) para manejar multicolinealidad y dummies.  
- **Random Forest** con muchos árboles para estabilidad; control suave de complejidad con `min_samples_*`.  
- **Énfasis en Recall/F1** por el costo de perder abandono (FN).

---

## 🧪 Cómo ejecutar
1. **Preprocesar** (one-hot + binarios a 0/1).  
2. **Split** estratificado (80/20).  
3. **Balancear *train*** (recomendado: **RUS**).  
4. **Escalar continuas** para Regresión Logística.  
5. **Entrenar** modelos seleccionados.  
6. **Evaluar** en test con *Accuracy, Precision, Recall, F1* y matriz de confusión.  
7. **Ajustar umbral** si el objetivo es maximizar Recall/F1/PR-AUC.

---

## 💼 Recomendaciones de retención

1. **Precio / Valor**  
   - Descuentos temporales (3–6 meses) a **mes-a-mes** con **cargos altos**.  
   - Bundles y topes de facturación; incentivos a **autopago**.

2. **Migración de contrato**  
   - Upsell a **Un año / Dos años** con mes bonificado / upgrade de velocidad.  
   - Enfocar en clientes **p(abandono) alta + mes-a-mes**.

3. **Calidad de servicio (fibra)**  
   - Monitoreo proactivo; prioridad de soporte a cuentas con **p(abandono) alta**.  
   - Reemplazo preventivo de CPE/router ante incidencias recurrentes.

4. **Onboarding (tenure bajo)**  
   - Programa de 60–90 días con check-ins, micro-encuestas, tutoriales.

5. **Facturación sin fricción**  
   - Alertas de consumo, explicadores de factura, fechas de pago flexibles.

---

## ⚠️ Limitaciones y siguientes pasos
- **Umbral de decisión** impacta fuertemente precisión/recall → optimizar según **costo de FN/FP**.  
- Posible **redundancia** entre variables de costo → evaluar reducción (p. ej., quedarse con `CargoMensual` + `Meses_Contrato`).  
- **Validación cruzada** con remuestreo dentro del *fold* (evitar fuga).  
- **Explainability por cliente** (SHAP) para recomendaciones personalizadas.  
- Explorar **XGBoost/LightGBM** y **calibración de probabilidades** (Platt/Isotónica).

---

## 📁 Estructura sugerida del repo
.
├── data/ # dataset(s)
├── notebooks/ # EDA, modelado, reportes
├── src/
│ ├── preprocessing.py # funciones de limpieza/encoding/escalado
│ ├── sampling.py # RUS/ROS/SMOTE
│ ├── models.py # definiciones de modelos/pipelines
│ └── eval.py # métricas, curvas, reports
├── outputs/ # métricas, gráficos, tablas
└── README.md


---

## ✍️ Autor
**Maio Carrizales**
