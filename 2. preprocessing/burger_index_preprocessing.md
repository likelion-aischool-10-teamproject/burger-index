# 브랜드별 매장 데이터 전처리

### 목표 
 - 브랜드별 크롤링 결과를 묶어서 데이터 분석할 수 있는 여러 데이터프레임을 생성 및 저장한다. 

### 주요 내용
1. 컬럼명 획일화 (예) "지점명", "매장명" → "지점명"으로
2. 주소 행정구역명 획일화 (예) "서울시","서울특별시","서울" → "서울특별시"로
3. 행정구역별(광역시도별/시군구별) 버거킹, KFC, 롯데리아, 맥도날드 매장수 세기 및 버거지수 계산
4. 각종 통계자료 합치기

### 최종 데이터프레임 
#### 1. `burger_df`
    - 크기 : (2306, 6)
    - 파일명 : *burger_df.csv* 또는 *burger_df.xlsx*로 저장
    - 구성  
    
|index|브랜드|주소|지점명|주소1(시,도)|주소2(시,구,군)|주소3(나머지)|
|---|---|---|---|---|---|---|
|0|버거킹| 서울특별시 강남구 선릉로 429	| 선릉역점	| 서울특별시| 강남구 | 선릉로 429|
|...|...|...|...|...|...|...|...|

#### 2. `nstore_df`
    - 크기 : (240, 9)
    - 파일명 : *nstore_df.csv* 또는 *nstore_df.xlsx*
    - 구성
|index|버거킹|KFC|맥도날드|롯데리아|BKM|버거지수|주소(주소1+주소2)|주소1(시,도)|주소2(시,구,군)|
|---|---|---|---|---|---|---|---|---|---|
|0|1| 0	|1| 6|2 | 0.3333| 강원도 강릉시| 강원도|강릉시|
|...|...|...|...|...|...|...|...|...|...|  
    	
#### 3. `nstore_stat_df`
    - 크기 :(17, 13)
    - 파일명 : *nstore_stat_df.csv* 또는 *nstore_stat_df.xlsx*
    - 구성 
    cf. 경제 지표 기준은 1인당

|index|버거킹|KFC|맥도날드|롯데리아|BKM|버거지수|시도별|지역내총생산|지역총소득| 개인소득|민간소비|인구|인구밀도|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|1| 0	|1| 6|2 | 0.3333| 강원도 |32061|29392|18997|16811|1521|90|
|...|...|...|...|...|...|...|...| ...| ...|  ...| ...| ...| ...|  

    

### 특이사항
- 롯데리아와 KFC의 경우 크롤링한 결과 자체가 `burger_df` 과 동일한 구성으로 되어있기 때문에 나머지 브랜드를 처리한 후에 `pandas.concat` 이용하여 추가
   



```python
import os
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

import warnings
warnings.filterwarnings(action='ignore')
```


```python
# output 파일 path 설정
output_path = './output/'
input_path = './input/'
crawl_path = '../1. web-crawling/'
```


```python
# 각 브랜드별 DataFrame 생성
burger_king_df = pd.read_excel(crawl_path+'BurgerKing/BurgerKing_all_store.xlsx')
kfc_df = pd.read_excel(crawl_path+'KFC/KFC_all_store.xlsx')
mcdonalds_df = pd.read_excel(crawl_path+'McDonalds/mcdonalds_all_store.xlsx')
lotteria_df = pd.read_excel(crawl_path+'Lotteria/lotteria_all_store.xlsx')
```


```python
burger_king_df.head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
    </tr>
  </tbody>
</table>
</div>




```python
kfc_df.head()
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
      <th>매장명</th>
      <th>매장 주소</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>원주단계DT</td>
      <td>강원 원주시 북원로 2266 (단계동) KFC원주단계DT</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>북원로 2266 (단계동) KFC원주단계DT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천이마트</td>
      <td>강원 춘천시 경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
    </tr>
    <tr>
      <th>2</th>
      <td>춘천석사</td>
      <td>강원 춘천시 영서로 2027 (석사동)</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>영서로 2027 (석사동)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>행신역</td>
      <td>경기 고양시 덕양구 충장로 8 (행신동)</td>
      <td>경기도</td>
      <td>고양시</td>
      <td>덕양구 충장로 8 (행신동)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>화정</td>
      <td>경기 고양시 덕양구 화신로272번길 57 (화정동)</td>
      <td>경기도</td>
      <td>고양시</td>
      <td>덕양구 화신로272번길 57 (화정동)</td>
    </tr>
  </tbody>
</table>
</div>




```python
mcdonalds_df.head()
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
      <th>매장명</th>
      <th>매장 주소</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>고양 삼송 DT</td>
      <td>경기 고양시 덕양구 고양대로 1948</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울시청</td>
      <td>서울 중구 남대문로9길 51 효덕빌딩</td>
    </tr>
    <tr>
      <th>2</th>
      <td>한신</td>
      <td>서울 노원구 한글비석로 57 (하계동)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>영천DT</td>
      <td>경북 영천시 호국로 141</td>
    </tr>
    <tr>
      <th>4</th>
      <td>대구동호DT</td>
      <td>대구 동구 안심로 403</td>
    </tr>
  </tbody>
</table>
</div>




```python
lotteria_df.head()
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
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>가평휴게소(상)</td>
      <td>경기도 가평군 설악면 미사리로 544</td>
      <td>경기도</td>
      <td>가평군</td>
      <td>설악면 미사리로 544</td>
    </tr>
    <tr>
      <th>1</th>
      <td>아산장재</td>
      <td>서울특별시 용산구 한강대로71길 47</td>
      <td>서울특별시</td>
      <td>용산구</td>
      <td>한강대로71길 47</td>
    </tr>
    <tr>
      <th>2</th>
      <td>수원파장</td>
      <td>경기도 수원시 장안구 파장로 91</td>
      <td>경기도</td>
      <td>수원시</td>
      <td>장안구 파장로 91</td>
    </tr>
    <tr>
      <th>3</th>
      <td>롯데더몰여수</td>
      <td>전라남도 여수시 국포1로 36</td>
      <td>전라남도</td>
      <td>여수시</td>
      <td>국포1로 36</td>
    </tr>
    <tr>
      <th>4</th>
      <td>김포마산</td>
      <td>경기도 김포시 김포한강8로148번길 5</td>
      <td>경기도</td>
      <td>김포시</td>
      <td>김포한강8로148번길 5</td>
    </tr>
  </tbody>
</table>
</div>




```python
burger_king_df.shape, kfc_df.shape, mcdonalds_df.shape, lotteria_df.shape
```




    ((423, 2), (187, 5), (405, 2), (1304, 5))




```python
# 각 DataFrame에 브랜드 열 생성
burger_king_df['브랜드'] = '버거킹'
kfc_df['브랜드'] = 'KFC'
mcdonalds_df['브랜드'] = '맥도날드'
lotteria_df['브랜드'] = '롯데리아'
```


```python
# 중복 데이터 확인
print( burger_king_df['지점명'].value_counts().sort_values(ascending=False).head(1) )
print()
print( kfc_df['매장명'].value_counts().sort_values(ascending=False).head(1) )
print()
print( mcdonalds_df['매장명'].value_counts().sort_values(ascending=False).head(1) )
print()
print( lotteria_df['지점명'].value_counts().sort_values(ascending=False).head(1) )
```

    대구월촌역FS점    2
    Name: 지점명, dtype: int64
    
    연제이마트    1
    Name: 매장명, dtype: int64
    
    김포감정DT점    1
    Name: 매장명, dtype: int64
    
    순천용당    1
    Name: 지점명, dtype: int64



```python
# 중복 제거
burger_king_df.drop_duplicates('지점명', keep='first', inplace=True)
kfc_df.drop_duplicates('매장명', keep='first', inplace=True)
mcdonalds_df.drop_duplicates('매장명', keep='first', inplace=True)
lotteria_df.drop_duplicates('지점명', keep='first', inplace=True)

burger_king_df.shape, kfc_df.shape, mcdonalds_df.shape, lotteria_df.shape
```




    ((410, 3), (187, 6), (405, 3), (1304, 6))




