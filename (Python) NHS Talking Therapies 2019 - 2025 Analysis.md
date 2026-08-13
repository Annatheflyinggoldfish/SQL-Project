# NHS Talking Therapies 2019 - 2025: National Trends, Demographic & Geographic Analysis (Preview) 

**A data analysis project examining service demand, treatment outcomes, demographic representation, and geographic equity in NHS Talking Therapies (IAPT) services across England.**

## Tools Used

**`Python`· `pandas` · `NumPy` · `SciPy`· `Plotly`**


## Overview

NHS Talking Therapies (formerly IAPT) is England's flagship programme for treating anxiety and depression through evidence-based psychological therapies. This project analyses whether the service access and treatment outcomes are equitable across different population groups and geographic areas, using publicly available NHS Digital data.

## Key Questions

**- How have service demand, appointments efficiency, and treatment outcome changed over time nationally?**

**- Are certain demographic groups (gender, ethnicity) over/under represented compared to the general population?**

**- Does deprivation (IMD) correlate with variation in service access and/or outcomes?**

**- Are there meaningful geographic disparities in service access/outcome across England's 42 Integrated Care Boards (ICBs)?**



### The analysis is structured as three linked notebooks (Welcome to download and run locally):

| Notebook | Focus | Key Techniques |
|---|---|---|
| [01 NHS Talking Therapies - Cleaning.ipynb](https://colab.research.google.com/drive/1LvEKX0ORIJt1LcAqN6YgoPYoXnbyyZmA#scrollTo=K2j33NjYn6zp) | Cleaning and structuring raw NHS Digital datasets | Data cleaning: handling missing values, schema standardisation |
| [02 NHS Talking Therapies - EDA part 1.ipynb](https://colab.research.google.com/drive/1SMcayvIqioYXsH-vVryktwTvnqZMyR15#scrollTo=hNVyojD7UT10) | National-level trend exploration | Time series analysis: service demand, appointments efficiency, and treatment outcome |
| [03 NHS Talking Therapies - EDA part 2.ipynb](https://colab.research.google.com/drive/1Ga34-NX5Gnakc6pC7JuKS13H8tPdXDp1#scrollTo=61WdBP8VV9Uf) | Demographic and geographic breakdown | Correlation analysis, representation ratios, geographic data merging |

---

## Methodology Highlights

- **Data standardisation with SQL**: Used DuckDB SQL queries to standardise column naming and structure across six years of data before merging into a single dataset.
- **Geographic Data merging**: Matched 42 ICBs to ONS population estimates using prefix-based matching to handle inconsistent naming conventions across datasets.
- **Representation ratios**: Built using England-only population denominators: 2021 Census (ethnicity) and ONS mid-year population estimates (ICB geography). Age-structure mismatch (NHS Talking Therapies mainly targets ages 16+) is flagged as a known limitation.
- **Statistical testing**: Applied correlation and representation-ratio analysis to assess demographic and geographic patterns across the dataset.

---

## Sample Outputs

**Note: Please refer to the notebooks above for full interactive charts and complete analysis**.

---

### **Service Demand & Treatment Funnel Overview**

<details>
<summary>View key code: Service Funnel Conversion Rates</summary>

```python
# Service Funnel Conversion Rates
conversion_df = pd.DataFrame({
    'Year': df_treatment['Year'],
    'Referral to Accessing (%)': df_treatment['Accessing Services'] / df_treatment['Referrals Received'] * 100,
    'Lost at Accessing (%)': 100 - df_treatment['Accessing Services'] / df_treatment['Referrals Received'] * 100,
    'Lost at Finishing (%)': (df_treatment['Accessing Services'] - df_treatment['Finished Course Treatment']) / df_treatment['Referrals Received'] * 100
}).set_index('Year')

print(f"Referrals Received --> Accessing Services: {conversion_df['Lost at Accessing (%)'].mean().round(1)}% loss.")
print(f"Accessing Services --> Finished Course Treatment: {conversion_df['Lost at Finishing (%)'].mean().round(1)}% loss.")
```

</details>

<img width="988" height="448" alt="image" src="https://github.com/user-attachments/assets/3b217cd0-10a1-496f-adcf-7ac992c6803f" />

### **Gender Breakdown Dashboard** 

<details>
<summary>View key code: referral volume by year</summary>

```python
# gender share of total referrals, aggregated by year
df_gender['Percentage of the Year'] = (
    df_gender['Count_ReferralsReceived']
    / df_gender.groupby('Year')['Count_ReferralsReceived'].transform('sum')
    * 100)

# aggregating data for plotting
gen_plot1 = df_gender[['Year','VariableA','Percentage of the Year']].groupby('VariableA')['Percentage of the Year'].mean().round(2)
```

</details>

<img width="962" height="552" alt="image" src="https://github.com/user-attachments/assets/54af85d6-62e1-4cb2-b7c3-de731a78a6b9" />

### **Ethnicity: Representation Ratio vs. Recovery Rate** 

<details>
<summary>View key code: representation ratio calculation</summary>

```python
# representation ratio calculation
eth_merged["Representation Ratio"] = (
    eth_merged["Percentage of the Year"] / eth_merged["england_pct"] * 100
).round(2)

# flag mismatch
missing = set(eth_plot1_1_labeled['Ethnicity']) - set(eth_merged['Ethnicity'])
if missing:
    print(f"Unmatched categories dropped from merge: {missing}")

# testing whether representation ratio correlates with recovery rate
pearson_r, pearson_p = stats.pearsonr(plot_deep_dive['Representation Ratio'], plot_deep_dive['Recovery Rate'])
spearman_r, spearman_p = stats.spearmanr(plot_deep_dive['Representation Ratio'], plot_deep_dive['Recovery Rate'])
```

</details>

<img width="923" height="484" alt="image" src="https://github.com/user-attachments/assets/66764560-0093-427e-945d-215fa340491e" />


### **IMD (Index of Multiple Deprivation) Dashboard** —

<details>
<summary>View key code: % of referrals ended before treatment began & referral share by IMD</summary>

```python
# % of referrals ended before treatment began
# denominator is "ended referrals" (not total referrals received)
df_dep['% Ended Before Treatment Began'] = (
    df_dep['Count Ended Before Treatment'] / df_dep['Count Ended Referrals'] * 100
    ).round(2)

# referral share of total, aggregated by year
df_dep['Percentage of the Year'] = (
    df_dep['Total Referrals Received']
    / df_dep.groupby('Year')['Total Referrals Received'].transform('sum')
    * 100)
```

</details>

<img width="1080" height="544" alt="image" src="https://github.com/user-attachments/assets/5d87250c-26d7-4d8e-ac29-4fdead489741" />


### **Geographic Representation Map**

<details>
<summary>View key code: matching ICB names across NHS and ONS datasets</summary>

```python
# using prefix to match names as NHS and ONS use inconsistent ICB naming systems
def match_name(name_clean, pop_names):
    for pn in pop_names:
        if pn.startswith(name_clean):
            return pn
    return None

pop_names_list = pop_merged['name_upper'].tolist()

geo_plot1['name_clean'] = geo_plot1['OrgName_clean'].str.replace(r'\.\.\.$', '', regex=True).str.rstrip('.')
geo_plot1['matched_name'] = geo_plot1['name_clean'].apply(lambda x: match_name(x, pop_names_list))

# flag any ICBs that failed to match
unmatched = geo_plot1[geo_plot1['ICB_population'].isna()]
print(f"ICBs with no population match: {len(unmatched)}")
```

</details>

<img width="700" height="455" alt="image" src="https://github.com/user-attachments/assets/b83a0671-6f0e-48f4-aacc-32ed58f2cce3" />

---

## Data Source

All datasets were drawn from publicly available sources. All data are aggregated, publicly published statistics; no patient-level or identifiable data is used.

* **NHS Talking Therapies data:** [NHS Digital, Psychological Therapies Annual Reports 2019/20–2024/25](https://digital.nhs.uk/data-and-information/publications/statistical/nhs-talking-therapies-for-anxiety-and-depression-annual-reports)

  *This is the cleaned, merged version of six annual [NHS Digital datasets](https://github.com/Annatheflyinggoldfish/Data-Analysis-Projects/blob/main/2019-2025%20NHS%20Therapy%20Analytics%20-%20Datasets/NHS_Talking_Therapies_Cleaned_All_Years.csv). See [`01_data_cleaning.ipynb`](https://colab.research.google.com/drive/1LvEKX0ORIJt1LcAqN6YgoPYoXnbyyZmA#scrollTo=K2j33NjYn6zp) for the full cleaning process.*
* **ICB geographic boundaries:** [Integrated Care Boards (April 2023) Boundaries EN BSC](https://geoportal.statistics.gov.uk/datasets/ons::integrated-care-boards-april-2023-boundaries-en-bsc/about)
* **Ethnicity reference data:** [Ethnic group, Census 2021 (TS021) - Office for National Statistics](https://www.ons.gov.uk/datasets/TS021/editions/2021/versions/3)
* **ONS population by ICB:** [ONS, Mid-2022 revised (Nov 2025) to Mid-2024: Integrated Care Boards (2024 geography edition)](https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/clinicalcommissioninggroupmidyearpopulationestimates/mid2022revisednov2025tomid2024integratedcareboards2024geography)

---

## About This Project

This project was built as part of my transition into data analysis, with a particular interest in public sector and health data.

I'm currently seeking junior data analyst roles, particularly within the NHS, the public sector, and non-profit organisations.

**Contact**: huanglp582@gmail.com · [LinkedIn](https://www.linkedin.com/in/huangliping)
