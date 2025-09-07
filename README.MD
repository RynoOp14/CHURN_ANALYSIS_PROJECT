Customer Churn — End-to-End Project (SQL → Power BI → Python)

This repository contains a complete churn analytics pipeline using SQL Server,Power BI, and Python.
It covers data preparation, visualization, and machine learning modeling to understand and predict customer churn.

📂 Repository Structure

* `SQLQuery1.sql` → Audit of NULL/missing values
* `SQLQuery2.sql` → Profiling queries (distribution, revenue share, distinct values)
* `SQLQuery3.sql` → Views for cohorts (`vw_ChurnData`, `vw_JoinData`)
* `SQLQuery4.sql` → Cleansed product table `prod_Churn`
* `project.pbix` → Power BI dashboards
* `churn_prediction.ipynb` → Jupyter Notebook for churn prediction
* `prediction_data.xlsx` → Excel export from SQL view (`vw_JoinData`) for ML input
* `/images/` → Add screenshots of Power BI dashboards here

---

1. SQL Server — Data Preparation

Run the queries in **SQL Server Management Studio (SSMS):**

1. `SQLQuery4.sql` → Creates `[dbo].[prod_Churn]` (handles nulls/defaults).
2. `SQLQuery3.sql` → Creates views: `vw_ChurnData` and `vw_JoinData`.
3. `SQLQuery1.sql` → Runs a NULL audit.
4. `SQLQuery2.sql` → Runs profiling queries for churn insights.

✅ After execution:

* `prod_Churn` is cleaned.
* Views `vw_ChurnData` and `vw_JoinData` are available for analysis.

---

2. Power BI — Visualization

2.1 Connect to SQL Server

* Get Data → SQL Server
* Import `prod_Churn`, `vw_ChurnData`, `vw_JoinData`
* Use Import mode for better performance

2.2 Create Core DAX Measures

```
Customer Count = COUNTROWS('prod_Churn')
Churned Customers = CALCULATE([Customer Count], 'prod_Churn'[Customer_Status] = "Churned")
Stayed Customers = CALCULATE([Customer Count], 'prod_Churn'[Customer_Status] = "Stayed")
Churn Rate = DIVIDE([Churned Customers], [Customer Count])
Total Revenue = SUM('prod_Churn'[Total_Revenue])
Avg Monthly Charge = AVERAGE('prod_Churn'[Monthly_Charge])
```

2.3 Suggested Dashboards

* Page 1 (Overview): KPI cards, Donut chart (Churned vs Stayed), Bar chart (Churn Category), Line chart (Tenure vs Charges)
* Page 2 (Profile): Stacked bar (Contract vs Status), Column chart (Churn by State), Matrix (Internet × Security)
* Page 3 (Churn Reasons): Bar chart of top churn reasons

📸 Add screenshots of dashboards to `/images/` and reference in README.

---

3. Python — Machine Learning

Notebook: `churn_prediction.ipynb`

3.1 Environment Setup

```
python -m venv .venv
.\.venv\Scripts\activate   # Windows
# or source .venv/bin/activate   # Mac/Linux
pip install -U pandas numpy scikit-learn matplotlib seaborn pyodbc joblib openpyxl sqlalchemy
```

3.2 Load Data

Option A — From SQL

```
import pyodbc, pandas as pd
conn = pyodbc.connect("DRIVER={ODBC Driver 17 for SQL Server};SERVER=localhost;DATABASE=db_Churn;Trusted_Connection=yes;")
df = pd.read_sql("SELECT * FROM dbo.prod_Churn", conn)
```

Option B — From Excel (`prediction_data.xlsx`)

```
import pandas as pd
df = pd.read_excel("prediction_data.xlsx")
```

3.3 Train Model

```
X = df.drop(columns=["Customer_Status"])
y = (df["Customer_Status"] == "Churned").astype(int)

from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

from sklearn.linear_model import LogisticRegression
model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
```

3.4 Evaluate Model

```
from sklearn.metrics import classification_report, roc_auc_score
pred = model.predict(X_test)
proba = model.predict_proba(X_test)[:, 1]

print(classification_report(y_test, pred))
print("ROC-AUC:", roc_auc_score(y_test, proba))
```

3.5 Save Model

```
import joblib
joblib.dump(model, "churn_model.joblib")
```

3.6 (Optional) Random Forest & SQL Output

```
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(n_estimators=200, random_state=42)
rf.fit(X_train, y_train)
print("RF ROC-AUC:", roc_auc_score(y_test, rf.predict_proba(X_test)[:, 1]))

from sqlalchemy import create_engine
engine = create_engine("mssql+pyodbc://@localhost/db_Churn?driver=ODBC+Driver+17+for+SQL+Server&trusted_connection=yes")
scored = X_test.copy()
scored["Churn_Prob"] = proba
scored.to_sql("ml_Scored", engine, schema="dbo", if_exists="replace", index=False)
```

---

🔁 Execution Order

1. Run SQL scripts → `prod_Churn` & views created
2. Build Power BI dashboards → KPIs & visuals from data
3. Run Notebook → Train churn models using SQL or Excel data
4. (Optional) Write predictions to SQL & refresh Power BI

---

✅ Checklist

* SQL scripts executed successfully
* Power BI visuals built with correct measures
* Notebook runs end-to-end without errors
* Model saved and/or predictions written back to SQL