```python
# 빈 DataFrame 생성 후 각 브랜드별 DataFrame 추가
burger_df = pd.DataFrame()

burger_df = pd.concat([burger_king_df, mcdonalds_df])
# burger_df = pd.concat([burger_df, kfc_df, lotteria_df])

# DataFrame 인덱스 재설정
burger_df.reset_index(drop=True, inplace=True)
```


```python
print(burger_df.shape)
burger_df.head()
```

    (815, 5)





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
      <th>매장명</th>
      <th>매장 주소</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 매장명, 매장 주소를 지점명, 주소로 통일
burger_df['지점명'].fillna(burger_df['매장명'], inplace=True)
burger_df['주소'].fillna(burger_df['매장 주소'], inplace=True)
print(burger_df.isnull().sum())
burger_df.head()
```

    지점명        0
    주소         0
    브랜드        0
    매장명      410
    매장 주소    410
    dtype: int64





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
      <th>매장명</th>
      <th>매장 주소</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 매장명, 매장 주소 열 삭제
burger_df.drop(['매장명', '매장 주소'], axis=1, inplace=True)
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>버거킹</td>
    </tr>
    <tr>
      <th>1</th>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>버거킹</td>
    </tr>
    <tr>
      <th>2</th>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>버거킹</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>버거킹</td>
    </tr>
    <tr>
      <th>4</th>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
      <td>버거킹</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>810</th>
      <td>대전카이스트점</td>
      <td>대전 유성구 대덕대로 535</td>
      <td>맥도날드</td>
    </tr>
    <tr>
      <th>811</th>
      <td>김천평화DT점</td>
      <td>경북 김천시 자산로 199</td>
      <td>맥도날드</td>
    </tr>
    <tr>
      <th>812</th>
      <td>대구태전 DT</td>
      <td>대구 북구 칠곡중앙대로 303</td>
      <td>맥도날드</td>
    </tr>
    <tr>
      <th>813</th>
      <td>강남 2호점</td>
      <td>서울 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
    </tr>
    <tr>
      <th>814</th>
      <td>개금점</td>
      <td>부산 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>맥도날드</td>
    </tr>
  </tbody>
</table>
<p>815 rows × 3 columns</p>
</div>




```python
# 주소를 시군구별로 나눠서 열 추가 후 재정렬
burger_df[['주소1', '주소2', '주소3']] = pd.DataFrame(burger_df['주소'].str.strip().str.split(' ', 2).tolist())
burger_df.reindex(columns=['브랜드', '지점명', '주소', '주소1', '주소2', '주소3'])
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
      <th>브랜드</th>
      <th>지점명</th>
      <th>주소</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>버거킹</td>
      <td>선릉역점</td>
      <td>서울특별시 강남구 선릉로 429</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>선릉로 429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>버거킹</td>
      <td>대치역점</td>
      <td>서울특별시 강남구 남부순환로 2936</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>남부순환로 2936</td>
    </tr>
    <tr>
      <th>2</th>
      <td>버거킹</td>
      <td>차병원사거리점</td>
      <td>서울특별시 강남구 봉은사로 179</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>봉은사로 179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>버거킹</td>
      <td>강남도곡점</td>
      <td>서울특별시 강남구 논현로 172 (도곡동 410-10) 1층</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>논현로 172 (도곡동 410-10) 1층</td>
    </tr>
    <tr>
      <th>4</th>
      <td>버거킹</td>
      <td>청담점</td>
      <td>서울특별시 강남구 선릉로 812</td>
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
      <th>810</th>
      <td>맥도날드</td>
      <td>대전카이스트점</td>
      <td>대전 유성구 대덕대로 535</td>
      <td>대전</td>
      <td>유성구</td>
      <td>대덕대로 535</td>
    </tr>
    <tr>
      <th>811</th>
      <td>맥도날드</td>
      <td>김천평화DT점</td>
      <td>경북 김천시 자산로 199</td>
      <td>경북</td>
      <td>김천시</td>
      <td>자산로 199</td>
    </tr>
    <tr>
      <th>812</th>
      <td>맥도날드</td>
      <td>대구태전 DT</td>
      <td>대구 북구 칠곡중앙대로 303</td>
      <td>대구</td>
      <td>북구</td>
      <td>칠곡중앙대로 303</td>
    </tr>
    <tr>
      <th>813</th>
      <td>맥도날드</td>
      <td>강남 2호점</td>
      <td>서울 강남구 테헤란로 107 메디타워2층</td>
      <td>서울</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
    </tr>
    <tr>
      <th>814</th>
      <td>맥도날드</td>
      <td>개금점</td>
      <td>부산 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>부산</td>
      <td>부산진구</td>
      <td>복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
    </tr>
  </tbody>
</table>
<p>815 rows × 6 columns</p>
</div>




```python
# 시도 값 확인
burger_df['주소1'].unique()
```




    array(['서울특별시', '부산광역시', '대구광역시', '인천광역시', '광주광역시', '대전광역시', '울산광역시',
           '경기도', '강원도', '충청북도', '충청남도', '전라북도', '전라남도', '경상북도', '경상남도',
           '제주특별자치도', '경기', '서울', '경북', '대구', '경남', '전남', '충남', '인천', '부산',
           '전북', '강원', '광주', '서울시', '충북', '울산', '대전'], dtype=object)




```python
# 줄여쓴 구역을 풀어서 다시 입력
burger_df['주소1'] = burger_df['주소1'].replace({'서울시':'서울특별시',
                                                '서울':'서울특별시',
                                                '부산':'부산광역시',
                                                '대구':'대구광역시',
                                                '인천':'인천광역시',
                                                '광주':'광주광역시',
                                                '대전':'대전광역시',
                                                '울산':'울산광역시',
                                                '세종':'세종특별자치시',
                                                '경기':'경기도',
                                                '강원':'강원도',
                                                '충북':'충청북도',
                                                '충남':'충청남도',
                                                '전북':'전라북도',
                                                '전남':'전라남도',
                                                '경북':'경상북도',
                                                '경남':'경상남도',
                                                '제주':'제주특별자치도'})

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
      <th>810</th>
      <td>대전카이스트점</td>
      <td>대전 유성구 대덕대로 535</td>
      <td>맥도날드</td>
      <td>대전광역시</td>
      <td>유성구</td>
      <td>대덕대로 535</td>
    </tr>
    <tr>
      <th>811</th>
      <td>김천평화DT점</td>
      <td>경북 김천시 자산로 199</td>
      <td>맥도날드</td>
      <td>경상북도</td>
      <td>김천시</td>
      <td>자산로 199</td>
    </tr>
    <tr>
      <th>812</th>
      <td>대구태전 DT</td>
      <td>대구 북구 칠곡중앙대로 303</td>
      <td>맥도날드</td>
      <td>대구광역시</td>
      <td>북구</td>
      <td>칠곡중앙대로 303</td>
    </tr>
    <tr>
      <th>813</th>
      <td>강남 2호점</td>
      <td>서울 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
    </tr>
    <tr>
      <th>814</th>
      <td>개금점</td>
      <td>부산 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>맥도날드</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
    </tr>
  </tbody>
</table>
<p>815 rows × 6 columns</p>
</div>




```python
# 주소1, 주소2, 주소3 을 합쳐서 주소를 다시 입력
burger_df['주소'] = burger_df[ ['주소1', '주소2', '주소3'] ].apply(' '.join, axis=1)
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
      <th>810</th>
      <td>대전카이스트점</td>
      <td>대전광역시 유성구 대덕대로 535</td>
      <td>맥도날드</td>
      <td>대전광역시</td>
      <td>유성구</td>
      <td>대덕대로 535</td>
    </tr>
    <tr>
      <th>811</th>
      <td>김천평화DT점</td>
      <td>경상북도 김천시 자산로 199</td>
      <td>맥도날드</td>
      <td>경상북도</td>
      <td>김천시</td>
      <td>자산로 199</td>
    </tr>
    <tr>
      <th>812</th>
      <td>대구태전 DT</td>
      <td>대구광역시 북구 칠곡중앙대로 303</td>
      <td>맥도날드</td>
      <td>대구광역시</td>
      <td>북구</td>
      <td>칠곡중앙대로 303</td>
    </tr>
    <tr>
      <th>813</th>
      <td>강남 2호점</td>
      <td>서울특별시 강남구 테헤란로 107 메디타워2층</td>
      <td>맥도날드</td>
      <td>서울특별시</td>
      <td>강남구</td>
      <td>테헤란로 107 메디타워2층</td>
    </tr>
    <tr>
      <th>814</th>
      <td>개금점</td>
      <td>부산광역시 부산진구 복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
      <td>맥도날드</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>복지로 70, 상가 103호 (개금동, 현대아이아파트)</td>
    </tr>
  </tbody>
