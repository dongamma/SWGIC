# VAR,ARIMA 예측-김유성

1열: 보성군-기흐변화에따른 농업용수예측
날짜: 2023년 10월 30일

1. AWS있는 데이터에 + 강수량 데이터의 강수량칼럼만 추가
2. 이후, 시계열 분석에 적합하게 Date 컬럼을 인덱스로 설정

| vgT | MinT | MaxT | AvgEn(hpa) | AvgPrs(hpa) | AvgWdSd | Rn | AvgPrs(kpa) | AvgEn(kpa) | g | r | e | u | ET | AWS | Precipitation(mm) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 2011-01-02 | -1.0 | -5.3 | 3.8 | 3.3 | 1025.4 | 3.8 | 2.63 | 102.54 | 0.33 | 0.041668 | 0.068189 | 0.606780 | 4.066 | 1.462971 | 10.935710 |
| 2011-01-03 | -0.7 | -2.7 | 2.3 | 3.2 | 1021.7 | 2.9 | 3.61 | 102.17 | 0.32 | 0.042487 | 0.067943 | 0.610819 | 3.103 | 1.456400 | 10.886591 |
| 2011-01-04 | 0.4 | -3.6 | 3.8 | 3.9 | 1021.8 | 3.4 | 5.67 | 102.18 | 0.39 | 0.045607 | 0.067950 | 0.635005 | 3.638 | 1.542873 | 11.532979 |
| 2011-01-05 | 1.6 | 0.0 | 4.3 | 4.2 | 1021.7 | 3.9 | 4.04 | 102.17 | 0.42 | 0.049235 | 0.067943 | 0.720697 | 4.173 | 1.688289 | 12.619957 |
| 2011-01-06 | -0.9 | -2.4 | 1.2 | 3.5 | 1024.7 | 4.5 | 5.22 | 102.47 | 0.35 | 0.041939 | 0.068143 | 0.589123 | 4.815 | 1.573865 | 11.764639 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 2021-05-30 | 19.7 | 12.8 | 26.1 | 14.4 | 1008.9 | 1.5 | 16.48 | 100.89 | 1.44 | 0.142406 | 0.067092 | 2.429825 | 1.605 | 5.222261 | 39.036403 |
| 2021-05-31 | 20.9 | 14.7 | 27.9 | 14.9 | 1010.9 | 1.7 | 16.58 | 101.09 | 1.49 | 0.151938 | 0.067225 | 2.715332 | 1.819 | 5.701662 | 42.619922 |
| 2021-06-01 | 21.0 | 13.9 | 27.6 | 15.2 | 1013.7 | 1.7 | 15.48 | 101.37 | 1.52 | 0.152757 | 0.067411 | 2.640521 | 1.819 | 5.290623 | 39.547409 |
| 2021-06-02 | 21.8 | 17.0 | 27.5 | 18.7 | 1012.7 | 1.7 | 13.12 | 101.27 | 1.87 | 0.159437 | 0.067345 | 2.804500 | 1.819 | 4.481388 | 33.498375 |
| 2021-06-03 | 20.1 | 18.7 | 21.8 | 22.5 | 1003.4 | 2.1 | 2.42 | 100.34 | 2.25 | 0.145525 | 0.066726 | 2.384237 | 2.247 | 0.780641 | 5.835291 |

3776 rows × 16 columns

그냥 데이터들로만 시각화 → ㅈ같이나옴.

```python
import matplotlib.pyplot as plt

# 데이터 예시
cols = ['AWS','Precipitation(mm)','AvgT']
data = bs_data[cols]

fig, ax1 = plt.subplots()

color = 'tab:blue'
ax1.set_xlabel('Date')
ax1.set_ylabel('AWS', color=color)
ax1.plot(data.index, data['AWS'], color=color)
ax1.tick_params(axis='y', labelcolor=color)

ax2 = ax1.twinx()  # 2nd y-axis
color = 'tab:orange'
ax2.set_ylabel('Precipitation(mm)', color=color)
ax2.plot(data.index, data['Precipitation(mm)'], color=color)
ax2.tick_params(axis='y', labelcolor=color)

ax3 = ax1.twinx()  # 3rd y-axis
color = 'tab:green'
# we already handled the x-label with ax1
ax3.spines['right'].set_position(('outward', 60))  # Move third y-axis to the right by 60 points
ax3.set_ylabel('AvgT', color=color)
ax3.plot(data.index, data['AvgT'], color=color)
ax3.tick_params(axis='y', labelcolor=color)

fig.tight_layout()  # To ensure that everything fits
plt.show()
```

![Untitled](VAR,ARIMA%20%E1%84%8B%E1%85%A8%E1%84%8E%E1%85%B3%E1%86%A8-%E1%84%80%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A5%E1%86%BC%2045552c6290e3466aac137755f117e9d1/Untitled.png)

