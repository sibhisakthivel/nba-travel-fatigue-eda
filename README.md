# NBA Travel and Fatigue Impact Analysis: Quantifying Schedule Density, Rest, and Performance Effects

## **[▶View Interactive EDA Report](https://sibhisakthivel.github.io/nba-travel-fatigue-eda/)**

## Overview

Modern NBA teams face demanding travel schedules, short rest periods, and dense game clusters that can significantly affect performance and injury risk. This project investigates how travel, rest, and schedule compression influence team outcomes across the NBA, using real schedule and performance data spanning multiple seasons.

The analysis aims to answer key questions such as:

- How frequently do teams experience high-density stretches like 4 games in 6 nights?

- How have back-to-backs, travel mileage, and time-zone changes evolved over the past decade?

- To what extent can schedule-related factors explain variations in team success?

To explore these questions, this project combines data engineering, exploratory data analysis, and predictive modeling within a reproducible Python workflow. Specifically, it:

1) Processes official NBA schedule and location data to quantify game density, rest days, and travel distance for each team and season.

2) Analyzes league-wide scheduling trends over time using custom-built metrics and interactive visualizations.

3) Builds dynamic team-level schedule visualizers that highlight periods of excessive travel or fatigue risk.

4) Applies logistic regression modeling to estimate the number of wins each team gained or lost due to schedule-driven advantages between 2019–2024.

Overall, this project serves as a comprehensive case study in sports analytics and applied data science, demonstrating how data can be leveraged to uncover hidden performance dynamics and evaluate fairness in league scheduling.

## Data Sources  

This project uses publicly available NBA datasets and internally derived features to quantify travel and fatigue metrics:  

| Dataset | Description | Key Columns |
|----------|--------------|--------------|
| **`schedule.csv`** | Historical NBA regular season schedules (2014–2024) used to compute season-level trends and schedule density statistics. | `season`, `team`, `gamedate`, `opponent`, `home` |
| **`schedule_24_partial.csv`** | Draft 2024–25 schedules for the **Oklahoma City Thunder (OKC)** and **Denver Nuggets (DEN)** — used for visualizing schedule structure and identifying fatigue-heavy stretches. | `team`, `gamedate`, `home`, `opponent` |
| **`locations.csv`** | Latitude and longitude coordinates of each NBA team’s home arena, used to calculate travel distance and time-zone crossings. | `team`, `latitude`, `longitude` |
| **`team_game_data.csv`** | Team-level box score data providing performance metrics such as **field goals attempted/made** and **three-point attempts/makes**, used for defensive efficiency calculations. | `season`, `gamedate`, `def_team`, `fgmade`, `fg3made`, `fgattempted` |

Additional derived tables and feature sets were generated during analysis (e.g., rolling travel mileage, back-to-back flags, and rest-day advantages) using these base datasets.

## Data Cleaning and Preprocessing  

Before analysis, all datasets were validated, standardized, and merged into a consistent season–team–date format.  
The preprocessing workflow included the following key steps:

- **File validation and type conversion**  
  - Ensured all `gamedate` columns were converted to `datetime` objects.  
  - Checked required columns (`team`, `season`, `gamedate`) before processing.

- **Handling missing or inconsistent records**  
  - Removed duplicate games and invalid rows.  
  - Imputed missing travel locations using known home-arena coordinates.

- **Standardizing schedule data**  
  - Sorted all schedules chronologically per team.  
  - Added rolling windows to compute metrics like *4-in-6* and *back-to-back* frequency.

- **Merging location and performance data**  
  - Joined `locations.csv` to schedule data to calculate travel distance via the Haversine formula.  
  - Integrated `team_game_data.csv` for advanced performance metrics such as defensive eFG%.

- **Feature extraction**  
  - Generated derived columns such as `rest_days`, `b2b_flag`, `rolling_travel_miles`, and time-zone change indicators.

This preprocessing pipeline produced a unified dataset ready for league-wide trend analysis, schedule visualization, and logistic regression modeling.

## Part I — Schedule Density Analysis  

The first stage of analysis focused on quantifying **schedule compression** across NBA seasons — specifically identifying how often teams face condensed stretches such as *4 games in 6 nights* or *5 in 7*. These high-density periods are a practical measure of potential fatigue and serve as the foundation for later performance modeling.  

### Methodology  
- Implemented the function **`count_x_in_y_stretches()`**, which calculates the number of games representing the *x-th game played in the past y days* for any team and season.  
- Applied this method to historical schedules from **2014–2024** to compute:  
  - **Team-specific counts** (e.g., OKC 2024–25 draft schedule)  
  - **Season-normalized averages** (per 82 games)  
  - **League-wide distributions** of 4-in-6 and 5-in-7 stretches  
- Included input validation, date handling, and normalization to ensure comparability between seasons with varying numbers of games.  

