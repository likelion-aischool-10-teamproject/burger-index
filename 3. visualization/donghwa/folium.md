```python
import os
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

import folium
from folium import plugins
import json
```


```python
input_df_path = '../2. preprocessing/output/'
input_path = './input/'
output_path = './output/'
```

## folium 
- 수업 때 배운 folium의 geojson과 MarkerCluster을 이용하여, 서울 지역의 버거매장 분포 지도를 그려보자
- 파일 : 
    - *서울시 행정구역 시군구 정보 (좌표계_ WGS1984).csv*
    - *seoul_muncipalities_geo.json*
    - *burger_df.csv*


```python
burger_df = pd.read_csv(input_df_path+"burger_df.csv")
print(burger_df.shape)
burger_df
```

    (2306, 6)





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
      <th>2301</th>
      <td>원주일산</td>
      <td>강원도 원주시 천사로 213</td>
      <td>롯데리아</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>천사로 213</td>
    </tr>
    <tr>
      <th>2302</th>
      <td>성남</td>
      <td>경기도 성남시 수정구 수정로 181</td>
      <td>롯데리아</td>
      <td>경기도</td>
      <td>성남시</td>
      <td>수정구 수정로 181</td>
    </tr>
    <tr>
      <th>2303</th>
      <td>대전중앙</td>
      <td>대전광역시 중구 선화동 3-3번지</td>
      <td>롯데리아</td>
      <td>대전광역시</td>
      <td>중구</td>
      <td>선화동 3-3번지</td>
    </tr>
    <tr>
      <th>2304</th>
      <td>홈서비스과천</td>
      <td>경기도 과천시 별양동 19-4</td>
      <td>롯데리아</td>
      <td>경기도</td>
      <td>과천시</td>
      <td>별양동 19-4</td>
    </tr>
    <tr>
      <th>2305</th>
      <td>홈서비스부암(부산역)</td>
      <td>부산광역시 부산진구 부암동 722-3번지</td>
      <td>롯데리아</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>부암동 722-3번지</td>
    </tr>
  </tbody>
</table>
<p>2306 rows × 6 columns</p>
</div>




