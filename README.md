![image.png](attachment:74d4b138-d81a-439b-af49-9adda0854c2c:image.png)

# 미션 목표 및 환경 세팅

---

<aside>
💡

자전거 대여 패턴을 분석하여 자전거 배치 및 운영 전략을 최적화하고, 대여 수요를 정확히 예측하는 것. 이를 통해 대여 시스템의 효율성을 높이고 사용자 만족도를 증가시키는 방법을 찾는 것이 이번 미션의 핵심!

이번 미션의 최종 목표는 RMSLE(Root Mean Squared Logarithmic Error)를 최대한 낮추는 것. 다양한 머신러닝 모델과 전략을 실험하여 가장 정확한 수요 예측 모델을 개발하기!

</aside>

위의 미션을 수행하기 위해 물리 컴퓨터에서 작업하는 것보다, 클라우드 컴퓨터 성능이 적합할 것이라고 판단하였기에, Kaggle Notebook에서 작업을 진행하였다.

# 데이터 설명

---

| 컬럼명 | 데이터 타입 | 설명 |
| --- | --- | --- |
| datetime | datetime | 자전거 대여 기록의 날짜 및 시간 (e.g : 2011-01-01 00:00:00) |
| season | int | 계절 (1:봄, 2:여름, 3:가을, 4:겨울) |
| holiday | int | 공휴일 여부 (0: 평일, 1: 공휴일) |
| workingday | int | 근무일 여부 (0: 주말/공휴일, 1: 근무일) |
| weather | int | 날씨 상황 (1: 맑음, 2: 구름낌/안개, 3: 약간의 비/눈, 4: 폭우/폭설) |
| temp | float | 실측 온도 (섭씨) |
| atemp | float | 체감 온도 (섭씨) |
| humidity | int | 습도 (%) |
| windspeed | float | 풍속 (m/s) |
| casual | int | 등록되지 않은 사용자의 대여 수 |
| registered | int | 등록된 사용자의 대여 수 |
| count | int | 총 대여 수 (종속 변수) |
- **train.csv** 파일에는 `count` 컬럼이 포함되어 있으며, 예측 대상인 종속변수.
- **test.csv** 파일에는 `casual`, `registered`, `count` 컬럼이 포함되어 있지 않음.
- **casual**과 **registered**는 자전거 대여 수요를 예측하는데 참고할만한 자료이며, `count`는 두 컬럼간의 합.

# 데이터 EDA

---

### 데이터의 크기

---

- train.csv : `(10886, 12)`
- test.csv : `(6493, 9)`

### 데이터의 존재 기간

---

- train.csv : `2011-01-01 00:00:00 ~ 2012-12-19 23:00:00`
- test.csv : `2011-01-20 00:00:00 ~ 2012-12-31 23:00:00`

## 연간 월별 미등록 사용자와 등록자 사용자의 자전거 대여 수

---

미등록 사용자와 등록 사용자의 자전거 대여 수 컬럼은 `train.csv`파일에만 있기에 `train.csv`파일을 사용하여 시각화 하였다.

![download.png](attachment:cf250dda-d176-4771-bd79-63664a285808:download.png)

- ResultDataFrame
    
    
    | month | casual_2011 | casual_2012 | cas_growth | reg_2011 | reg_2012 | reg_growth |
    | --- | --- | --- | --- | --- | --- | --- |
    | 1 | 2009 | 5224 | 161.15 | 21544 | 51088 | 137.13 |
    | 2 | 3776 | 5521 | 46.21 | 29068 | 60748 | 108.98 |
    | 3 | 7910 | 17146 | 116.76 | 30825 | 77620 | 151.80 |
    | 4 | 12229 | 27584 | 125.56 | 38288 | 89301 | 133.23 |
    | 5 | 15865 | 25420 | 60.22 | 63848 | 95014 | 48.81 |
    | 6 | 19600 | 28974 | 47.82 | 70176 | 101983 | 45.32 |
    | 7 | 26145 | 24802 | -5.13 | 66703 | 96967 | 45.37 |
    | 8 | 17580 | 28290 | 60.92 | 65716 | 101930 | 55.10 |
    | 9 | 18311 | 27590 | 50.67 | 60793 | 105835 | 74.09 |
    | 10 | 17159 | 20928 | 21.96 | 62363 | 106984 | 71.55 |
    | 11 | 10155 | 15198 | 49.66 | 60734 | 90353 | 48.76 |
    | 12 | 5079 | 9621 | 89.42 | 56104 | 89356 | 59.26 |

<aside>
📌

casual 이용객 총 증가율 : 51.66%

registered 이용객 총 증가율 : 70.43%

</aside>

- 2011년 대비 2012년 casual 이용객의 총 증가율은 약 51.66% 증가하였고, registered 이용객의 총 증가율은 약 70.43% 증가하였다.
- 2012년 7월의 casual 유저의 수는 2011년 대비 감소한 추세를 보였다.

## 연간 시간대별 자전거 평균 대여량

---

![download.png](attachment:10b51ec8-92e3-41a6-8282-001576445183:download.png)

<aside>
📌

출근 시간대인 8시와 퇴근 시간대인 17~18시에 많은 대여량을 볼 수 있다.

</aside>

## 날씨별 자전거 총 대여량

---

![download.png](attachment:20a5921a-05d5-4ecd-b5d0-4858edb45e20:download.png)

<aside>
📌

- weather 컬럼의 각 라벨링별 의미
    - 1 : 맑음
    - 2 : 흐림/안개
    - 3 : 약간의 비/눈
    - 4 : 폭우/폭설
- 데이터의 총 대여량은 2011~2012년 총합산값.
- 날씨가 좋아지지 않을수록, 자전거의 대여량은 많이 감소하는 추세를 보인다.
</aside>

## 계졀별 자전거 대여량 분포

---

`count` 컬럼은 `train.csv`파일에만 있기에 `train.csv`파일을 사용하여 시각화 하였다.

![download.png](attachment:e2c9ccf0-f4d7-4679-845a-0fc6e174e9d5:download.png)

<aside>
📌

- season 컬럼의 각 라벨링별 의미
    - 1 : 봄
    - 2 : 여름
    - 3 : 가을
    - 4 : 겨울
- 대여량 분포는 2011~2012년 사이의 범위를 포함.
- 여름과 가을에서 가장 많은 대여량을 보이고 있음.
</aside>