</table>
<p>815 rows × 6 columns</p>
</div>




```python
lotteria_df.head()
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
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
      <th>브랜드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>가평휴게소(상)</td>
      <td>경기도 가평군 설악면 미사리로 544</td>
      <td>경기도</td>
      <td>가평군</td>
      <td>설악면 미사리로 544</td>
      <td>롯데리아</td>
    </tr>
    <tr>
      <th>1</th>
      <td>아산장재</td>
      <td>서울특별시 용산구 한강대로71길 47</td>
      <td>서울특별시</td>
      <td>용산구</td>
      <td>한강대로71길 47</td>
      <td>롯데리아</td>
    </tr>
    <tr>
      <th>2</th>
      <td>수원파장</td>
      <td>경기도 수원시 장안구 파장로 91</td>
      <td>경기도</td>
      <td>수원시</td>
      <td>장안구 파장로 91</td>
      <td>롯데리아</td>
    </tr>
    <tr>
      <th>3</th>
      <td>롯데더몰여수</td>
      <td>전라남도 여수시 국포1로 36</td>
      <td>전라남도</td>
      <td>여수시</td>
      <td>국포1로 36</td>
      <td>롯데리아</td>
    </tr>
    <tr>
      <th>4</th>
      <td>김포마산</td>
      <td>경기도 김포시 김포한강8로148번길 5</td>
      <td>경기도</td>
      <td>김포시</td>
      <td>김포한강8로148번길 5</td>
      <td>롯데리아</td>
    </tr>
  </tbody>
</table>
</div>




```python
kfc_df.rename(columns={"매장명":"지점명", "매장 주소":"주소"}, inplace=True)
kfc_df.head()
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
      <th>주소1</th>
      <th>주소2</th>
      <th>주소3</th>
      <th>브랜드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>원주단계DT</td>
      <td>강원 원주시 북원로 2266 (단계동) KFC원주단계DT</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>북원로 2266 (단계동) KFC원주단계DT</td>
      <td>KFC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천이마트</td>
      <td>강원 춘천시 경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
      <td>KFC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>춘천석사</td>
      <td>강원 춘천시 영서로 2027 (석사동)</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>영서로 2027 (석사동)</td>
      <td>KFC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>행신역</td>
      <td>경기 고양시 덕양구 충장로 8 (행신동)</td>
      <td>경기도</td>
      <td>고양시</td>
      <td>덕양구 충장로 8 (행신동)</td>
      <td>KFC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>화정</td>
      <td>경기 고양시 덕양구 화신로272번길 57 (화정동)</td>
      <td>경기도</td>
      <td>고양시</td>
      <td>덕양구 화신로272번길 57 (화정동)</td>
      <td>KFC</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 롯데리아, KFC 데이터 추가
burger_df = pd.concat([burger_df, kfc_df, lotteria_df]) 
burger_df.reset_index(drop=True, inplace=True)
burger_df.head()
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
  </tbody>
</table>
</div>




```python
# csv, xlsx 파일로 내보내기
burger_df.to_csv(output_path+"burger_df.csv", index=False)
burger_df.to_excel(output_path+"burger_df.xlsx", index=False)

os.listdir(output_path)
```




    ['nstore_stat_df.xlsx',
     '.DS_Store',
     'nstore_stat_df_2.xlsx',
     'nstore_df.csv',
     'nstore_stat_df_2.csv',
     'nstore_df.xlsx',
     'nstore_stat_df.csv',
     'burger_df.csv',
     'burger_df.xlsx']



> `burger_df` 파일 전처리 완료


```python
# 주소1+2 컬럼 만들기
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
      <th>2301</th>
      <td>원주일산</td>
      <td>강원도 원주시 천사로 213</td>
      <td>롯데리아</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>천사로 213</td>
      <td>강원도 원주시</td>
    </tr>
    <tr>
      <th>2302</th>
      <td>성남</td>
      <td>경기도 성남시 수정구 수정로 181</td>
      <td>롯데리아</td>
      <td>경기도</td>
      <td>성남시</td>
      <td>수정구 수정로 181</td>
      <td>경기도 성남시</td>
    </tr>
    <tr>
      <th>2303</th>
      <td>대전중앙</td>
      <td>대전광역시 중구 선화동 3-3번지</td>
      <td>롯데리아</td>
      <td>대전광역시</td>
      <td>중구</td>
      <td>선화동 3-3번지</td>
      <td>대전광역시 중구</td>
    </tr>
    <tr>
      <th>2304</th>
      <td>홈서비스과천</td>
      <td>경기도 과천시 별양동 19-4</td>
      <td>롯데리아</td>
      <td>경기도</td>
      <td>과천시</td>
      <td>별양동 19-4</td>
      <td>경기도 과천시</td>
    </tr>
    <tr>
      <th>2305</th>
      <td>홈서비스부암(부산역)</td>
      <td>부산광역시 부산진구 부암동 722-3번지</td>
      <td>롯데리아</td>
      <td>부산광역시</td>
      <td>부산진구</td>
      <td>부암동 722-3번지</td>
      <td>부산광역시 부산진구</td>
    </tr>
  </tbody>
</table>
<p>2306 rows × 7 columns</p>
</div>




```python
# 브랜드별로 데이터 묶기
bking_grp = burger_df[burger_df["브랜드"]=="버거킹"]
kfc_grp = burger_df[burger_df["브랜드"]=="KFC"]
mc_grp = burger_df[burger_df["브랜드"]=="맥도날드"]
lotte_grp = burger_df[burger_df["브랜드"]=="롯데리아"]

bking_grp.shape, kfc_grp.shape, mc_grp.shape, lotte_grp.shape # 갯수 확인
```




    ((410, 7), (187, 7), (405, 7), (1304, 7))




```python
# 시구군별로 매장수 세기
bking_cnt =  bking_grp.groupby("주소1+2").count()["지점명"]
kfc_cnt = kfc_grp.groupby("주소1+2").count()["지점명"]
mc_cnt = mc_grp.groupby("주소1+2").count()["지점명"]
lotte_cnt = lotte_grp.groupby("주소1+2").count()["지점명"]

bking_cnt.shape, kfc_cnt.shape, mc_cnt.shape, lotte_cnt.shape
```




    ((129,), (90,), (132,), (224,))




```python
# column 이름을 Series.name 에 저장
bking_cnt.name = "버거킹"
kfc_cnt.name = "KFC"
mc_cnt.name = "맥도날드"
lotte_cnt.name = "롯데리아"
```


```python
# Series 데이터 확인
print(bking_cnt)
print()
print(kfc_cnt)
print()
print(mc_cnt)
print()
print(lotte_cnt)
print()
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
    
    주소1+2
    강원도 강릉시      6
    강원도 고성군      1
    강원도 동해시      3
    강원도 삼척시      1
    강원도 속초시      3
                ..
    충청북도 제천시     4
    충청북도 증평군     1
    충청북도 진천군     2
    충청북도 청주시    31
    충청북도 충주시     8
    Name: 롯데리아, Length: 224, dtype: int64
    



```python
# Series 합치기
nstore_df = pd.concat([bking_cnt, kfc_cnt, mc_cnt, lotte_cnt], axis=1) # 시리즈 합치기, 합치면서 dtypes=float으로 변하는 것 같다
nstore_df = nstore_df.fillna(0) # 결측치 0으로 채우기 
nstore_df = nstore_df.astype("int") # 자료형 타입 int로 바꾸기

print(nstore_df.info())
nstore_df
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 224 entries, 강원도 강릉시 to 충청북도 증평군
    Data columns (total 4 columns):
     #   Column  Non-Null Count  Dtype
    ---  ------  --------------  -----
     0   버거킹     224 non-null    int64
     1   KFC     224 non-null    int64
     2   맥도날드    224 non-null    int64
     3   롯데리아    224 non-null    int64
    dtypes: int64(4)
    memory usage: 8.8+ KB
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
      <th>롯데리아</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원도 강릉시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>15</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>7</td>
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
      <th>충청북도 단양군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 보은군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 영동군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>충청북도 옥천군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>충청북도 증평군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>224 rows × 4 columns</p>
</div>




```python
# 시군구 중복 확인
len(nstore_df.index), len(set(nstore_df.index)) 
```




    (224, 224)




```python
# 버거지수 만들기 전 롯데리아 매장이 없는 지역 확인
nstore_df[nstore_df["롯데리아"]==0]
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
      <th>롯데리아</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