### Results and Findings  
- The **Oklahoma City Thunder** were scheduled for **26 4-in-6 stretches** in the 2024–25 draft schedule.  
- Across the **2014–2024** dataset, teams averaged approximately **25 such stretches per season** (normalized to 82 games).  
- Historical extremes:  
  - **Most compressed** — Charlotte Hornets (≈ 28 per season)  
  - **Least compressed** — New York Knicks (≈ 22 per season)  
- Variability of ±3–4 stretches around the mean is consistent with normal scheduling randomness rather than systematic bias.  

### Interpretation  
While small differences in density exist among teams, the overall distribution suggests **no major structural imbalance** in league scheduling.  
The frequency of these intense windows provides a quantitative baseline for evaluating fatigue exposure, later integrated into defensive-efficiency and win-probability modeling.

## Part II — Defensive Efficiency Case Study  

To explore how schedule-related fatigue may influence on-court outcomes, this section examines the relationship between **opponent rest** and **defensive performance**. Specifically, we measure how a team’s defensive efficiency changes when facing opponents on the second night of a back-to-back.  

### Methodology  
- Implemented the helper function **`defensive_efg()`**, which calculates **Defensive Effective Field Goal Percentage (eFG%)** for a given team and season.  
- The function computes:  
  \[
  eFG\% = \frac{(FGM + 0.5 \times 3PM)}{FGA}
  \]
  where all values represent opponent shooting against the defensive team.  
- Two calculations were performed for each team-season:  
  1. Overall defensive eFG% across all games.  
  2. Defensive eFG% **only when opponents were on the second night of a back-to-back**.  
- The function includes error handling, flexible column detection (`off_team` or `off_team_name`), and supports both full-season and conditional filters.  

### Results and Findings  
- Across multiple teams and seasons, **defensive eFG% tends to improve slightly when opponents are on the second night of a back-to-back**, typically by **0.5–1.0 percentage points**.  
- The effect magnitude varies by season and team, but the trend is consistent: **rest-disadvantaged opponents shoot less efficiently**.  
- This result aligns with the hypothesis that **fatigue impairs offensive execution**, especially in games with limited recovery time or travel demands.  

### Interpretation  
This analysis provides an early link between **schedule fatigue and performance outcomes**.  
It demonstrates that short-term rest disadvantages—such as back-to-backs—can measurably affect opponent efficiency.  
These findings reinforce the need to model fatigue-related variables explicitly when estimating win probabilities in later sections.

## Part III — League-Wide Scheduling Trends  

This section analyzes how **NBA scheduling structure has evolved over the past decade**, focusing on travel intensity, rest frequency, and game density.  
By aggregating schedule-level metrics across all teams and seasons, we can visualize league-wide efforts to reduce fatigue and balance competitive fairness.  

### Methodology  
- Created a function **`compute_league_metrics_per_season()`** to calculate per-season league averages for:  
  - **Average 4-in-6 stretch count** (schedule density)  
  - **Average back-to-back count** (short-rest frequency)  
  - **Average rolling 5-game travel distance** (travel burden)  
- Each metric was aggregated across all 30 teams per season (2014–2024).  
- Combined these statistics into an interactive **Plotly dual-axis visualization**, showing multi-year trends in game density, travel, and rest.  

### Results and Findings  
- **Schedule compression has steadily declined** since 2014, with notable reductions in back-to-back games and 4-in-6 stretches.  
- **Travel distance fluctuated** annually but dropped significantly in 2020 due to COVID-related scheduling adjustments before stabilizing in subsequent seasons.  
- Recent years show a **slight rebound** in travel and density, suggesting a balance between reducing fatigue and maintaining scheduling flexibility.  

### Interpretation  
League-wide scheduling reforms appear to have achieved their goal of **reducing player fatigue and improving rest equity**.  
The clear downward trend in schedule density underscores the NBA’s data-driven approach to optimizing season logistics.  
These findings provide important historical context for the subsequent team-level visualization and win-probability modeling stages.

## Part IV — Team-Level Schedule Visualization Tools  

To analyze how travel and rest impact specific teams, this section introduces a suite of **interactive visualization functions** for inspecting season schedules.  
These tools combine spatial, temporal, and fatigue-related data into a single interpretable view that highlights high-density travel stretches and potential fatigue risk zones.  

### Methodology  
Developed several reusable Python utilities to quantify and visualize schedule-level fatigue:  

- **`rolling_travel_distance()`** – Calculates cumulative and rolling travel mileage using the Haversine formula between consecutive game locations.  
- **`rolling_rest_days()`** – Computes rest gaps between games, flags back-to-backs, and builds rolling rest-day windows.  
- **`flag_n_in_m_fullwindow()`** – Detects dense stretches such as 3-in-5, 4-in-6, or 5-in-7 game clusters within a moving calendar window.  
- **`plotly_schedule_with_home_strip()`** – Produces an interactive Plotly chart combining:  
  - Shaded schedule-density bands (e.g., 4-in-6, 5-in-7, B2B)  
  - Rolling 7-day travel line (miles)  
  - Time-zone change markers  
  - Home/Away color strip for full-season visualization  

