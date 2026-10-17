# Financial Health Prediction for Small Businesses

Can we predict how financially healthy a small business is from a few survey answers? This project cleans, explores and models survey data from small business owners in Southern Africa to predict whether their financial health is **low**, **medium** or **high**.

**Read the article:** [Predicting the Financial Health of Small Businesses with Python](https://dev.to/sharonnyabuto/predicting-the-financial-health-of-small-businesses-with-python-108m)

## Dataset

The data comes from the [data.org Financial Health Prediction Challenge](https://zindi.world/competitions/dataorg-financial-health-prediction-challenge), hosted on [Zindi](https://zindi.world).

- **Countries:** Eswatini, Lesotho, Malawi and Zimbabwe
- **Size:** 9,618 business owners, 39 columns. Some columns are completely empty for one country (for example, there is no gender data for Lesotho).
- **Target:** `Target`, the financial health category (`low`, `medium` or `high`). The challenge organisers based this on four areas: savings and assets, debt and repayment ability, resilience to shocks, and access to credit and financial services.
- **File:** `Data/financial_health.csv`, included in this repository exactly as downloaded from Zindi.
- **Licence:** the challenge rules allow the data to be used and shared under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). If you reuse the data, credit the source above and keep this licence on it.

## What the notebook does

1. **Checks the data** for size, duplicates, missing values and column types.
2. **Cleans the text answers** so that the same answer is always spelled the same way, and checks the missing values country by country (some questions were not asked in some countries).
3. **Cleans the numbers:** converts money to US dollars (the data uses four currencies), drops columns that cannot be used and removes rows where personal income is higher than business turnover.
4. **Explores the data** (owners' age and gender, tax compliance, cash flow and credit).
5. **Tests which columns are linked to financial health** using standard statistical tests.
6. **Splits the data** 80/20 into training and test sets.
7. **Fills the missing values after the split, using the training data only.** Text columns with more than 35% missing answers **in at least one country** get an `unknown` category, the other text columns get the most common answer, and number columns get the median. The same values are then used to fill the test data, so the test data stays truly unseen.
8. **Encodes and rescales:** turns the text answers into numbers and rescales the number columns, again learning from the training data only.
9. **Trains and compares seven kinds of model:** Logistic Regression, Decision Tree, K-Nearest Neighbours, Random Forest, Gradient Boosting, XGBoost and SVM.

## Results

The models were tested on 1,473 businesses they had not seen during training (20% of the data). Only 79 of them (about 5%) are in the `high` group.

| Model | Accuracy | Macro F1 | Recall on `high` |
|-------|:--------:|:--------:|:----------------:|
| Logistic Regression | 0.86 | 0.75 | 0.41 |
| Decision Tree | **0.87** | 0.77 | 0.53 |
| K-Nearest Neighbours | 0.77 - 0.79 | 0.62 | 0.24 - 0.28 |
| Random Forest | 0.81 | 0.59 | 0.13 |
| Gradient Boosting | **0.87** | **0.79** | **0.54** |
| XGBoost | **0.87** | 0.78 | 0.51 |
| SVM | 0.83 | 0.66 | 0.24 |

> The three K-Nearest Neighbours runs (Euclidean, Manhattan and Minkowski) are shown as a range. Euclidean and Minkowski give identical results, because Minkowski with its default setting is the same as Euclidean.

**How to read the table**

- **Accuracy** is the share of businesses classified correctly.
- **Macro F1** is the average score across the three groups, so the small `high` group counts as much as the large `low` group.
- **Recall on `high`** is the share of truly high-health businesses that the model found.

Only just over 5% of the businesses are in the `high` group, and about 65% are in the `low` group. Because of this, accuracy alone can look good even when a model is poor at finding the `high` group, so macro F1 and the recall on `high` are the more useful scores.

**Takeaways**

- Gradient Boosting scored best overall (macro F1 0.79), followed closely by XGBoost (0.78) and the Decision Tree (0.77). The gaps between the top models are small and come from a single split.
- No model found more than about half of the `high` businesses (best recall 0.54).
- The Random Forest was always right when it said `high` (precision 1.00) but found only 13% of them, so it was very cautious.
- Four models scored between 0.86 and 0.87 on accuracy, yet their recall on `high` ranged from 0.41 to 0.54.
- The simple models (Logistic Regression, Decision Tree) came close to the boosting models.

Scores come from my own train/test split, not from the Zindi leaderboard.

## Project structure

The notebook loads the data from `../Data/financial_health.csv`, so the two folders must sit side by side.

```text
.
├── Data/
│   └── financial_health.csv
│   └── VariableDefinitions.csv
├── Notebooks/
│   └── fin_health.ipynb
├── requirements.txt
└── README.md
```

## Getting started

1. Clone the repository:

   ```bash
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

2. Create and activate a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Windows: .venv\Scripts\activate
   ```

3. Install the libraries:

   ```bash
   pip install -r requirements.txt
   ```

4. Open the notebook and run the cells from top to bottom:

   ```bash
   jupyter notebook Notebooks/fin_health.ipynb
   ```

Run the cells in order, because the cleaning steps change the same table step by step. The notebook was written with Python 3.13 and a recent version of pandas.

## Assumptions and limitations

Several choices in this project are my own judgement and have not been independently reviewed. Anyone building on this work should check them against the variable definitions that come with the challenge data:

- In `current_problem_cash_flow`, the value `0` is treated as `no`.
- In `keeps_financial_records`, a plain `yes` is treated as `yes, always`.
- Personal income is assumed to be monthly and business turnover annual, so rows with income above turnover are removed. Business expenses are dropped because it is unclear whether they are monthly or annual.
- One fixed exchange rate per country is used to convert to US dollars.
- Text columns with more than 35% missing answers in at least one country get an `unknown` category. The 35% cut-off, and applying it country by country, are judgement calls.
- During the exploration and statistical tests, rows with a missing answer are left out for that column, because the gaps are only filled after the split.

Known limitations;

- Some questions were not asked in some countries, so the model cannot learn what those answers mean for that country. The `unknown` category also lines up with the country.
- The share of `high` businesses differs a lot between countries (over 12% in Eswatini, under 1% in Lesotho), so the model may rely heavily on the country.
- Rows where personal income is higher than business turnover were removed, so the model has never seen such rows.

## Acknowledgements

- Data: [data.org](https://data.org) Financial Health Prediction Challenge on [Zindi](https://zindi.world)
