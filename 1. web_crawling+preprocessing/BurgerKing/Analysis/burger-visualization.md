```python
import pandas as pd
import folium
import os
import numpy as np
from folium import plugins
import json
```

## folium 
- 수업 때 배운 folium의 geojson과 MarkerCluster을 이용하여, 서울 지역의 버거 분포 지도를 그려보자
- 파일 : 
    - *서울시 행정구역 시군구 정보 (좌표계_ WGS1984).csv*
    - *seoul_muncipalities_geo.json*
    - *burger_df.csv*


```python
burger_df = pd.read_csv("../2. preprocessing/burger_df.csv")
print(burger_df.shape)
burger_df
```

    (1002, 6)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지점명</th>
      <th>주소</th>
      <th>브랜드</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 812</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>997</th>
      <td>대전카이스트점</td>
      <td>대전광역시 유성구 대덕대로 535</td>
      <td>맥도날드</td>
      <td>대전광역시</td>
      <td>유성구</td>
      <td>대덕대로 535</td>
    </tr>
    <tr>
      <th>998</th>
      <td>김천평화DT점</td>
      <td>경상북도 김천시 자산로 199</td>
      <td>맥도날드</td>
      <td>경상북도</td>
      <td>김천시</td>
      <td>자산로 199</td>
    </tr>
    <tr>
      <th>999</th>
      <td>대구태전 DT</td>
      <td>대구광역시 북구 칠곡중앙대로 303</td>
      <td>맥도날드</td>
      <td>대구광역시</td>
      <td>북구</td>
      <td>칠곡중앙대로 303</td>
    </tr>
    <tr>
      <th>1000</th>
      <td>강남 2호점</td>
      <td>서울특별시 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
    </tr>
    <tr>
      <th>1001</th>
      <td>개금점</td>
      <td>부산광역시 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>맥도날드</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
    </tr>
  </tbody>
</table>
<p>1002 rows × 6 columns</p>
</div>




```python
loc_df = pd.read_csv("서울시 행정구역 시군구 정보 (좌표계_ WGS1984).csv", encoding='cp949')
loc_df[["시군구명_한글","위도","경도"]]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>시군구명_한글</th>
      <th>위도</th>
      <th>경도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>도봉구</td>
      <td>37.665861</td>
      <td>127.031767</td>
    </tr>
    <tr>
      <th>1</th>
      <td>은평구</td>
      <td>37.617612</td>
      <td>126.922700</td>
    </tr>
    <tr>
      <th>2</th>
      <td>동대문구</td>
      <td>37.583801</td>
      <td>127.050700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>동작구</td>
      <td>37.496504</td>
      <td>126.944307</td>
    </tr>
    <tr>
      <th>4</th>
      <td>금천구</td>
      <td>37.460097</td>
      <td>126.900155</td>
    </tr>
    <tr>
      <th>5</th>
      <td>구로구</td>
      <td>37.495486</td>
      <td>126.858121</td>
    </tr>
    <tr>
      <th>6</th>
      <td>종로구</td>
      <td>37.599100</td>
      <td>126.986149</td>
    </tr>
    <tr>
      <th>7</th>
      <td>강북구</td>
      <td>37.646995</td>
      <td>127.014716</td>
    </tr>
    <tr>
      <th>8</th>
      <td>중랑구</td>
      <td>37.595379</td>
      <td>127.093967</td>
    </tr>
    <tr>
      <th>9</th>
      <td>강남구</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>10</th>
      <td>강서구</td>
      <td>37.565762</td>
      <td>126.822656</td>
    </tr>
    <tr>
      <th>11</th>
      <td>중구</td>
      <td>37.557945</td>
      <td>126.994190</td>
    </tr>
    <tr>
      <th>12</th>
      <td>강동구</td>
      <td>37.549208</td>
      <td>127.146482</td>
    </tr>
    <tr>
      <th>13</th>
      <td>광진구</td>
      <td>37.548144</td>
      <td>127.085753</td>
    </tr>
    <tr>
      <th>14</th>
      <td>마포구</td>
      <td>37.562291</td>
      <td>126.908780</td>
    </tr>
    <tr>
      <th>15</th>
      <td>서초구</td>
      <td>37.476953</td>
      <td>127.037810</td>
    </tr>
    <tr>
      <th>16</th>
      <td>성북구</td>
      <td>37.606991</td>
      <td>127.023218</td>
    </tr>
    <tr>
      <th>17</th>
      <td>노원구</td>
      <td>37.655264</td>
      <td>127.077120</td>
    </tr>
    <tr>
      <th>18</th>
      <td>송파구</td>
      <td>37.504853</td>
      <td>127.114482</td>
    </tr>
    <tr>
      <th>19</th>
      <td>서대문구</td>
      <td>37.582037</td>
      <td>126.935667</td>
    </tr>
    <tr>
      <th>20</th>
      <td>양천구</td>
      <td>37.527062</td>
      <td>126.856153</td>
    </tr>
    <tr>
      <th>21</th>
      <td>영등포구</td>
      <td>37.520641</td>
      <td>126.913924</td>
    </tr>
    <tr>
      <th>22</th>
      <td>관악구</td>
      <td>37.465399</td>
      <td>126.943807</td>
    </tr>
    <tr>
      <th>23</th>
      <td>성동구</td>
      <td>37.550675</td>
      <td>127.040962</td>
    </tr>
    <tr>
      <th>24</th>
      <td>용산구</td>
      <td>37.531101</td>
      <td>126.981074</td>
    </tr>
  </tbody>
</table>
</div>




```python