# 버거지수 컬럼 만들기
nstore_df["BKM"]=nstore_df["버거킹"]+nstore_df["KFC"]+nstore_df["맥도날드"]
nstore_df["버거지수"] = nstore_df["BKM"]/nstore_df["롯데리아"] 
```


```python
# 인덱스를 주소1(시,도), 주소2(시,군,구)로 나누기, 인덱싱 재설정은 나중에 한다
nstore_df["주소"] = nstore_df.index
nstore_df[["주소1","주소2"]] = nstore_df["주소"].str.strip().str.split().tolist()
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
      <th>주소</th>
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
      <td>6</td>
      <td>2</td>
      <td>0.333333</td>
      <td>강원도 강릉시</td>
      <td>강원도</td>
      <td>강릉시</td>
    </tr>
    <tr>
      <th>강원도 속초시</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0.666667</td>
      <td>강원도 속초시</td>
      <td>강원도</td>
      <td>속초시</td>
    </tr>
    <tr>
      <th>강원도 원주시</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>15</td>
      <td>6</td>
      <td>0.400000</td>
      <td>강원도 원주시</td>
      <td>강원도</td>
      <td>원주시</td>
    </tr>
    <tr>
      <th>강원도 춘천시</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>7</td>
      <td>6</td>
      <td>0.857143</td>
      <td>강원도 춘천시</td>
      <td>강원도</td>
      <td>춘천시</td>
    </tr>
    <tr>
      <th>강원도 홍천군</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>1.000000</td>
      <td>강원도 홍천군</td>
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
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>충청북도 단양군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 단양군</td>
      <td>충청북도</td>
      <td>단양군</td>
    </tr>
    <tr>
      <th>충청북도 보은군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 보은군</td>
      <td>충청북도</td>
      <td>보은군</td>
    </tr>
    <tr>
      <th>충청북도 영동군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 영동군</td>
      <td>충청북도</td>
      <td>영동군</td>
    </tr>
    <tr>
      <th>충청북도 옥천군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 옥천군</td>
      <td>충청북도</td>
      <td>옥천군</td>
    </tr>
    <tr>
      <th>충청북도 증평군</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 증평군</td>
      <td>충청북도</td>
      <td>증평군</td>
    </tr>
  </tbody>
</table>
<p>224 rows × 9 columns</p>
</div>



#### 주소2(시,군,구) 중복 확인
- 이 후에 주소2를 기준으로 인구나 인구밀도 정보를 넣어도 되는지 알기 위해서 중복을 확인한다.
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

    224
    203
    203





    중구      6
    동구      5
    서구      5
    북구      4
    남구      4
    고성군     2
    강서구     2
    의정부시    1
    함평군     1
    부여군     1
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
    부산광역시 중구    부산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 동구    광주광역시
    대구광역시 동구    대구광역시
    대전광역시 동구    대전광역시
    울산광역시 동구    울산광역시
    부산광역시 동구    부산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 서구    광주광역시
    대구광역시 서구    대구광역시
    대전광역시 서구    대전광역시
    인천광역시 서구    인천광역시
    부산광역시 서구    부산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 북구    광주광역시
    대구광역시 북구    대구광역시
    부산광역시 북구    부산광역시
    울산광역시 북구    울산광역시
    Name: 주소1, dtype: object 
    
    광주광역시 남구    광주광역시
    대구광역시 남구    대구광역시
    부산광역시 남구    부산광역시
    울산광역시 남구    울산광역시
    Name: 주소1, dtype: object 
    
    강원도 고성군      강원도
    경상남도 고성군    경상남도
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
nstore_df.reset_index(drop=True, inplace=True) 
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
      <th>주소</th>
      <th>주소1</th>
      <th>주소2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>6</td>
      <td>2</td>
      <td>0.333333</td>
      <td>강원도 강릉시</td>
      <td>강원도</td>
      <td>강릉시</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0.666667</td>
      <td>강원도 속초시</td>
      <td>강원도</td>
      <td>속초시</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>15</td>
      <td>6</td>
      <td>0.400000</td>
      <td>강원도 원주시</td>
      <td>강원도</td>
      <td>원주시</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>7</td>
      <td>6</td>
      <td>0.857143</td>
      <td>강원도 춘천시</td>
      <td>강원도</td>
      <td>춘천시</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>1.000000</td>
      <td>강원도 홍천군</td>
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
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>219</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 단양군</td>
      <td>충청북도</td>
      <td>단양군</td>
    </tr>
    <tr>
      <th>220</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 보은군</td>
      <td>충청북도</td>
      <td>보은군</td>
    </tr>
    <tr>
      <th>221</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 영동군</td>
      <td>충청북도</td>
      <td>영동군</td>
    </tr>
    <tr>
      <th>222</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 옥천군</td>
      <td>충청북도</td>
      <td>옥천군</td>
    </tr>
    <tr>
      <th>223</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 증평군</td>
      <td>충청북도</td>
      <td>증평군</td>
    </tr>
  </tbody>
</table>
<p>224 rows × 9 columns</p>
</div>




```python
# csv, xlsx 파일로 내보내기
nstore_df.to_csv(output_path+"nstore_df.csv", index=False)
nstore_df.to_excel(output_path+"nstore_df.xlsx", index=False)

os.listdir(output_path)
```




    ['nstore_stat_df.xlsx',
     '.DS_Store',
     'nstore_stat_df_2.xlsx',
     'nstore_df.csv',
     'nstore_stat_df_2.csv',
     'nstore_df.xlsx',
     'nstore_stat_df.csv',
     'burger_df.csv',
     'burger_df.xlsx']



> `nstore_df` 파일 전처리 완료

## 광역시도별 경제 지표 가져오기
- 출처 : [통계청](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1C86&conn_path=I2)  
- 파일명 : *시도별_1인당_지역내총생산\__지역총소득\__개인소득_20210702225439.csv* 
- 설명 : (2019년 기준) 전국 16개 시도별 `1인당 지역내총생산`, `1인당 지역총소득`,`1인당 개인소득`,`1인당 민간소비``


```python
eco_index = pd.read_csv(input_path+"시도별_1인당_지역내총생산__지역총소득__개인소득_20210702225439.csv", encoding='cp949', skiprows=1)
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

### 방식 1.  nstore_df에 eco_index를 join한다.
- 결과 : 실패

### 방식 2. `eco_index`에 `nstore_df`를 join하는 방식


