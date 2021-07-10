```python
from selenium import webdriver
from bs4 import BeautifulSoup
import time
import pandas as pd
import os
import platform
```


```python
# OS 별로 웹드라이버 로드(windows, mac)
if platform.system() == "Windows":
    driver = webdriver.Chrome('../chromedriver_90')
elif platform.system()=="Darwin":
    driver = webdriver.Chrome('../chromedriver')
else:
    print("Unknown System")
    
url = 'https://www.kfckorea.com/store/findStore'
driver.get(url)
time.sleep(3)
```


```python
# 지역검색
loc_a = driver.find_element_by_xpath('//*[@id="app"]/div[2]/div/div/div[2]/div/div[2]/div/div/ul/li[2]/a')
loc_a.click()
time.sleep(1)
```


```python
# 시도 선택
loc_list1 = driver.find_element_by_xpath('//*[@id="app"]/div[2]/div/div/div[2]/div/div[2]/div/div/div/div/div[1]/select[1]')

# 시도 리스트 길이
list1_len = len(loc_list1.text.split()[1:])

for i in range(2, 2 + list1_len) :

    store_name = []
    store_addr = []
    
    loc_list1.click()
    time.sleep(1)

    xpath1 = f'//*[@id="app"]/div[2]/div/div/div[2]/div/div[2]/div/div/div/div/div[1]/select[1]/option[{i}]'
    loc_list1_cont = driver.find_element_by_xpath(xpath1)
    loc_list1_cont.click()
    time.sleep(1)

    loc_list1.click()
    time.sleep(1)

    # 구군 선택
    loc_list2 = driver.find_element_by_xpath('//*[@id="app"]/div[2]/div/div/div[2]/div/div[2]/div/div/div/div/div[1]/select[2]')
    list2_len = len(loc_list2.find_elements_by_xpath('option')[1:])
    
    for j in range(2, 2 + list2_len) :

        loc_list2.click()
        time.sleep(1)

        xpath2 = f'//*[@id="app"]/div[2]/div/div/div[2]/div/div[2]/div/div/div/div/div[1]/select[2]/option[{j}]'
        loc_list2_cont = driver.find_element_by_xpath(xpath2)
        loc_list2_cont.click()
        time.sleep(1)

        loc_list2.click()
        time.sleep(1)
        
        print(loc_list2_cont.text)

        # 매장정보 수집
        page = driver.page_source
        soup = BeautifulSoup(page, 'lxml')

        store_list = soup.select_one('div.swiper-slide').select('ul.store-item')

        if len(store_list) > 0 :

            for o in store_list :
                
                # 매장명
                temp_store_name = o.select_one('a').text.strip()
                store_name.append(temp_store_name)

                # 매장주소
                temp_store_addr = o.select_one('li.point').find_next().text
                store_addr.append(temp_store_addr)
                
    print(len(store_name), len(store_addr))
        
    dat_df = pd.DataFrame({'매장명':store_name,
                           '매장 주소':store_addr})
    

    
    # 폴더 확인 후 없으면 만듦
    path = './data/'
    if not os.path.isdir(path):                                                           
        os.mkdir(path)
        
#     file_name 에 연월일, 시분초 추가시    
#     now_time = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
#     file_name = f'KFC매장_{loc_list1_cont.text}_{now_time}.xlsx'

    file_name = f'KFC매장_{loc_list1_cont.text}.xlsx'
    dat_df.to_excel(path + file_name, index=False)
    
    if file_name in os.listdir('./data') :
        print(f'{file_name} 저장 완료')
    else : 
        print(f'{file_name} 저장 실패')


print('데이터 수집 완료')
```

    강남구
    강동구
    강북구
    강서구
    관악구
    광진구
    구로구
    금천구
    노원구
    도봉구
    동대문구
    동작구
    마포구
    서대문구
    서초구
    성동구
    성북구
    송파구
    양천구
    영등포구
    용산구
    은평구
    종로구
    중구
    중랑구
    73 73
    KFC매장_서울.xlsx 저장 완료
    강서구
    금정구
    남구
    동구
    동래구
    부산진구
    북구
    사상구
    사하구
    서구
    수영구
    연제구
    영도구
    중구
    해운대구
    기장군
    9 9
    KFC매장_부산.xlsx 저장 완료
    달서구
    동구
    북구
    서구
    수성구
    중구
    달성군
    남구
    10 10
    KFC매장_대구.xlsx 저장 완료
    계양구
    남구
    남동구
    동구
    부평구
    서구
    연수구
    중구
    강화군
    옹진군
    11 11
    KFC매장_인천.xlsx 저장 완료
    광산구
    남구
    동구
    북구
    서구
    3 3
    KFC매장_광주.xlsx 저장 완료
    대덕구
    동구
    서구
    유성구
    중구
    5 5
    KFC매장_대전.xlsx 저장 완료
    남구
    동구
    북구
    중구
    울주군
    2 2
    KFC매장_울산.xlsx 저장 완료
    세종시
    1 1
    KFC매장_세종.xlsx 저장 완료
    강릉시
    동해시
    삼척시
    속초시
    원주시
    춘천시
    태백시
    고성군
    양구군
    양양군
    영월군
    인제군
    정선군
    철원군
    평창군
    홍천군
    화천군
    횡성군
    3 3
    KFC매장_강원.xlsx 저장 완료
    고양시 덕양구
    고양시 일산동구
    고양시 일산서구
    과천시
    광명시
    광주시
    구리시
    군포시
    김포시
    남양주시
    동두천시
    부천시 소사구
    부천시 오정구
    부천시 원미구
    성남시 분당구
    성남시 수정구
    성남시 중원구
    수원시 권선구
    수원시 영통구
    수원시 장안구
    수원시 팔달구
    시흥시
    안산시 단원구
    안산시 상록구
    안성시
    안양시 동안구
    안양시 만안구
    양주시
    오산시
    용인시 기흥구
    용인시 수지구
    용인시 처인구
    의왕시
    의정부시
    이천시
    파주시
    평택시
    포천시
    하남시
    화성시
    가평군
    양평군
    여주군
    연천군
    50 50
    KFC매장_경기.xlsx 저장 완료
    거제시
    김해시
    밀양시
    사천시
    양산시
    진주시
    창원시 마산합포구
    창원시 마산회원구
    창원시 성산구
    창원시 의창구
    창원시 진해구
    통영시
    거창군
    고성군
    남해군
    산청군
    의령군
    창녕군
    하동군
    함안군
    함양군
    합천군
    4 4
    KFC매장_경남.xlsx 저장 완료
    경산시
    경주시
    구미시
    김천시
    문경시
    상주시
    안동시
    영주시
    영천시
    포항시 남구
    포항시 북구
    고령군
    군위군
    봉화군
    성주군
    영덕군
    영양군
    예천군
    울릉군
    울진군
    의성군
    청도군
    청송군
    칠곡군
    4 4
    KFC매장_경북.xlsx 저장 완료
    광양시
    나주시
    목포시
    순천시
    여수시
    강진군
    고흥군
    곡성군
    구례군
    담양군
    무안군
    보성군
    신안군
    영광군
    영암군
    완도군
    장성군
    장흥군
    진도군
    함평군
    해남군
    화순군
    2 2
    KFC매장_전남.xlsx 저장 완료
    군산시
    김제시
    남원시
    익산시
    전주시 덕진구
    전주시 완산구
    정읍시
    고창군
    무주군
    부안군
    순창군
    완주군
    임실군
    장수군
    진안군
    2 2
    KFC매장_전북.xlsx 저장 완료
    제주시
    서귀포시
    1 1
    KFC매장_제주.xlsx 저장 완료
    계룡시
    공주시
    논산시
    당진시
    보령시
    서산시
    아산시
    천안시 동남구
    천안시 서북구
    금산군
    부여군
    서천군
    예산군
    청양군
    태안군
    홍성군
    5 5
    KFC매장_충남.xlsx 저장 완료
    제천시
    청주시 상당구
    청주시 흥덕구
    충주시
    괴산군
    단양군
    보은군
    영동군
    옥천군
    음성군
    증평군
    진천군
    청원군
    청주시 서원구
    청주시 청원구
    2 2
    KFC매장_충북.xlsx 저장 완료
    데이터 수집 완료



