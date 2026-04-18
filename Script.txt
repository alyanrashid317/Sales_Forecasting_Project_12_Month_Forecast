import pandas as pd
from prophet import Prophet

# 1. Prepare the data from Power BI
# We use .copy() to avoid 'SettingWithCopy' warnings
df = dataset[['Month-Year', 'Month-Year Sales']].copy()
df.columns = ['ds', 'y']

# 2. Convert to datetime and sort
# 'coerce' handles any messy date strings by turning them into NaT
df['ds'] = pd.to_datetime(df['ds'], errors='coerce')
df = df.dropna(subset=['ds']).sort_values('ds')

# 3. Initialize and Fit the Model
model = Prophet()
model.fit(df)

# 4. Create Future Dates
# freq='MS' ensures monthly steps; periods=12 adds one year
future = model.make_future_dataframe(periods=12, freq='MS')

# 5. Predict
forecast = model.predict(future)

# 6. Add a 'Type' column to distinguish Actuals from Forecasts
# This is huge for Power BI visuals later
last_real_date = df['ds'].max()
forecast['Type'] = ['Actual' if x = last_real_date else 'Forecast' for x in forecast['ds']]

# 7. Final Output Table for Power BI
# We include 'yhat' (the prediction) and the confidence intervals
final_forecast = forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper', 'Type']]