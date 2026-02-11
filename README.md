# Statistics and Machine Learning Exam Project

## 1. Project Overview

This project investigates the UCI Diabetes dataset through both unsupervised and supervised learning. 
Irregular, event-based medical records are transformed into structured representations 
to explore latent behavioral patterns and to predict short-term glucose dynamics. 
The study emphasizes data preprocessing, robustness to noise, and model interpretability 
in a real-world healthcare context.

---

## 2. Problem Statement

This project explores the **UCI Diabetes** dataset, which contains irregular, 
event-based records of daily diabetes management behaviors and blood glucose outcomes across multiple patients. 
The data are aggregated at the daily level to transform heterogeneous event sequences into fixed-length feature vectors 
summarizing glycemic level and variability, treatment intensity, monitoring activity, and risk-related events.

**Unsupervised learning** methods are applied to these daily representations 
to identify latent behavioral–outcome regimes, such as stable days, intervention-heavy days, 
and highly unstable high-risk days, without relying on predefined labels.

**Supervised learning** methods are applied directly to the event-level data to 
predict the next blood glucose measurement based on recent management actions and physiological context. 
By framing each glucose measurement as a prediction target and using preceding insulin, meal, exercise, 
and glucose events within a fixed lookback window as input features, the task evaluates 
how well short-term glucose dynamics can be inferred from logged behavioral sequences.

---

## 3. Environment Setup

This project uses **uv** for Python environment and dependency management. <br />
`uv` provides fast, reproducible installs and a simple workflow suitable for both beginners and experienced users.

### 3.1 Clone the Repository

```bash
git clone https://github.com/SakigamiYang/srh-stat-and-ml-exam.git
cd srh-stat-and-ml-exam
```

### 3.2 Create Virtual Environment and Sync Dependencies

Create a virtual environment:

```bash
uv venv
```

Activate the environment:

```bash
# macOS / Linux
source .venv/bin/activate
```

```powershell
# Windows
.venv\Scripts\activate
```

Sync project dependencies:

```bash
uv sync
```

This installs all required dependencies defined in `pyproject.toml` and ensures a reproducible environment via `uv.lock`.

---

## 4. Dataset Information