#데이터 전처리 ( log변환 및 차분 )

```python

import matplotlib.pyplot as plt

# 년도별로 데이터 그룹화
grouped_data = data.resample('Y').mean()

fig, ax1 = plt.subplots(figsize=(10, 6))

color = 'tab:blue'
ax1.set_xlabel('Year')
ax1.set_ylabel('AWS', color=color)
ax1.plot(grouped_data.index, grouped_data['AWS'], color=color, marker='o')
ax1.tick_params(axis='y', labelcolor=color)

ax2 = ax1.twinx()
color = 'tab:orange'
ax2.set_ylabel('Precipitation(mm)', color=color)
ax2.plot(grouped_data.index, grouped_data['Precipitation(mm)'], color=color, marker='o')
ax2.tick_params(axis='y', labelcolor=color)

ax3 = ax1.twinx()
color = 'tab:green'
ax3.spines['right'].set_position(('outward', 60))
ax3.set_ylabel('AvgT', color=color)
ax3.plot(grouped_data.index, grouped_data['AvgT'], color=color, marker='o')
ax3.tick_params(axis='y', labelcolor=color)

fig.tight_layout()
plt.show()
```

![Untitled](VAR,ARIMA%20%E1%84%8B%E1%85%A8%E1%84%8E%E1%85%B3%E1%86%A8-%E1%84%80%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A5%E1%86%BC%2045552c6290e3466aac137755f117e9d1/Untitled%201.png)

- 변수별 정상성 검정 (ADF검정)

```python
import pandas as pd
import matplotlib.pyplot as plt

# 1차 차분 적용
data_diff = data.diff().dropna()

# ADF 테스트를 재실행하여 차분 후의 정상성 확인을 위한 코드 (옵션)
# from statsmodels.tsa.stattools import adfuller
# result_AWS = adfuller(data_diff['AWS'])
# result_AvgT = adfuller(data_diff['AvgT'])
# print('--Test statistic for diff AWS--')
# print('ADF Statistic:', result_AWS[0])
# print('p-value:', result_AWS[1])
# print('\n--Test statistic for diff AvgT--')
# print('ADF Statistic:', result_AvgT[0])
# print('p-value:', result_AvgT[1])

# 차분 후의 데이터를 그래프로 표현
fig, ax = plt.subplots(3, 1, figsize=(10, 8))

data_diff['AWS'].plot(ax=ax[0], title='First Difference of AWS')
data_diff['Precipitation(mm)'].plot(ax=ax[1], title='First Difference of Precipitation(mm)')
data_diff['AvgT'].plot(ax=ax[2], title='First Difference of AvgT')

plt.tight_layout()
plt.show()
```

![Untitled](VAR,ARIMA%20%E1%84%8B%E1%85%A8%E1%84%8E%E1%85%B3%E1%86%A8-%E1%84%80%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A5%E1%86%BC%2045552c6290e3466aac137755f117e9d1/Untitled%202.png)

```python
import pandas as pd
from statsmodels.tsa.stattools import adfuller

# 1차 차분 적용
data_diff = data.diff().dropna()

# AWS에 대한 ADF 테스트
result_AWS = adfuller(data_diff['AWS'])
print('--Test statistic for diff AWS--')
print('ADF Statistic:', result_AWS[0])
print('p-value:', result_AWS[1])
print('--------------------------------')

# Precipitation(mm)에 대한 ADF 테스트 (이미 정상성을 가진다고 확인했으므로 생략해도 됨.)
result_Precipitation = adfuller(data_diff['Precipitation(mm)'])
print('--Test statistic for diff Precipitation(mm)--')
print('ADF Statistic:', result_Precipitation[0])
print('p-value:', result_Precipitation[1])
print('--------------------------------')

# AvgT에 대한 ADF 테스트
result_AvgT = adfuller(data_diff['AvgT'])
print('--Test statistic for diff AvgT--')
print('ADF Statistic:', result_AvgT[0])
print('p-value:', result_AvgT[1])
print('--------------------------------')
```

<aside>
💡

```
--Test statistic for diff AWS--
ADF Statistic: -22.008923764803242
p-value: 0.0
--------------------------------
--Test statistic for diff Precipitation(mm)--
ADF Statistic: -18.66712985240571
p-value: 2.0458051530987248e-30
--------------------------------
--Test statistic for diff AvgT--
ADF Statistic: -8.359967745417519
p-value: 2.841044687748634e-13
```

1. **diff AWS**:
    - **ADF Statistic:** -22.0089
    - **p-value:** 0.0
        - 이 통계치와 p-value를 통해 AWS의 차분 데이터가 정상성을 가진다고 판단할 수 있음.
        - p-value가 0.05보다 작기 때문에, null hypothesis(해당 시계열이 non-stationary하다는 가설)를 기각 즉, AWS의 차분 데이터는 정상성을 가진다는 결론을 내릴 수 있음.
