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
|상관관계 분석 및 시각화| 2021.07.02~07.07(4일)|브랜드별 매장 분포 지도 시각화, 상관관계 분석|
|심화 주제 진행 | 2021.07.08 (1일)|맘스터치 매장 분포 지도 시각화, 상관관계 분석|
|프로젝트 정리 및 발표| 2021.07.09 (1일)| 발표 자료, 프로젝트 보고서|

##  1차 결과 보고 (2021.07.02)
1.  맥도날드, 버거킹, KFC 전국 매장수 웹 크롤링 완료, 롯데리아 업데이트 진행 중
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
- 브랜드별 상관관계 : 예) 버거킹이 많은 지역에는 KFC도 많을까?
- 버거지수와 인구밀도/경제지표의 상관관계
- 추가) 전국 시구군별 매장수/밀도 colormap, markercluster