```

    (1002, 6)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지점명</th>
      <th>주소</th>
      <th>브랜드</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 812</td>
    </tr>
  </tbody>
</table>
</div>




```python
grp_seoul = burger_df.loc[burger_df["주소1"]=="서울특별시",:]
grp_seoul
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지점명</th>
      <th>주소</th>
      <th>브랜드</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 812</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>985</th>
      <td>잠실역</td>
      <td>서울특별시 송파구 송파대로 558</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>송파구</td>
      <td>송파대로 558</td>
    </tr>
    <tr>
      <th>989</th>
      <td>등촌 DT</td>
      <td>서울특별시 강서구 양천로 546</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강서구</td>
      <td>양천로 546</td>
    </tr>
    <tr>
      <th>990</th>
      <td>마리오 아울렛</td>
      <td>서울특별시 금천구 디지털로 185 마리오아울렛1</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>금천구</td>
      <td>디지털로 185 마리오아울렛1</td>
    </tr>
    <tr>
      <th>996</th>
      <td>미아역</td>
      <td>서울특별시 강북구 도봉로 204 미아역 맥도날드</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강북구</td>
      <td>도봉로 204 미아역 맥도날드</td>
    </tr>
    <tr>
      <th>1000</th>
      <td>강남 2호점</td>
      <td>서울특별시 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
    </tr>
  </tbody>
</table>
<p>280 rows × 6 columns</p>
</div>




```python
seoul_data = pd.merge(grp_seoul, loc_df, left_on="주소2", right_on="시군구명_한글", how='left')
seoul_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지점명</th>
      <th>주소</th>
      <th>브랜드</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
      <th>순번</th>
      <th>시군구코드</th>
      <th>시군구명_한글</th>
      <th>시군구명_영문</th>
      <th>ESRI_PK</th>
      <th>위도</th>
      <th>경도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 812</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>275</th>
      <td>잠실역</td>
      <td>서울특별시 송파구 송파대로 558</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>송파구</td>
      <td>송파대로 558</td>
      <td>19</td>
      <td>11710</td>
      <td>송파구</td>
      <td>Songpa-gu</td>
      <td>19</td>
      <td>37.504853</td>
      <td>127.114482</td>
    </tr>
    <tr>
      <th>276</th>
      <td>등촌 DT</td>
      <td>서울특별시 강서구 양천로 546</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강서구</td>
      <td>양천로 546</td>
      <td>11</td>
      <td>11500</td>
      <td>강서구</td>
      <td>Gangseo-gu</td>
      <td>10</td>
      <td>37.565762</td>
      <td>126.822656</td>
    </tr>
    <tr>
      <th>277</th>
      <td>마리오 아울렛</td>
      <td>서울특별시 금천구 디지털로 185 마리오아울렛1</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>금천구</td>
      <td>디지털로 185 마리오아울렛1</td>
      <td>5</td>
      <td>11545</td>
      <td>금천구</td>
      <td>Geumcheon-gu</td>
      <td>4</td>
      <td>37.460097</td>
      <td>126.900155</td>
    </tr>
    <tr>
      <th>278</th>
      <td>미아역</td>
      <td>서울특별시 강북구 도봉로 204 미아역 맥도날드</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강북구</td>
      <td>도봉로 204 미아역 맥도날드</td>
      <td>8</td>
      <td>11305</td>
      <td>강북구</td>
      <td>Gangbuk-gu</td>
      <td>7</td>
      <td>37.646995</td>
      <td>127.014716</td>
    </tr>
    <tr>
      <th>279</th>
      <td>강남 2호점</td>
      <td>서울특별시 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
      <td>10</td>
      <td>11680</td>
      <td>강남구</td>
      <td>Gangnam-gu</td>
      <td>9</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
  </tbody>
</table>
<p>280 rows × 13 columns</p>
</div>




```python
xloc = list(seoul_data["위도"])
yloc = list(seoul_data["경도"])
pop_ups = list(seoul_data["지점명"])

dat_loc = np.array([xloc, yloc]).T
dat_loc[:10]
```




    array([[ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091],
           [ 37.4959854, 127.0664091]])




```python
with open('seoul_muncipalities_geo.json',mode='rt',encoding='utf-8') as f:
  geo = json.loads(f.read()) 
  f.close()

burger_map = folium.Map([37.5838699,127.0565831], zoom_start=11)
plugins.MarkerCluster(dat_loc, popups=pop_ups).add_to(burger_map)

folium.GeoJson(
    geo,
  name= 'seoul_municipalities' ).add_to(burger_map)

burger_map.save("burger_map_seoul.html")
burger_map

```








```python
import os 
os.listdir()
```




    ['seoul_muncipalities_geo.json',
     '.DS_Store',
     'corona_webdata_visualization.ipynb',
     'burger_map_seoul.html',
     '.ipynb_checkpoints',
     'burgerking_folium.ipynb',
     '서울시 행정구역 시군구 정보 (좌표계_ WGS1984).csv']




```python
burger_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1002 entries, 0 to 1001
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   지점명     1002 non-null   object
     1   주소      1002 non-null   object
     2   브랜드     1002 non-null   object
     3   주소1     1002 non-null   object
     4   주소2     1002 non-null   object
     5   주소3     1002 non-null   object
    dtypes: object(6)
    memory usage: 47.1+ KB