2. **diff Precipitation(mm)**:
    - **ADF Statistic:** -18.6671
    - **p-value:** 매우 작은 값 (2.0458051530987248e-30)
        - Precipitation(mm)의 차분 데이터 역시 정상성을 가진다는 결론을 내릴 수 있음.
        - p-value가 0.05보다 작기 때문에 해당 데이터는 정상적이라고 판단됨
3. **diff AvgT**:
    - **ADF Statistic:** -8.3599
    - **p-value:** 매우 작은 값 (2.841044687748634e-13)
        - AvgT의 차분 데이터도 정상성을 가진다고 판단할 수 있음.
        - 이 경우에도 p-value가 0.05보다 작아서, 해당 데이터는 정상적이라는 결론을 내릴 수 있음.
</aside>

- VAR 모델

기본 정보:
Model: 사용된 모델은 VAR (Vector Autoregression)임.
Method: 사용된 추정 방법은 OLS (Ordinary Least Squares)임.
Date & Time: 분석을 수행한 날짜와 시간임.
No. of Equations: 분석 대상이 되는 변수의 수임. 3개의 변수(AWS, Precipitation(mm), AvgT)가 있음.
AIC, BIC, HQIC: 모델의 적합도를 평가하는 정보 기준 값입니다. 값이 낮을수록 모델의 적합도가 높다고 판단할 수 있있음.
회귀 결과:
각 변수에 대한 회귀 결과가 제시되어 있있음.
AWS:

const: 상수항의 추정치는 -0.011564이며, 이 값은 통계적으로 유의하지 않음.(p-value: 0.598).
Precipitation(mm):

const: 상수항의 추정치는 0.002051이며, 이 값은 통계적으로 유의하지 않음.(p-value: 0.985).
AvgT:

const: 상수항의 추정치는 0.006550이며, 이 값은 통계적으로 유의하지 않습니다(p-value: 0.495).
잔차의 상관 행렬:
잔차의 상관 행렬은 각 변수의 잔차 간의 상관 관계를 보여줌.

AWS의 잔차와 Precipitation(mm)의 잔차 사이에는 -0.875648의 상관 관계가 있으며, 이는 둘 사이에 강한 음의 상관 관계가 있음을 나타냄.
AWS의 잔차와 AvgT의 잔차 사이에는 0.302546의 상관 관계가 있음.
Precipitation(mm)의 잔차와 AvgT의 잔차 사이에는 -0.151910의 상관 관계가 있음.
결론:
모델 결과에서 상수항만이 유의한 회귀 계수로 나타났으며, 각 변수의 값에 대한 추가적인 정보가 주어지지 않음. 즉, 각 변수간의 시차에 따른 관계는 고려되지 않았음.  

<aside>
💡 *maxlags=1로 설정했기 때문에 과거의 데이터를 기반으로 현재 데이터를 설명하려고 했지만, 상수항 외의 변수들은 모델에 포함되지 않았음. 이는 데이터의 패턴이나 시계열 특성이 상수항만을 필요로 하거나, 사용된 데이터 셋이 너무 작아서 (Nobs: 9.00000) 다른 변수들을 포함시키지 못했을 수 있음.*

</aside>

```python
# VAR 
print("VAR Lag Order:", var_lag_order)
print("X_train shape:", X_train.shape)
print("X_test shape:", X_test.shape)

# Lag order에 따른 X_train의 마지막 데이터 포인트
last_data_points = X_train.values[-var_lag_order:]

# 예측 수행
forecasted_values = var_fit.forecast(y=last_data_points, steps=len(X_test))

print(forecasted_values)

# 데이터가 작아서 예측이 불가함.
```

→에러뜸 데이터가 너무 작아서…

그래서 ARIMA모델로 변경

```python
from statsmodels.tsa.arima.model import ARIMA
#ARIMA 

# AWS에 대한 예측
model_aws = ARIMA(X_train['AWS'], order=(1,1,1)) # 예: AR=1, 차분=1, MA=1
result_aws = model_aws.fit()
forecast_aws = result_aws.forecast(steps=len(X_test))

# Precipitation(mm)에 대한 예측
model_prec = ARIMA(X_train['Precipitation(mm)'], order=(1,1,1))
result_prec = model_prec.fit()
forecast_prec = result_prec.forecast(steps=len(X_test))

# AvgT에 대한 예측
model_avgt = ARIMA(X_train['AvgT'], order=(1,1,1))
result_avgt = model_avgt.fit()
forecast_avgt = result_avgt.forecast(steps=len(X_test))

print("Forecasted AWS:", forecast_aws)
print("Forecasted Precipitation(mm):", forecast_prec)
print("Forecasted AvgT:", forecast_avgt)
```

