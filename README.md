Greece Electricity Prices Project


Multi-Factor analysis and forecasting of Greek Wholesale
electricity prices (Aug 2022-Aug 2026)
combining demand, generation mix, cross-border flows
Built with Python , SQL, and Power Bi


DATA USED:

From ENTSO-E: Day-Ahead Prices (Hourly/15 min)
              Actual Load(Hourly)
              Generation by type(solar, wind, gas, lignite, hydro) (Hourly)
              Cross Border Flows (BG , MK , IT) (Hourly)

    
     
From Yahoo Finance (yfinance):
            *TTF natural gas futures (Daily)


All sources loaded into SQLite and the analysis dataset is built
with SQL join ~35000 hourly observations



Features engineered:
       1.residual load(load - solar - wind)
       2.price lags(24h , 168h)
       3.rolling price/gas averages
       4.hourly encoding(sin/cos)
       5.crisis-regime flag
       6.calendar features



Did a train/test Split for the forecast models
(Train to Feb 2025)
(Test Feb-Jul 2025)


Model Results 
Baseline (same hour last week)

Model             MAE (€/MWh) RMSE (€/MWh)  vs baseline


Baseline            28.13  41.71  
Linear Regression   21.07  28.27               25.1% 
LightGBM            20.20  28.83               28.2% 











MAE/RMSE used instead of MAPE because prices cross zero
(Negative Price Hours) making percentage errors meaningless


KEY FINDINGS



1.Residual Load is the dominant price driver-not solar directly
 While solar correlates strongly(negatively) with price 
 (can also be seen from duck curve with solar)
 feature importance ranks residual load 1st and raw solar 11th out    
 of 16
 Solar's effect operates through residual load as it reduces the
 demand that expensive generators must cover.
 Relationship is monotonic but non linear
 **(Spearman 0.56 vs Pearson 0.43)**


2.The merit order bends at ~4000 MW residual load
  Below it prices are stable (p90-median spread ~€140)
  Above it spike risk rises sharply (spread €200-450)
  This is the threshold where Greece exhausts cheap generations and  
  dispatches expensive thermal plant

3.The duck curve: evening prices are 3.27x midday(2024+).
  Trough €55.66 at 12:00(solar peak), peak €182.22 at 20:00 (solar        
  gone demand high)

4.TTF gas sets the level while the residual load sets the shape
  TTF-price correlation: 0.97(monthly) , but only 0.32(hourly)
  withing a stable year. Gas determines where prices sit month to 
  month 

5.An apparent seasonal anomaly was the 2022 gas crisis.
  Autumn initially showed the highest evening peaks 
  contradicting expectations. First hypothesis:
  (earlier sunsets mean higher autumn residual load)
  tested and REJECTED
  autumn residual load is only 3d highest at 17:00
  (6,344 MW, winter 5,831 MW, autumn 5,026 MW)
  Actual Cause: TTF peaked near €340/MWh in autumn 2022
  vs normal (€30-50) inflating the 4-year autumn average
  Decided to keep 2022 in training with TTF as feature rather
  than dropping it
  Crisis flag ranked 15th out of 16 in importance
  TTF alone explains the regime

 6.The duck curve is deepening
  Midday solar: 2,965 MW (2023) → 3,838 MW (2025). Negative-price     
  hours: 1 (2022) → 12 (2025). Evening–midday spread: €65 (2023) →  
  €111 (2025). Expanding solar hollows out midday prices while  
  evenings stay anchored to gas costs.

 7.Gas generation and price rise together monotonically
  median price climbs from €18 at minimal gas dispatch to €172
  at ~4700MWm with the p90-median gap widening across the
  range-spike risk concentrates in high-gas hours.
  the relationship is diagnostic rather that casual:
  gas is dispatched because the market is tight and
  that tightness is what sets the price 
  the supply-side mirror of residual load, which is why gas ranks
  3d in feature importance to residual load's 1st
  ***BINS ABOVE 5000 MW CONTAIN FAR TOO FEW OBSERVATIONS TO   
  INTERPRET***

 8.Wind Generation has little effect on median prices
  (flat at ~€90-100 across the wind range) but strongly suppresses
  price spikes: p90 falls from ~€180 in calm hours to ~130 in windy
  ones. Unlike solar, whose fixed daily timing systematically
  depresses midday prices, wind's value lies in the evenings it
  happens to cover-reducing spike risk rather that shifting
  the typical price

 9.Weekends are 21.6% cheaper:
  With the largest gaps at peak hours (-€53 at 20:00, -€49 at  
  08:00) weekday industrial load pushes the system past the 4000MW
  threshold while weekend load doesn't


