# **Data Analysis & Modeling Report**

## **1. Column Descriptions**

- **sessionid**: Unique identifier for a game session; may repeat when a session spans multiple days or when duplicate logs exist.  
- **playerid**: Unique player identifier.  
- **serverdate_first / serverdate_last**: Server-side timestamps marking the start and end of the logged session; mostly same‑day, with a few multi‑day anomalies.  
- **clientdate_first / clientdate_last**: Client-side timestamps for the session; used as the reliable reference for session uptime.  
- **dateid**: Integer representation of the calendar date derived from server timestamps.  
- **countrycode**: ISO 3166‑1 alpha‑2 country code for the player.  
- **gameversion**: Game version identifier; constant across the dataset and removed from modeling.  
- **playersessions_index**: Sequential index of each player’s sessions ordered by time.  
- **playdays_index**: Index of records belonging to the same multi-record session.  
- **sessionuptime_duration**: Duration between clientdate_first and clientdate_last (seconds).  
- **cum_playtime_duration**: Cumulative playtime; inconsistent relative to session_playtime_duration and excluded from modeling.  
- **session_playtime_duration**: Actual playtime during the session (seconds).  
- **population**: Binary categorical variable indicating two experimental groups (A/B test).

---

## **2. Data Quality Review & Cleaning**

### **Detected Anomalies and Resolutions**

#### **a. Client–Server Timestamp Inconsistencies**
- ~359 records show >1‑week discrepancy between client and server timestamps, likely due to device clock issues or delayed synchronization.  
- Retained, as the inconsistencies do not systematically distort session-level modeling.

#### **b. Multi‑day Server Timestamps**
- 16 records show serverdate_first and serverdate_last differing by up to 6 weeks despite short session durations (<10 hours).  
- Treated as logging errors; retained due to low volume and minimal impact on target definition.

#### **c. Invalid Country Codes & Negative Playtime**
- 307 records with countrycode = `"RU5514"` and session_playtime_duration = -1.  
- Removed due to invalid categorical value and clearly erroneous playtime.

#### **d. Missing Country Codes**
- A few isolated records with missing countrycode belonging to players with no other sessions.  
- Dropped due to lack of contextual information and no impact on other player histories.

#### **e. Zero Uptime but Positive Playtime**
- Some records show sessionuptime_duration = 0 while playtime > 0.  
- Recomputed uptime using client timestamps.

#### **f. Duplicate Session Records**
- Multiple records occasionally represent the same session (same playerid and clientdate_first).  
- Consolidated by grouping and aggregating using max/min/mode depending on field semantics.

---

## **3. Exploratory Insights**

### **a. Seasonality of Playtime**
![total_playtime_distribution](total_playtime_distribution.png")
Daily total playtime shows noticeable fluctuations, including increases in early May and early July.  
These may correspond to internal events (updates, promotions) or external factors (holidays, school breaks).  
Both A/B populations follow similar trends, though Population 1 shows higher engagement in early June—worth further statistical investigation.

### **b. Geographic Distribution**
![player distribution across countries](player_distribution_across_countries.png")
The largest absolute player counts come from the US and China.  
However, when normalized by country population, several European countries, Canada, and Australia show higher per‑capita engagement.  
This pattern may reflect regional marketing effectiveness or stronger franchise affinity.

---

## **4. Data Preprocessing for Modeling**

### **Feature Engineering**
- **time_since_joined**: Player tenure at each session (seconds).  
- **play_ratio**: session_playtime_duration / sessionuptime_duration.  
- **gap**: Time since previous session for each player.  
- **Rolling & Expanding Stats**: Player‑level mean and standard deviation of gap and play_ratio (expanding = historical; rolling = last 3 sessions).

### **Target Definition**
- **last_session** = True when the gap between a session and the player’s maximum observed date exceeds a player‑specific statistical threshold.  
- This approach avoids leakage and approximates churn using only historical information.

---

## **5. Statistical Analysis & Feature Validation**

### **Correlation & Association Tests**
- **playersessions_index** (r = -0.113) and **time_since_joined** (r = -0.099) negatively correlate with churn → experienced players churn less.  
- Chi‑square tests show strong associations for **countrycode** (χ² = 780.56) and **population** (χ² = 80.35), indicating demographic/geographic influence.

### **Feature Importance**
- Logistic regression highlights countrycode and session‑level metrics as key predictors.

### **Class Imbalance**
- Only **7.2%** of sessions are labeled as last_session → required class weighting and careful evaluation.

### **Train/Validation/Test Split**
- Split by **unique players** (60/20/20) to avoid leakage across sessions.  
- Class distribution preserved across splits.

---

## **6. Modeling Approach & Results**

### **Preprocessing**
- Encoded categorical variables.  
- Imputed missing values.  
- Ensured consistent feature sets across splits.

### **Benchmark Model: Logistic Regression**
- Class‑weighted logistic regression.  
- **Validation AUC:** 0.652  
- **Validation F1:** 0.186  

### **Comparison Model: Random Forest**
- Tuned for class imbalance.  
- **Validation AUC:** 0.687  
- **Validation F1:** 0.016  
- Top features include playersessions_index and session-level metrics.  
- Low F1 reflects difficulty predicting the minority class.

### **Interpretation**
- Both models outperform naive baselines.  
- Predictive power is limited by data sparsity, timestamp inconsistencies, and lack of richer behavioral features.  
- Tenure, session frequency, and geography emerge as meaningful churn indicators.

---

## **7. Additional Features That Would Be Useful**

To improve churn prediction, the following would be valuable:

- **In‑game behavior metrics and engagement**: actions per minute, match outcomes, progression, difficulty level, event participation.  
- **Social features**: friend interactions, party play, clan/guild membership.  
- **Economic data**: purchases, currency balances, spending patterns.