```python
# 웹드라이버 종료
driver.close()
```


```python
# 전체 KFC 매장 파일 리스트
path = './data/'

file_list = os.listdir(path)
file_list
```




    ['KFC매장_강원.xlsx',
     'KFC매장_경기.xlsx',
     'KFC매장_경남.xlsx',
     'KFC매장_경북.xlsx',
     'KFC매장_광주.xlsx',
     'KFC매장_대구.xlsx',
     'KFC매장_대전.xlsx',
     'KFC매장_부산.xlsx',
     'KFC매장_서울.xlsx',
     'KFC매장_세종.xlsx',
     'KFC매장_울산.xlsx',
     'KFC매장_인천.xlsx',
     'KFC매장_전남.xlsx',
     'KFC매장_전북.xlsx',
     'KFC매장_제주.xlsx',
     'KFC매장_충남.xlsx',
     'KFC매장_충북.xlsx']




```python
# 전체 KFC 매장 합치기
kfc_df = pd.DataFrame({})

for i in file_list 
    temp_df = pd.read_excel(path + i)
    kfc_df = pd.concat([kfc_df, temp_df], ignore_index=True)
    
kfc_df
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
      <td>원주단계DT</td>
      <td>강원 원주시 북원로 2266 (단계동) KFC원주단계DT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천이마트</td>
      <td>강원 춘천시 경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
    </tr>
    <tr>
      <th>2</th>
      <td>춘천석사</td>
      <td>강원 춘천시 영서로 2027 (석사동)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>행신역</td>
      <td>경기 고양시 덕양구 충장로 8 (행신동)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>화정</td>
      <td>경기 고양시 덕양구 화신로272번길 57 (화정동)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>일산장항</td>
      <td>경기 고양시 일산동구 정발산로 27 (장항동)</td>
    </tr>
    <tr>
      <th>6</th>
      <td>주엽점</td>
      <td>경기 고양시 일산서구 중앙로 1419 (주엽동)</td>
    </tr>
    <tr>
      <th>7</th>
      <td>일산후곡</td>
      <td>경기 고양시 일산서구 일산로 577 (일산동) 문화촌상가 1층</td>
    </tr>
    <tr>
      <th>8</th>
      <td>철산역</td>
      <td>경기 광명시 철산로 25 (철산동) 한영빌딩</td>
    </tr>
    <tr>
      <th>9</th>
      <td>하안동</td>
      <td>경기 광명시 하안로 289 (하안동)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>구리돌다리</td>
      <td>경기 구리시 경춘로 221 (인창동)</td>
    </tr>
    <tr>
      <th>11</th>
      <td>산본역</td>
      <td>경기 군포시 산본로323번길 16-14 (산본동) 2층</td>
    </tr>
    <tr>
      <th>12</th>
      <td>군포산본</td>
      <td>경기 군포시 고산로 681 (산본동)</td>
    </tr>
    <tr>
      <th>13</th>
      <td>김포한강</td>
      <td>경기 김포시 김포한강7로 93 (구래동)</td>
    </tr>
    <tr>
      <th>14</th>
      <td>김포</td>
      <td>경기 김포시 돌문로 43 (사우동) 1층</td>
    </tr>
    <tr>
      <th>15</th>
      <td>덕소</td>
      <td>경기 남양주시 와부읍 덕소로97번길 3</td>
    </tr>
    <tr>
      <th>16</th>
      <td>부천역</td>
      <td>경기 부천시 부천로 1 (심곡본동) 지하1층</td>
    </tr>
    <tr>
      <th>17</th>
      <td>부천중동</td>
      <td>경기 부천시 중동로254번길 95 (중동)</td>
    </tr>
    <tr>
      <th>18</th>
      <td>웅진플레이도시</td>
      <td>경기 부천시 조마루로 2 (상동)</td>
    </tr>
    <tr>
      <th>19</th>
      <td>부천상동HP</td>
      <td>경기 부천시 길주로 118 (상동) 삼성홈플러스1층 (상동)</td>
    </tr>
    <tr>
      <th>20</th>
      <td>정자역</td>
      <td>경기 성남시 분당구 느티로 27 (정자동)</td>
    </tr>
    <tr>
      <th>21</th>
      <td>야탑역</td>
      <td>경기 성남시 분당구 성남대로916번길 7 (야탑동)</td>
    </tr>
    <tr>
      <th>22</th>
      <td>서현역</td>
      <td>경기 성남시 분당구 분당로53번길 19 (서현동)</td>
    </tr>
    <tr>
      <th>23</th>
      <td>분당정자</td>
      <td>경기 성남시 분당구 정자로 88 (정자동) 성심빌딩 1층</td>
    </tr>
    <tr>
      <th>24</th>
      <td>미금역</td>
      <td>경기 성남시 분당구 성남대로 151 (구미동) 엠코헤리츠1층</td>
    </tr>
    <tr>
      <th>25</th>
      <td>위례중앙</td>
      <td>경기 성남시 수정구 위례광장로 104 (창곡동)</td>
    </tr>
    <tr>
      <th>26</th>
      <td>성남태평</td>
      <td>경기 성남시 수정구 수정로 185 (태평동) 1층</td>
    </tr>
    <tr>
      <th>27</th>
      <td>영통씨네마</td>
      <td>경기 수원시 영통구 청명남로 40 (영통동) 영통시네마 1층</td>
    </tr>
    <tr>
      <th>28</th>
      <td>수원역광장</td>
      <td>경기 수원시 팔달구 향교로 5 (매산로1가) 2층</td>
    </tr>
    <tr>
      <th>29</th>
      <td>아주대</td>
      <td>경기 수원시 팔달구 아주로 37 (우만동) 아록빌딩 1층</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>충무로역</td>
      <td>서울 중구 퇴계로 213 (충무로4가)</td>
    </tr>
    <tr>
      <th>158</th>
      <td>먹골역</td>
      <td>서울 중랑구 공릉로 8 (묵동, 대길빌딩)</td>
    </tr>
    <tr>
      <th>159</th>
      <td>사가정</td>
      <td>서울 중랑구 사가정로 389 (면목동) 1층</td>
    </tr>
    <tr>
      <th>160</th>
      <td>망우동</td>
      <td>서울 중랑구 망우로 410 (망우동)</td>
    </tr>
    <tr>
      <th>161</th>
      <td>세종이마트</td>
      <td>세종특별자치시 금송로 687 (가람동)</td>
    </tr>
    <tr>
      <th>162</th>
      <td>울산현대</td>
      <td>울산 남구 삼산중로 71 (달동)</td>
    </tr>
    <tr>
      <th>163</th>
      <td>울산명덕</td>
      <td>울산 동구 방어진순환도로 909 (서부동)</td>
    </tr>
    <tr>
      <th>164</th>
      <td>계산동</td>
      <td>인천 계양구 계양대로 214 (계산동) 에이스타운 1층</td>
    </tr>
    <tr>
      <th>165</th>
      <td>인하대</td>
      <td>인천 미추홀구 인하로77번길 8 (용현동) 1층</td>
    </tr>
    <tr>
      <th>166</th>
      <td>인천논현</td>
      <td>인천 남동구 논고개로 61 (논현동)</td>
    </tr>
    <tr>
      <th>167</th>
      <td>인천구월</td>
      <td>인천 남동구 남동대로 784 (구월동) 1,2층</td>
    </tr>
    <tr>
      <th>168</th>
      <td>간석HP</td>
      <td>인천 남동구 경원대로 971 (간석동) 홈플러스 2층</td>
    </tr>
    <tr>
      <th>169</th>
      <td>부평역</td>
      <td>인천 부평구 광장로 16 (부평동) 지하</td>
    </tr>
    <tr>
      <th>170</th>
      <td>검단역</td>
      <td>인천 서구 완정로 172 (마전동)</td>
    </tr>
    <tr>
      <th>171</th>
      <td>인천청라</td>
      <td>인천 서구 중봉대로586번길 19 (연희동)</td>
    </tr>
    <tr>
      <th>172</th>
      <td>인천스퀘어원</td>
      <td>인천 연수구 청능대로 210 (동춘동) SQUARE 1 외부광장동 1층</td>
    </tr>
    <tr>
      <th>173</th>
      <td>인천공항교통센터</td>
      <td>인천 중구 공항로 271 (운서동) 지하 1F T2-02</td>
    </tr>
    <tr>
      <th>174</th>
      <td>동인천이마트</td>
      <td>인천 중구 인중로 134 (신생동)  동인천이마트 1층 푸드코트 內</td>
    </tr>
    <tr>
      <th>175</th>
      <td>목포이마트</td>
      <td>전남 목포시 옥암로 138 (옥암동) 목포이마트 1층 푸드코트 內</td>
    </tr>
    <tr>
      <th>176</th>
      <td>순천연향동</td>
      <td>전남 순천시 연향중앙상가길 9 (연향동)</td>
    </tr>
    <tr>
      <th>177</th>
      <td>전주영화</td>
      <td>전북 전주시 완산구 전주객사4길 24-14 (고사동)</td>
    </tr>
    <tr>
      <th>178</th>
      <td>전북도청</td>
      <td>전북 전주시 완산구 효자로 194 (효자동3가) 로자벨시티 106호</td>
    </tr>
    <tr>
      <th>179</th>
      <td>서귀포중문DT</td>
      <td>제주특별자치도 서귀포시 중문관광로 90 (색달동)</td>
    </tr>
    <tr>
      <th>180</th>
      <td>당진</td>
      <td>충남 당진시 당진중앙2로 101 (읍내동) 1층</td>
    </tr>
    <tr>
      <th>181</th>
      <td>아산터미널</td>
      <td>충남 아산시 번영로 223 (모종동) 아산고속터미널 1층 102~104호</td>
    </tr>
    <tr>
      <th>182</th>
      <td>천안터미널</td>
      <td>충남 천안시 동남구 만남로 43 (신부동) 터미널(승차장) 내</td>
    </tr>
    <tr>
      <th>183</th>
      <td>천안쌍용</td>
      <td>충남 천안시 서북구 충무로 205 (쌍용동)</td>
    </tr>
    <tr>
      <th>184</th>
      <td>천안불당</td>
      <td>충남 천안시 서북구 불당26로 50 (불당동, 천안불당지웰시티푸르지오2단지) F10...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>청주메가폴리스</td>
      <td>충북 청주시 흥덕구 풍산로 15 (가경동) 메가폴리스 1층</td>
    </tr>
    <tr>
      <th>186</th>
      <td>청주수곡DT</td>
      <td>충북 청주시 서원구 수영로7번길 7 (수곡동)</td>
    </tr>
  </tbody>
</table>
<p>187 rows × 2 columns</p>
</div>



### 세종시는 구가 없기 떄문에 전처리 필요
 - '세종특별자치시'의 주소2를 '세종특별자치시'로 넣어주기 위한 전처리


```python
kfc_df[['주소1', '주소2', '주소3']] = pd.DataFrame(kfc_df['매장 주소'].str.split(' ', 2).tolist())
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
      <td>강원</td>
      <td>원주시</td>
      <td>북원로 2266 (단계동) KFC원주단계DT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천이마트</td>
      <td>강원 춘천시 경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
      <td>강원</td>
      <td>춘천시</td>
      <td>경춘로 2341 (온의동) 이마트 춘천점 1층 푸드코트 內</td>
    </tr>
    <tr>
      <th>2</th>
      <td>춘천석사</td>
      <td>강원 춘천시 영서로 2027 (석사동)</td>
      <td>강원</td>
      <td>춘천시</td>
      <td>영서로 2027 (석사동)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>행신역</td>
      <td>경기 고양시 덕양구 충장로 8 (행신동)</td>
      <td>경기</td>
      <td>고양시</td>
      <td>덕양구 충장로 8 (행신동)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>화정</td>
      <td>경기 고양시 덕양구 화신로272번길 57 (화정동)</td>
      <td>경기</td>
      <td>고양시</td>
      <td>덕양구 화신로272번길 57 (화정동)</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 주소1에 이상치를 확인