## 실제 온도, 체감 온도, 습도, 풍속에 따른 자전거 총 대여량 분포

---

실제 온도, 체감 온도, 습도, 풍속에 따른 자전거 총 대여량 분포를 histplot을 그려 분포를 관찰하였다.

### 실제 온도에 따른 자전거 총 대여량 분포 (temp)

---

![download.png](attachment:34c225b1-f70f-408f-be62-d733b8708f9e:download.png)

- Result DataFrame
    
    
    | bin_start | bin_end | count |
    | --- | --- | --- |
    | 0.820 | 4.838 | 69 |
    | 4.838 | 8.856 | 648 |
    | 8.856 | 12.874 | 1440 |
    | 12.874 | 16.892 | 1891 |
    | 16.892 | 20.910 | 1587 |
    | 20.910 | 24.928 | 1753 |
    | 24.928 | 28.946 | 1901 |
    | 28.946 | 32.964 | 1194 |
    | 32.964 | 36.982 | 355 |
    | 36.982 | 41.000 | 48 |
    

<aside>
📌

실제 온도가 약 8 ~ 32도 사이 구간에 많은 대여량이 포착됨.

</aside>

### 체감 온도에 따른 자전거 총 대여량 분포 (atamp)

---

![download.png](attachment:1ecfb6de-3046-4d05-9b8f-ef4e8b458fd7:download.png)

- Result DataFrame
    
    
    | bin_start | bin_end | count |
    | --- | --- | --- |
    | 0.7600 | 5.2295 | 44 |
    | 5.2295 | 9.6990 | 406 |
    | 9.6990 | 14.1685 | 1243 |
    | 14.1685 | 18.6380 | 1679 |
    | 18.6380 | 23.1075 | 1790 |
    | 23.1075 | 27.5770 | 1962 |
    | 27.5770 | 32.0465 | 1832 |
    | 32.0465 | 36.5160 | 1425 |
    | 36.5160 | 40.9855 | 440 |
    | 40.9855 | 45.4550 | 65 |

<aside>
📌

체감 온도가 약 9 ~ 36도 사이 구간에 많은 대여량이 포착됨.

</aside>

### 습도에 따른 자전거 총 대여량 분포 (humidity)

---

![download.png](attachment:034dc6c9-88d8-4def-9301-c379e762ffff:download.png)

- Result DataFrame
    
    
    | bin_start | bin_end | count |
    | --- | --- | --- |
    | 0.0 | 10.0 | 23 |
    | 10.0 | 20.0 | 45 |
    | 20.0 | 30.0 | 364 |
    | 30.0 | 40.0 | 1039 |
    | 40.0 | 50.0 | 1727 |
    | 50.0 | 60.0 | 1842 |
    | 60.0 | 70.0 | 1748 |
    | 70.0 | 80.0 | 1736 |
    | 80.0 | 90.0 | 1676 |
    | 90.0 | 100.0 | 686 |

<aside>
📌

습도가 30 ~ 90% 사이에 많은 대여량이 포착됨.

</aside>

### 풍속에 따른 자전거 대여량 분포 (windspeed)

---

![download.png](attachment:69453cb7-0970-4007-8f9c-03a46a27f33b:download.png)

- Result DataFrame
    
    
    | bin_start | bin_end | count |
    | --- | --- | --- |
    | 0.00000 | 5.69969 | 1313 |
    | 5.69969 | 11.39938 | 4083 |
    | 11.39938 | 17.09907 | 2827 |
    | 17.09907 | 22.79876 | 1540 |
    | 22.79876 | 28.49845 | 696 |
    | 28.49845 | 34.19814 | 280 |
    | 34.19814 | 39.89783 | 107 |
    | 39.89783 | 45.59752 | 31 |
    | 45.59752 | 51.29721 | 6 |
    | 51.29721 | 56.99690 | 3 |

<aside>
📌

풍속이 0 ~ 22.5m/s 내에 많은 대여량이 포착됨.

</aside>

## 범주형 데이터 간 상관관계 분석

---

범주형 변수는 다음과 같다.

<aside>
💡

- season
- holiday
- workingday
- weather
- datetime을 year, month, day, hour로 구분한 컬럼
</aside>

이 변수들을 사용하여 카이제곱 검정을 실시하였고, 대립가설인 “**강한 연관이 있다.**”  의 조건을 충족하는 p-value값이 0.05이하인 값들만 추출한 결과, 다음과 같다.

### train_df의 카이제곱 검정결과

| Variable 1 | Variable 2 | Chi2 Statistic | p-value |
| --- | --- | --- | --- |
| season | holiday | 20.823388 | 1.145516e-04 |
| season | weather | 49.158656 | 1.549925e-07 |
| season | month | 32658.000000 | 0.000000e+00 |
| holiday | workingday | 679.830361 | 7.274718e-150 |
| holiday | month | 341.279082 | 1.695226e-66 |
| holiday | day | 292.159124 | 1.967560e-51 |
| workingday | weather | 16.162519 | 1.050217e-03 |
| workingday | month | 64.723871 | 1.214468e-09 |
| workingday | day | 95.196217 | 1.665054e-12 |
| weather | year | 13.414695 | 3.820469e-03 |
| weather | month | 247.560382 | 1.050538e-34 |
| weather | day | 191.622423 | 2.730175e-17 |
| weather | hour | 133.348920 | 5.430890e-06 |

<aside>
📌

train_df에서 강한 연관성이 있는 변수

- `season` -> `holiday`, `waether`, `month`
- `holiday` -> `workingday`, `month`, `day`
- `workingday` -> `weather`, `month`, `day`
- `weather` -> `year`, `month`, `day`, `hour`
</aside>

### test_df의 카이제곱 검정결과

| Variable 1 | Variable 2 | Chi2 Statistic | p-value |
| --- | --- | --- | --- |
| season | holiday | 105.083854 | 1.253472e-22 |
| season | workingday | 52.195517 | 2.721058e-11 |
| season | weather | 173.617925 | 1.087270e-32 |
| season | month | 17581.734489 | 0.000000e+00 |
| season | day | 79.937217 | 9.047547e-06 |
| holiday | workingday | 421.654275 | 1.065063e-93 |
| holiday | month | 410.636832 | 3.372796e-81 |
| holiday | day | 86.432986 | 8.306194e-14 |
| workingday | weather | 33.275090 | 2.817826e-07 |
| workingday | month | 85.398959 | 1.321284e-13 |
| workingday | day | 61.476363 | 4.925023e-09 |
| weather | month | 284.271288 | 9.391568e-42 |
| weather | day | 157.640598 | 3.507383e-18 |
| weather | hour | 121.800350 | 9.194184e-05 |
| month | day | 304.885711 | 7.440361e-18 |