```python
new_nstore_df = nstore_df.groupby("주소1").sum()[["버거킹","KFC","맥도날드","롯데리아"]] 
new_nstore_df # 롯데리아가 없는 시,도가 있는지 직접 확인
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
      <th>롯데리아</th>
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
      <td>54</td>
    </tr>
    <tr>
      <th>경기도</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>317</td>
    </tr>
    <tr>
      <th>경상남도</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>86</td>
    </tr>
    <tr>
      <th>경상북도</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>68</td>
    </tr>
    <tr>
      <th>광주광역시</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>42</td>
    </tr>
    <tr>
      <th>대구광역시</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>66</td>
    </tr>
    <tr>
      <th>대전광역시</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>51</td>
    </tr>
    <tr>
      <th>부산광역시</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>82</td>
    </tr>
    <tr>
      <th>서울특별시</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>181</td>
    </tr>
    <tr>
      <th>세종특별자치시</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>10</td>
    </tr>
    <tr>
      <th>울산광역시</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>31</td>
    </tr>
    <tr>
      <th>인천광역시</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>74</td>
    </tr>
    <tr>
      <th>전라남도</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>51</td>
    </tr>
    <tr>
      <th>전라북도</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>53</td>
    </tr>
    <tr>
      <th>제주특별자치도</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>14</td>
    </tr>
    <tr>
      <th>충청남도</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>67</td>
    </tr>
    <tr>
      <th>충청북도</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>57</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_nstore_df["BKM"]= new_nstore_df["버거킹"]+new_nstore_df["KFC"]+new_nstore_df["맥도날드"]
new_nstore_df["버거지수"] = new_nstore_df["BKM"]/new_nstore_df["롯데리아"]
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
    </tr>
    <tr>
      <th>주소1</th>
      <th></th>
      <th></th>
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
      <td>54</td>
      <td>19</td>
      <td>0.351852</td>
    </tr>
    <tr>
      <th>경기도</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>317</td>
      <td>246</td>
      <td>0.776025</td>
    </tr>
    <tr>
      <th>경상남도</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>86</td>
      <td>57</td>
      <td>0.662791</td>
    </tr>
    <tr>
      <th>경상북도</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>68</td>
      <td>44</td>
      <td>0.647059</td>
    </tr>
    <tr>
      <th>광주광역시</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>42</td>
      <td>28</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <th>대구광역시</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>66</td>
      <td>55</td>
      <td>0.833333</td>
    </tr>
    <tr>
      <th>대전광역시</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>51</td>
      <td>32</td>
      <td>0.627451</td>
    </tr>
    <tr>
      <th>부산광역시</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>82</td>
      <td>70</td>
      <td>0.853659</td>
    </tr>
    <tr>
      <th>서울특별시</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>181</td>
      <td>280</td>
      <td>1.546961</td>
    </tr>
    <tr>
      <th>세종특별자치시</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>10</td>
      <td>1</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <th>울산광역시</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>31</td>
      <td>21</td>
      <td>0.677419</td>
    </tr>
    <tr>
      <th>인천광역시</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>74</td>
      <td>52</td>
      <td>0.702703</td>
    </tr>
    <tr>
      <th>전라남도</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>51</td>
      <td>17</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <th>전라북도</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>53</td>
      <td>26</td>
      <td>0.490566</td>
    </tr>
    <tr>
      <th>제주특별자치도</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>14</td>
      <td>10</td>
      <td>0.714286</td>
    </tr>
    <tr>
      <th>충청남도</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>67</td>
      <td>24</td>
      <td>0.358209</td>
    </tr>
    <tr>
      <th>충청북도</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>57</td>
      <td>20</td>
      <td>0.350877</td>
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
      <th>롯데리아</th>
      <th>BKM</th>
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
      <td>54</td>
      <td>19</td>
      <td>0.351852</td>
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
      <td>317</td>
      <td>246</td>
      <td>0.776025</td>
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
      <td>86</td>
      <td>57</td>
      <td>0.662791</td>
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
      <td>68</td>
      <td>44</td>
      <td>0.647059</td>
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
      <td>42</td>
      <td>28</td>
      <td>0.666667</td>
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
      <td>66</td>
      <td>55</td>
      <td>0.833333</td>
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
      <td>51</td>
      <td>32</td>
      <td>0.627451</td>
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
      <td>82</td>
      <td>70</td>
      <td>0.853659</td>
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
      <td>181</td>
      <td>280</td>
      <td>1.546961</td>
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
      <td>10</td>
      <td>1</td>
      <td>0.100000</td>
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
      <td>31</td>
      <td>21</td>
      <td>0.677419</td>
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
      <td>74</td>
      <td>52</td>
      <td>0.702703</td>
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
      <td>51</td>
      <td>17</td>
      <td>0.333333</td>
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
      <td>53</td>
      <td>26</td>
      <td>0.490566</td>
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
      <td>14</td>
      <td>10</td>
      <td>0.714286</td>
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
      <td>67</td>
      <td>24</td>
      <td>0.358209</td>
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
      <td>57</td>
      <td>20</td>
      <td>0.350877</td>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
    </tr>
  </tbody>
</table>
</div>



## 광역시도별 인구수/인구밀도 데이터 가져오기
- 출처 : 시도별 인구수/인구밀도 [통계청](https://www.index.go.kr/potal/stts/idxMain/selectPoSttsIdxSearch.do?idx_cd=1007&stts_cd=100701&freq=Y)
- 파일명: *stat_100701.xls*
- 설명 : (2020년 기준) 전국 16개 시도별 `인구`[천명], `인구밀도`[명/km^2]


```python
pop_df_all = pd.read_excel(input_path+"stat_100701.xls",skiprows=3)
pop_df = pop_df_all[1:-5] # 불필요한 행 제거
pop_df = pop_df[["Unnamed: 0", "인구.1","인구밀도.1"]] # 불필요한 열 제거
pop_df = pop_df.rename(columns={"Unnamed: 0": "행정구역", "인구.1":"인구","인구밀도.1":"인구밀도"}) # 컬럼명 수정
# 행정구역 이름 동일화
pop_df = pop_df.replace({'서울':'서울특별시',
                        '부산':'부산광역시',
                        '대구':'대구광역시',
                        '인천':'인천광역시',
                        '광주':'광주광역시',
                        '대전':'대전광역시',
                        '울산':'울산광역시',
                        '세종':'세종특별자치시',
                        '경기':'경기도',
                        '강원':'강원도',
                        '충북':'충청북도',
                        '충남':'충청남도',
                        '전북':'전라북도',
                        '전남':'전라남도',
                        '경북':'경상북도',
                        '경남':'경상남도',
                        '제주':'제주특별자치도'})

pop_df
# pop_df.info()
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
      <th>행정구역</th>
      <th>인구</th>
      <th>인구밀도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>9,766</td>
      <td>16,136</td>
    </tr>
    <tr>
      <th>2</th>
      <td>부산광역시</td>
      <td>3,424</td>
      <td>4,447</td>
    </tr>
    <tr>
      <th>3</th>
      <td>대구광역시</td>
      <td>2,458</td>
      <td>2,782</td>
    </tr>
    <tr>
      <th>4</th>
      <td>인천광역시</td>
      <td>2,924</td>
      <td>2,750</td>
    </tr>
    <tr>
      <th>5</th>
      <td>광주광역시</td>
      <td>1,495</td>
      <td>2,984</td>
    </tr>
    <tr>
      <th>6</th>
      <td>대전광역시</td>
      <td>1,528</td>
      <td>2,832</td>
    </tr>
    <tr>
      <th>7</th>
      <td>울산광역시</td>
      <td>1,159</td>
      <td>1,092</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세종특별자치시</td>
      <td>266</td>
      <td>571</td>
    </tr>
    <tr>
      <th>9</th>
      <td>경기도</td>
      <td>12,786</td>
      <td>1,255</td>
    </tr>
    <tr>
      <th>10</th>
      <td>강원도</td>
      <td>1,521</td>
      <td>90</td>
    </tr>
    <tr>
      <th>11</th>
      <td>충청북도</td>
      <td>1,609</td>
      <td>217</td>
    </tr>
    <tr>
      <th>12</th>
      <td>충청남도</td>
      <td>2,153</td>
      <td>262</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전라북도</td>
      <td>1,829</td>
      <td>227</td>
    </tr>
    <tr>
      <th>14</th>
      <td>전라남도</td>
      <td>1,795</td>
      <td>146</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경상북도</td>
      <td>2,675</td>
      <td>141</td>
    </tr>
    <tr>
      <th>16</th>
      <td>경상남도</td>
      <td>3,339</td>
      <td>317</td>
    </tr>
    <tr>
      <th>17</th>
      <td>제주특별자치도</td>
      <td>635</td>
      <td>343</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 행정구역 이름 동일한지 확인