### Results and Applications  
- Applied the tool to the **Oklahoma City Thunder (OKC)** and **Denver Nuggets (DEN)** 2024–25 draft schedules.  
- The resulting visuals clearly distinguish home stretches, long road trips, and periods of overlapping fatigue risk.  
- Each team’s chart provides a season-long snapshot of **travel burden, rest distribution, and schedule irregularities** at a glance.  

### Interpretation  
This visualization framework enables analysts and coaching staff to **quickly identify critical travel stretches and rest bottlenecks**.  
By unifying schedule density, geography, and rest dynamics, it offers a powerful diagnostic layer that supports both **performance monitoring** and **logistical decision-making**.

## Part V — Schedule-Adjusted Win Modeling  

Building on the descriptive analyses, this stage applies **predictive modeling** to estimate how much a team’s win total is influenced by schedule-related factors such as travel, rest, and game density.  
The goal is to isolate the portion of team performance attributable to schedule structure rather than on-court strength.

### Methodology  
- Constructed a logistic regression model to predict game outcomes (win/loss) across **2019–2024** seasons.  
- The model incorporated both **team strength** and **schedule-driven** variables:  
  - *Team & opponent performance metrics:* rolling and season-to-date net rating, offensive/defensive rating, win percentage.  
  - *Schedule context:* rest days, back-to-back and 4-in-6 indicators, rolling 7-day travel distance, time-zone crossings, and home/away status.  
- Implemented a **counterfactual “neutralization” procedure**:  
  - Each game was predicted twice — once with actual schedule features, and once with neutralized (season-median) schedule conditions.  
  - The difference (ΔP) represents the change in win probability due solely to schedule factors.  
- Aggregated ΔP across all games to estimate **“schedule-adjusted wins”** per team.  
- Evaluated the model using leave-one-season-out (LOSO) validation with metrics:  
  - **Brier = 0.225**, **LogLoss = 0.644**, **AUC = 0.683**, indicating solid calibration and discriminatory power.  

### Results and Findings  
- Teams typically gained or lost between **±1–2 wins per season** due to schedule advantages or disadvantages.  
- **Positive schedule effects** were associated with greater rest, minimal travel, and fewer compressed stretches.  
- **Negative effects** corresponded to extended road trips, multiple time-zone shifts, and dense game clusters.  
- League-centered estimates (mean-zeroed across all teams):  
  - *Most helped:* Teams with more balanced rest/travel schedules gained roughly **+1.4 wins**.  
  - *Most hurt:* Teams facing heavy travel and tight turnarounds lost about **–1.5 wins**.  

### Interpretation  
This modeling approach quantifies the tangible performance impact of schedule design.  
By simulating a neutral schedule baseline, we isolate how fatigue exposure translates to measurable changes in win probability.  
The results confirm that even small differences in travel and rest accumulation can subtly but consistently influence season outcomes — a key insight for both **sports analytics** and **schedule optimization**.

## Key Insights and Future Implementations  

### Key Insights  
This project demonstrates how data-driven analysis can reveal subtle but measurable effects of **schedule design, travel, and rest** on NBA team performance.  
By combining descriptive statistics, visualization, and predictive modeling, several clear themes emerge:  

- **Schedule compression directly affects performance consistency.**  
  High-density stretches (4-in-6, 5-in-7) increase fatigue exposure and correlate with reduced defensive efficiency.  

- **League scheduling reforms have reduced player fatigue.**  
  Back-to-back frequency and 4-in-6 occurrences have steadily declined since 2014, reflecting a shift toward rest-optimized scheduling.  

- **Fatigue has quantifiable performance costs.**  
  Even marginal differences in rest and travel can influence win probability by 1–2 games per season — often enough to affect playoff seeding.  

### Future Implementations  
To extend this study and enhance practical applications, several directions are planned:  

- **Automated Data Pipeline**  
  - Develop a fully automated ETL workflow that ingests new schedule data, computes fatigue metrics, and updates analyses in real time.  

- **Player-Level Integration**  
  - Incorporate player box-score and tracking data to model individual fatigue responses and recovery effects.  

- **Predictive Simulation**  
  - Couple this framework with the *NBA Player Performance Prediction Model* to generate fatigue-adjusted player and team forecasts.  

- **Interactive Dashboard**  
  - Deploy a Streamlit or Plotly-Dash web interface that visualizes schedule density, travel mileage, and predicted fatigue risk for any team or date range.  

---  

This project serves as a comprehensive example of **end-to-end sports analytics**, uniting engineering, visualization, and modeling to quantify the hidden impact of travel and fatigue in professional basketball.

---

## Author  
**Sibhi Sakthivel**  
M.S. Molecular Science & Software Engineering, UC Berkeley  
sibhisak@gmail.com
sibhisakthivel@berkeley.edu
[LinkedIn](https://www.linkedin.com/in/sibhi-sakthivel-3ab23113b)
[GitHub](https://github.com/sibhisakthivel)
