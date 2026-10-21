# Maji Ndogo Farm Yield Prediction

**Predicting crop yield per hectare from geographic, weather, and soil data, after finding and fixing two data quality issues before any model was trained**

> Interactive dashboard: [dashboard/farm_yield_dashboard.html](dashboard/farm_yield_dashboard.html)

## In short

- **What:** I predicted crop yield per hectare for 5,654 farm fields, after finding that the column named "yield" was the wrong thing to predict.
- **Data:** 5,654 fields, 8 crop types, and 5 provinces, plus 1,843 free-text weather station messages.
- **Tools:** Python (pandas, scikit-learn, regex), Plotly.
- **Main findings:**
  - Two data issues had to be fixed first: two columns were swapped, and the obvious target mostly measured farm size.
  - Tea had the highest yield per hectare, and volcanic and loamy soils produced the best yields.
  - The best model (Gradient Boosting) had an R² of 0.96, meaning it explains about 96% of the variation in yield. This is unusually high and most likely reflects that the data was created for a course.

---

## Problem

The dataset covers 5,654 farm fields across 5 provinces and 8 crop types, with geographic, weather, soil, and farm management data. The obvious column to predict is called `Annual_yield`, but it turned out to be the wrong one (explained below).

In this project, I worked out what the correct target was, prepared a clean set of features, and built a model that predicts yield per hectare and shows which environmental factors affect it most.

**Research questions:**
- What does "yield" actually mean in this dataset, and is the column with the most obvious name the right one to model?
- Which environmental factors (soil, weather, pollution, geography) are related to yield?
- Can yield per hectare be predicted from field characteristics, and how well?

---

## Dataset

Source: `Maji_Ndogo_farm_survey_small.db` (4 linked tables covering geographic, weather, soil and crop, and farm management data for 5,654 fields) and `Weather_Station_Dataset.zip` (1,843 free-text weather station messages).

---

## Two data issues I found before modelling

1. **`Crop_type` and `Annual_yield` were swapped in the source data.** The column labelled `Crop_type` held numbers, and the column labelled `Annual_yield` held crop names, some with typos such as "wheatn", "teaa", and "cassaval". I confirmed and fixed this in [`notebooks/01_data_cleaning.ipynb`](notebooks/01_data_cleaning.ipynb).
2. **`Annual_yield` is not a useful target to predict.** It has a correlation of 0.95 with `Plot_size`, because it is the total output of the farm, which naturally grows with farm size. The `Standard_yield` column sounds like a fixed benchmark, but it turned out to be exactly `Annual_yield / Plot_size` (correlation of 1.0, checked in [`notebooks/02_exploratory_analysis.ipynb`](notebooks/02_exploratory_analysis.ipynb)). That makes it the yield per hectare, which does not depend on farm size, so it is the correct target.

If I had built the model on `Annual_yield`, it would mostly have been predicting farm size.

---

## Methods

| Step | What I did | Files |
|---|---|---|
| **Cleaning** | Fixed the swapped columns, corrected crop name typos, joined the 4 tables, extracted values from 10 different weather message formats (English and Chinese) using regex, and checked the results against the clean data | [`notebooks/01_data_cleaning.ipynb`](notebooks/01_data_cleaning.ipynb) |
| **Exploratory analysis** | Identified the correct target, compared yield by crop, province, and soil type, and looked at how environmental factors relate to yield | [`notebooks/02_exploratory_analysis.ipynb`](notebooks/02_exploratory_analysis.ipynb) |
| **Machine learning** | Trained 4 regression models (Linear, Ridge, Random Forest, Gradient Boosting) to predict yield per hectare | [`notebooks/03_machine_learning.ipynb`](notebooks/03_machine_learning.ipynb) |
| **Dashboard** | An interactive HTML dashboard | [`dashboard/farm_yield_dashboard.html`](dashboard/farm_yield_dashboard.html) |

---

## Results