<aside>
📌

test_df에서 강한 연관성이 있는 변수

- `season` -> `holiday`, `workingday`, `weather`, `month`, `day`
- `holiday` ->`workingday`, `month`, `day`
- `workingday` -> `weather`, `month`, `day`
- `weather` -> `month`, `day`, `hour`
- `month` -> `day`
</aside>

<aside>
📌

- 가장 강한 상관관계를 가지는 컬럼은 다음과 같다.
    - `season` → `holiday`, `weather`, `month`
    - `holiday` → `workingday`, `month`, `day`
    - `workingday` → `weather`, `month`, `day`
    - `weather` → `month`, `day`, `hour`
- train_df와 test_df의 상관관계의 유사도가 매우 비슷하다.
</aside>

선형 회귀 시, 다중공선성을 위해 범주형 변수들을 One-Hot Encoding을 하여 연관성을 줄일 수 있다.

## 연속형 데이터 간 상관관계 분석

---

연속형 변수는 다음과 같다.

<aside>
💡

- temp
- atemp
- humidity
- windspeed
</aside>

이 변수들을 사용하여 `corr()`를 분석하였다.

`count` 의 경우, 종속변수이기 때문에 상관관계 분석에 포함시키지 않았다.

**train_df의 연속형 변수의 상관관계 히트맵**

![download.png](attachment:a653b997-38dd-4df4-8704-24703785173f:download.png)

**test_df의 연속형 변수의 상관관계 히트맵**

![download.png](attachment:7ca7849d-d5f6-4a85-ab81-e125a7ba1dc3:download.png)

<aside>
📌

- `temp`와 `atemp` 사이의 강한 연관을 가지고 있음을 나타낸다.
- train_df와 test_df의 상관관계의 유사도가 매우 비슷하다.
</aside>

선형 회귀 시, 다중공선성을 위해 `temp` 컬럼과 `atemp` 컬럼을 다음과 같은 방법으로 처리할 수 있다.

- `temp`, `atemp` 컬럼 둘 중 하나를 삭제
- `temp`, `atemp` 컬럼 사이를 뺀 값인 **`temp_diff`**라는 새로운 변수 추가 후 `temp`, `atemp` 컬럼 삭제

## 범주형-연속형 데이터 간 상관관계 분석

---

범주형 데이터와 연속형 데이터 들을 토대로 ANOVA 분석을 진행하였다.

아래는 ANOVA 분석을 진행하고, 대립가설인 “**강한 연관이 있다.**” 의 조건을 충족하는 p-value값이 0.05이하인 값들만 추출한 결과, 다음과 같다.

### train_df의 ANOVA 분석 검정결과

| categorical Variable | numeric Variable | f-statistic | p-value |
| --- | --- | --- | --- |
| season | temp | 3441.968407 | 0.000000e+00 |
| season | atemp | 3267.590270 | 0.000000e+00 |
| season | humidity | 39.363540 | 3.351976e-25 |
| season | windspeed | 62.247336 | 1.137033e-39 |
| holiday | temp | 33.684692 | 6.786683e-09 |
| holiday | atemp | 34.177581 | 5.274847e-09 |
| holiday | humidity | 6.492755 | 1.085434e-02 |
| workingday | temp | 61.038716 | 6.484148e-15 |
| workingday | atemp | 69.878234 | 7.643663e-17 |
| workingday | humidity | 23.138684 | 1.541301e-06 |
| workingday | windspeed | 18.764952 | 1.500870e-05 |
| weather | temp | 72.912214 | 2.247343e-46 |
| weather | atemp | 78.574004 | 6.363219e-50 |
| weather | humidity | 512.294088 | 8.577962e-299 |
| weather | windspeed | 29.389217 | 7.245947e-19 |
| year | humidity | 55.905060 | 8.609213e-14 |
| month | temp | 2074.740441 | 0.000000e+00 |
| month | atemp | 1838.253093 | 0.000000e+00 |
| month | humidity | 33.951492 | 3.611495e-71 |
| month | windspeed | 26.484307 | 1.161826e-54 |
| day | humidity | 13.139530 | 3.907926e-25 |
| day | windspeed | 12.211348 | 4.210147e-23 |
| hour | temp | 13.934885 | 9.217988e-53 |
| hour | atemp | 12.579029 | 9.748807e-47 |
| hour | humidity | 72.688216 | 4.651576e-301 |
| hour | windspeed | 14.987973 | 1.888750e-57 |

<aside>
📌

train_df 내 범주형-연속형 데이터 간의 상관관계가 강한 데이터 컬럼  모음

`season` → `temp`, `atemp`, `humidity`, `windspeed`

`holiday` → `temp`, `atemp`, `humidity`

`workingday` → `temp`, `atemp`, `humidity`

`weather` → `temp`, `atemp`, `humidity`, `windspeed`

`year` → `humidity`

`month` → `temp`, `atemp`, `humidity`, `windspeed`

`day` → `humidity`, `windspeed`

`hour` → `temp`, `atemp`, `humidity`, `windspeed`

</aside>

### test_df의 ANOVA 분석 검정결과

| categorical Variable | numeric Variable | f-statistic | p-value |
| --- | --- | --- | --- |
| season | temp | 3441.968407 | 0.000000e+00 |
| season | atemp | 3267.590270 | 0.000000e+00 |
| season | humidity | 39.363540 | 3.351976e-25 |
| season | windspeed | 62.247336 | 1.137033e-39 |
| holiday | temp | 33.684692 | 6.786683e-09 |
| holiday | atemp | 34.177581 | 5.274847e-09 |
| holiday | humidity | 6.492755 | 1.085434e-02 |
| workingday | temp | 61.038716 | 6.484148e-15 |
| workingday | atemp | 69.878234 | 7.643663e-17 |
| workingday | humidity | 23.138684 | 1.541301e-06 |
| workingday | windspeed | 18.764952 | 1.500870e-05 |
| weather | temp | 72.912214 | 2.247343e-46 |
| weather | atemp | 78.574004 | 6.363219e-50 |
| weather | humidity | 512.294088 | 8.577962e-299 |
| weather | windspeed | 29.389217 | 7.245947e-19 |
| year | humidity | 55.905060 | 8.609213e-14 |
| month | temp | 2074.740441 | 0.000000e+00 |
| month | atemp | 1838.253093 | 0.000000e+00 |
| month | humidity | 33.951492 | 3.611495e-71 |
| month | windspeed | 26.484307 | 1.161826e-54 |
| day | humidity | 13.139530 | 3.907926e-25 |
| day | windspeed | 12.211348 | 4.210147e-23 |
| hour | temp | 13.934885 | 9.217988e-53 |
| hour | atemp | 12.579029 | 9.748807e-47 |
| hour | humidity | 72.688216 | 4.651576e-301 |
| hour | windspeed | 14.987973 | 1.888750e-57 |

