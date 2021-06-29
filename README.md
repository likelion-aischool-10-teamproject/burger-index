team project 02. burger-index
# 버거 지수는 실제로 도시의 발전을 나타낼까?

버거 지수 = (맥도날드 매장수 + 버거킹 매장수 +  KFC 매장수)/롯데리아 매장수

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
3. 상관관계 분석 전 변수 
3-1. 공식 모델링 (예. 맥도날드+버거킹+KFC+맘스터치)/롯데리아  
3-2. 브랜드별 가중치 계산 → 수치화   
4. 상관관계 분석  