Power BI Dashboard

A five-page interactive dashboard built on the same dataset, with slicers for year, 
week, and date range.

Page 1 — The Duck Curve**
Hour × month price heatmap with a year slicer, showing the midday trough deepening 
year over year. Cards for midday average, evening average, and the peak-to-trough ratio.

Page 2 — The Merit Order**
Binned residual load against median and p90 price, with the 4,000 MW threshold marked. 
Weekday versus weekend price comparison by hour.

Page 3 — The Gas Connection**
TTF gas and electricity price plotted together across the full period, showing the 
2022 crisis and subsequent normalisation. Negative-price hours per year.

Page 4 — Two Kinds of Renewable**
Solar and wind plotted against price on identical scales, showing that solar depresses 
the median while wind suppresses the p90. Midday solar capacity growth by year.

Page 5 — Forecasting the Price**
Actual versus predicted prices across the test period with a week slicer, LightGBM 
feature importance, and model metrics.


<img width="2027" height="1158" alt="Heatmap Showing The Duck Curve" src="https://github.com/user-attachments/assets/8e23aaef-128d-4f9d-86a8-79ff71b60744" />

<img width="2011" height="1128" alt="Residual Load" src="https://github.com/user-attachments/assets/535e9672-126d-4755-95b9-3941b505e544" />

<img width="2021" height="1121" alt="Gas" src="https://github.com/user-attachments/assets/a892d69b-c111-4dd5-b504-64544e3fea1f" />


<img width="2006" height="1132" alt="Solar and Wind" src="https://github.com/user-attachments/assets/106175ea-cdb7-4c62-bca4-b547fc319a52" />

<img width="2010" height="1122" alt="Forecasting" src="https://github.com/user-attachments/assets/38c0cb83-043a-47a9-acce-325c987c6af0" />











***Some More Graphs that were Done with python that add detail***





<img width="1500" height="900" alt="solar_vs_price" src="https://github.com/user-attachments/assets/33b07691-a38c-46e6-a668-53e13b13eba5" />
<img width="1385" height="884" alt="price_vs_residual_load" src="https://github.com/user-attachments/assets/3034447b-3095-4e25-8dca-3830e59fc338" />


<img width="1500" height="750" alt="price_distribution" src="https://github.com/user-attachments/assets/a5218a08-2b2c-4424-a800-f50e70ca9b5c" />

<img width="2100" height="750" alt="duck_curve_by_year" src="https://github.com/user-attachments/assets/15c1efac-0ae7-4338-8e92-7c401af7d71d" />

***IN THE LAST GRAPH THE DUCK CURVE IS MUCH MORE VISIBLE, SOLAR GROWS YEAR BY YEAR AND CONSEQUENTLY THE MIDDAY PRICES DECREASE MORE AND MORE AS YEARS GO BY****









DATA QUALITY ISSUES FOUND AND SOLVED

 1.Phantom 74% missing generation data** — actually an artifact: Greek prices switched from hourly to **15-minute settlement during 2025 (56,950 price rows vs ~35,000 hours). Sub-hourly price rows found no hourly generation match in the join, appearing as NaN. Fixed by resampling prices to hourly means. Real generation gaps: <10 rows.


 2.SQLite `strftime()` silently converts tz-aware timestamps to UTC** — hour/month features derived in SQL were shifted 2–3h from Athens time. All time features now derived in pandas from localized datetimes.


 3.Generation data ends Jul 2025** (ENTSO-E reporting lag) while prices run to Aug 2026 — modeling window restricted accordingly.
yfinance writes multi-level CSV headers** — flattened before export.

CHARTS


 1.Price distribution histogram
 2.Solar vs price scatter
 3.Duck curve by season
 4.Duck curve by year (two-panel, price + solar)
 5.Gas and TTF relationships
 6.Merit order curve 
 7.Actual vs predicted (Feb 2025)
 8.Feature importance
 9.Error by hour
 10.Price vs gas (binned)
 11.Price vs wind (binned)


TOOLS USED

Python (pandas, scikit-learn, LightGBM, matplotlib) · SQL (SQLite) · Power BI · entsoe-py · yfinance