<aside>
📌

test_df 내 범주형-연속형 데이터 간의 상관관계가 강한 데이터 컬럼  모음

`season` → `temp`, `atemp`, `humidity`, `windspeed`

`holiday` → `temp`, `atemp`, `humidity`

`workingday` → `temp`, `atemp`, `humidity`

`weather` → `temp`, `atemp`, `humidity`, `windspeed`

`year` → `humidity`

`month` → `temp`, `atemp`, `humidity`, `windspeed`

`day` → `humidity`, `windspeed`

`hour` → `temp`, `atemp`, `humidity`, `windspeed`

</aside>

<aside>
📌

- `season`, `holiday`, `workingday`, `weather`, `year`, `month`, `hour`와 **`temp`, `atemp`, `humidity`, `windspeed`** 간의 관계는 거의 모든 경우에서 유의미한 차이를 보임.
- 현재 모든 범주형 변수가 연속형 변수에 유의미한 영향을 끼치는 것으로 확인할 수 있다.
- train_df와 test_df의 ANOVA 분석 결과가 비슷하게 나오는 경향을 파악할 수 있다.
</aside>

## EDA 분석 종합 인사이트

---

- **미등록 사용자보다 등록된 사용자의 대여량이 가장 많은 수치를 기록함.**
- **출근 시간대와 퇴근 시간대에 자전거 대여량의 평균이 가장 높음.**
- **날씨가 좋아지지 않을수록 대여량의 수치는 급격하게 감소함.**
- **계절이 여름과 가을일 때 많은 대여량의 분포를 보임.**
- **습도가 30~90%에서 많은 대여량을 보이는데, 실내에서 생활할 때 필요한 적정 습도 기준은 실내 온도 24℃에서 최소 40%, 15℃에서 최대 70%라고 기준을 두고 있는 것으로 보아, 이용객들은 실내 적정 습도 범위 내에서 많은 대여량을 보이고 있음.**
    - [이에 대한 링크](https://www.inha.com/page/health/medicine/148039)
- **데이터 분포의 패턴 경우, 학습 데이터와 검증 데이터 둘 다 비슷한 패턴과 데이터간 연관 추세를 보이고 있음.**
    - 어느 한 지역에서만 추출한 데이터로 추측이 가능함.
    - 그 지역의 지리적인 특성을 가지고 있기 때문에 비슷한 추세를 보일 수 있을 것임.
- **위의 내용으로 인해 과적합의 위험은 감소하지만, 모델의 일반화 성능의 평가가 어렵다.**
    - 차후 전국 단위의 데이터를 추출할 시, 지역 데이터를 추가하여 머신러닝 모델을 다시 학습시켜 일반화 성능을 향상 시킬 수 있음.

# 선형 회귀 머신러닝 모델링 전 데이터 전처리

---

선형 회귀를 진행하기 전, 데이터 전처리를 다음과 같은 단계를 거쳐 전처리를 진행하였다.

1. 명목형 데이터 One-Hot Encoding 진행 후 상관계수 계산
    1. 다중공선성이 해결되지 않으면 해당 컬럼을 삭제
2. 연속형 데이터 간 VIF 계산 후, 파생 변수인 `temp_diff` (`temp` - `atemp`) 생성
    1. 다중공선성이 해결되지 않으면 PCA 진행 후 다시 VIF값 확인
3. 범주형 - 연속형 간 ANOVA 분석 다시 실행

<aside>
💡

### 왜 명목형 변수에만 One-Hot Encoding을 진행했는가?

- 명목형 변수들을 가진 컬럼들은 현재 라벨링된 수치값을 가지고 있는데, 선형 모델에서 라벨링된 수치값을 수치적인 관계로 잘못 해석할 수 있다.
- 순서형 변수는 순서가 중요한 범주형 변수이다. 1월과 2월은 숫자 차이가 1이지만, 이 차이는 실제로 시간적으로 연속적인 관계를 나타내는데, 이를 One-Hot Encoding을 하게 되면 **순서적 의미**를 잃게 된다.
- 순서형 변수는 **수치적인 순서**가 중요한 의미를 갖기 때문에, 이를 그대로 선형 회귀에 입력하는 것이 유효하다.
</aside>

<aside>
💡

### VIF(다중공선성)의 중요성

- 회귀분석 결과에서 각 변수의 회귀계수 추정이 부정확해질 수 있음.
- 구조방정식을 이용한 매개모형등을 분석시 분산을 크게 추정하여 2종 오류를 발생시킬 수 있음.
- 불안정한 추정치로 해석의 문제를 일으킬 수 있음.
</aside>

## 다중공선성 방지를 위한 명목형 데이터 전처리

---

선형 모델을 적용하기 위해 명목형 변수인 `season`, `weather` 컬럼을 **One-Hot Encoding** 한 뒤, 이산형 데이터와 순서형 데이터의 상관계수를 파악하기 위해 Point-biserial, 피어슨 상관계수를 통해 데이터간의 상관관계를 도출하였다.

- train_df의 One-Hot Encoding한 데이터와 순서형 데이터의 상관계수 파악 결과
    
    <Point-biserial>
    
    | 변수 | year 상관계수 | year p-value | month 상관계수 | month p-value | day 상관계수 | day p-value | hour 상관계수 | hour p-value |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | season_2 | -0.0024 | 0.7986 | -0.2556 | 6.60e-162 | 0.0010 | 0.9154 | -0.0027 | 0.7755 |
    | season_3 | -0.0016 | 0.8676 | 0.2484 | 1.05e-152 | 0.0009 | 0.9218 | -0.0026 | 0.7853 |
    | season_4 | -0.0022 | 0.8157 | 0.7531 | 0.0000 | 0.0005 | 0.9571 | -0.0030 | 0.7572 |
    | weather_2 | 0.0191 | 0.0467 | 0.0190 | 0.0471 | 0.0120 | 0.2104 | -0.0507 | 1.23e-07 |
    | weather_3 | -0.0308 | 0.0013 | -0.0004 | 0.9674 | -0.0189 | 0.0480 | 0.0140 | 0.1433 |
    | weather_4 | 0.0095 | 0.3192 | -0.0154 | 0.1089 | -0.0017 | 0.8562 | 0.0090 | 0.3504 |
    
    ![download.png](attachment:62f2f742-8aea-49ae-b3ca-aa694f39a2a9:download.png)
    
- test_df의 One-Hot Encoding한 데이터와 순서형 데이터의 상관계수 파악 결과
    
    <Point-biserial>
    
    | 변수 | year 상관계수 | year p-value | month 상관계수 | month p-value | day 상관계수 | day p-value | hour 상관계수 | hour p-value |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | season_2 | -0.0029 | 0.8176 | -0.4260 | 1.16e-284 | 0.0211 | 0.0884 | -0.0032 | 0.7936 |
    | season_3 | 8.26e-05 | 0.9947 | 0.1149 | 1.58e-20 | -0.0196 | 0.1145 | -0.0046 | 0.7100 |
    | season_4 | -0.0171 | 0.1677 | 0.5739 | 0.0000 | 0.0135 | 0.2755 | -0.0003 | 0.9832 |
    | weather_2 | -0.0022 | 0.8574 | 0.0056 | 0.6530 | -0.0831 | 1.95e-11 | -0.0504 | 4.90e-05 |
    | weather_3 | -0.0329 | 0.0079 | -0.0085 | 0.4910 | -0.0340 | 0.0062 | 0.0217 | 0.0808 |
    | weather_4 | -0.0001 | 0.9918 | -0.0285 | 0.0217 | -0.0091 | 0.4652 | -0.0078 | 0.5319 |
    
    ![download.png](attachment:7fc763ce-3196-41a4-88e6-c71c195685f0:download.png)
    

<aside>
📌

- One-Hot Encoding을 실행하여도 `season` 과 `month` 간의 상관관계가 있다는 것을 확인.
- `season` 컬럼을 삭제하는 것이 다중공선성이 해결될 것으로 보임.
</aside>

- season 컬럼을 제거한 train_df의 상관계수 파악 결과
    
    
    | 변수 | year 상관계수 | year p-value | month 상관계수 | month p-value | day 상관계수 | day p-value | hour 상관계수 | hour p-value |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | weather_2 | 0.0191 | 0.0467 | 0.0190 | 0.0471 | 0.0120 | 0.2104 | -0.0507 | 1.23e-07 |
    | weather_3 | -0.0308 | 0.0013 | -0.0004 | 0.9674 | -0.0190 | 0.0480 | 0.0140 | 0.1433 |
    | weather_4 | 0.0095 | 0.3192 | -0.0154 | 0.1089 | -0.0017 | 0.8562 | 0.0090 | 0.3504 |
    
    ![download.png](attachment:0f7a7572-f1c9-4a67-9c4a-ad5b1fc7b1b8:download.png)
    
- season 컬럼을 제거한 test_df의 상관계수 파악 결과
    
    
    | 변수 | year 상관계수 | year p-value | month 상관계수 | month p-value | day 상관계수 | day p-value | hour 상관계수 | hour p-value |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | weather_2 | -0.0022 | 0.8574 | 0.0056 | 0.6530 | -0.0831 | 1.95e-11 | -0.0504 | 4.90e-05 |
    | weather_3 | -0.0329 | 0.0079 | -0.0085 | 0.4910 | -0.0340 | 0.0062 | 0.0217 | 0.0808 |
    | weather_4 | -0.0001 | 0.9918 | -0.0285 | 0.0217 | -0.0091 | 0.4652 | -0.0078 | 0.5319 |
    
    ![download.png](attachment:a4fb550c-1376-4ecd-b561-f197070e6df7:download.png)
    

<aside>
📌

- 다중공선성이 많이 해결된 것을 볼 수 있다.
</aside>

## 선형 모델을 시작하기 전 연속형 데이터 전처리

---

EDA를 진행하며 확인한 기존 연속형 데이터인 `temp` , `atemp`, `humidity`, `windspeed`간의 상관관계를 관찰했고, 그 결과  `temp` , `atemp` 간의 상관관계가 매우 큰 것을 확인할 수 있었다. 

다중공선성을 줄이기 위해 `temp` , `atemp` 의 차이인 `temp_diff` 라는 파생 변수를 생성하고 두 컬럼을 삭제한 다음, One-Hot Encoding되어 이산형 데이터로 만들어진 범주형 데이터까지 합산하여 피어슨 상관계수와 VIF 값을 같이 확인하였다.

- 파생변수를 생성한 train_df의 피어슨 상관계수와 VIF
    
    ![download.png](attachment:0db7c091-c52b-4d31-b997-d9e123d5f613:download.png)
    
    | columns | vif |
    | --- | --- |
    | temp_diff | 4.677462 |
    | humidity | 6.340641 |
    | windspeed | 2.391240 |
    | weather_2 | 1.517309 |
    | weather_3 | 1.251026 |
    | weather_4 | 1.000403 |
- 파생변수를 생성한 train_df의 피어슨 상관계수와 VIF
    
    ![download.png](attachment:324d9606-cfd2-4c7d-a92a-52eebe61c8c8:download.png)
    
    | columns | vif |
    | --- | --- |
    | temp_diff | 5.872320 |
    | humidity | 8.124797 |
    | windspeed | 2.462594 |
    | weather_2 | 1.569522 |
    | weather_3 | 1.356557 |
    | weather_4 | 1.003112 |

<aside>
📌

- 파생 변수를 통해 피어슨 상관계수의 값이 많이 떨어진 것을 확인할 수 있음
- VIF값이 10 미만으로 다중공선성이 많이 줄어들었지만, VIF값이 5 이상 존재하는 데이터가 존재하여 모델링 시 유의할 컬럼으로 지정.
</aside>

## 연속형 데이테를 다양한 스케일러 적용 후 데이터 분포 시각화

---

스케일러는 연속형 변수에만 적용하는 것이 원칙이고, 현재 종속변수는 `count` 이므로 포함시키지 않고, 독립 변수인 `temp_diff`, `humidity`, `windspeed`에만 적용하였다.

One-Hot Encoding 데이터는 이미 0과 1로 나뉘어져 있는 이산형 값이기 때문에 따로 스케일링을 진행하지 않았다.

### StandardScaler

---

![download.png](attachment:ad1e595c-8349-4610-b293-9208562ac443:download.png)

![download.png](attachment:6ce47ea6-8cac-414f-8017-9d7d481ce21e:download.png)

### MinMaxScaler

---

![download.png](attachment:f37cbf2b-d55d-4272-b228-d073df9eeb17:download.png)

![download.png](attachment:2dc2b6bf-d1aa-4ec1-ac0c-f8d817d67173:download.png)

### RobustScaler

---

![download.png](attachment:c0f2f027-05f9-40ea-8c9b-74c29b68160d:download.png)

![download.png](attachment:790399a1-bfc3-4992-94a9-96dabfb380a2:download.png)

### 적용하지 않은 스케일러

---

- 현재 Raw 데이터에 음수값(-)이 존재하지 않아, MaxAbsScaler를 적용하지 않았다.
- 데이터를 row 방향으로 스케일링 하는 방법인 Normalizer는 이 데이터에 맞지 않아 적용하지 않았다.

# 선형 회귀분석 모델링

---

GridSearchCV를 통해 최적의 파라미터 값을 찾아내었다.

또한 평가 점수를 RMSLE로 평가를 하기 때문에 target 변수에 `log1p` 값을 적용하여 계산하였다.

## Linear Regressor

---

```
Result of StandardScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.24283344598786255

Result of MinMaxScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.24283344598786383

Result of RobustScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.2428334459878639
```

<aside>
💡

### **Best Scaler**

- StandardScaler
    - `{'fit_intercept': True, 'positive': False}`
    - `0.24283344598786255`
</aside>

- Feature Importance 그래프
    
    ![download.png](attachment:8e779d97-57ea-4406-a69a-dae8736c75a7:download.png)
    

## Lasso

---

```
Result of StandardScaler
Best Parameters : {'fit_intercept': False, 'positive': True}
Best Score : 0.26574507804388914

Result of MinMaxScaler
Best Parameters : {'fit_intercept': False, 'positive': True}
Best Score : 0.26574507804388914

Result of RobustScaler
Best Parameters : {'fit_intercept': False, 'positive': True}
Best Score : 0.26574507804388914
```

<aside>
💡

### **Best Scaler**

- 세 스케일러 모두 동일
    - `{'fit_intercept': False, 'positive': True}`
    - `0.26574507804388914`
</aside>

- Feature Importance 그래프
    
    ![download.png](attachment:c71b6ed5-d83d-4ffe-b294-5a00feb4219c:download.png)
    

## Ridge

---

```
Result of StandardScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.2428329942289224

Result of MinMaxScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.2428431232866118

Result of RobustScaler
Best Parameters : {'fit_intercept': True, 'positive': False}
Best Score : 0.24283315114678167
```

<aside>
💡

### **Best Scaler**

- StandardScaler
    - `{'fit_intercept': True, 'positive': False}`
    - `0.2428329942289224`
</aside>

- Feature Importance 그래프
    
    ![download.png](attachment:5d1669e9-6801-4ba3-97ec-934384d29a66:download.png)
    

# 결정 트리 머신러닝 모델 구성

---

결정 트리 머신러닝 모델은 다음과 같은 특성을 가진다.

<aside>
💡

- 비선형 모델을 가지고 있어 특성 간의 관계를 자유롭게 학습하는 방식이기에, 스케일링의 영향을 받지 않는다.
- 라벨링 되어있는 범주형 변수들도 그 값들의 순서나 크기를 중요하게 고려하지 않기 때문에, One-Hot Encoding을 굳이 진행하지 않아도 된다.
</aside>

이하 각 모델별 결과값 도출은 다음과 같이 진행 하였다.

- input 데이터는 다음과 같은 데이터를 input하여 결과값을 도출하였다.
    - datetime을 연/월/일/시간별로 구분한 Raw 데이터
    - 선형회귀에 사용한 Scaled 데이터
- 내가 설정한 GridSearch에 지정할 파라미터 값을 넣는다.
- GridSearchCV를 이용하여 가장 최적의 파라미터 값을 찾는다.
- 모델의 평가 점수는 RSMLE 값을 도출한다.
- 재현성을 위해 random_state값을 42로 고정하였다.

## RandomForest

---

RandomForest는 여러 개의 결정트리(Decision Tree)를 활용한 배깅(Bagging) 방식의 대표적인 알고리즘이다.

### RandomForest의 장단점

### RandomForest의 장점

- 여러 트리를 사용하여 평균을 내는 방식으로, 과적합에 덜 민감함.
- 각 트리의 예측을 평균내므로 상대적으로 쉽게 해석할 수 있음.
- 모델이 학습하면서 변수 중요도를 제공하므로 특징 선택에 유용.

### RandomForest의 단점

- 트리가 많아지면 모델이 커지고 예측 시간이 길어짐.
- 많은 트리가 포함되므로 예측 시 속도가 상대적으로 느려질 수 있음.
- 기본적인 하이퍼파라미터로는 성능을 크게 개선하기 어려울 수 있음.

### RandomForest를 사용하여 나타난 결과값

```python
# 파라미터값
param = {
	'n_estimators': [100, 200, 300],
	'max_depth': [5, 10, 15],
	'min_samples_split': [2, 5, 10],
	'min_samples_leaf': [1, 2, 4]
}
```

<aside>
💡

**가장 높게 나온 데이터 : Raw Data**

`Best Parameters`

- `max_depth` : `15`
- `min_samples_leaf` : `4`
- `min_samples_split` : `5`
- `n_estimators` : `300`

`Best Score (RMSLE)` : `0.10821955492791364`

</aside>

- 각 데이터 별 Best Parameters와 Best Score
    - Raw Data
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 2, 'n_estimators': 300}
    Best Score : 0.10859456191822163
    ```
    
    - StandardScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 300}
    Best Score : 0.11293346966590563
    ```
    
    - MinMaxScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 100}
    Best Score : 0.11262592437379748
    ```
    
    - RobustScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 200}
    Best Score : 0.11252590373856475
    ```
    
