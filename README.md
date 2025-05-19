Livestock Market Chain Analyzer (LMCA)

This project is a comprehensive data analytics platform designed to study and evaluate the live animal market chain in the Gursum district of Ethiopia. The tool integrates economic theory, statistical modeling, and data visualization to support research findings from an MSc thesis using the Structure-Conduct-Performance (S-C-P) framework and the Heckman Two-Stage Selection Model.


---

Features

Data Preprocessing: Load and clean survey data from producers and traders

Exploratory Data Analysis (EDA): Summarize and visualize household and trader characteristics

Market Structure Analysis:

Calculate CR4 (Concentration Ratio of top 4 firms)

Estimate Herfindahl-Hirschman Index (HHI)

Identify barriers to market entry


Market Conduct Evaluation:

Analyze pricing strategies

Study buyer/seller negotiation behavior


Market Performance Assessment:

Calculate Gross Marketing Margins (TGMM)

Measure producer share in different market channels


Econometric Modeling:

Implement the Heckman Two-Stage Model

Model determinants of market participation and supply in TLU


Reporting & Visualization:

Generate PDF/HTML summaries

Export plots for publication




---

Project Structure

livestock-market-analyzer/
│
├── data/                      # Input datasets (CSV)
│   └── example_producers_data.csv
│
├── notebooks/                 # Exploratory and model Jupyter notebooks
│   └── 01_EDA_and_Model.ipynb
│
├── app/                       # Optional Streamlit interface
│   └── streamlit_app.py
│
├── models/                    # Trained model files (e.g., heckman_model.pkl)
│
├── utils/                     # Helper modules for analysis
│   ├── market_structure.py
│   ├── econometrics.py
│   └── visualizations.py
│
├── requirements.txt           # Python dependencies
├── README.md                  # Project documentation
└── LICENSE                    # Open source license


---

Installation

# Clone the repository
git clone https://github.com/your-username/livestock-market-analyzer.git
cd livestock-market-analyzer

# (Optional) Set up a virtual environment
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

# Install required packages
pip install -r requirements.txt


---

Usage

1. Run Exploratory Analysis

Open and run the notebook in notebooks/01_EDA_and_Model.ipynb:

Summary stats

Histograms

Correlation matrix


2. Market Structure Analysis

Use market_structure.py to compute:

CR4, HHI, and barriers to entry


3. Econometric Model

In the same notebook or econometrics.py, run:

Heckman 2-stage model (using statsmodels)

Analyze impact of variables on market participation and quantity supplied


4. Launch the Web App (Optional)

streamlit run app/streamlit_app.py


---

Example Model Output

Probit (Participation):

Positive factors: family size, extension contact, veterinary services

Negative factors: age of household head, distance to market


OLS (Supply):

Positive: livestock owned, family size

Negative: distance to market



---

License

This project is licensed under the MIT License. You are free to use, modify, and distribute it with attribution.


---

Author

Jemal Tahir Mumed
Haramaya University, 2024
MSc in Agribusiness and Value Chain Management

Feel free to cite this tool in academic research or use it for community impact projects in livestock economics.


---

Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.


---

Acknowledgements

Special thanks to Haramaya University, my advisors, and the Gursum District participants whose input helped shape this work.

Livestock Market Chain Analyzer - Step 3: Market Structure Analysis

import pandas as pd

Load the dataset

df = pd.read_csv("data/example_producers_data.csv")

Step 3.1: Calculate Concentration Ratio (CR4)

Assumed 'market_share' column for simplicity. Replace with actual logic if needed.

For real data, you'd sum animal volumes by trader, sort, and compute top 4 share.

market_share = [25, 20, 15, 10, 8, 7, 5, 5, 3, 2]  # Example: percent share of top 10 traders cr4 = sum(market_share[:4]) print(f"CR4 (Concentration Ratio of top 4 traders): {cr4}%")

Step 3.2: Calculate Herfindahl-Hirschman Index (HHI)

hhi = sum([s**2 for s in market_share]) print(f"HHI (Herfindahl Index): {hhi}")

Interpretation based on values:

- CR4 > 50% suggests strong oligopoly