```python
burger_df["주소1+2"] = burger_df["주소1"]+" "+burger_df["주소2"]
burger_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>지점명</th>
      <th>주소</th>
      <th>브랜드</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
      <th>주소1+2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 812</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>997</th>
      <td>대전카이스트점</td>
      <td>대전광역시 유성구 대덕대로 535</td>
      <td>맥도날드</td>
      <td>대전광역시</td>
      <td>유성구</td>
      <td>대덕대로 535</td>
      <td>대전광역시 유성구</td>
    </tr>
    <tr>
      <th>998</th>
      <td>김천평화DT점</td>
      <td>경상북도 김천시 자산로 199</td>
      <td>맥도날드</td>
      <td>경상북도</td>
      <td>김천시</td>
      <td>자산로 199</td>
      <td>경상북도 김천시</td>
    </tr>
    <tr>
      <th>999</th>
      <td>대구태전 DT</td>
      <td>대구광역시 북구 칠곡중앙대로 303</td>
      <td>맥도날드</td>
      <td>대구광역시</td>
      <td>북구</td>
      <td>칠곡중앙대로 303</td>
      <td>대구광역시 북구</td>
    </tr>
    <tr>
      <th>1000</th>
      <td>강남 2호점</td>
      <td>서울특별시 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
      <td>서울특별시 강남구</td>
    </tr>
    <tr>
      <th>1001</th>
      <td>개금점</td>
      <td>부산광역시 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>맥도날드</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>부산광역시 부산진구</td>
    </tr>
  </tbody>
</table>
<p>1002 rows × 7 columns</p>
</div>



# PART I. 최종 DataFrame 생성

브랜드끼리의 상관관계, 버거지수와 인구/인구밀도/경제지표의 상관관계를 분석하기 위해서 최종적으로 원하는 DataFrame은 아래와 같이 생겼다.

|index|시+구,군|맥도날드|버거킹|KFC|롯데리아|버거지수|인구/인구밀도/경제지표|
|---|---|---|---|---|---|---|---|
|0|서울특별시 강남구| 11|7 |11 | 20|1.45 | 44865 |
|...|...|...|...|...|...|...|...|



```python
# 브랜드 종류 확인
burger_df["브랜드"].unique()
```




    array(['버거킹', 'KFC', '맥도날드'], dtype=object)




```python
# 브랜드별로 데이터 묶기
bking_grp = burger_df[burger_df["브랜드"]=="버거킹"]
kfc_grp = burger_df[burger_df["브랜드"]=="KFC"]
mc_grp = burger_df[burger_df["브랜드"]=="맥도날드"]
# lotte_grp = burger_df[burger_df["브랜드"]=="롯데리아"]

bking_grp.shape, kfc_grp.shape, mc_grp.shape # 갯수 확인
```




    ((410, 7), (187, 7), (405, 7))




```python
# 시구군별로 매장수 세기
bking_cnt =  bking_grp.groupby("주소1+2").count()["지점명"]
kfc_cnt = kfc_grp.groupby("주소1+2").count()["지점명"]
mc_cnt = mc_grp.groupby("주소1+2").count()["지점명"]
# lotte_cnt = lotte_grp.groupby("주소1+2").count()["지점명"]

bking_cnt.shape, kfc_cnt.shape, mc_cnt.shape
```




    ((129,), (90,), (132,))




```python
# column 이름을 Series.name 에 저장
bking_cnt.name = "버거킹"
kfc_cnt.name = "KFC"
mc_cnt.name = "맥도날드"
# lotte_cnt.name = "롯데리아"
```


```python
# Series 데이터 확인
print(bking_cnt)
print()
print(kfc_cnt)
print()
print(mc_cnt)
print()
# print(lotte_cnt)
# print()
```

    주소1+2
    강원도 강릉시     1
    강원도 속초시     1
    강원도 원주시     3
    강원도 춘천시     2
    강원도 홍천군     2
               ..
    충청남도 홍성군    1
    충청북도 음성군    1
    충청북도 진천군    1
    충청북도 청주시    7
    충청북도 충주시    1
    Name: 버거킹, Length: 129, dtype: int64
    
    주소1+2
    강원도 원주시         1
    강원도 춘천시         2
    경기도 고양시         5
    경기도 광명시         2
    경기도 구리시         1
                   ..
    제주특별자치도 서귀포시    1
    충청남도 당진시        1
    충청남도 아산시        1
    충청남도 천안시        3
    충청북도 청주시        2
    Name: KFC, Length: 90, dtype: int64
    
    주소1+2
    강원도 강릉시     1
    강원도 동해시     1
    강원도 속초시     1
    강원도 원주시     2
    강원도 춘천시     2
               ..
    충청남도 천안시    4
    충청남도 홍성군    1
    충청북도 제천시    1
    충청북도 청주시    6
    충청북도 충주시    1
    Name: 맥도날드, Length: 132, dtype: int64
    



```python
# Series 합치기
nstore_df = pd.concat([bking_cnt, kfc_cnt, mc_cnt], axis=1) # 시리즈 합치기, 합치면서 dtypes=float으로 변하는 것 같다
nstore_df = nstore_df.fillna(0) # 결측치 0으로 채우기 
nstore_df = nstore_df.astype("int") # 자료형 타입 int로 바꾸기

print(nstore_df.info())
nstore_df
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 145 entries, 강원도 강릉시 to 충청북도 제천시
    Data columns (total 3 columns):
     #   Column  Non-Null Count  Dtype
    ---  ------  --------------  -----
     0   버거킹     145 non-null    int64
     1   KFC     145 non-null    int64
     2   맥도날드    145 non-null    int64
    dtypes: int64(3)
    memory usage: 4.5+ KB
    None





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도 강릉시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 홍천군</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>부산광역시 수영구</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>부산광역시 영도구</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>울산광역시 울주군</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청남도 논산시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 제천시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 3 columns</p>
</div>




```python
len(nstore_df.index), len(set(nstore_df.index)) # 시군구 중복 확인
```




    (145, 145)