- Feature Importance 그래프
    
    ![download.png](attachment:afe00eac-b8e8-4365-84ff-4679b5aaf2fe:download.png)
    

## Gradient Boosting

---

GradientBoost는 Gradient(또는 잔차(Residual))를 이용하여 이전 모형의 약점을 보완하는 새로운 모형을 순차적으로 적합한 뒤 이들을 선형 결함하여 얻어진 모형을 생성하는 지도 학습 알고리즘.

### Gradient Boosting의 장단점

### Gradient Boosting의 장점

- 여러 약한 학습기를 결합하여 매우 높은 예측 성능을 자랑함.
- 모델이 순차적으로 학습하면서 오차를 보완하므로 과적합에 강함.
- 여러 유형의 손실 함수 및 모델을 설정할 수 있어 다양한 문제에 적합.

### Gradient Boosting의 단점

- 순차적으로 모델을 학습하므로, XGBoost나 LightGBM보다 학습 속도가 느릴 수 있음.
- 모델 성능을 최적화하려면 많은 하이퍼파라미터를 튜닝해야 할 수 있음.
- 하이퍼파라미터 조정이 잘못되면 과적합될 가능성이 있음.

### Gradient Boosting를 사용하여 나타난 결과값

```python
param = {
	'n_estimators': [100, 200, 300],
	'learning_rate': [0.01, 0.05, 0.1],
	'max_depth': [3, 5, 7],
	'subsample': [0.8, 0.9, 1.0]
}
```

