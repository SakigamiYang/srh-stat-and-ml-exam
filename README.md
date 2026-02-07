# Statistics and Machine Learning Exam Project

## 1. Project Overview

[TODO]

---

## 2. Problem Statement

[TODO]

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