```python
# 버거지수 컬럼 만들기
nstore_df["버거지수"] = nstore_df["버거킹"]+nstore_df["KFC"]+nstore_df["맥도날드"] #/nstore_df["롯데리아"] 
nstore_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도 강릉시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
    </tr>
    <tr>
      <th>강원도 홍천군</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>부산광역시 수영구</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>부산광역시 영도구</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>울산광역시 울주군</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청남도 논산시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 제천시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 4 columns</p>
</div>




```python
# 인덱스를 주소1(광역시,도), 주소2(시,군,구)로 나누기
nstore_df["주소1"] = nstore_df.index
nstore_df[["주소1","주소2"]] = nstore_df["주소1"].str.strip().str.split().tolist()
nstore_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
      <th>주소1</th>
      <th>주소2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도 강릉시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>강릉시</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>속초시</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>원주시</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>춘천시</td>
    </tr>
    <tr>
      <th>강원도 홍천군</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>강원도</td>
      <td>홍천군</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>부산광역시 수영구</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>부산광역시</td>
      <td>수영구</td>
    </tr>
    <tr>
      <th>부산광역시 영도구</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>부산광역시</td>
      <td>영도구</td>
    </tr>
    <tr>
      <th>울산광역시 울주군</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>울산광역시</td>
      <td>울주군</td>
    </tr>
    <tr>
      <th>충청남도 논산시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청남도</td>
      <td>논산시</td>
    </tr>
    <tr>
      <th>충청북도 제천시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청북도</td>
      <td>제천시</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 6 columns</p>
</div>



#### 주소2(시,군,구) 중복 확인
- 이 후에 주소2를 기준으로 인구나 인구밀도 정보를 넣어도 되는지 확인한다
- 결과 : 중복있음. 주소2를 기준으로 인구나 인구밀도 정보를 넣으면 안된다. 
- 대신, 인덱스 값은 중복이 없으니 기준으로 사용할 것이다. reset_index를 통해 숫자형 인덱스를 지정하고 기존의 값을 새로운 컬럼에 저장한다. 

#### cf. 중복 제거 방식 2가지
1. set 사용하여 길이 비교
2. pandas.drop_duplicates() 메서드 사용


```python
# 주소2(시,군,구) 중복 확인
print(len(nstore_df['주소2']))
print(len(set(nstore_df["주소2"]))) # 중복 제거 - 1. set 사용
print(len(nstore_df["주소2"].drop_duplicates())) # 중복 제거 - 2. pandas.drop_duplicates() 사용

nstore_df['주소2'].value_counts().head(10) # 어느 도/시에서 겹치는거지? 아래의 셀에서 확인
```

    145
    127
    127





    중구     5
    동구     5
    남구     4
    북구     4
    서구     4
    강서구    2
    음성군    1
    김해시    1
    순천시    1
    익산시    1
    Name: 주소2, dtype: int64




```python
# 주소2(시,군,구) 이름이 겹치는 주소1(시,도) 
list_duplicates = list(nstore_df['주소2'].value_counts().head(6).index)
list_duplicates

for one in list_duplicates :
    print(nstore_df[nstore_df["주소2"]==one]["주소1"], "\n")

```

    대구광역시 중구    대구광역시
    대전광역시 중구    대전광역시
    서울특별시 중구    서울특별시
    울산광역시 중구    울산광역시
    인천광역시 중구    인천광역시
    Name: 주소1, dtype: object 
    
    광주광역시 동구    광주광역시
    대구광역시 동구    대구광역시
    대전광역시 동구    대전광역시
    울산광역시 동구    울산광역시
    부산광역시 동구    부산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 남구    광주광역시
    대구광역시 남구    대구광역시
    부산광역시 남구    부산광역시
    울산광역시 남구    울산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 북구    광주광역시
    대구광역시 북구    대구광역시
    부산광역시 북구    부산광역시
    울산광역시 북구    울산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 서구    광주광역시
    대구광역시 서구    대구광역시
    대전광역시 서구    대전광역시
    인천광역시 서구    인천광역시
    Name: 주소1, dtype: object 
    
    서울특별시 강서구    서울특별시
    부산광역시 강서구    부산광역시
    Name: 주소1, dtype: object 
    


#### cf. 인덱싱 재설정 `reset_index(drop=False, inplace=False)`
- drop : 기존의 인덱스 값을 삭제할 것인가?
    - False(default) : 삭제하지 않고 새로운 컬럼에 저장
    - True : 삭제
- inplace : 기존의 데이터프레임에 저장할 것인가?
    - False(default) : 기존의 데이터프레임에는 저장하지 않고 출력만 
    - True : 기존의 데이터프레임에 저장


