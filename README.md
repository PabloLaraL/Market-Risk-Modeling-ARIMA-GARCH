# Modelado de Volatilidad y Value-at-Risk (VaR)
## Framework ARIMA–GARCH con Backtesting Estadístico (Kupiec & Christoffersen)

### Resumen
Este repositorio implementa un **pipeline completo de riesgo de mercado** basado en volatilidad condicional, integrando:

- Transformación de precios a **retornos logarítmicos**
- Diagnóstico de estacionariedad (ADF) y estructura temporal (ACF/PACF)
- Filtrado de la media mediante **ARIMA** (innovaciones / residuos)
- Detección de heterocedasticidad condicional (ARCH-LM)
- Estimación de volatilidad con **GARCH / EGARCH / GJR-GARCH** y distribuciones **Normal / t / skew-t**
- Forecast de volatilidad (horizonte \(H\))
- Cálculo de VaR: **Paramétrico condicional**, **Histórico**, **Monte Carlo** (uni y multi)
- Validación formal mediante **Backtesting**:
  - **Kupiec** (cobertura incondicional)
  - **Christoffersen** (independencia)
  - Cobertura condicional (test conjunto)

Caso de estudio: **Banco de Chile (CHILE.SN)** con datos diarios.

---

## 1. Motivación (hechos estilizados)
Las series financieras típicamente presentan:

- Precios **no estacionarios**
- Retornos generalmente **estacionarios en media**
- **Volatility clustering** y **heterocedasticidad condicional**
- **Colas pesadas** (subestimación de extremos bajo Normal)
- Persistencia de volatilidad
- Posible **asimetría** ante shocks negativos (leverage)

Este proyecto modela explícitamente estas propiedades y valida el VaR con tests estándar.

---

## 2. Metodología

### 2.1 Datos y retornos
Se parte de precios diarios \(P_t\) y se construyen retornos logarítmicos:

\[
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
\]

> **Buenas prácticas:** por defecto se usa **Adj Close** (ajustado por dividendos y splits) para evitar saltos artificiales en retornos.

### 2.2 Filtrado de media (ARIMA)
Se modela la media condicional para remover dependencia lineal:

\[
r_t = \mu_t + \varepsilon_t
\]

donde \(\varepsilon_t\) son innovaciones.  
Diagnóstico: ACF y Ljung–Box sobre residuos.

### 2.3 Volatilidad condicional (GARCH-family)
Se evalúa evidencia ARCH con ARCH-LM y se estiman modelos de volatilidad, por ejemplo:

**GARCH(1,1)**  
\[
\sigma_t^2 = \omega + \alpha\,\varepsilon_{t-1}^2 + \beta\,\sigma_{t-1}^2
\]

**EGARCH(1,1)** (log-varianza)  
\[
\log(\sigma_t^2) = \omega + \beta\log(\sigma_{t-1}^2) + \alpha|z_{t-1}| + \gamma z_{t-1}
\]

**GJR-GARCH(1,1)** (asimetría)  
\[
\sigma_t^2 = \omega + \alpha\varepsilon_{t-1}^2 + \gamma\,\varepsilon_{t-1}^2\,\mathbb{1}_{\{\varepsilon_{t-1}<0\}} + \beta\sigma_{t-1}^2
\]

Se comparan distribuciones para \(z_t\): Normal, t y skew-t.

### 2.4 Forecast de volatilidad
Se calcula \(\hat{\sigma}_{t+h}\) para un horizonte \(H\), observando convergencia a la volatilidad de largo plazo y diferencias entre especificaciones.

---

## 3. Value-at-Risk (VaR)

### 3.1 VaR paramétrico condicional (ARIMA + GARCH)
\[
VaR^{(\alpha)}_{t+1} = \mu_{t+1} + q_{\alpha}\,\sigma_{t+1}
\]
donde \(q_{\alpha}\) es el cuantil de la distribución asumida (p.ej. t con \(\nu\)).

### 3.2 VaR histórico (rolling)
Cuantil empírico de retornos en una ventana (p.ej. 250 días).

### 3.3 VaR Monte Carlo
Simulación de retornos:
\[
r^{(sim)}_{t+1} = \mu_{t+1} + \sigma_{t+1} z^{(sim)}
\]
y cálculo del cuantil empírico de la distribución simulada.
Incluye extensión multivariada con Cholesky (shocks correlacionados).

---

## 4. Backtesting
Se define violación cuando \(r_t < VaR_t\).

- **Kupiec (Unconditional Coverage):** frecuencia de violaciones ≈ \(\alpha\)
- **Christoffersen (Independence):** ausencia de clustering de violaciones
- **Conditional Coverage:** test conjunto

---

## 5. Cómo ejecutar

### Opción A — Jupyter Notebook (recomendado)
1. Crear entorno e instalar dependencias:
```bash
pip install -r requirements.txt
```
2. Abrir el notebook:
```bash
jupyter notebook notebook.ipynb
```

### Opción B — Script reproducible (src)
El archivo `src/run_pipeline.py` permite correr el pipeline en modo batch (sin interfaz).

---

## 6. Estructura del repo
- `notebook.ipynb`: notebook limpio (sin outputs) con el pipeline completo.
- `src/run_pipeline.py`: ejecución reproducible del pipeline (CLI).
- `requirements.txt`: dependencias.

---

## 7. Limitaciones y extensiones
**Limitaciones:** univariado, correlación estática en el bloque multivariado, sin ES.  
**Extensiones:** Expected Shortfall (Basel III), DCC-GARCH, cambios de régimen, stress testing.

---

## Disclaimer
Proyecto con fines académicos y de investigación. No constituye recomendación de inversión.