- **Tea has the highest yield per hectare (0.645), while banana and coffee have the lowest (about 0.46).**
- **Sokoto has the highest average yield per hectare (0.588).** It is also the province with the lowest water access in my [water access analytics](https://github.com/Valerie-komen/Maji-Ndogo-Water-Access-Analytics) project, but these are two different datasets, so I do not claim any link between them.
- **Volcanic and loamy soils produce higher yields than sandy, rocky, and peaty soils.**
- **Pollution level (r = -0.29) and pH (r = -0.20) have the strongest individual relationship with yield.** Rainfall and temperature on their own show almost no linear relationship.
- **Gradient Boosting predicts yield per hectare with a test R² of 0.963 and a cross-validated R² of 0.957 (±0.003).** This is a strong and stable result, but it is unusually clean for real agricultural data. It is more likely a sign that this is a course-generated dataset than something you would expect from real field data. I have noted this in the notebook.

![Farm Yield Dashboard](images/dashboard.png)

---

## Machine Learning

Target: `Standard_yield` (yield per hectare, see the data issues above for why). Features: geographic (elevation, latitude and longitude, slope, province), weather (rainfall, temperature), soil (fertility, type, pH), pollution level, and crop type. I left `Plot_size` and `Annual_yield` out of the features because the target is calculated from them.

| Model | Test R² | CV R² (5-fold) | Test RMSE |
|---|---|---|---|
| Linear Regression | 0.655 | 0.628 | 0.068 |
| Ridge Regression | 0.655 | 0.628 | 0.068 |
| Random Forest | 0.914 | 0.915 | 0.034 |
| **Gradient Boosting** | **0.963** | **0.957** | **0.022** |

The tree-based models performed much better than the linear ones, which shows that the relationship between these features and yield is not a straight line. The most important features (rainfall, tea as a crop, latitude, pH, elevation, and pollution level) make sense from a farming point of view and stayed consistent across models.

![Predicted vs Actual](images/predicted_vs_actual.png)
![Feature Importance](images/feature_importance.png)

---

## Extracting data from weather station messages

`Weather_station_data.csv` contains free-text sensor messages written in a mix of English and Chinese formats. My first regex pattern only matched 40% of the messages. I went through the unmatched messages and found there were **10 different formats**, not 4: three measurements (temperature, rainfall, and pollution), each written in three or four ways. After updating the patterns, all messages were matched. I then checked the extracted rainfall values against the clean field data, and they were very close (for example, station 2 had 1,691mm from the messages and 1,706mm from the clean data).

---

## Installation

```bash
git clone https://github.com/Valerie-komen/Maji-Ndogo-Farm-Yield-Prediction.git
cd Maji-Ndogo-Farm-Yield-Prediction
pip install -r requirements.txt
```

## Reproducibility

1. **Data cleaning:** [`notebooks/01_data_cleaning.ipynb`](notebooks/01_data_cleaning.ipynb) → `data/cleaned_farm_data.csv`.
2. **Exploratory analysis:** [`notebooks/02_exploratory_analysis.ipynb`](notebooks/02_exploratory_analysis.ipynb).
3. **Machine learning:** [`notebooks/03_machine_learning.ipynb`](notebooks/03_machine_learning.ipynb).
4. **Dashboard:** open [`dashboard/farm_yield_dashboard.html`](dashboard/farm_yield_dashboard.html) in a browser.

**Note on raw data:** the source `.db` file and the weather station `.zip` are ALX coursework materials, so I have not included them here. `data/cleaned_farm_data.csv` is the cleaned, analysis-ready output.

All notebook outputs in this repository are from real runs of the code.

---

## Repository Structure

```
Maji-Ndogo-Farm-Yield-Prediction/
├── README.md
├── requirements.txt
├── data/
│   ├── cleaned_farm_data.csv
│   └── parsed_weather_messages.csv
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_exploratory_analysis.ipynb
│   └── 03_machine_learning.ipynb
├── dashboard/
│   └── farm_yield_dashboard.html
└── images/
```

---

## Limitations

- There is no data on farming practices, seed variety, or irrigation, which probably matter more than the factors available here. Even with a high R², the model should not be read as a full explanation of what drives yield.
- The very high R² is most likely because this is a course-generated dataset rather than real field data, so it should not be taken as a realistic accuracy for agricultural models in practice.
- The weather station messages only cover 5 stations linked to specific fields, so not every field has a matching station message.

---

*If you have questions or feedback, feel free to open an issue or connect with me on [LinkedIn](https://www.linkedin.com/in/valerie-komen-468a93355/).*
