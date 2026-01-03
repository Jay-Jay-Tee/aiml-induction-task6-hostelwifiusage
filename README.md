# 📶 Hostel Wi-Fi Performance Analysis & Prediction  
(Exploratory Data Analysis + Feature Engineering + Regression)

**GDSC AI/ML Inductions - Intermediate Task 2**

![task](https://img.shields.io/badge/Task-Intermediate%202-blue)
![type](https://img.shields.io/badge/Type-Regression-brightgreen)
![model](https://img.shields.io/badge/Model-Random%20Forest-orange)
![metric](https://img.shields.io/badge/R²-0.7594-success)

---

## 👑 Problem Statement & Motivation

Hostel Wi-Fi performance often feels unpredictable: fast at some times and painfully slow at others. However, raw internet logs do not directly record Wi-Fi speed or latency. Instead, they capture session-level behavior such as start time, duration, upload/download volumes, and termination reasons.

The goal of this project is to:
- Understand usage patterns that lead to Wi-Fi congestion  
- Identify peak pain hours and heavy usage behavior  
- Predict Wi-Fi performance at a given time using historical session data  

---

## ✨ Dataset Understanding & Cleaning Decisions

Each row in the dataset corresponds to one internet session.  
Key columns include:

- start_time: when the session began  
- usage_time: duration of the session  
- upload, download, total_transfer: data usage  
- seession_break_reason: how the session ended  

### Missing Values

Only 9 missing values were found, all in seession_break_reason.

**Decision:**  
These were filled with "Unknown" instead of dropping rows.

**Why?**
- Dropping rows would discard valid session behavior  
- Missing break reason does not imply missing usage data  
- Treating it as "Unknown" avoids incorrect assumptions  

---

## 🌟 Defining Wi-Fi Performance (Target Variable)

### Problem
There is no explicit Wi-Fi speed or latency column.

### Solution
A proxy metric was defined:

    transfer_rate = total_transfer / usage_seconds

### Why?
- Measures how much data was successfully transferred per second  
- Lower values typically indicate congestion or throttling  
- Higher values indicate smoother network conditions  

This proxy is commonly used when direct throughput measurements are unavailable.

---

## ✌️ Exploratory Data Analysis (EDA): What the Data Revealed

EDA was performed to understand behavior before modeling.

### EDA 1: Total Data Transfer by Hour (Peak Usage)

**Finding:**
- Clear peaks during evening hours (≈ 18–22)  
- Lowest usage during early morning (≈ 3–6)  

**Interpretation:**  
Wi-Fi load strongly follows student routines such as streaming, gaming, and assignment work.

---

### EDA 2: Distribution of Session Transfer Rate (Session Intensity)

**Finding:**
- Distribution is heavily right-skewed  
- Most sessions are low intensity  
- A small number of sessions have extremely high transfer rates  

**Interpretation:**  
Congestion is driven by a small fraction of very heavy sessions.

---

### EDA 3: Data-Hungry Users

**Finding:**  
A small subset of users accounts for a disproportionately large share of total transfer.

**Interpretation:**  
Congestion is driven by concentrated behavior, not uniform usage.

---

### EDA 4: Session Break Reasons

**Finding:**
- Idle-Timeout overwhelmingly dominates  
- Other reasons are rare  

**Interpretation:**  
Many devices remain connected while idle, motivating its use as a behavioral feature.

---

### EDA 5: Actual Transfer Rate by Hour

**Finding:**
- Average transfer rate drops during peak usage hours  
- Best performance occurs late night / early morning  

**Interpretation:**  
High usage directly correlates with worse user experience, validating the proxy target.

---

## 🎯 Feature Engineering: Why Each Feature Exists

Raw columns do not directly represent learnable patterns.

### Time-Based Features
- hour, dayofweek, is_weekend  

**Why:**  
Wi-Fi usage follows daily and weekly cycles.

---

### Cyclic Encoding
- hour_sin, hour_cos  

**Why:**  
Time is cyclic (23:00 and 00:00 are adjacent).  
Numerical encoding alone breaks this continuity.

---

### Session Intensity Features
- log_total_transfer  
- log_usage_seconds  

**Why logs are necessary:**
- Raw values are extremely skewed  
- A few sessions dominate scale  
- Logs compress magnitude while preserving order  

---

### Behavioral Ratios
- upload_ratio  
- download_ratio  

**Why:**  
The same total transfer can represent very different behaviors:
- Upload-heavy → assignments  
- Download-heavy → streaming  

---

### Stability Indicator
- idle_timeout (binary)  

**Why only Idle-Timeout:**  
It dominates the dataset; other categories are too rare and noisy.

---

## 🚀 Why Random Forest Regressor Was Used

This problem involves:
- Non-linear relationships  
- Mixed feature types  
- Heavy-tailed distributions  
- Noise and outliers  

**Why Random Forest fits well:**
- Captures non-linear thresholds (e.g., evening spikes)  
- No feature scaling required  
- Robust to extreme values  
- Strong performance on tabular data  
- Provides feature importance  

---

## 🌠 Why Time-Based Train/Test Split Was Used

Instead of random splitting, data was split chronologically.

**Why this matters:**
- Random splitting allows training on future sessions  
- Causes temporal leakage  
- Inflates performance unrealistically  

**Time-based split ensures:**
- Train = past  
- Test = future  
- Realistic evaluation  

---

## 🎖️ Model Results & Interpretation

**Metrics:**
- MAE = 508.21  
- R² = 0.7594  

These values are strong given real-world noise.

Errors arise due to:
- Rare spikes  
- Unpredictable behavior  
- Infrastructure variability  

---

## 🔔 Predicted vs Actual Behavior

- Predictions follow overall trends well  
- Extreme spikes are smoothed (expected)  
- Worst predicted hours align with actual worst hours  

This confirms the model learned real congestion patterns, not noise.

---

## 🎊 Final Conclusion

This project shows that:
- Wi-Fi congestion is behavior-driven and time-dependent  
- A small fraction of sessions dominates network stress  
- Feature engineering is critical for meaningful learning  
- Random Forest regression reliably predicts congestion windows  
- Time-aware evaluation is essential for honest performance  

---

## 🎖️ Practical Recommendation and Final Result

**Best time for downloads or streaming:**  
Late night to early morning (≈ 1–6 AM)

---

## ******** NOTE ********

This project does not directly measure Wi-Fi latency or instantaneous speed. Instead, Wi-Fi performance is inferred using a session-level proxy:

    transfer_rate = total_transfer / usage_seconds

Low transfer rate does not always imply poor Wi-Fi performance.

- During off-peak hours, low transfer rate reflects idle sessions  
- During peak hours, low transfer rate indicates congestion  

The regression model predicts expected session throughput given time and usage context, enabling identification of congestion windows rather than direct lag measurement.
