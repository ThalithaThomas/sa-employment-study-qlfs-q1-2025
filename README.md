# sa-employment-study-qlfs-q1-2025
Data analysis of South African labour market trends (QLFS Q1 2025) including provincial, gender, and population group disparities with statistical validation.

South African Employment Statistics Analysis (QLFS 2025) 📊


Project Overview
This project analyzes South Africa’s 2025 Quarterly Labour Force Survey (QLFS) data to uncover labor market dynamics. The analysis answers key business questions about unemployment disparities across provinces, gender, and population groups. It demonstrates end-to-end data analysis skills: from data cleaning in Power Query to statistical testing and visualization with Python and Power BI.

🎯 Business Questions
Which provinces have the highest and lowest unemployment rates and volumes? What factors might explain these differences?

Where is the gender gap in unemployment the widest?

Which population groups are most and least employed nationally?

🛠 Skills Demonstrated
Data Wrangling – Cleaned and standardized raw CSV data with Power Query and Excel

Statistical Analysis – Used Python (pandas, SciPy) for descriptive stats, chi-square tests, and confidence intervals

Visualization & Storytelling – Designed interactive Power BI dashboards for provincial and national insights

Highlights

Built an interactive Power BI dashboard used to compare unemployment trends across 9 provinces.

Applied statistical significance testing to confirm observed disparities (p < 0.001).

Delivered actionable policy recommendations for gender, provincial, and demographic disparities.

🧹 Data Cleaning & Processing (Power Query)
Initial Data Import

Loaded raw QLFS 2025 dataset via Power Query in Excel.

Column Header Cleanup

Standardized headers, removed extra rows, ensuring a single clean header.

Province Column Creation

Extracted valid province names from Population_Group column using conditional logic, separating geographic from demographic data.

Data Filtering

Filtered Population_Group to retain demographic categories only (e.g., Black African, Coloured, Indian/Asian, White, Total), removing geographic entries now stored in Province.

Unit Conversion

Recognized population figures were in thousands; multiplied relevant columns by 1,000 to obtain absolute values.

Final Dataset Structure

Clean dataset with clear Province and Population_Group columns plus population metrics ready for analysis.

🐍 Data Analysis & Statistical Process (Python)
import pandas as pd
import numpy as np
from scipy.stats import chi2_contingency, norm

# Load cleaned data
df = pd.read_excel('QLFS_2025_cleaned.xlsx')
df.columns = df.columns.str.lower()

# Helper function to calculate rates
def calc_rate(numerator, denominator):
    return np.where(denominator != 0, np.round((numerator / denominator) * 100, 1), np.nan)
Question 1: Provincial Unemployment Rates & Volumes
# Aggregate unemployment and employment by province
province_summary = df.groupby('province').agg({
    'male_unemployed':'sum', 'male_employed':'sum', 'male_economically_active':'sum',
    'female_unemployed':'sum', 'female_employed':'sum', 'female_economically_active':'sum',
    'employed':'sum', 'unemployed':'sum', 'total_economically_active':'sum'
}).reset_index()

# Calculate unemployment and employment rates
province_summary['unemployment_rate'] = calc_rate(province_summary['unemployed'], province_summary['total_economically_active'])
province_summary['employment_rate'] = calc_rate(province_summary['employed'], province_summary['total_economically_active'])
province_summary['male_unemployment_rate'] = calc_rate(province_summary['male_unemployed'], province_summary['male_economically_active'])
province_summary['female_unemployment_rate'] = calc_rate(province_summary['female_unemployed'], province_summary['female_economically_active'])

# Identify provinces with highest and lowest unemployment rates and volumes
max_rate_province = province_summary.loc[province_summary['unemployment_rate'].idxmax()]
min_rate_province = province_summary.loc[province_summary['unemployment_rate'].idxmin()]

max_vol_province = province_summary.loc[province_summary['unemployed'].idxmax()]
min_vol_province = province_summary.loc[province_summary['unemployed'].idxmin()]
Statistical Validation — Chi-Square Test
# Prepare contingency table for chi-square test between highest and lowest unemployment provinces
contingency_table = [
    [int(max_rate_province['unemployed']), int(max_rate_province['employed'])],
    [int(min_rate_province['unemployed']), int(min_rate_province['employed'])]
]

chi2, p_value, _, _ = chi2_contingency(contingency_table)
print(f"Chi-square test p-value: {p_value:.5e}")
Confidence Interval Function
def calculate_unemployment_ci(unemployed, total_active, confidence=0.95):
    if total_active == 0:
        return np.nan, np.nan
    p = unemployed / total_active
    z = norm.ppf(1 - (1 - confidence) / 2)
    se = np.sqrt(p * (1 - p) / total_active)
    margin = z * se
    lower = max(0, (p - margin) * 100)
    upper = min(100, (p + margin) * 100)
    return lower, upper

# Calculate and compare 95% confidence intervals
max_lower, max_upper = calculate_unemployment_ci(max_rate_province['unemployed'], max_rate_province['total_economically_active'])
min_lower, min_upper = calculate_unemployment_ci(min_rate_province['unemployed'], min_rate_province['total_economically_active'])