- HHI < 1000 = competitive, 1000–1800 = moderately concentrated, >1800 = highly concentrated

Step 3.3: Identify Barriers to Entry (qualitative example)

barriers_to_entry = ["Lack of capital", "Limited licenses", "Poor infrastructure"] print("Barriers to market entry:") for barrier in barriers_to_entry: print(f"- {barrier}")

print("\nStep 3 complete: Market structure analysis done (CR4, HHI, entry barriers).")
Livestock Market Chain Analyzer - Step 4: Heckman Two-Stage Model

import pandas as pd import statsmodels.api as sm import statsmodels.formula.api as smf

Load the dataset

df = pd.read_csv("data/example_producers_data.csv")

Step 4.1: First Stage - Probit Model for Market Participation

Binary outcome: 1 if participated in market, 0 otherwise

probit_model = smf.probit('market_participation ~ age + family_size + distance_to_market + livestock_owned_tlu', data=df).fit() print("\n--- Probit Model (Stage 1) ---") print(probit_model.summary())

Step 4.2: Compute Inverse Mills Ratio (IMR)

df['mills_ratio'] = sm.distributions.norm.pdf(probit_model.fittedvalues) / sm.distributions.norm.cdf(probit_model.fittedvalues)

Step 4.3: Second Stage - OLS Model for Quantity Supplied (only for participants)

selection = df[df['market_participation'] == 1].copy() ols_model = smf.ols('market_supply_tlu ~ age + family_size + distance_to_market + livestock_owned_tlu + mills_ratio', data=selection).fit()

print("\n--- OLS Model (Stage 2) ---") print(ols_model.summary())

Step 4.4: Interpretations

Check significance of mills_ratio to determine presence of selection bias

if ols_model.pvalues['mills_ratio'] < 0.05: print("\nSelection bias detected and corrected using Heckman model.") else: print("\nNo significant selection bias detected.")

print("\nStep 4 complete: Heckman two-stage model executed.")

Livestock Market Chain Analyzer - Step 5: Streamlit Web App

Save this code as app/streamlit_app.py

import streamlit as st import pandas as pd import statsmodels.api as sm import statsmodels.formula.api as smf import matplotlib.pyplot as plt import seaborn as sns

st.set_page_config(page_title="Livestock Market Analyzer", layout="wide") st.title("Livestock Market Chain Analyzer - Gursum District")

Upload CSV file

data_file = st.file_uploader("Upload producer CSV data", type=["csv"])

if data_file is not None: df = pd.read_csv(data_file) st.success("Data uploaded successfully!")

# Show preview
st.subheader("Data Preview")
st.dataframe(df.head())

# Descriptive statistics
st.subheader("Descriptive Statistics")
st.write(df.describe())

# Correlation Matrix
st.subheader("Correlation Matrix")
corr = df.corr()
fig, ax = plt.subplots()
sns.heatmap(corr, annot=True, cmap="coolwarm", ax=ax)
st.pyplot(fig)

# Probit Model
st.subheader("Probit Model: Market Participation")
probit_model = smf.probit('market_participation ~ age + family_size + distance_to_market + livestock_owned_tlu', data=df).fit(disp=False)
st.text(probit_model.summary())

# Inverse Mills Ratio
df['mills_ratio'] = sm.distributions.norm.pdf(probit_model.fittedvalues) / sm.distributions.norm.cdf(probit_model.fittedvalues)

# OLS Model for supply (on participants only)
selection = df[df['market_participation'] == 1].copy()
ols_model = smf.ols('market_supply_tlu ~ age + family_size + distance_to_market + livestock_owned_tlu + mills_ratio', data=selection).fit()
st.subheader("OLS Model: Quantity Supplied")
st.text(ols_model.summary())

# Interpretation of IMR
if ols_model.pvalues['mills_ratio'] < 0.05:
    st.success("Selection bias detected and corrected.")
else:
    st.info("No significant selection bias detected.")

else: st.info("Awaiting CSV file upload...")

streamlit run app/streamlit_app.py
cd path/to/livestock-market-analyzer
git init
git add .
git commit -m "Initial commit: Livestock Market Chain Analyzer"