kfc_df['주소1'].unique()
```




    array(['강원', '경기', '경남', '경북', '광주', '대구', '대전', '부산', '서울', '세종특별자치시',
           '울산', '인천', '전남', '전북', '제주특별자치도', '충남', '충북'], dtype=object)




```python
# 주소1의 줄임말을 풀어서 치환
addr = {'서울':'서울특별시',
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
        '제주':'제주특별자치도'}

kfc_df['주소1'] = kfc_df['주소1'].replace(addr)

kfc_df['주소1'].unique()
```




    array(['강원도', '경기도', '경상남도', '경상북도', '광주광역시', '대구광역시', '대전광역시', '부산광역시',
           '서울특별시', '세종특별자치시', '울산광역시', '인천광역시', '전라남도', '전라북도', '제주특별자치도',
           '충청남도', '충청북도'], dtype=object)




```python
kfc_df.loc[kfc_df['주소1'] == '세종특별자치시'].
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
      <th>161</th>
      <td>세종이마트</td>
      <td>세종특별자치시 금송로 687 (가람동)</td>
      <td>세종특별자치시</td>
      <td>금송로</td>
      <td>687 (가람동)</td>
    </tr>
  </tbody>
</table>
</div>




```python
addr1 = addr2 = '세종특별자치시'
addr3 = '금송로 687 (가람동)'
addr = f'{addr1} {addr3}'

kfc_df.loc[ kfc_df['매장명'] == '세종이마트', '매장 주소': ] = addr, addr1, addr2, addr3
```


```python
kfc_df.loc[ kfc_df['매장명'] == '세종이마트']
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
      <th>161</th>
      <td>세종이마트</td>
      <td>세종특별자치시 금송로 687 (가람동)</td>
      <td>세종특별자치시</td>
      <td>세종특별자치시</td>
      <td>금송로 687 (가람동)</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 파일 출력
kfc_df.to_excel('KFC_all_store.xlsx', index=False)
```


```python

```