ci_overlap = not (max_lower > min_upper or min_lower > max_upper)
Question 2: Gender Gap in Unemployment
# Calculate gender unemployment gaps per province
province_summary['gender_gap'] = province_summary['female_unemployment_rate'] - province_summary['male_unemployment_rate']

# Rank provinces by gender gap
gender_gap_rank = province_summary.sort_values('gender_gap', ascending=False)
Question 3: Employment by Population Group Nationally
national_employment = []
for group in df['population_group'].unique():
    grp = df[df['population_group'] == group]
    employed = grp['employed'].sum()
    active = grp['total_economically_active'].sum()
    unemployed = grp['unemployed'].sum()
    employment_rate = (employed / active) * 100 if active > 0 else np.nan
    unemployment_rate = (unemployed / active) * 100 if active > 0 else np.nan
    national_employment.append({
        'group': group,
        'employed': employed,
        'active': active,
        'employment_rate': employment_rate,
        'unemployment_rate': unemployment_rate
    })

# Sort by employment volume and rate
employment_by_volume = sorted(national_employment, key=lambda x: x['employed'], reverse=True)
employment_by_rate = sorted(national_employment, key=lambda x: x['employment_rate'], reverse=True)
📈 Key Insights & Interpretation


Provincial Unemployment Differences
North West has the highest unemployment rate (40.4%) with 596,071 unemployed.

Western Cape has the lowest (19.6%) with 696,808 unemployed.

The 20.8 percentage point gap is statistically significant (chi-square p < 0.001).

Confidence intervals do not overlap, confirming significance.



Gender Gap in Unemployment
Women face higher unemployment in every province.

The widest gap is in North West (11.5 percentage points).

Rural provinces like Limpopo and Free State also show severe disparities.

Average gender gap nationally is 5.6 percentage points.



Employment by Population Group
Black African population holds 75.8% of jobs but has the lowest employment rate (63%).

White population has the highest employment rate (92.7%) despite fewer jobs (10.5%).

This highlights systemic inequalities in employment access.
Strong Recommendations Based on Findings
1. Provincial Unemployment Disparities
Evidence:

North West’s unemployment rate (40.4%) is 20.8pp higher than Western Cape’s (p < 0.001, CI non-overlap).

Over 596,000 unemployed in North West versus 696,000 in Western Cape, despite WC’s larger workforce.

✅ Recommendations

Focus Area	Key Actions
Tackle Regional Unemployment Gaps	
- Stimulate high-unemployment provinces (North West, Eastern Cape, Free State) with investments in manufacturing, agribusiness, and renewable energy.
- Improve transport, broadband, and logistics to reduce geographic isolation.
- Adapt Western Cape’s tourism/agritech model to other provinces.
  
Close Gender Employment Gaps	- Launch women-focused job programs in rural areas.
- Provide childcare and transport subsidies.
- Set gender equity hiring targets in public works and infrastructure projects.
  
Address Inequalities Across Population Groups
- Expand skills training and apprenticeships for unemployed Black African workers.
- Offer hiring incentives for underrepresented groups.
- Enforce anti-discrimination in recruitment.
  
Leverage Economic Hubs & Growth Sectors
- Use Gauteng’s scale to expand infrastructure, logistics, and manufacturing.
- Strengthen rural–urban economic linkages.
- Invest in agriculture, agro-processing, tourism, and green economy jobs.
  
Strengthen Labour Market Data & Coordination
- Create quarterly labour market dashboards.
- Build public–private training partnerships.
- Establish regional employment hubs for job matching and SME support.

 Visualisations
The analysis is supported by interactive Power BI dashboards and Python-based statistical charts to uncover trends in unemployment and employment distribution.
## 📊 Power BI Dashboard
You can view the interactive dashboard here: [View Dashboard](https://app.powerbi.com/links/5CziXLMPje?ctid=97e0f4cf-7980-44e9-81e2-2472e6bfa4f4&pbi_source=linkShare)


📊 Key Dashboard Pages:

Provincial Unemployment Disparities – Compares unemployment rates & volumes across all provinces.
<img width="634" height="358" alt="Screenshot 2025-08-11 at 14 05 51" src="https://github.com/user-attachments/assets/c27197cf-1955-4b1f-a093-b732af8f344f" />


Employment by Population Group – Shows employment rate and volume by demographic group per province.
<img width="641" height="362" alt="Screenshot 2025-08-11 at 14 06 54" src="https://github.com/user-attachments/assets/72afa4fd-c17b-4012-88f8-fd551b9d0afd" />

Gender Gap Analysis – Highlights male vs. female unemployment rates with provincial rankings.
<img width="646" height="360" alt="Screenshot 2025-08-11 at 14 09 41" src="https://github.com/user-attachments/assets/9f5626d4-c7d7-4565-aca0-ecf518340989" />

Geographic Patterns & Economic Belts Analysis
<img width="637" height="357" alt="Screenshot 2025-08-11 at 15 28 09" src="https://github.com/user-attachments/assets/5bc4cdf9-c374-4984-aa33-1ac7287ca464" />
  
🛠 Technology Stack
Data Processing: Power Query, Python

Visualization: Power BI

Statistical Analysis: Python (Pandas, NumPy, SciPy)
Takeaway: This project demonstrates how integrated data engineering, statistical testing, and interactive visualization can turn raw government survey data into targeted, evidence-based policy recommendations.
