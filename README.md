# Noob2Pro CF



This project is an analytical tool which fetches data directly from the codeforces api, analyses the data and provides some useful insights. Then, it recommends topicwise questions using scipy.stats.linregress. 

## Features

- **Live Codeforces API integration**: user's submission history, rating history, and profile info
- **Topic-wise performance stats**: acceptance rate, average/max/median rating, unique problems solved, WA/TLE/CE counts.
- **Strong / Average / Weak topic classification**: based on acceptance rate thresholds (>=70%: strong, <50%: weak).
- **Forgotten topic alerts**: warning after 14 days; marks as forgotten after 30 days.
- **Rating trend**
- **Per-topic regression-based recommendations**: uses `scipy.stats.linregress` on your solved-problem ratings to predict the next difficulty level, then fetches matching problems from Codeforces and displays them as links on the table.
- **Priority scoring**: ranks topics so you can focus on weakest ones.
- **Multi-user support**
- **Dashboard**: 7 matplotlib charts rendered in one dashboard.


> **Note:** The last 500 submissions are fetched per user. Users with fewer contests may have less reliable trend data (at least 3 contests are required for a rating trend to be computed).

## Project Structure
```
.
├── python_project.ipynb       # Main notebook 
└── files/                     # Auto-created on first run to hold CSV exports
    ├── cf_user_info_<handle>.csv
    ├── cf_submissions_<handle>.csv
    ├── cf_rating_history_<handle>.csv
    └── cf_recs_<handle>.csv
```


## Motivation 
The codeforces platform does not tell you which topics you are weak in, which problems you should solve next. It is the basic human nature that you like to do the things you are good at. So, when a person practices questions on codeforces, he/she is more likely to keep practicing the strong topics and therefore neglecting the weak ones. 
 
The key questions a serious competitor should be asking are:
- Which topics have low acceptance rates in my submissions and need more practice?
- Which topics have I completely neglected recently?
- Given my solved-problem history in a topic, what difficulty should I target next?
- Is my contest rating trending upward or stagnating?

## Problem Definition 
Given a Codeforces handle, automatically fetch the user's submission history and contest rating history, analyse performance across topics, identify weak and forgotten areas, and recommend the optimal next problems to solve (difficulty calibrated by a linear regression model trained on the user's own solved-problem ratings.)

## Relevance and problem complexity 
Random practice leads to slow improvement. This tool provides personalized guidance.

Dynamic target setting: the recommended difficulty must adapt to the user's current level in each topic independently, not just their overall rating.

| Layer | Complexity |
|---|---|
| Data ingestion | Live REST API with retry logic, persistent HTTP sessions, rate-aware error handling |
| Feature engineering | 12+ derived features per topic from raw submission records |
| Topic modelling | Many-to-one tag mapping with grouping logic |
| Recommendation engine | uses per-topic OLS regression using scipy.stats to get the target rating|
| Visualisation | 7-chart matplotlib dashboard with normalised heatmap |
| Output | Table with clickable problem links + CSV exports |


## User guidelines
### Install Dependencies
```bash
pip install numpy pandas scipy matplotlib requests ipython
```
Run all cells and enter the users codeforces handles separated by commas.
For each handle the notebook will fetch and save raw data to `files/` (cf_user_info_handle.csv, cf_submissions_handle.csv,cf_rating_history_handle.csv,  cf_recs_handle.csv)


## Readability and ease of navigation 
The code structure is clearly described by the markdown cells and comments. Charts are self explanatory and have legends wherever required. 

## Generalizability & Extensibility

### Class-Based Architecture

1. `CFClient`: API communication, ease to add new methods for any Codeforces endpoint (e.g., fetch friends) 
2. `UserSession`: Per-user state container, ease to add new fields (e.g., peer comparison, streak data)
3. `CPAnalyserEngine`: Ease to add new pipeline stages (e.g., generate study plan) as methods

### Swappable Components

- **Recommendation model**: get_topic_regression function uses scipy.stats.linregress (OLS). This can be swapped for polynomial regression, ridge regression, or any sklearn estimator with a one-function change.
- **Tag mapping**: CF_TAG_MAP and TOPIC_TO_CF_TAG are plain dictionaries. New tags or category merges require only dictionary edits, no logic changes.
- **Chart functions**: each of the 7 chart functions is independent. New charts can be added to render_dashboard without modifying any existing function.

### Multi-User Support

UserSession contains data for a single user. The tool analyses multiple users sequentially with zero state leakage between them.


## Topics covered in course

### Python and OOPS
- **Classes and objects**:  CFClient, UserSession, CPAnalyserEngine
- **Modular functions**: the project is decomposed into small functions.
- **Data Structures**: lists, dictionaries, sets
- **Lambda functions**: used inside groupby().agg() for custom aggregation (e.g., counting OK verdicts).
- **Exception handling**: try/except in CFClient.cf_get with retry logic.


### NumPy
- `np.arange`: index arrays for regression x-axis.
- `np.clip`: constrains predicted ratings to [800, 3500].
- `np.std`: standard deviation of rating history.
- `np.where`: severity classification in detect_forgotten_topics.
- `np.linalg.norm`: L2-norm for calculating priority score.
- `np.array`: conversion of pandas Series to arrays for regression.

### Pandas
- `pd.DataFrame`: dataframe for submissions, stats, and recommendations.
- `groupby().agg()`: multi-column aggregation for topic stats.
- `merge`: joining accepted-problem stats back onto the base stats table.
- `fillna`, `round`
- `assign`, `dt.to_period`: datetime feature engineering
- `to_csv`: exporting data
- `to_html`: rendering clickable hyperlinks in the recommendations table.
- `json_normalize`: flattening nested API JSON into tabular form.

### A glance from test run:
![alt text](image-1.png)

### SciPy
- **`scipy.stats.linregress`**: 
  - `rating_trend`: OLS over contest rating history (slope, intercept, R²).
  - `get_topic_regression`: OLS over per-topic accepted problem ratings to predict target difficulty.


### Matplotlib
Seven distinct chart types using:

- `ax.barh`,`ax.bar` for bar graphs.
- `ax.axvline` to signify thresholds.
- `ax.pie` with `autopct` for pie charts.
- `ax.plot` 
- `ax.imshow` with `RdYlGn` colormap, `plt.colorbar` for heatmap
![alt text](image.png)

### File I/O
- `os.makedirs`: directory creation.
- `pd.DataFrame.to_csv`: writing structured output
- `pd.json_normalize`: converting nested JSON to flat CSV

---