<aside>
💡

**가장 높게 나온 데이터 : Raw Data**

`Best Parameters`

- `learing_rate` : `0.05`
- `max_depth` : `5`
- `n_estimators` : `300`
- `subsample` : `0.9`

`Best Score (RMSLE)` : `0.09669434892860171`

</aside>

- 각 데이터 별 Best Parameters와 Best Score
    - Raw Data
    
    ```
    Best Parameters : {'learning_rate': 0.05, 'max_depth': 5, 'n_estimators': 300, 'subsample': 0.9}
    Best Score : 0.09669434892860171
    ```
    
    - StandardScaler
    
    ```
    Best Parameters : {'learning_rate': 0.1, 'max_depth': 3, 'n_estimators': 300, 'subsample': 0.8}
    Best Score : 0.10299486431662197
    ```
    
    - MinMaxScaler
    
    ```
    Best Parameters : {'learning_rate': 0.1, 'n_estimators': 200, 'num_leaves': 30, 'subsample': 0.8}
    Best Score : 0.10448370048638442
    ```
    
    - RobustScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 200}
    Best Score : 0.11252590373856475
    ```
    
- Feature Importance 그래프
    
    ![download.png](attachment:afe00eac-b8e8-4365-84ff-4679b5aaf2fe:download.png)
    

## XGBoost

---

XGBoost는 부스팅 기법을 개선한 고성능 머신러닝 라이브러리이다. 속도와 효율성이 높으며 그래디언트 부스팅 알고리즘을 기반으로 다양한 기능을 제공한다.

### XGBoost의 장단점

### XGBoost의 장점

- GradientBoost를 개선하여 학습 속도가 매우 빨라, 대규모 데이터셋에 적합함.
- 다양한 하이퍼파라미터를 설정하여 성능을 최적화할 수 있음.
- L1/L2/ 정규화와 조기 종료 등의 방법으로 과적합을 방지.

### XGBoost의 단점

- 대규모 데이터셋에는 메모리 소모가 커질 수 있음.
- 모델 성정 및 하이퍼파라미터 튜닝이 복잡할 수 있어, 초보자에게는 어려울 수 있음.
- 선형 회귀 문제보다는 비선형 데이터에 적합함.

### XGBoost를 사용하여 나타난 결과값

```python
param = {
	'n_estimators': [100, 200, 300],
	'learning_rate': [0.01, 0.05, 0.1],
	'max_depth': [3, 5, 7],
	'colsample_bytree': [0.8, 0.9, 1.0]
}
```

<aside>
💡

**가장 높게 나온 데이터 : Raw Data**

`Best Parameters`

- `colsample_bytree` : `0.9`
- `learning_rate` : `0.1`
- `max_depth` : `5`
- `n_estimators` : `200`

`Best Score (RMSLE)` : `0.09782854081356063`

</aside>

- 각 데이터 별 Best Parameters와 Best Score
    - Raw Data
    
    ```
    Best Parameters : {'colsample_bytree': 0.9, 'learning_rate': 0.1, 'max_depth': 5, 'n_estimators': 200}
    Best Score : 0.09782854081356063
    ```
    
    - StandardScaler
    
    ```
    Best Parameters : {'colsample_bytree': 0.8, 'learning_rate': 0.05, 'max_depth': 5, 'n_estimators': 300}
    Best Score : 0.10266053002552127
    ```
    
    - MinMaxScaler
    
    ```
    Best Parameters : {'colsample_bytree': 0.8, 'learning_rate': 0.05, 'max_depth': 5, 'n_estimators': 300}
    Best Score : 0.10266053002552127
    ```
    
    - RobustScaler
    
    ```
    Best Parameters : {'colsample_bytree': 0.8, 'learning_rate': 0.05, 'max_depth': 5, 'n_estimators': 300}
    Best Score : 0.10266053002552127
    ```
    
- Feature Importance 그래프
    
    ![download.png](attachment:2f0364fb-5c0b-47e1-95e9-7059ca40b894:download.png)
    

## LightGBM

---

LightGBM은 일반 GBM 계열의 균형 트리 분할(level-wise) 방식을 사용하지 않고, 리프 중심 트리 분할(leaf-wise) 방식을 사용하여 트리가 깊어지기 위해 소용되는 시간과 메모리를 절약하는 방식을 채택한 Gradient Boosting 알고리즘이다.

### LightGBM의 장단점

### LightGBM의 장점

- XGBoost보다 더 빠르고 메모리 효율적인 학습을 지원.
- 분산 학습을 지원하여 매우 큰 데이터셋도 효율적으로 처리할 수 있음.
- 범주형 변수를 원-핫 인코딩 없이 다룰 수 있어 데이터 전처리 부담이 적음.

### LightGBM의 단점

- 학습 속도가 빠르지만, 하이퍼파라미터 튜닝을 잘못하면 과적합될 수 있음.
- LightGBM은 대규모 데이터셋에서 빠르지만, 메모리 소모가 커질 수 있음.
- 데이터의 크기가 작을 때 과적합이 쉽게 일어남. (공식 문서에서 10,000개 이하의 데이터를 작다고 표현)

### LightGBM를 사용하여 나타난 결과값

```python
param = {
	'n_estimators': [100, 200, 300],
	'learning_rate': [0.01, 0.05, 0.1],
	'num_leaves': [20, 30, 40],
	'subsample': [0.8, 0.9, 1.0]
}
```

<aside>
💡

**가장 높게 나온 데이터 : Raw Data**

`Best Parameters`

- `max_depth` : `15`
- `min_samples_leaf` : `2`
- `min_samples_split` : `2`
- `n_estimators` : `300`

`Best Score (RMSLE)` : `0.10859456191822163`

</aside>

- 각 데이터 별 Best Parameters와 Best Score
    - Raw Data
    
    ```
    Best Parameters : {'learning_rate': 0.05, 'n_estimators': 300, 'num_leaves': 20, 'subsample': 0.8}
    Best Score : 0.09980283289547538
    ```
    
    - StandardScaler
    
    ```
    Best Parameters : {'learning_rate': 0.1, 'n_estimators': 200, 'num_leaves': 30, 'subsample': 0.8}
    Best Score : 0.10433081566340582
    ```
    
    - MinMaxScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 100}
    Best Score : 0.11262592437379748
    ```
    
    - RobustScaler
    
    ```
    Best Parameters : {'max_depth': 15, 'min_samples_leaf': 2, 'min_samples_split': 5, 'n_estimators': 200}
    Best Score : 0.11252590373856475
    ```
    
