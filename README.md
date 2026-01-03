📶 Hostel Wi-Fi Performance Analysis & Prediction
(Exploratory Data Analysis + Feature Engineering + Regression)

👑 Problem Statement & Motivation

Hostel Wi-Fi performance often feels unpredictable: fast at some times and painfully slow at others. However, raw internet logs do not directly record Wi-Fi speed or latency. Instead, they capture session-level behavior such as start time, duration, upload/download volumes, and termination reasons.

The goal of this project is to:
Understand usage patterns that lead to Wi-Fi congestion
Identify peak pain hours and heavy usage behavior
Predict Wi-Fi performance at a given time, using historical session data

✨ Dataset Understanding & Cleaning Decisions

Each row in the dataset corresponds to one internet session. Key columns include:
start_time: when the session began
usage_time: duration of the session
upload, download, total_transfer: data usage
seession_break_reason: how the session ended

Missing Values
Only 9 missing values were found, all in seession_break_reason.

Decision:
These were filled with "Unknown" instead of dropping rows.

Why?
Dropping rows would discard valid session behavior
Missing break reason does not imply missing usage data
Treating it as "Unknown" avoids incorrect assumptions

🌟 Defining Wi-Fi Performance (Target Variable)

Problem
There is no explicit Wi-Fi speed or latency column.

Solution
We define a proxy metric:

transfer_rate=total_transfer/usage_seconds

Why?
It measures how much data was successfully transferred per second
Lower values typically indicate congestion or throttling
Higher values indicate smoother network conditions

This proxy is commonly used when direct throughput measurements are unavailable.

✌️ Exploratory Data Analysis (EDA): What the Data Revealed

EDA was performed to understand behavior before modeling.

EDA 1: Total Data Transfer by Hour (Peak Usage)

Finding:
Clear peaks during evening hours (≈18–22)
Lowest usage during early morning (≈3–6)

Interpretation:
Wi-Fi load strongly follows student routines: evenings coincide with streaming, gaming, and assignment work.

This confirms that time is a critical factor.

EDA 2: Distribution of Session Transfer Rate (Session Intensity)

Finding:
Distribution is heavily right-skewed
Most sessions are low intensity
A small number of sessions have extremely high transfer rates

Interpretation:
Wi-Fi congestion is not caused by average users, but by a small fraction of very heavy sessions.

A few users cause most of the load.

EDA 3: Data-Hungry Users

Finding:
A small subset of users accounts for a disproportionately large share of total transfer

Interpretation:
Congestion is driven by concentrated behavior, not uniform usage.

This reinforces the need for models that handle outliers and skew.

EDA 4: Session Break Reasons

Finding:
Idle-Timeout overwhelmingly dominates
Other reasons (Lost-Carrier, Lost-Service, User-Request) are rare

Interpretation:
Many devices remain connected while idle.

This motivates using Idle-Timeout as a behavioral feature.

EDA 5: Actual Transfer Rate by Hour

Finding:
Average transfer rate drops during peak usage hours
Best performance occurs late night / early morning

Interpretation:
High usage directly correlates with worse user experience, validating the proxy target.

🎯 Feature Engineering: Why Each Feature Exists

Raw columns are not learnable patterns. Feature engineering transforms them into meaningful signals.

Time-Based Features
hour, dayofweek, is_weekend

Why:
Wi-Fi usage follows daily and weekly cycles.

Cyclic Encoding
hour_sin, hour_cos

Why:
Time is cyclic (23:00 and 00:00 are adjacent).
Numerical encoding alone breaks this continuity.

This prevents the model from learning incorrect distance relationships.

Session Intensity Features
log_total_transfer
log_usage_seconds

Why logs are necessary:
Raw values are extremely skewed
A few sessions are orders of magnitude larger
Logs compress scale while preserving order.

Behavioral Ratios
upload_ratio
download_ratio

Why:
Same total transfer can represent very different behaviors:

Upload-heavy -> assignments and working papers

Download-heavy -> streaming

Stability Indicator
idle_timeout (binary)

Why only Idle-Timeout:

Dominates dataset
Other categories are too rare and noisy

🚀 Why Random Forest Regressor Was Used

This problem involves:
Non-linear relationships
Mixed feature types
Heavy-tailed distributions
Noise and outliers

Why Random Forest fits perfectly:
Captures non-linear thresholds (e.g. evening spikes)
No feature scaling required
Robust to extreme values
Works well on tabular data
Provides feature importance

🌠 Why Time-Based Train/Test Split Was Used

Instead of train_test_split, data was split chronologically.

Why this matters:
Random splitting would allow the model to:
Train on future sessions
Test on past sessions

This causes temporal leakage and unrealistically high performance.

Time-based split ensures:
Train = past
Test = future

Evaluation reflects real-world deployment

🎖️ Model Results & Interpretation

Metrics:
MAE = 508.21
R²  = 0.7594

These represent good values of R² and MAE over real-world noisy deployment data.

The error is expected due to:
rare spikes
unpredictable behavior
infrastructure noise

🔔 Predicted vs Actual Behavior

Predictions follow overall trends well

Extreme spikes are smoothed (expected)

Worst predicted hours align with actual worst hours

This confirms the model learned real congestion patterns, not noise.

🎊 Final Conclusion

This project shows that:
Wi-Fi congestion is behavior-driven and time-dependent
A small fraction of sessions dominates network stress
Feature engineering is critical for meaningful learning
Random Forest regression can reliably predict congestion windows
Time-aware evaluation is essential for honest performance

🎖️ Practical Recommendation and Our Final Bonus Result

📺 Best time for downloads or streaming:
Late night to early morning (≈1–6 AM)

******** NOTE: ********

This project does not directly measure Wi-Fi latency or instantaneous speed, as such metrics are not available in the dataset. Instead, Wi-Fi performance is inferred using a session-level proxy defined as:

transfer_rate = total_transfer / usage_seconds

This quantity represents the average throughput achieved by a session and is used as an indicator of network strain under concurrent usage.

It is important to note that low transfer rate does not always imply poor Wi-Fi performance. During off-peak hours (e.g., early morning), many sessions are idle or background connections, resulting in low transfer rates despite minimal congestion. Conversely, during evening peak hours, sessions are actively transferring large amounts of data; although congestion exists, the average session transfer rate may appear higher because users are aggressively utilizing available bandwidth.

Therefore, in this analysis:
Low transfer rate during peak hours is interpreted as Wi-Fi congestion and slow user experience.
Low transfer rate during off-peak hours reflects idle usage rather than poor network capacity.

The regression model predicts expected session throughput given time and usage context, allowing identification of periods of high network strain rather than direct measurement of lag.