set(pop_df["행정구역"]) == set(df_2["시도별"])
```




    True




```python
# 두 데이터셋 합치기
nstore_pop_df = pd.merge(df_2, pop_df, left_on="시도별", right_on='행정구역', how='left') 
nstore_pop_df
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
      <th>행정구역</th>
      <th>인구</th>
      <th>인구밀도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9</td>
      <td>3</td>
      <td>7</td>
      <td>54</td>
      <td>19</td>
      <td>0.351852</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
      <td>강원도</td>
      <td>1,521</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>317</td>
      <td>246</td>
      <td>0.776025</td>
      <td>경기도</td>
      <td>36133</td>
      <td>38466</td>
      <td>20482</td>
      <td>17399</td>
      <td>경기도</td>
      <td>12,786</td>
      <td>1,255</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>86</td>
      <td>57</td>
      <td>0.662791</td>
      <td>경상남도</td>
      <td>33690</td>
      <td>32140</td>
      <td>18939</td>
      <td>16426</td>
      <td>경상남도</td>
      <td>3,339</td>
      <td>317</td>
    </tr>
    <tr>
      <th>3</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>68</td>
      <td>44</td>
      <td>0.647059</td>
      <td>경상북도</td>
      <td>40272</td>
      <td>34873</td>
      <td>18611</td>
      <td>16217</td>
      <td>경상북도</td>
      <td>2,675</td>
      <td>141</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>42</td>
      <td>28</td>
      <td>0.666667</td>
      <td>광주광역시</td>
      <td>27548</td>
      <td>30964</td>
      <td>20532</td>
      <td>18231</td>
      <td>광주광역시</td>
      <td>1,495</td>
      <td>2,984</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>66</td>
      <td>55</td>
      <td>0.833333</td>
      <td>대구광역시</td>
      <td>23744</td>
      <td>27798</td>
      <td>19210</td>
      <td>17850</td>
      <td>대구광역시</td>
      <td>2,458</td>
      <td>2,782</td>
    </tr>
    <tr>
      <th>6</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>51</td>
      <td>32</td>
      <td>0.627451</td>
      <td>대전광역시</td>
      <td>28364</td>
      <td>31548</td>
      <td>20498</td>
      <td>18025</td>
      <td>대전광역시</td>
      <td>1,528</td>
      <td>2,832</td>
    </tr>
    <tr>
      <th>7</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>82</td>
      <td>70</td>
      <td>0.853659</td>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
      <td>부산광역시</td>
      <td>3,424</td>
      <td>4,447</td>
    </tr>
    <tr>
      <th>8</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>181</td>
      <td>280</td>
      <td>1.546961</td>
      <td>서울특별시</td>
      <td>44865</td>
      <td>49121</td>
      <td>23440</td>
      <td>21891</td>
      <td>서울특별시</td>
      <td>9,766</td>
      <td>16,136</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>10</td>
      <td>1</td>
      <td>0.100000</td>
      <td>세종특별자치시</td>
      <td>35826</td>
      <td>36983</td>
      <td>19789</td>
      <td>16762</td>
      <td>세종특별자치시</td>
      <td>266</td>
      <td>571</td>
    </tr>
    <tr>
      <th>10</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>31</td>
      <td>21</td>
      <td>0.677419</td>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
      <td>울산광역시</td>
      <td>1,159</td>
      <td>1,092</td>
    </tr>
    <tr>
      <th>11</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>74</td>
      <td>52</td>
      <td>0.702703</td>
      <td>인천광역시</td>
      <td>30425</td>
      <td>32571</td>
      <td>19332</td>
      <td>16451</td>
      <td>인천광역시</td>
      <td>2,924</td>
      <td>2,750</td>
    </tr>
    <tr>
      <th>12</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>51</td>
      <td>17</td>
      <td>0.333333</td>
      <td>전라남도</td>
      <td>43323</td>
      <td>35532</td>
      <td>18711</td>
      <td>16104</td>
      <td>전라남도</td>
      <td>1,795</td>
      <td>146</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>53</td>
      <td>26</td>
      <td>0.490566</td>
      <td>전라북도</td>
      <td>28740</td>
      <td>28260</td>
      <td>18725</td>
      <td>16022</td>
      <td>전라북도</td>
      <td>1,829</td>
      <td>227</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>14</td>
      <td>10</td>
      <td>0.714286</td>
      <td>제주특별자치도</td>
      <td>30720</td>
      <td>30834</td>
      <td>18734</td>
      <td>16953</td>
      <td>제주특별자치도</td>
      <td>635</td>
      <td>343</td>
    </tr>
    <tr>
      <th>15</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>67</td>
      <td>24</td>
      <td>0.358209</td>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
      <td>충청남도</td>
      <td>2,153</td>
      <td>262</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>57</td>
      <td>20</td>
      <td>0.350877</td>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
      <td>충청북도</td>
      <td>1,609</td>
      <td>217</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 인구, 인구밀도 type : object → int
nstore_pop_df["인구"] = nstore_pop_df["인구"].str.replace(",","",regex=False)
nstore_pop_df["인구"] = nstore_pop_df["인구"].astype(int)

nstore_pop_df["인구밀도"] = nstore_pop_df["인구밀도"].str.replace(",","",regex=False)
nstore_pop_df["인구밀도"] = nstore_pop_df["인구밀도"].astype(int)

nstore_pop_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 17 entries, 0 to 16
    Data columns (total 14 columns):
     #   Column      Non-Null Count  Dtype  
    ---  ------      --------------  -----  
     0   버거킹         17 non-null     int64  
     1   KFC         17 non-null     int64  
     2   맥도날드        17 non-null     int64  
     3   롯데리아        17 non-null     int64  
     4   BKM         17 non-null     int64  
     5   버거지수        17 non-null     float64
     6   시도별         17 non-null     object 
     7   1인당 지역내총생산  17 non-null     int64  
     8   1인당 지역총소득   17 non-null     int64  
     9   1인당 개인소득    17 non-null     int64  
     10  1인당 민간소비    17 non-null     int64  
     11  행정구역        17 non-null     object 
     12  인구          17 non-null     int64  
     13  인구밀도        17 non-null     int64  
    dtypes: float64(1), int64(11), object(2)
    memory usage: 2.0+ KB



```python
# 행정구역 컬럼 지우기
nstore_pop_df.drop("행정구역", axis=1, inplace=True)
nstore_pop_df
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
      <th>시도별</th>
      <th>1인당 지역내총생산</th>
      <th>1인당 지역총소득</th>
      <th>1인당 개인소득</th>
      <th>1인당 민간소비</th>
      <th>인구</th>
      <th>인구밀도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9</td>
      <td>3</td>
      <td>7</td>
      <td>54</td>
      <td>19</td>
      <td>0.351852</td>
      <td>강원도</td>
      <td>32061</td>
      <td>29392</td>
      <td>18997</td>
      <td>16811</td>
      <td>1521</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100</td>
      <td>50</td>
      <td>96</td>
      <td>317</td>
      <td>246</td>
      <td>0.776025</td>
      <td>경기도</td>
      <td>36133</td>
      <td>38466</td>
      <td>20482</td>
      <td>17399</td>
      <td>12786</td>
      <td>1255</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24</td>
      <td>4</td>
      <td>29</td>
      <td>86</td>
      <td>57</td>
      <td>0.662791</td>
      <td>경상남도</td>
      <td>33690</td>
      <td>32140</td>
      <td>18939</td>
      <td>16426</td>
      <td>3339</td>
      <td>317</td>
    </tr>
    <tr>
      <th>3</th>
      <td>25</td>
      <td>4</td>
      <td>15</td>
      <td>68</td>
      <td>44</td>
      <td>0.647059</td>
      <td>경상북도</td>
      <td>40272</td>
      <td>34873</td>
      <td>18611</td>
      <td>16217</td>
      <td>2675</td>
      <td>141</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15</td>
      <td>3</td>
      <td>10</td>
      <td>42</td>
      <td>28</td>
      <td>0.666667</td>
      <td>광주광역시</td>
      <td>27548</td>
      <td>30964</td>
      <td>20532</td>
      <td>18231</td>
      <td>1495</td>
      <td>2984</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20</td>
      <td>10</td>
      <td>25</td>
      <td>66</td>
      <td>55</td>
      <td>0.833333</td>
      <td>대구광역시</td>
      <td>23744</td>
      <td>27798</td>
      <td>19210</td>
      <td>17850</td>
      <td>2458</td>
      <td>2782</td>
    </tr>
    <tr>
      <th>6</th>
      <td>14</td>
      <td>5</td>
      <td>13</td>
      <td>51</td>
      <td>32</td>
      <td>0.627451</td>
      <td>대전광역시</td>
      <td>28364</td>
      <td>31548</td>
      <td>20498</td>
      <td>18025</td>
      <td>1528</td>
      <td>2832</td>
    </tr>
    <tr>
      <th>7</th>
      <td>20</td>
      <td>9</td>
      <td>41</td>
      <td>82</td>
      <td>70</td>
      <td>0.853659</td>
      <td>부산광역시</td>
      <td>27409</td>
      <td>29388</td>
      <td>19680</td>
      <td>18030</td>
      <td>3424</td>
      <td>4447</td>
    </tr>
    <tr>
      <th>8</th>
      <td>111</td>
      <td>73</td>
      <td>96</td>
      <td>181</td>
      <td>280</td>
      <td>1.546961</td>
      <td>서울특별시</td>
      <td>44865</td>
      <td>49121</td>
      <td>23440</td>
      <td>21891</td>
      <td>9766</td>
      <td>16136</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>10</td>
      <td>1</td>
      <td>0.100000</td>
      <td>세종특별자치시</td>
      <td>35826</td>
      <td>36983</td>
      <td>19789</td>
      <td>16762</td>
      <td>266</td>
      <td>571</td>
    </tr>
    <tr>
      <th>10</th>
      <td>9</td>
      <td>2</td>
      <td>10</td>
      <td>31</td>
      <td>21</td>
      <td>0.677419</td>
      <td>울산광역시</td>
      <td>65352</td>
      <td>54969</td>
      <td>22550</td>
      <td>18482</td>
      <td>1159</td>
      <td>1092</td>
    </tr>
    <tr>
      <th>11</th>
      <td>23</td>
      <td>11</td>
      <td>18</td>
      <td>74</td>
      <td>52</td>
      <td>0.702703</td>
      <td>인천광역시</td>
      <td>30425</td>
      <td>32571</td>
      <td>19332</td>
      <td>16451</td>
      <td>2924</td>
      <td>2750</td>
    </tr>
    <tr>
      <th>12</th>
      <td>7</td>
      <td>2</td>
      <td>8</td>
      <td>51</td>
      <td>17</td>
      <td>0.333333</td>
      <td>전라남도</td>
      <td>43323</td>
      <td>35532</td>
      <td>18711</td>
      <td>16104</td>
      <td>1795</td>
      <td>146</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>2</td>
      <td>11</td>
      <td>53</td>
      <td>26</td>
      <td>0.490566</td>
      <td>전라북도</td>
      <td>28740</td>
      <td>28260</td>
      <td>18725</td>
      <td>16022</td>
      <td>1829</td>
      <td>227</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>1</td>
      <td>8</td>
      <td>14</td>
      <td>10</td>
      <td>0.714286</td>
      <td>제주특별자치도</td>
      <td>30720</td>
      <td>30834</td>
      <td>18734</td>
      <td>16953</td>
      <td>635</td>
      <td>343</td>
    </tr>
    <tr>
      <th>15</th>
      <td>9</td>
      <td>5</td>
      <td>10</td>
      <td>67</td>
      <td>24</td>
      <td>0.358209</td>
      <td>충청남도</td>
      <td>52402</td>
      <td>40953</td>
      <td>18955</td>
      <td>16037</td>
      <td>2153</td>
      <td>262</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>57</td>
      <td>20</td>
      <td>0.350877</td>
      <td>충청북도</td>
      <td>42653</td>
      <td>34484</td>
      <td>18823</td>
      <td>15802</td>
      <td>1609</td>
      <td>217</td>
    </tr>
  </tbody>
</table>
</div>




```python
# csv, xlsx 파일로 내보내기