| Tag           | Value                                                                                                                                                                                                     |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Dataset       | [UCI - Diabetes](https://doi.org/10.24432/C5T59G)                                                                                                                                                         |
| DOI           | 10.24432/C5T59G                                                                                                                                                                                           |
| \# Instances  | 70 files <br /> <br /> Each file contains a sequence of diabetes treatment events for the same patient over several months (approximately 4–5 months), with treatment actions encoded into 20 categories. |
| \# Features   | 20 <br /> (\# of Treatment Event Categories)                                                                                                                                                              |
| Data Format   | TSV <br /> <br /> Columns: <br /> (1) Date in MM-DD-YYYY format <br /> (2) Time in XX:YY format <br /> (3) Code <br /> (4) Value                                                                          |

---

## 5. Samples Generation

Code: [raw_data_to_samples.ipynb](notebooks/raw_data_to_samples.ipynb)

### 5.1 Unsupervised Learning Samples

- Data Contract: [diabetes-daily-features.md](docs/data-contracts/unsupervised-learning/diabetes-daily-features.md)
- Shape: `(3378, 170)`

### 5.2 Supervised Learning Samples

- Data Contract: [diabetes-event-window-features.md](docs/data-contracts/supervised-learning/diabetes-event-window-features.md)
- Shape: `(10612, 30)`
- Target: `y_next_glucose`

---

## 6. Learning Results

### 6.1 Unsupervised Learning Results

Daily feature vectors were analyzed using **UMAP** for dimensionality reduction followed by **DBSCAN** for clustering. 
Model selection jointly considered the number of clusters, noise fraction, and Silhouette score.

With `eps = 1.0` and `min_samples = 10`, DBSCAN identified **6 clusters** with **near-zero noise** 
and a **Silhouette score of approximately 0.43**, indicating a well-separated structure. 
The clusters differ systematically in glucose level and variability, insulin usage, event frequency, 
and daily activity span, suggesting distinct behavioral–outcome regimes 
such as stable days, intervention-heavy days, and more volatile patterns.

- Notebook: [unsupervised_learning.ipynb](notebooks/unsupervised_learning.ipynb)

### 6.2 Supervised Learning Results

With `n_estimators = 400` and `min_samples_leaf = 2`, a Random Forest regressor was applied to predict 
the next blood glucose value using event-window features. The model achieved moderate performance, 
with an RMSE of approximately 77 on the validation set and 74 on the test set, and R² values around 0.10–0.12. 
IQR-based denoising led to a small but consistent improvement, indicating limited sensitivity to extreme values. 
The prediction–actual plots show that the model captures some non-linear structure 
but still tends to regress toward the mean, suggesting that short-term glucose dynamics remain difficult to model 
with the current feature representation.

- Notebook: [supervised_learning.ipynb](notebooks/supervised_learning.ipynb)

---

## 7. Data Preprocessing and Statistical Analysis

Prior to both unsupervised and supervised learning, the raw UCI Diabetes event logs underwent systematic preprocessing 
and validation to ensure data consistency, interpretability, and experimental control.

### 7.1 Data Cleaning and Validation

The original dataset consists of irregular, event-based medical records collected from multiple patients. 
Several files were found to contain malformed records, including invalid timestamps, missing codes, 
or non-numeric values where measurements were expected. These files were excluded entirely 
to avoid introducing ambiguous noise. For the remaining files, records with invalid dates, times, or values 
were filtered out during parsing.

Event timestamps were normalized to a consistent `HH:MM` format, and all numeric values were explicitly cast to 
ensure type correctness. This step guarantees that downstream feature construction operates on clean 
and well-defined inputs.

### 7.2 Feature Construction and Missing Values

For supervised learning, features were constructed from fixed lookback windows 
preceding each glucose measurement event. Due to the nature of diabetes management data, 
the absence of certain actions (e.g., insulin administration) within a window is clinically meaningful and expected. 
Therefore, missing values in selected insulin-related features were filled with zeros 
rather than treated as invalid samples.

All numerical features were subsequently scaled using min–max normalization to place them on a comparable range, 
while temporal categorical features (`hour_of_day`, `day_of_week`) were encoded using one-hot encoding. 
Identifier and timestamp fields were excluded from modeling.

### 7.3 Statistical Analysis

Basic descriptive statistics, including mean, variance, and skewness, were computed for numerical features 
to examine their distributional properties. Correlation analysis was performed to identify dependencies 
among features and to assess potential redundancy. Visualizations such as histograms, boxplots, 
and correlation heatmaps were used to support exploratory analysis and to verify the effects of preprocessing steps.

### 7.4 Noise Injection and Denoising

To evaluate model robustness, synthetic noise was manually injected into a small fraction (1%) of the training data 
by replacing selected glucose-related values with extreme outliers. This controlled perturbation enables 
a direct comparison between noisy and denoised training conditions without relying on the original data distribution.

Denoising was performed using an interquartile range (IQR)–based clipping strategy, 
where values outside ([Q1 − 1.5 \cdot IQR, ; Q3 + 1.5 \cdot IQR]) were clipped to the corresponding bounds. 
This approach preserves all samples while mitigating the influence of extreme values 
in a statistically principled manner.

---

## 8. Potential Improvements and Alternative Problem Formulations

The supervised learning results indicate that predicting the exact next blood glucose value 
from short-term behavioral logs is inherently challenging. Blood glucose dynamics 
are influenced by many unobserved physiological and contextual factors, 
and the available event-level features provide only a partial view of the underlying state. 
As a result, model performance is constrained by information limits rather than by the choice of model alone.

To address this limitation, several alternative problem formulations are proposed as future improvements. 
These approaches aim to reduce target noise and increase predictability by redefining the learning objective.

### 8.1 Predicting Glucose Change (Δ Glucose)

Instead of predicting the absolute next glucose value, the task can be reformulated to predict the change in glucose 
relative to the most recent measurement:

$$
\Delta BG = BG_{next} - BG_{last}
$$

This formulation removes individual baseline effects and focuses the model on short-term glucose dynamics. 
It is more closely aligned with a state-transition perspective 
and is typically more stable than absolute-value regression.

**Pros:**

- Reduces inter-individual variability
- Often yields higher explanatory power
- Maintains continuous regression structure

**Cons:**

- Still sensitive to timing irregularities
- Requires careful interpretation of magnitude

### 8.2 Directional Glucose Prediction (Classification)

The regression task can be further simplified into a classification problem 
by predicting the direction of glucose change (e.g., increase vs. decrease, or decrease / stable / increase). 
This reduces sensitivity to measurement noise and focuses on clinically relevant trends.

**Pros:**

- More robust to noise and outliers
- Enables intuitive evaluation metrics (accuracy, F1-score)
- Clear decision-oriented interpretation

**Cons:**

- Discards information about change magnitude
- Requires threshold selection for class boundaries

### 8.3 Glucose Range or Risk-Level Prediction (Ordinal Classification)

Another alternative is to predict discrete glucose ranges 
(e.g., `< 70` - hypoglycemic, `70 – 140` - normal, `140 – 180` - elevated, `> 180` - high) 
instead of exact values. This formulation aligns more closely with clinical decision-making 
and reduces the impact of extreme values.

**Pros:**

- Strong clinical interpretability
- Less sensitive to extreme measurement errors
- Naturally supports ordinal or multiclass models

**Cons:**

- Loss of fine-grained numerical detail
- Class imbalance may require additional handling

---

## 9. References

The proposed improvements and alternative formulations are grounded in established work 
on medical time-series modeling, robustness to noise, and the inherent uncertainty of glucose dynamics, 
as summarized in the references below.

[1] Frank, A. (2010). UCI machine learning repository. http://archive.ics.uci.edu/ml.
<br />**(Reference for the UCI Diabetes dataset used in this project.)**

[2] Shahar, Y. (1997). A framework for knowledge-based temporal abstraction. Artificial intelligence, 90(1-2), 79-133.
<br />**(Supports the event-based representation and window-based feature construction from irregular medical records.)**

[3] Zeevi, D., Korem, T., Zmora, N., Israeli, D., Rothschild, D., Weinberger, A., ... & Segal, E. (2015). Personalized nutrition by prediction of glycemic responses. Cell, 163(5), 1079-1094.
<br />**(Demonstrates that blood glucose responses are influenced by many unobserved factors, explaining the intrinsic difficulty of glucose prediction.)**

[4] Contreras, I., & Vehi, J. (2018). Artificial intelligence for diabetes management and decision support: literature review. Journal of medical Internet research, 20(5), e10775.
<br />**(Provides background on the limitations and challenges of short-term glucose prediction.)**

[5] Breiman, L. (2001). Random forests. Machine learning, 45(1), 5-32.
<br />**(Theoretical foundation for using Random Forests to model non-linear relationships in supervised learning.)**

[6] Friedman, J. (2009). The elements of statistical learning: Data mining, inference, and prediction. (No Title).
<br />**(General reference for bias–variance tradeoff, regression behavior, and non-linear models.)**

[7] Galton, F. (1886). Regression towards mediocrity in hereditary stature. The Journal of the Anthropological Institute of Great Britain and Ireland, 15, 246-263.
<br />**(Original source of the ‘regression to the mean’ phenomenon observed in prediction results.)**

[8] M Bishop, C. (2006). Pattern recognition and machine learning.
<br />**(Supports alternative problem formulations such as classification and ordinal prediction instead of direct regression.)**

[9] Huber, P. (1981). Robust statistics. new york: John wiley and sons. HuberRobust statistics1981.
<br />**(Theoretical basis for robustness to noise and the use of outlier mitigation techniques such as IQR clipping.)**
