team project 02. burger-index
# 버거 지수는 실제로 도시의 발전을 나타낼까?


도시 발전 지표로서의 버거 지수 = (맥도날드 매장수 + 버거킹 매장수 +  KFC 매장수)/롯데리아 매장수

## 목표
1차 : 버거지수와 지역별 인구수(혹은 인구밀도, 기타 경제지표)와의 상관관계  
2차 : 맘스터치 매장수를 포함한 버거지수 

## 과정
### 1차. 기존의 버거지수
1. [맥도날드](https://www.mcdonalds.co.kr/kor/main.do), [버거킹](https://www.burgerking.co.kr/#/home), [KFC](https://www.kfckorea.com/), [롯데리아](http://www.lotteria.com/index.asp)의 전국 매장수 웹 크롤링  
1-추가. 브랜드별 매장 분포 시각화 : 행정구역별 매장수 colormap, markercluster
2. 수집 데이터 전처리 (예. '서울특별시', '서울시' 등 동일한 행정구역 명칭을 하나로 통합/분류)
2. 지역별 인구수 혹은 인구밀도, 기타 경제지표 찾기 [통계청](https://kostat.go.kr/portal/korea/index.action)
3. 상관관계 분석
4. 시각화 : 산점도, 회귀직선

### 2차. 업그레이드 버거지수
1. [맘스터치](https://www.momstouch.co.kr/) 전국 매장수 웹 크롤링
2. 수집 데이터 전처리  
3. 상관관계 분석 전 변수 정의  
3-1. 버거지수 공식 모델링 (예. 맥도날드+버거킹+KFC+맘스터치)/롯데리아  
3-2. 브랜드별 가중치 계산 → 수치화   
4. 상관관계 분석  
5. 시각화 : 산점도, 회귀직선

## 계획
|과정|일정|예상 결과|
|--|--|--|
|웹 크롤링| 2021.06.28~07.01 (4일)|브랜드별 매장 주소(csv, xlsx 파일)|
|전처리| 2021.07.02 (1일) | 모든 버거 매장 정보(csv, xlsx 파일) |
|상관관계 분석 및 시각화| 2021.07.05~07.07(3일)|브랜드별 매장 분포 지도 시각화, 상관관계 분석|
|심화 주제 진행 | 2021.07.08 (1일)|맘스터치 매장 분포 지도 시각화, 상관관계 분석|
|프로젝트 정리 및 발표| 2021.07.09 (1일)| 발표 자료, 프로젝트 보고서|

## 주요 이슈 및 해결법
### 웹데이터 수집
- 롯데리아 웹데이터 수집
  - 롯데리아 홈페이지 페이지 로딩 속도 저하로 정확한 데이터 수집 불가
    - 현재 페이지 상태를 확인하여 페이지 이동이 될 때까지 while과 sleep을 사용하여 대기<br><br>
  - urllib의 urlopen 사용 시 서버 접근 제한이 되는 경우 발생
    - requests 모듈 사용하여 headers에 user-agent 입력<br><br>
  - 롯데리아 홈페이지에 다수 지점이 주소가 아닌 위치정보가 포함되어 있어 주소정보 확인 불가
    - 예시 )
      - 감일 : 감일백제로 109 1층
      - 번동D/T : 번동사거리 현대주유소 위치
      - 천안두정 : 두정역에서 100M, 주공8단지 정문 건너편에 위치
      - 수원정자 : 홈배달 가능합니디
    - 롯데리아 지점 명으로 네이버에서 다시 검색하여 주소데이터 수집<br><br>
  - 네이버에서 지점 명으로 검색 시 결과페이지 형식이 다른 때도 있어 오류
    - 예시 )
      - 단일 지점이 나오고 지도가 있음
      - 단일 지점이 나오고 지도가 없음
      - 지점이 목록으로 나옴
      - 지점이 목록으로 나오고 페이지 구조가 다름
      - 주소의 표시 형식이 다름
    - 각각의 형식에 맞춰서 예외처리 적용<br><br>
  - 네이버에 등록된 지점 명과 롯데이라 홈페이지의 지점 명이 달라서 검색 안 됨
    - 예시 )
      - 롯데리아 판교D/T
      - 롯데리아 전북진안
    - 네이버에서 검색할 수 있도록 지점명 전처리<br><br>
  - 롯데리아 홈페이지에 없어진 지점에 대한 정보가 존재
    - 네이버 검색 시 검색된 지점 명과 등록된 지점 명을 비교하여 없는 지점 None 처리<br><br>

### 전처리
### 상관관계 분석
### 시각화
- 지도시각화
  - 수집한 시군구 지도데이터에 시도가 아님에도 구까지 분류된 지역이 있음
    - 지도데이터의 가공이 가능한 [웹페이지](https://mapshaper.org/)를 이용하여 지도데이터 가공<br><br>

##  1차 결과 보고 (2021.07.02)
1.  맥도날드, 버거킹, KFC 전국 매장수 웹 크롤링 완료, 롯데리아 업데이트 진행 중
 - 공식 홈페이지를 통해 지점명과 주소를 가져옴
2.  맥도날드, 버거킹, KFC 전처리 완료
3.  시각화

- 전국 브랜드별 매장 수

![image](https://user-images.githubusercontent.com/38090151/124228920-d6552f80-db47-11eb-8ad1-4c234196b435.png)

- 전국 브랜드별 매장 수 + 롯데리아 추가

|수직바|
|---|
|![bar](https://user-images.githubusercontent.com/38090151/124228898-cf2e2180-db47-11eb-8787-c7d8fcbcf742.png) | 

|수평바|
|----|
|![barh](https://user-images.githubusercontent.com/38090151/124228902-d05f4e80-db47-11eb-97cf-e5d5bdfc38f8.png) |

|파이차트|
|-----|
|![pie](https://user-images.githubusercontent.com/38090151/124228904-d0f7e500-db47-11eb-9b11-478980b1f3f2.png) |


- 전국 광역시/특별시별 매장 수

![image (1)](https://user-images.githubusercontent.com/38090151/124228919-d5bc9900-db47-11eb-9527-ad10213b5227.png)


- 서울특별시 버거 지도


![image (2)](https://user-images.githubusercontent.com/38090151/124228906-d1907b80-db47-11eb-9df7-10a7a2a7a97d.png)


4. 추후 분석 방향성 공유
- 최종 데이터프레임 : 시+구군별 맥도날드/버거킹/KFC/롯데리아 매장수, 버거지수, 인구밀도 또는 경제지표
- 분석 1순위 
  - 브랜드별 상관관계 : 예) 버거킹이 많은 지역에는 KFC도 많지 않을까?
  - 버거지수와 인구밀도/경제지표의 상관관계 예) 버거 지수가 높을수록 더 발전된 도시일까?
- 분석 2순위
  - 전국 시구군별 매장수/밀도 colormap, markercluster