nstore_pop_df.to_csv(output_path+"nstore_stat_df.csv", index=False)
nstore_pop_df.to_excel(output_path+"nstore_stat_df.xlsx", index=False)

os.listdir(output_path)
```




    ['nstore_stat_df.xlsx',
     '.DS_Store',
     'nstore_stat_df_2.xlsx',
     'nstore_df.csv',
     'nstore_stat_df_2.csv',
     'nstore_df.xlsx',
     'nstore_stat_df.csv',
     'burger_df.csv',
     'burger_df.xlsx']



> `nstore_stat_df` 파일 전처리 완료

## 시군구별 인구수/면적 데이터 가져오기
#### 1. 데이터셋 1
- 출처 : 시군구별 인구수 [통계청](https://kosis.kr/statHtml/statHtml.do?orgId=315&tblId=TX_315_2009_H1001&conn_path=I2)
- 파일명: *도시지역_인구현황_시군구__20210703233117.csv* 또는 *도시지역_인구현황_시군구__20210703233121.xlsx*
- 설명 : (2019년 기준) 전국 시군구별 `인구[명]`

#### 2. 데이터셋 2
- 출처 : 시군구별 면적 [통계청](https://kosis.kr/statHtml/statHtml.do?orgId=315&tblId=TX_315_2009_H1009)
- 파일명 : *행정구역_현황_20210707012706.csv* 또는 *행정구역_현황_20210707012706.xlsx*
- 설명 : (2019년 기준) 전국 시군구별 `면적[m^2]`


```python
# 시군구별 인구수 데이터 가져오기
pop_all_df = pd.read_csv(input_path+'도시지역_인구현황_시군구__20210703233117.csv', encoding='cp949', skiprows=3)
pop_df = pop_all_df[["전국","소계","51849861"]]
pop_df.rename(columns={"51849861":"인구수"}, inplace=True)

# 시도별 소계 제거
idx_sum = pop_df.loc[pop_df["소계"]=='소계',:].index
pop_df.drop(index=idx_sum, inplace=True) 

# 시군구 컬럼 만들기
pop_df["시군구"] = pop_df["전국"]+" "+pop_df["소계"]
pop_df
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
      <th>전국</th>
      <th>소계</th>
      <th>인구수</th>
      <th>시군구</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>151290</td>
      <td>서울특별시 종로구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울특별시</td>
      <td>중구</td>
      <td>126171</td>
      <td>서울특별시 중구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울특별시</td>
      <td>용산구</td>
      <td>228670</td>
      <td>서울특별시 용산구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울특별시</td>
      <td>성동구</td>
      <td>300889</td>
      <td>서울특별시 성동구</td>
    </tr>
    <tr>
      <th>5</th>
      <td>서울특별시</td>
      <td>광진구</td>
      <td>351350</td>
      <td>서울특별시 광진구</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>240</th>
      <td>경상남도</td>
      <td>함양군</td>
      <td>39637</td>
      <td>경상남도 함양군</td>
    </tr>
    <tr>
      <th>241</th>
      <td>경상남도</td>
      <td>거창군</td>
      <td>62179</td>
      <td>경상남도 거창군</td>
    </tr>
    <tr>
      <th>242</th>
      <td>경상남도</td>
      <td>합천군</td>
      <td>45204</td>
      <td>경상남도 합천군</td>
    </tr>
    <tr>
      <th>244</th>
      <td>제주특별자치도</td>
      <td>제주시</td>
      <td>489405</td>
      <td>제주특별자치도 제주시</td>
    </tr>
    <tr>
      <th>245</th>
      <td>제주특별자치도</td>
      <td>서귀포시</td>
      <td>181584</td>
      <td>제주특별자치도 서귀포시</td>
    </tr>
  </tbody>
</table>
<p>229 rows × 4 columns</p>
</div>




```python
# 시군구별 면적 데이터 가져오기
area_all_df = pd.read_csv(input_path+"행정구역_현황_20210707012706.csv", encoding='cp949',skiprows=3)
area_df = area_all_df[["전국","소계","100401285021"]]
area_df.rename(columns={"100401285021":"면적"},inplace=True)

# 시도별 소계 제거
idx_sum = area_df.loc[area_df["소계"]=='소계',:].index
area_df.drop(index=idx_sum, inplace=True) 

# 면적 단위 환산 m^2 → km^2
area_df['면적'] = area_df["면적"]/10**6

# 시군구 컬럼 만들기
area_df["시군구"] = area_df["전국"]+" "+area_df["소계"]
area_df
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
      <th>전국</th>
      <th>소계</th>
      <th>면적</th>
      <th>시군구</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>23.913280</td>
      <td>서울특별시 종로구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울특별시</td>
      <td>중구</td>
      <td>9.959983</td>
      <td>서울특별시 중구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울특별시</td>
      <td>용산구</td>
      <td>21.866384</td>
      <td>서울특별시 용산구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울특별시</td>
      <td>성동구</td>
      <td>16.859343</td>
      <td>서울특별시 성동구</td>
    </tr>
    <tr>
      <th>5</th>
      <td>서울특별시</td>
      <td>광진구</td>
      <td>17.062949</td>
      <td>서울특별시 광진구</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>240</th>
      <td>경상남도</td>
      <td>함양군</td>
      <td>725.485214</td>
      <td>경상남도 함양군</td>
    </tr>
    <tr>
      <th>241</th>
      <td>경상남도</td>
      <td>거창군</td>
      <td>803.312949</td>
      <td>경상남도 거창군</td>
    </tr>
    <tr>
      <th>242</th>
      <td>경상남도</td>
      <td>합천군</td>
      <td>983.507159</td>
      <td>경상남도 합천군</td>
    </tr>
    <tr>
      <th>244</th>
      <td>제주특별자치도</td>
      <td>제주시</td>
      <td>978.668959</td>
      <td>제주특별자치도 제주시</td>
    </tr>
    <tr>
      <th>245</th>
      <td>제주특별자치도</td>
      <td>서귀포시</td>
      <td>871.558431</td>
      <td>제주특별자치도 서귀포시</td>
    </tr>
  </tbody>
</table>
<p>229 rows × 4 columns</p>
</div>




```python
# 행정구역 겹치지 않는지 확인
print( len(set(pop_df["시군구"]))==len(pop_df["시군구"]))
print( len(set(area_df["시군구"]))==len(area_df["시군구"]))