```python
# 인덱싱 재설정
# nstore_df.reset_index(drop=False, inplace=True) 
nstore_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
      <th>주소1</th>
      <th>주소2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강원도 강릉시</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>강릉시</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강원도 속초시</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>속초시</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강원도 원주시</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>원주시</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강원도 춘천시</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>춘천시</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강원도 홍천군</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>강원도</td>
      <td>홍천군</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>140</th>
      <td>부산광역시 수영구</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>부산광역시</td>
      <td>수영구</td>
    </tr>
    <tr>
      <th>141</th>
      <td>부산광역시 영도구</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>부산광역시</td>
      <td>영도구</td>
    </tr>
    <tr>
      <th>142</th>
      <td>울산광역시 울주군</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>울산광역시</td>
      <td>울주군</td>
    </tr>
    <tr>
      <th>143</th>
      <td>충청남도 논산시</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청남도</td>
      <td>논산시</td>
    </tr>
    <tr>
      <th>144</th>
      <td>충청북도 제천시</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청북도</td>
      <td>제천시</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 7 columns</p>
</div>



# PART II. 버거지수와 경제 지표의 상관관계
## 경제 지표 가져오기
- 출처 : [통계청](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1C86&conn_path=I2)  
- 파일명 : *시도별_1인당_지역내총생산\__지역총소득\__개인소득_20210702225439.csv* 
- 설명 : (2019년 기준) 전국 16개 시도별 `1인당 지역내총생산`, `1인당 지역총소득`,`1인당 개인소득`,`1인당 민간소비`


```python
eco_index = pd.read_csv("시도별_1인당_지역내총생산__지역총소득__개인소득_20210702225439.csv", encoding='cp949', skiprows=1)
eco_index
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>전국</td>
      <td>37208</td>
      <td>37530</td>
      <td>20400</td>
      <td>17962</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>44865</td>
      <td>49121</td>
      <td>23440</td>
      <td>21891</td>
    </tr>
    <tr>
      <th>2</th>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
    </tr>
    <tr>
      <th>3</th>
      <td>대구광역시</td>
      <td>23744</td>
      <td>27798</td>
      <td>19210</td>
      <td>17850</td>
    </tr>
    <tr>
      <th>4</th>
      <td>인천광역시</td>
      <td>30425</td>
      <td>32571</td>
      <td>19332</td>
      <td>16451</td>
    </tr>
    <tr>
      <th>5</th>
      <td>광주광역시</td>
      <td>27548</td>
      <td>30964</td>
      <td>20532</td>
      <td>18231</td>
    </tr>
    <tr>
      <th>6</th>
      <td>대전광역시</td>
      <td>28364</td>
      <td>31548</td>
      <td>20498</td>
      <td>18025</td>
    </tr>
    <tr>
      <th>7</th>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세종특별자치시</td>
      <td>35826</td>
      <td>36983</td>
      <td>19789</td>
      <td>16762</td>
    </tr>
    <tr>
      <th>9</th>
      <td>경기도</td>
      <td>36133</td>
      <td>38466</td>
      <td>20482</td>
      <td>17399</td>
    </tr>
    <tr>
      <th>10</th>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>11</th>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
    </tr>
    <tr>
      <th>12</th>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전라북도</td>
      <td>28740</td>
      <td>28260</td>
      <td>18725</td>
      <td>16022</td>
    </tr>
    <tr>
      <th>14</th>
      <td>전라남도</td>
      <td>43323</td>
      <td>35532</td>
      <td>18711</td>
      <td>16104</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경상북도</td>
      <td>40272</td>
      <td>34873</td>
      <td>18611</td>
      <td>16217</td>
    </tr>
    <tr>
      <th>16</th>
      <td>경상남도</td>
      <td>33690</td>
      <td>32140</td>
      <td>18939</td>
      <td>16426</td>
    </tr>
    <tr>
      <th>17</th>
      <td>제주특별자치도</td>
      <td>30720</td>
      <td>30834</td>
      <td>18734</td>
      <td>16953</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 행정구역 이름이 같은지 확인
print(set(eco_index["시도별"][1:]) == set(nstore_df["주소1"].unique()))
```

    True


### 두가지 방식
경제지표가 __시군구 단위__가 아니라 __시도 단위__이기 때문에 분석 방법에 <U>두가지</U>가 있다.  
#### 1. `nstore_df`에 `eco_index`를 join하는 방식
- 어떤 구/군인가에 상관없이 동일한 시/도에 포함되면 동일한 경제지표를 가지게 된다.  
- 방식 2.보다는 구/군에 따른 데이터 수가 많기 때문에 산점도를 그렸을 때 더 세부적인 버거지수 비교가 가능하다.
- 시의 경제 지표가 그에 포함된 구의 경제지표를 대표하기엔 다소 무리가 있다. 각 시/도 안에서도 경제적인 수준의 편차가 존재하기 때문에.  

#### 2. `eco_index`에 `nstore_df`를 join하는 방식
- 해당 시/도당 하나의 버거지수
- 버거지수-경제지표 산점도에 데이터 수가 줄어든다.
- 논리적으로는 이 방식이 더 맞아보인다.

## 방식 1.  nstore_df에 eco_index를 join한다.


```python
df_1 = pd.merge(nstore_df, eco_index, left_on="주소1", right_on='시도별', how='left')
df_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강원도 강릉시</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>강릉시</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강원도 속초시</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>강원도</td>
      <td>속초시</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강원도 원주시</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강원도 춘천시</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강원도 홍천군</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>강원도</td>
      <td>홍천군</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>140</th>
      <td>부산광역시 수영구</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>부산광역시</td>
      <td>수영구</td>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
    </tr>
    <tr>
      <th>141</th>
      <td>부산광역시 영도구</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>부산광역시</td>
      <td>영도구</td>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
    </tr>
    <tr>
      <th>142</th>
      <td>울산광역시 울주군</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>울산광역시</td>
      <td>울주군</td>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
    </tr>
    <tr>
      <th>143</th>
      <td>충청남도 논산시</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청남도</td>
      <td>논산시</td>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
    </tr>
    <tr>
      <th>144</th>
      <td>충청북도 제천시</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>충청북도</td>
      <td>제천시</td>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 12 columns</p>
</div>




```python
df_1.columns
```




    Index(['index', '버거킹', 'KFC', '맥도날드', '버거지수', '주소1', '주소2', '시도별',
           '1인당 지역내총생산', '1인당 지역총소득', '1인당 개인소득', '1인당 민간소비'],
          dtype='object')




```python
import plotly.express as px

px.scatter(df_1, x='버거지수', y='1인당 지역내총생산', trendline='ols')
# px.scatter(df_1, x='버거지수', y='1인당 지역총소득', color='시도별')
```