<aside>
💡

```
Forecasted AWS: 2021-12-31    0.010259
Freq: A-DEC, dtype: float64
Forecasted Precipitation(mm): 2021-12-31    0.055888
Freq: A-DEC, dtype: float64
Forecasted AvgT: 2021-12-31    0.006363
Freq: A-DEC, dtype: float64
```

- **AWS**: AWS는 농업용수공급량.

-**예측된 농업용수 공급량값은 0.010259임**.

-이 수치는 해당 날짜의 공급량이 평균적으로 약간의 증가나 감소를 보일 것을 의미할 수 있음.

- **Precipitation(mm)**: 이 지표는 강수량. 예측된 강수량은 0.055888mm임.

-**이 값은 해당 날짜에 매우 적은 양의 강수가 예상된다는 것을 나타냄.**

-이런 낮은 강수량은 대체로 가벼운 이슬비 또는 거의 비가 오지 않는 상황을 의미할 수 있음.

- **AvgT**: 이 지표는 평균 온도.

-**예측된 평균 온도 값은 0.006363임.**

-이 값은 평균 온도가 매우 약간의 변화를 보일 것으로 예상됩니다. AvgT 값이 Celsius 또는 Fahrenheit 단위로 표현된다면, 이런 작은 변화는 실제로는 온도에 큰 변동이 없다는 것을 의미할 수 있습니다.

</aside>

```python
import matplotlib.pyplot as plt

# 예측값
data = {
    'AWS': 0.010259,
    'Precipitation(mm)': 0.055888,
    'AvgT': 0.006363
}

# 그래프 그리기
labels = list(data.keys())
values = list(data.values())

plt.figure(figsize=(10,6))
plt.bar(labels, values, color=['blue', 'green', 'red'])
plt.title("Forecasted Values for 2021-12-31")
plt.ylabel("Values")
plt.xlabel("Metrics")
plt.grid(axis='y')

for i, v in enumerate(values):
    plt.text(i, v + 0.001, "{:.5f}".format(v), ha='center', va='bottom')

plt.show()
```

![Untitled](VAR,ARIMA%20%E1%84%8B%E1%85%A8%E1%84%8E%E1%85%B3%E1%86%A8-%E1%84%80%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A5%E1%86%BC%2045552c6290e3466aac137755f117e9d1/Untitled%203.png)

<aside>
💡

# **AWS (Agricultural Water Supply, 블루 바)**

### **농업용수(AWS)의 예측 값은 약 0.01026입니다.**

### **AWS는 농작물의 성장에 필요한 물의 공급량을 나타내며, 이 값은 2021년 12월 31일에 농업에 필요한 물의 공급량을 예측한 것**

### **Precipitation(mm) (그린 바)**

### **강수량의 예측 값은 약 0.05589mm**

### **이 값은 세 가지 중에서 가장 큰 값으로, 이날의 강수량이 농업용수(AWS)와 평균 온도(AvgT)에 비해 상대적으로 높을 것으로 예상**

### **AvgT (레드 바)**

### **평균 온도의 예측 값은 약 0.00636**

### **이 값은 세 가지 중에서 가장 작은 값으로, 이날의 평균 온도 변화가 작을 것으로 예상**

</aside>

- ARIMA를 통한 mse

 

```python
import pandas as pd
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from statsmodels.tsa.arima.model import ARIMA
from sklearn.metrics import mean_squared_error

# 차분 수행
diff_bs_data = bs_data['AWS'].diff().dropna()

# 'AWS' 컬럼에 대한 2차 차분
diff2 = diff_bs_data.diff().dropna()

# 'AWS' 컬럼에 대한 3차 차분
diff3 = diff2.diff().dropna()

# ACF와 PACF 플롯 그리기
# 차분 이후의 ACF와 PACF 플롯 그리기
plot_acf(diff_bs_data)
plot_pacf(diff_bs_data)

# 위의 플롯을 바탕으로 p, d, q 값을 설정
p = 1 # 플롯을 보고 적절한 값을 선택
d = 0  # 이미 정상성을 갖게 만들었다면 0
q = 1  # 플롯을 보고 적절한 값을 선택

# ARIMA 모델 학습
model = ARIMA(diff_bs_data, order=(p,d,q))
model_fit = model.fit()

# 예측 및 성능 검증
predictions = model_fit.forecast(steps=10)  # 10일치 예측
mse = mean_squared_error(diff_bs_data[-10:], predictions)
print('MSE:', mse)
```

![xxxx.png](VAR,ARIMA%20%E1%84%8B%E1%85%A8%E1%84%8E%E1%85%B3%E1%86%A8-%E1%84%80%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A5%E1%86%BC%2045552c6290e3466aac137755f117e9d1/xxxx.png)