- Feature Importance 그래프
    
    ![download.png](attachment:2444235c-2c9a-4d03-bb30-d634757ff3f5:download.png)
    

## 모델링 분석 인사이트

---

- **RandomForest, GradientBoosting, XGBoost, LightGBM모델 중 XGBoost에서 score가 가장 낮게 나왔으므로, XGBoost가 가장 예측력이 좋은 모델이다.**
- **모든 모델에서 변수의 중요도는** `hour`**컬럼이 압도적으로 높은 값을 보여준다.**
- **그 다음으로** `workingday`, `temp`, `year`, `month` **순서대로 중요도를 보여준다.**

# 분석 결론

---

EDA를 통한 분석과 모델링을 통한 분석 인사이트를 통해 다음과 같은 분석 결론을 내릴 수 있었다.

## 출퇴근 시간, 직장인들을 위한 수요 증가 반영하기 위한 솔루션

---

출퇴근 시간에 자전거 대여량이 급증하므로, 이에 대한 공급 조절 및 운영 전략이 필요합니다. 이를 해결하기 위한 방안은 다음과 같다.

### **1. 출퇴근이 많은 장소(회사, 주택가 및 아파트)에 자전거 다수 배치**

- 대중교통과 연계된 자전거 정류장 확대 (지하철역, 버스 정류장 근처)
- 대여소 위치를 데이터 기반으로 최적화하여 수요가 많은 곳에 집중 배치
- AI 기반 실시간 수요 예측을 활용해 자전거 재배치 자동화