<div>                            <div id="1f580160-97ff-41b9-a2f2-71730353affc" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("1f580160-97ff-41b9-a2f2-71730353affc")) {                    Plotly.newPlot(                        "1f580160-97ff-41b9-a2f2-71730353affc",                        [{"hovertemplate":"\ubc84\uac70\uc9c0\uc218=%{x}<br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","symbol":"circle"},"mode":"markers","name":"","orientation":"v","showlegend":false,"type":"scatter","x":[2,2,6,6,2,25,2,8,2,6,6,9,9,2,16,23,28,6,7,3,9,5,1,6,18,13,5,5,11,2,5,13,5,11,3,8,7,21,5,7,11,3,1,3,1,12,7,5,1,10,5,2,15,3,5,9,3,13,5,2,12,10,6,6,3,6,5,11,6,5,5,6,10,29,10,7,14,9,8,9,8,11,6,8,10,11,10,21,4,10,19,11,13,8,9,13,13,9,11,3,3,3,4,12,5,8,6,10,7,3,1,4,2,4,3,5,4,15,2,6,3,2,2,12,2,1,1,15,2,1,4,2,1,1,1,1,1,2,1,2,3,1,1,1,1],"xaxis":"x","y":[32061,32061,32061,32061,32061,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,36133,33690,33690,33690,33690,33690,33690,40272,40272,40272,40272,40272,40272,40272,40272,27548,27548,27548,27548,27548,23744,23744,23744,23744,23744,23744,23744,23744,28364,28364,28364,28364,27409,27409,27409,27409,27409,27409,27409,27409,27409,27409,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,44865,65352,65352,65352,65352,30425,30425,30425,30425,30425,30425,30425,43323,43323,43323,43323,43323,43323,28740,28740,28740,28740,30720,52402,52402,52402,52402,52402,42653,42653,42653,42653,35826,30720,52402,32061,36133,33690,33690,40272,28364,27409,27409,27409,27409,65352,52402,42653],"yaxis":"y"},{"hovertemplate":"<b>OLS trendline</b><br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0 = 47.2341 * \ubc84\uac70\uc9c0\uc218 + 36935<br>R<sup>2</sup>=0.000825<br><br>\ubc84\uac70\uc9c0\uc218=%{x}<br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0=%{y} <b>(trend)</b><extra></extra>","legendgroup":"","marker":{"color":"#636efa","symbol":"circle"},"mode":"lines","name":"","showlegend":false,"type":"scatter","x":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,4,4,4,4,4,4,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,6,6,6,6,6,6,6,6,6,6,6,6,6,6,7,7,7,7,7,7,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,11,11,11,11,11,11,11,11,12,12,12,12,13,13,13,13,13,13,14,15,15,15,16,18,19,21,21,23,25,28,29],"xaxis":"x","y":[36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,36982.20257062773,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37029.43667478057,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37076.6707789334,37123.904883086245,37123.904883086245,37123.904883086245,37123.904883086245,37123.904883086245,37123.904883086245,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37171.13898723908,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37218.37309139192,37265.607195544755,37265.607195544755,37265.607195544755,37265.607195544755,37265.607195544755,37265.607195544755,37312.8412996976,37312.8412996976,37312.8412996976,37312.8412996976,37312.8412996976,37312.8412996976,37312.8412996976,37360.07540385043,37360.07540385043,37360.07540385043,37360.07540385043,37360.07540385043,37360.07540385043,37360.07540385043,37360.07540385043,37407.309508003265,37407.309508003265,37407.309508003265,37407.309508003265,37407.309508003265,37407.309508003265,37407.309508003265,37407.309508003265,37454.54361215611,37454.54361215611,37454.54361215611,37454.54361215611,37454.54361215611,37454.54361215611,37454.54361215611,37454.54361215611,37501.77771630894,37501.77771630894,37501.77771630894,37501.77771630894,37549.01182046178,37549.01182046178,37549.01182046178,37549.01182046178,37549.01182046178,37549.01182046178,37596.24592461462,37643.48002876746,37643.48002876746,37643.48002876746,37690.71413292029,37785.18234122597,37832.4164453788,37926.88465368448,37926.88465368448,38021.352861990155,38115.82107029583,38257.52338275434,38304.75748690718],"yaxis":"y"}],                        {"legend":{"tracegroupgap":0},"margin":{"t":60},"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"\ubc84\uac70\uc9c0\uc218"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0"}}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('1f580160-97ff-41b9-a2f2-71730353affc');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


[comments] 버거지수가 달라도 동일한 시,도 안에 있으면 동일한 경제 지표로 표시되기 때문에 의미있는 결과를 얻을 수 없었다.  
이번 방식은 실패!

## 방식 2. `eco_index`에 `nstore_df`를 join하는 방식