# 행정구역 이름 같은지 확인
print( set(pop_df["시군구"]) == set(area_df["시군구"]))

# 행정구역 길이 확인
print(len(set(pop_df["시군구"])))
# pop_df['시군구'].unique()
```

    True
    True
    True
    229



```python
# 인구수 + 면적 데이터 합치기
pop_area_df = pd.merge(pop_df,area_df, on='시군구', how='left')[["전국_x", "소계_x","인구수","시군구","면적"]]
pop_area_df.rename(columns={"전국_x":'광역시도',"소계_x":"시군구","시군구":"주소"}, inplace=True)

# 인구밀도 구하기 명/km^2
pop_area_df["인구밀도"] = pop_area_df["인구수"]/pop_area_df["면적"]
pop_area_df

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
      <th>광역시도</th>
      <th>시군구</th>
      <th>인구수</th>
      <th>주소</th>
      <th>면적</th>
      <th>인구밀도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울특별시</td>
      <td>종로구</td>
      <td>151290</td>
      <td>서울특별시 종로구</td>
      <td>23.913280</td>
      <td>6326.610151</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울특별시</td>
      <td>중구</td>
      <td>126171</td>
      <td>서울특별시 중구</td>
      <td>9.959983</td>
      <td>12667.792706</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울특별시</td>
      <td>용산구</td>
      <td>228670</td>
      <td>서울특별시 용산구</td>
      <td>21.866384</td>
      <td>10457.604696</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울특별시</td>
      <td>성동구</td>
      <td>300889</td>
      <td>서울특별시 성동구</td>
      <td>16.859343</td>
      <td>17847.018119</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울특별시</td>
      <td>광진구</td>
      <td>351350</td>
      <td>서울특별시 광진구</td>
      <td>17.062949</td>
      <td>20591.399529</td>
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
      <th>224</th>
      <td>경상남도</td>
      <td>함양군</td>
      <td>39637</td>
      <td>경상남도 함양군</td>
      <td>725.485214</td>
      <td>54.635159</td>
    </tr>
    <tr>
      <th>225</th>
      <td>경상남도</td>
      <td>거창군</td>
      <td>62179</td>
      <td>경상남도 거창군</td>
      <td>803.312949</td>
      <td>77.403209</td>
    </tr>
    <tr>
      <th>226</th>
      <td>경상남도</td>
      <td>합천군</td>
      <td>45204</td>
      <td>경상남도 합천군</td>
      <td>983.507159</td>
      <td>45.962045</td>
    </tr>
    <tr>
      <th>227</th>
      <td>제주특별자치도</td>
      <td>제주시</td>
      <td>489405</td>
      <td>제주특별자치도 제주시</td>
      <td>978.668959</td>
      <td>500.072058</td>
    </tr>
    <tr>
      <th>228</th>
      <td>제주특별자치도</td>
      <td>서귀포시</td>
      <td>181584</td>
      <td>제주특별자치도 서귀포시</td>
      <td>871.558431</td>
      <td>208.344035</td>
    </tr>
  </tbody>
</table>
<p>229 rows × 6 columns</p>
</div>




```python
# 시군구별 버거 매장 갯수 nstore_df 의 주소 이름이 같은지 확인
print(set(nstore_df['주소']) == set(pop_area_df['주소']))

# 서로 다른 주소 확인
print(set(nstore_df["주소"])-set(pop_area_df['주소'])) # nstore_df에는 있고, pop_area_df에는 없는 지역
print(set(pop_area_df['주소']) - set(nstore_df['주소'])) # pop_area_df에는 있고, nstore_df에는 없는 지역

```

    False
    set()
    {'전라남도 신안군', '경상북도 청송군', '경상북도 영양군', '인천광역시 동구', '경상북도 군위군'}



```python
# 브랜드별 매장수 데이터 셋과 인구/인구밀도 데이터 셋 합치기
final_df = pd.merge(nstore_df, pop_area_df, on='주소', how='inner')
final_df = final_df[['버거킹', 'KFC', '맥도날드', '롯데리아', 'BKM', '버거지수', '주소', '주소1', '주소2', '인구수', '면적', '인구밀도']]
final_df
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
      <th>롯데리아</th>
      <th>BKM</th>
      <th>버거지수</th>
      <th>주소</th>
      <th>주소1</th>
      <th>주소2</th>
      <th>인구수</th>
      <th>면적</th>
      <th>인구밀도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>6</td>
      <td>2</td>
      <td>0.333333</td>
      <td>강원도 강릉시</td>
      <td>강원도</td>
      <td>강릉시</td>
      <td>213442</td>
      <td>1040.783864</td>
      <td>205.078122</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0.666667</td>
      <td>강원도 속초시</td>
      <td>강원도</td>
      <td>속초시</td>
      <td>81786</td>
      <td>105.760448</td>
      <td>773.313668</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>15</td>
      <td>6</td>
      <td>0.400000</td>
      <td>강원도 원주시</td>
      <td>강원도</td>
      <td>원주시</td>
      <td>349215</td>
      <td>868.240238</td>
      <td>402.210108</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>7</td>
      <td>6</td>
      <td>0.857143</td>
      <td>강원도 춘천시</td>
      <td>강원도</td>
      <td>춘천시</td>
      <td>281291</td>
      <td>1116.373900</td>
      <td>251.968449</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>1.000000</td>
      <td>강원도 홍천군</td>
      <td>강원도</td>
      <td>홍천군</td>
      <td>69150</td>
      <td>1820.310462</td>
      <td>37.988025</td>
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
      <th>219</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 단양군</td>
      <td>충청북도</td>
      <td>단양군</td>
      <td>29756</td>
      <td>780.158660</td>
      <td>38.140960</td>
    </tr>
    <tr>
      <th>220</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 보은군</td>
      <td>충청북도</td>
      <td>보은군</td>
      <td>32949</td>
      <td>584.207531</td>
      <td>56.399478</td>
    </tr>
    <tr>
      <th>221</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 영동군</td>
      <td>충청북도</td>
      <td>영동군</td>
      <td>48738</td>
      <td>846.016198</td>
      <td>57.608826</td>
    </tr>
    <tr>
      <th>222</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 옥천군</td>
      <td>충청북도</td>
      <td>옥천군</td>
      <td>51023</td>
      <td>537.221176</td>
      <td>94.975780</td>
    </tr>
    <tr>
      <th>223</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>충청북도 증평군</td>
      <td>충청북도</td>
      <td>증평군</td>
      <td>37392</td>
      <td>81.797648</td>
      <td>457.128058</td>
    </tr>
  </tbody>
</table>
<p>224 rows × 12 columns</p>
</div>




```python
# csv, xlsx 파일로 내보내기
final_df.to_csv(output_path+"nstore_stat_df_2.csv", index=False)
final_df.to_excel(output_path+"nstore_stat_df_2.xlsx", index=False)

os.listdir(output_path)

```




    ['nstore_stat_df.xlsx',
     '.DS_Store',
     'nstore_stat_df_2.xlsx',
     'nstore_df.csv',
     'nstore_stat_df_2.csv',
     'nstore_df.xlsx',
     'nstore_stat_df.csv',
     'burger_df.csv',
     'burger_df.xlsx']



> `nstore_stat_df_2` 파일 전처리 완료


```python

```
