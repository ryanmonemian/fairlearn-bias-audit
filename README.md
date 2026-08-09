# Fairlearn Bias Audit: COMPAS Recidivism Dataset

Auditing a recidivism prediction classifier for racial bias using Fairlearn, 
built on ProPublica's COMPAS dataset.

## Overview

This project trains a baseline classifier to predict two-year recidivism, 
then uses Fairlearn to measure whether the model treats defendants of 
different races differently. It also attempts to reduce any disparity found 
using constrained optimization, and compares the results against COMPAS's 
own real-world risk score.

## Key Findings

- The baseline model's overall false positive rate was 28.3%, but this was 
  not evenly distributed: 36.3% for African-American defendants who did not 
  reoffend, versus 20.2% for Caucasian defendants who did not reoffend.
- Demographic parity difference: 0.189. Equalized odds difference: 0.161 
  (African-American vs. Caucasian defendants).
- Mitigation with Fairlearn's `ExponentiatedGradient` reduced both fairness 
  gaps, at a cost of about 2.5 percentage points of overall accuracy. The 
  gap closed because false positive rates rose for both groups, not because 
  outcomes improved for African-American defendants. Results from this step 
  vary slightly between runs due to randomness in the algorithm itself.
- COMPAS's own risk score was slightly more accurate (65.5%) than the 
  baseline model, but showed a larger racial disparity on every fairness 
  metric measured (DPD: 0.259, EOD: 0.225).
- Permutation importance showed `priors_count` was by far the most 
  influential feature in the model's predictions, raising the concern that 
  it may indirectly reflect uneven policing across communities.

Full methodology, reasoning, and results are documented in the notebook and 
model card linked below.

## Project Structure

```
fairlearn-bias-audit/
├── data/
│   └── compas-scores-two-years.csv
├── notebooks/
│   └── bias_audit.ipynb       # full analysis
├── MODEL_CARD.md              # model card (see below)
├── README.md
├── LICENSE
├── requirements.txt
```

## Model Card

See [model_card.md](./model_card.md) for intended use, limitations, ethical 
considerations, and full quantitative results.

## Setup

Clone the repo and set up a virtual environment:

```bash
git clone https://github.com/ryanmonemian/fairlearn-bias-audit.git
cd fairlearn-bias-audit
python -m venv venv
```

Activate the environment:
- Windows (PowerShell): `venv\Scripts\activate`
- macOS/Linux: `source venv/bin/activate`

Install dependencies:
```bash
pip install -r requirements.txt
```

Open `notebooks/bias_audit.ipynb` in Jupyter or VS Code and select the 
`venv` kernel to run it.

## Tech Stack

Python, pandas, scikit-learn, Fairlearn, Jupyter

## Data Source

Larson, Jeff, et al. "Compas-Analysis." GitHub, ProPublica, 2016, 
github.com/propublica/compas-analysis.

Angwin, Julia, et al. "How We Analyzed the COMPAS Recidivism Algorithm." 
ProPublica, 23 May 2016, 
propublica.org/article/how-we-analyzed-the-compas-recidivism-algorithm.

## License

MIT License