### **2. 모바일 앱을 활용한 예약 및 실시간 모니터링 시스템 도입**

- 모바일 앱에서 실시간으로 대여 가능 자전거 수를 확인하도록 시스템 개선
- 출퇴근 시간대 예약 기능 추가해 이용자의 대기 시간 단축
- AI 추천 시스템을 도입해 사용자 맞춤형 대여소 및 경로 안내

### **3. 출퇴근 시간대 할인 및 구독 플랜 프로모션 진행**

- 정기 구독 플랜 제공 (예: 월간 출퇴근 전용 패스)
- 피크 타임(출퇴근 시간) 전용 할인 쿠폰 제공하여 이용 활성화
- 기업 제휴를 통한 직원 전용 할인 및 복지 혜택 제공

## 날씨에 따른 자전거 수리 비용 절감

---

날씨가 악화될수록 대여량이 감소하는 경향을 보이며, 이와 함께 자전거의 손상 가능성도 증가한다. 이를 최소화하기 위한 방안은 다음과 같다.

### **1. 자전거 보관소 설치 및 보호 시설 강화**

- 우천 및 폭설 시 자전거를 안전하게 보관할 수 있도록 지붕형 대여소 확충
- 실시간 날씨 데이터를 활용한 대여소별 자동 보호 시스템 적용 (예: 강풍 시 자동 차단)
- 자전거 보관소 내 CCTV 및 보안 시스템 강화하여 도난 및 파손 방지

### **2. 자전거 유지보수 관리 강화**

- 주기적인 장비 점검 및 정비 일정을 자동화 (IoT 센서를 활용한 실시간 고장 감지)
- 악천후 이후 집중 정비 시스템 도입하여 장비 수명을 연장
- 날씨에 따른 부품 손상 분석을 통해 비용 절감 전략 수립

## 전국 단위 데이터 추가

---

현재 특정 지역 데이터를 기반으로 분석이 진행되고 있는 것으로 보이며, 보다 일반화된 수요 예측 모델을 만들기 위해 전국 단위 데이터 추가가 필요하다. 이를 위한 전략은 다음과 같다.

### **1. 다양한 도시로 서비스 확장**

- 대도시뿐만 아니라 중소도시에도 자전거 대여 시스템 확대
- 지역별 특성을 반영한 맞춤형 대여 시스템 도입 (예: 관광지 중심 자전거 대여소)
- 지역별 교통 인프라 및 이용 패턴을 분석하여 최적의 서비스 제공

### **2. 전국 단위 데이터 수집 및 활용**

- 각 지역별 기후, 인구밀도, 도로 환경 등을 반영한 수요 예측 모델 고도화
- 전국 데이터를 활용한 머신러닝 모델 재학습을 통해 예측 정확도 향상
- 정부 및 지자체와 협업하여 공공 데이터 연계 및 정책 지원 확보

### **3. 수요 기반 맞춤형 서비스 제공**

- 지역별 이용 패턴에 맞춘 요금제 및 대여 방식 도입 (예: 관광지에서는 시간제 요금제, 출퇴근 도시는 구독제)
- 스마트시티 연계 프로젝트로 자전거 공유 시스템을 도시 교통 정책과 통합
