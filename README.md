# NHIS Access-to-Care Analysis — Datathon 2026

Analysis of the CDC NHIS Adult Summary Health Statistics dataset: access barriers, 2025 forecasts, and at-risk subgroup identification using weighted linear regression, K-Means, and Isolation Forest.

**Full documentation** (methods, results, rubric-aligned report): **[report/submission.md](report/submission.md)**

**Slideshow**: [https://drive.google.com/file/d/1dmw7SZtqFao00QV-3P_GQnk-KL1H7qwn/view]

---

## How to run

Open the notebook listed under analysis.ipynb after cloning this repo. 

**Requirements**: Python 3.8+, pandas, numpy, matplotlib, scikit-learn

```bash
# Open and run in Jupyter
jupyter notebook analysis.ipynb
```

Or run non-interactively (generates all figures and writes the report):

```bash
jupyter nbconvert --to notebook --execute analysis.ipynb
```

- **Input**: `Access_to_Care_Dataset.csv`
- **Output**: `figures/` (25 PNGs), `report/submission.md`