```python
eco_index
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>전국</td>
      <td>37208</td>
      <td>37530</td>
      <td>20400</td>
      <td>17962</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>44865</td>
      <td>49121</td>
      <td>23440</td>
      <td>21891</td>
    </tr>
    <tr>
      <th>2</th>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
    </tr>
    <tr>
      <th>3</th>
      <td>대구광역시</td>
      <td>23744</td>
      <td>27798</td>
      <td>19210</td>
      <td>17850</td>
    </tr>
    <tr>
      <th>4</th>
      <td>인천광역시</td>
      <td>30425</td>
      <td>32571</td>
      <td>19332</td>
      <td>16451</td>
    </tr>
    <tr>
      <th>5</th>
      <td>광주광역시</td>
      <td>27548</td>
      <td>30964</td>
      <td>20532</td>
      <td>18231</td>
    </tr>
    <tr>
      <th>6</th>
      <td>대전광역시</td>
      <td>28364</td>
      <td>31548</td>
      <td>20498</td>
      <td>18025</td>
    </tr>
    <tr>
      <th>7</th>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세종특별자치시</td>
      <td>35826</td>
      <td>36983</td>
      <td>19789</td>
      <td>16762</td>
    </tr>
    <tr>
      <th>9</th>
      <td>경기도</td>
      <td>36133</td>
      <td>38466</td>
      <td>20482</td>
      <td>17399</td>
    </tr>
    <tr>
      <th>10</th>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>11</th>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
    </tr>
    <tr>
      <th>12</th>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전라북도</td>
      <td>28740</td>
      <td>28260</td>
      <td>18725</td>
      <td>16022</td>
    </tr>
    <tr>
      <th>14</th>
      <td>전라남도</td>
      <td>43323</td>
      <td>35532</td>
      <td>18711</td>
      <td>16104</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경상북도</td>
      <td>40272</td>
      <td>34873</td>
      <td>18611</td>
      <td>16217</td>
    </tr>
    <tr>
      <th>16</th>
      <td>경상남도</td>
      <td>33690</td>
      <td>32140</td>
      <td>18939</td>
      <td>16426</td>
    </tr>
    <tr>
      <th>17</th>
      <td>제주특별자치도</td>
      <td>30720</td>
      <td>30834</td>
      <td>18734</td>
      <td>16953</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_nstore_df = nstore_df.groupby("주소1").sum()[["버거킹","KFC","맥도날드"]] # + "롯데리아"
new_nstore_df["버거지수"] = new_nstore_df["버거킹"]+new_nstore_df["KFC"]+new_nstore_df["맥도날드"] #/new_nstore_df["롯데리아"]
new_nstore_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
    </tr>
    <tr>
      <th>주소1</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도</th>
      <td>9</td>
      <td>3</td>
      <td>7</td>
      <td>19</td>
    </tr>
    <tr>
      <th>경기도</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>246</td>
    </tr>
    <tr>
      <th>경상남도</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>57</td>
    </tr>
    <tr>
      <th>경상북도</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>44</td>
    </tr>
    <tr>
      <th>광주광역시</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>28</td>
    </tr>
    <tr>
      <th>대구광역시</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>55</td>
    </tr>
    <tr>
      <th>대전광역시</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>32</td>
    </tr>
    <tr>
      <th>부산광역시</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>70</td>
    </tr>
    <tr>
      <th>서울특별시</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>280</td>
    </tr>
    <tr>
      <th>세종특별자치시</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>울산광역시</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>21</td>
    </tr>
    <tr>
      <th>인천광역시</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>52</td>
    </tr>
    <tr>
      <th>전라남도</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>17</td>
    </tr>
    <tr>
      <th>전라북도</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>26</td>
    </tr>
    <tr>
      <th>제주특별자치도</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>10</td>
    </tr>
    <tr>
      <th>충청남도</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>24</td>
    </tr>
    <tr>
      <th>충청북도</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_2 = pd.merge(new_nstore_df, eco_index, left_on=new_nstore_df.index, right_on='시도별', how='left')
df_2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
      <th>버거지수</th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9</td>
      <td>3</td>
      <td>7</td>
      <td>19</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>246</td>
      <td>경기도</td>
      <td>36133</td>
      <td>38466</td>
      <td>20482</td>
      <td>17399</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>57</td>
      <td>경상남도</td>
      <td>33690</td>
      <td>32140</td>
      <td>18939</td>
      <td>16426</td>
    </tr>
    <tr>
      <th>3</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>44</td>
      <td>경상북도</td>
      <td>40272</td>
      <td>34873</td>
      <td>18611</td>
      <td>16217</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>28</td>
      <td>광주광역시</td>
      <td>27548</td>
      <td>30964</td>
      <td>20532</td>
      <td>18231</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>55</td>
      <td>대구광역시</td>
      <td>23744</td>
      <td>27798</td>
      <td>19210</td>
      <td>17850</td>
    </tr>
    <tr>
      <th>6</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>32</td>
      <td>대전광역시</td>
      <td>28364</td>
      <td>31548</td>
      <td>20498</td>
      <td>18025</td>
    </tr>
    <tr>
      <th>7</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>70</td>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
    </tr>
    <tr>
      <th>8</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>280</td>
      <td>서울특별시</td>
      <td>44865</td>
      <td>49121</td>
      <td>23440</td>
      <td>21891</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>세종특별자치시</td>
      <td>35826</td>
      <td>36983</td>
      <td>19789</td>
      <td>16762</td>
    </tr>
    <tr>
      <th>10</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>21</td>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
    </tr>
    <tr>
      <th>11</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>52</td>
      <td>인천광역시</td>
      <td>30425</td>
      <td>32571</td>
      <td>19332</td>
      <td>16451</td>
    </tr>
    <tr>
      <th>12</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>17</td>
      <td>전라남도</td>
      <td>43323</td>
      <td>35532</td>
      <td>18711</td>
      <td>16104</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>26</td>
      <td>전라북도</td>
      <td>28740</td>
      <td>28260</td>
      <td>18725</td>
      <td>16022</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>10</td>
      <td>제주특별자치도</td>
      <td>30720</td>
      <td>30834</td>
      <td>18734</td>
      <td>16953</td>
    </tr>
    <tr>
      <th>15</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>24</td>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>20</td>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
    </tr>
  </tbody>
</table>
</div>




```python
import plotly.express as px