```python
loc_df = pd.read_csv(input_path+"서울시 행정구역 시군구 정보 (좌표계_ WGS1984).csv", encoding='cp949')
loc_df = loc_df[["시군구코드","시군구명_한글", "위도","경도"]]
loc_df
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
      <th>시군구코드</th>
      <th>시군구명_한글</th>
      <th>위도</th>
      <th>경도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11320</td>
      <td>도봉구</td>
      <td>37.665861</td>
      <td>127.031767</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11380</td>
      <td>은평구</td>
      <td>37.617612</td>
      <td>126.922700</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11230</td>
      <td>동대문구</td>
      <td>37.583801</td>
      <td>127.050700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11590</td>
      <td>동작구</td>
      <td>37.496504</td>
      <td>126.944307</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11545</td>
      <td>금천구</td>
      <td>37.460097</td>
      <td>126.900155</td>
    </tr>
    <tr>
      <th>5</th>
      <td>11530</td>
      <td>구로구</td>
      <td>37.495486</td>
      <td>126.858121</td>
    </tr>
    <tr>
      <th>6</th>
      <td>11110</td>
      <td>종로구</td>
      <td>37.599100</td>
      <td>126.986149</td>
    </tr>
    <tr>
      <th>7</th>
      <td>11305</td>
      <td>강북구</td>
      <td>37.646995</td>
      <td>127.014716</td>
    </tr>
    <tr>
      <th>8</th>
      <td>11260</td>
      <td>중랑구</td>
      <td>37.595379</td>
      <td>127.093967</td>
    </tr>
    <tr>
      <th>9</th>
      <td>11680</td>
      <td>강남구</td>
      <td>37.495985</td>
      <td>127.066409</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11500</td>
      <td>강서구</td>
      <td>37.565762</td>
      <td>126.822656</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11140</td>
      <td>중구</td>
      <td>37.557945</td>
      <td>126.994190</td>
    </tr>
    <tr>
      <th>12</th>
      <td>11740</td>
      <td>강동구</td>
      <td>37.549208</td>
      <td>127.146482</td>
    </tr>
    <tr>
      <th>13</th>
      <td>11215</td>
      <td>광진구</td>
      <td>37.548144</td>
      <td>127.085753</td>
    </tr>
    <tr>
      <th>14</th>
      <td>11440</td>
      <td>마포구</td>
      <td>37.562291</td>
      <td>126.908780</td>
    </tr>
    <tr>
      <th>15</th>
      <td>11650</td>
      <td>서초구</td>
      <td>37.476953</td>
      <td>127.037810</td>
    </tr>
    <tr>
      <th>16</th>
      <td>11290</td>
      <td>성북구</td>
      <td>37.606991</td>
      <td>127.023218</td>
    </tr>
    <tr>
      <th>17</th>
      <td>11350</td>
      <td>노원구</td>
      <td>37.655264</td>
      <td>127.077120</td>
    </tr>
    <tr>
      <th>18</th>
      <td>11710</td>
      <td>송파구</td>
      <td>37.504853</td>
      <td>127.114482</td>
    </tr>
    <tr>
      <th>19</th>
      <td>11410</td>
      <td>서대문구</td>
      <td>37.582037</td>
      <td>126.935667</td>
    </tr>
    <tr>
      <th>20</th>
      <td>11470</td>
      <td>양천구</td>
      <td>37.527062</td>
      <td>126.856153</td>
    </tr>
    <tr>
      <th>21</th>
      <td>11560</td>
      <td>영등포구</td>
      <td>37.520641</td>
      <td>126.913924</td>
    </tr>
    <tr>
      <th>22</th>
      <td>11620</td>
      <td>관악구</td>
      <td>37.465399</td>
      <td>126.943807</td>
    </tr>
    <tr>
      <th>23</th>
      <td>11200</td>
      <td>성동구</td>
      <td>37.550675</td>
      <td>127.040962</td>
    </tr>
    <tr>
      <th>24</th>
      <td>11170</td>
      <td>용산구</td>
      <td>37.531101</td>
      <td>126.981074</td>
    </tr>
  </tbody>
</table>
</div>




```python
grp_seoul = burger_df.loc[burger_df["주소1"]=="서울특별시",:]
# grp_seoul

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
      <th>시군구코드</th>
      <th>시군구명_한글</th>
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
      <td>11680</td>
      <td>강남구</td>
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
      <td>11680</td>
      <td>강남구</td>
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
      <td>11680</td>
      <td>강남구</td>
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
      <td>11680</td>
      <td>강남구</td>
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
      <td>11680</td>
      <td>강남구</td>
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
    </tr>
    <tr>
      <th>456</th>
      <td>마들역</td>
      <td>서울특별시 노원구 동일로 1528</td>
      <td>롯데리아</td>
      <td>서울특별시</td>
      <td>노원구</td>
      <td>동일로 1528</td>
      <td>11350</td>
      <td>노원구</td>
      <td>37.655264</td>
      <td>127.077120</td>
    </tr>
    <tr>
      <th>457</th>
      <td>언더랜드</td>
      <td>서울특별시 송파구 올림픽로 240</td>
      <td>롯데리아</td>
      <td>서울특별시</td>
      <td>송파구</td>
      <td>올림픽로 240</td>
      <td>11710</td>
      <td>송파구</td>
      <td>37.504853</td>
      <td>127.114482</td>
    </tr>
    <tr>
      <th>458</th>
      <td>소공2호</td>
      <td>서울특별시 중구 남대문로 81</td>
      <td>롯데리아</td>
      <td>서울특별시</td>
      <td>중구</td>
      <td>남대문로 81</td>
      <td>11140</td>
      <td>중구</td>
      <td>37.557945</td>
      <td>126.994190</td>
    </tr>
    <tr>
      <th>459</th>
      <td>동서울터미널</td>
      <td>서울특별시 광진구 구의3동546-1</td>
      <td>롯데리아</td>
      <td>서울특별시</td>
      <td>광진구</td>
      <td>구의3동546-1</td>
      <td>11215</td>
      <td>광진구</td>
      <td>37.548144</td>
      <td>127.085753</td>
    </tr>
    <tr>
      <th>460</th>
      <td>봉천</td>
      <td>서울특별시 관악구 관악로212</td>
      <td>롯데리아</td>
      <td>서울특별시</td>
      <td>관악구</td>
      <td>관악로212</td>
      <td>11620</td>
      <td>관악구</td>
      <td>37.465399</td>
      <td>126.943807</td>
    </tr>
  </tbody>
</table>
<p>461 rows × 10 columns</p>
</div>




```python
dat_loc = np.array([seoul_data["위도"].tolist(), seoul_data["경도"].tolist()]).T
pop_ups = seoul_data["지점명"].tolist()
print(dat_loc.shape)
dat_loc[:10]
```

    (461, 2)





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
with open(input_path+'seoul_muncipalities_geo.json',mode='rt',encoding='utf-8') as f:
  geo = json.loads(f.read()) 
  f.close()

burger_map = folium.Map([37.540275, 126.982510], zoom_start=11)
plugins.MarkerCluster(dat_loc, popups=pop_ups).add_to(burger_map)

folium.GeoJson(
    geo,
  name= 'seoul_municipalities' ).add_to(burger_map)

burger_map.save(output_path+"burger-map-seoul.html")
burger_map
```








```python
os.listdir()
```




    ['output_23_0.png',
     'folium.md',
     'folium.ipynb',
     '.DS_Store',
     'output_18_0.png',
     'input',
     'output_39_0.png',
     'output',
     'correlation.md',
     'output_19_0.png',
     'output_8_0.png',
     'output_33_0.png',
     'output_17_0.png',
     'output_13_0.png',
     'output_11_0.png',
     '.ipynb_checkpoints',
     'output_32_0.png',
     'output_32_1.png',
     'output_16_0.png',
     'correlation.ipynb',
     'output_36_0.png']




```python

```