px.scatter(df_2, x='버거지수', y='1인당 지역내총생산', trendline='ols')
```


<div>                            <div id="768f16cb-4c6b-47c8-bacd-884001faccdf" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("768f16cb-4c6b-47c8-bacd-884001faccdf")) {                    Plotly.newPlot(                        "768f16cb-4c6b-47c8-bacd-884001faccdf",                        [{"hovertemplate":"\ubc84\uac70\uc9c0\uc218=%{x}<br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","symbol":"circle"},"mode":"markers","name":"","orientation":"v","showlegend":false,"type":"scatter","x":[19,246,57,44,28,55,32,70,280,1,21,52,17,26,10,24,20],"xaxis":"x","y":[32061,36133,33690,40272,27548,23744,28364,27409,44865,35826,65352,30425,43323,28740,30720,52402,42653],"yaxis":"y"},{"hovertemplate":"<b>OLS trendline</b><br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0 = 7.93895 * \ubc84\uac70\uc9c0\uc218 + 36210.1<br>R<sup>2</sup>=0.003505<br><br>\ubc84\uac70\uc9c0\uc218=%{x}<br>1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0=%{y} <b>(trend)</b><extra></extra>","legendgroup":"","marker":{"color":"#636efa","symbol":"circle"},"mode":"lines","name":"","showlegend":false,"type":"scatter","x":[1,10,17,19,20,21,24,26,28,32,44,52,55,57,70,246,280],"xaxis":"x","y":[36218.0665791063,36289.51715108268,36345.089818175424,36360.96772305906,36368.90667550088,36376.845627942705,36400.66248526816,36416.5403901518,36432.418295035444,36464.174104802725,36559.44153410457,36622.95315363913,36646.77001096459,36662.64791584823,36765.85429759189,38163.10992735225,38433.03431037413],"yaxis":"y"}],                        {"legend":{"tracegroupgap":0},"margin":{"t":60},"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"\ubc84\uac70\uc9c0\uc218"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"1\uc778\ub2f9 \uc9c0\uc5ed\ub0b4\ucd1d\uc0dd\uc0b0"}}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('768f16cb-4c6b-47c8-bacd-884001faccdf');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


[comments] 롯데리아 없이도 의미있는 결과를 기대했는데, 데이터포인트도 많지 않고 상관관계를 보여줄만한 그래프, R값이 나오지 않았다... 롯데리아 결과가 나오면 다시 해봐야겠다.

# PART III. 버거지수와 인구수/인구밀도의 상관관계
롯데리아 결과가 나오고 해야겠다...

## 인구수/인구밀도 데이터 가져오기
- 출처 : [통계청](https://kosis.kr/statHtml/statHtml.do?orgId=315&tblId=TX_315_2009_H1001&conn_path=I2) ← 이거쓰면 될 것 같긴한데.. 확실하진 않다! 더 찾아보기

# PART IV. 브랜드 사이의 상관관계
KFC가 많은 곳에 버거킹도 많지 않을까? 버거킹이 적은 곳엔 롯데리아가 많지 않을까? 하는 물음에 답을 해줄 수 있을 것이다.


```python
brand_df = nstore_df.set_index(nstore_df["index"])[["버거킹","KFC","맥도날드"]]
brand_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도 강릉시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>강원도 홍천군</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>부산광역시 수영구</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>부산광역시 영도구</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>울산광역시 울주군</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청남도 논산시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 제천시</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 3 columns</p>
</div>




```python
brand_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>145.000000</td>
      <td>145.000000</td>
      <td>145.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.827586</td>
      <td>1.289655</td>
      <td>2.793103</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.328402</td>
      <td>1.585162</td>
      <td>2.341938</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>4.000000</td>
      <td>2.000000</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>12.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 한글 패치
import matplotlib.pyplot as plt
from matplotlib import font_manager, rc
import numpy as np

plt.rc('font', family='AppleGothic')
```


```python
plt.bar(brand_df.sum().index, brand_df.sum())
```




    <BarContainer object of 3 artists>




    
![png](output_45_1.png)
    



```python
# 수치로 확인
brand_df.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>버거킹</th>
      <th>KFC</th>
      <th>맥도날드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>버거킹</th>
      <td>1.000000</td>
      <td>0.655219</td>
      <td>0.734599</td>
    </tr>
    <tr>
      <th>KFC</th>
      <td>0.655219</td>
      <td>1.000000</td>
      <td>0.659753</td>
    </tr>
    <tr>
      <th>맥도날드</th>
      <td>0.734599</td>
      <td>0.659753</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMK = brand_df.to_numpy().T
BMK.shape # index - 0:버거킹, 1:KFC, 2:맥도날드
```




    (3, 145)




```python
#매장별 상관관계 분석하기
import scipy.stats
import numpy as np

fig = plt.figure(figsize=(12,12))

def plot_nstores3(b1, b2, label1, label2):
    plt.scatter(brand_df[b1] + np.random.random(len(brand_df)),
                brand_df[b2] + np.random.random(len(brand_df)),
                edgecolor='none', alpha=0.75, s=6, c='black')
    plt.xlim(-1, 15 if (b1 != '롯데리아') & (b1 != '맘스터치') else 35)
    plt.ylim(-1, 15 if (b2 != '롯데리아') & (b2 != '맘스터치') else 35)
    plt.xlabel(label1)
    plt.ylabel(label2)

    r = scipy.stats.pearsonr(brand_df[b1], brand_df[b2])

    if r[0]>=0.75:
        color='red'
    elif r[0]<0.5:
        color='blue'
    else:
        color='black'

    plt.annotate('r={:.3f}'.format(r[0]), (9 if (b1 != '롯데리아') & (b1 != '맘스터치') else 20,
                                          12.5 if (b2 != '롯데리아') &(b2 != '맘스터치') else 20),
                 fontsize=14, color=color)
bgbrands = [
            ('버거킹', '버거킹'), ('KFC', 'KFC'),('맥도날드', '맥도날드')
]

for a in range(len(bgbrands) - 1):
    for b in range(1, len(bgbrands)):
        if a >= b:
            continue
        ax = fig.add_subplot(len(bgbrands)-1, len(bgbrands)-1, a * 2 + b)
        acol, alabel = bgbrands[a]
        bcol, blabel = bgbrands[b]
        plot_nstores3(bcol, acol, blabel, alabel)

plt.tight_layout()

```


    
![png](output_48_0.png)
    



```python

```