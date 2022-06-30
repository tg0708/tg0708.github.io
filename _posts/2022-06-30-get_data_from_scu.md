---
layout: single
# title:  "get_data_from_scu"
categories: Project
tag: [Web Scraping, Selenium]
toc : true
author_profile : false
---

# Import


```python
from custom import selenium_custom # custom.py
from custom import IDPW # custom.py
from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
import warnings
warnings.filterwarnings('ignore')
import time
# xpath 대기 함수
def wait_by_xpath(xpath):
   element = WebDriverWait(driver,10).until(EC.presence_of_element_located((By.XPATH, xpath)))
   return element
```



# Login

## Create a privacy object


```python
# 로그인 및 주민번호 정보 객체 생성
logdata = IDPW.Get_logdata()
resident_num = IDPW.Get_resident_num()
log_id = logdata.get_id('서울사이버')
log_pw = logdata.get_pw('서울사이버')
certi_pw = logdata.get_pw('공인인증서')
resident_num = resident_num.back()
```

## Webdriver Setting


```python
# check driver version & path 
selenium_custom.driver_update()
driver_path = 'C:\\Users\\tgkang\\PyData\\크롤링\\103\\chromedriver.exe'

# Remove AutomationExtension
option = webdriver.ChromeOptions()
option.add_experimental_option("useAutomationExtension", False)
option.add_experimental_option("excludeSwitches", ['enable-automation'])
```

## Login


```python
# Excute driver
driver = webdriver.Chrome(driver_path, chrome_options=option)
url = "https://sign.iscu.ac.kr/ptl/sso/SsoLoginForm.scu"
driver.get(url)

# Maximize
driver.maximize_window()

# Wait xpath element
wait_by_xpath('//*[@id="userId"]')

# Input ID
element = driver.find_element_by_xpath('//*[@id="userId"]')
element.clear()
element.click()
element.send_keys(log_id)

# Input PW
element = driver.find_element_by_xpath('//*[@id="password"]')
element.clear()
element.click()
element.send_keys(log_pw)

# Click_login
element =  driver.find_element_by_xpath('//*[@id="detailLoginForm"]/div/div[2]/div[3]/div[1]/a')
element.click()

# Wait xpath element
wait_by_xpath('//*[@id="studentCertForm"]')

# 주민번호 뒷자리
# Switch ifame
driver.switch_to.frame('studentCertForm')

# Input resident number
element = driver.find_element_by_xpath('//*[@id="socNo2"]')
element.clear()
element.click()
element.send_keys(resident_num)

# Click login
element = driver.find_element_by_xpath('//*[@id="detailForm"]/div/div[2]/a/span')
element.click()
```

# Get data

## Subject list


```python
# Switch to default 
driver.switch_to.default_content()
# {subject :  xpath}
subject_list = {} 
# convert subject list to xpath
elements = driver.find_elements_by_xpath('//*[@id="lmsSubjList"]/table/tbody/tr')
for idx, element in enumerate(elements):
    sub_name = element.find_element_by_xpath('th[2]/a').text
    sub_element = f'//*[@id="lmsSubjList"]/table/tbody/tr[{idx+1}]/th[2]/a'
    subject_list[sub_name] = sub_element
    print(element.find_element_by_xpath('th[2]/a').text)
```

    데이터과학입문
    데이터베이스
    빅데이터기초수학
    파이썬프로그래밍


## Select a subject


```python
# Dictionary -> List
input_list = list(subject_list.keys())
# Select_subject
subject_select = input(f"과목을 입력하세요 : {input_list}")
# return seleted xpath
element = driver.find_element_by_xpath(subject_list[subject_select])
# Move to seleted page
element.click()
# wait
time.sleep(2)
```

    과목을 입력하세요 : ['데이터과학입문', '데이터베이스', '빅데이터기초수학', '파이썬프로그래밍']데이터과학입문


## Lecture list


```python
# Switch frame
driver.switch_to.frame('mainView')
dict_info = {}
# convert lecture list to xpath
elements = driver.find_elements_by_xpath('//*[@id="content"]/div[5]/div[2]/table/tbody/tr/td[2]')
for idx, element in enumerate(elements):
    print(idx + 1, element.text)
input_idx = input("강좌번호 입력 : ")
# return seleted lecture xpath
class_button = '//*[@id="content"]/div[5]/div[2]/table/tbody/tr[' + input_idx + ']/td[3]/button'
class_button
```

    1 오리엔테이션
    2 미래 AI 기술의 변화
    3 빅데이터 - 스포츠 분야를 중심으로
    4 분석(分析, Analysis)
    5 전략
    6 글쓰기(자기소개서와 사업계획서를 중심으로)
    7 창의성 - 우리 시대 최고로 각광받는 개인역량
    8 중간고사
    9 역량 - 우리 시대 CEO들이 가장 애타게 찾는 인재들의 핵심역량
    10 리더십 - 자기 자신과 타인에 관한
    11 창업 - 대한민국 55세 이후 일자리 비율 1위(자영업)
    12 행복
    13 시련대처 - 아무도 원치 않지만 피할 수 없는 인생의 위기들
    14 자기분석
    15 기말고사
    강좌번호 입력 : 1
    '//*[@id="content"]/div[5]/div[2]/table/tbody/tr[1]/td[3]/button'



## Select a lecture


```python
# Click lecture
driver.find_element_by_xpath(class_button).click()
time.sleep(2)
# Switch frame
driver.switch_to.window(driver.window_handles[1])

# Certification login 
element = driver.find_element_by_xpath('//*[@id="LoginLayer"]/div/map/area[1]')
element.click()

# Wait untill Certi password box element come
wait_by_xpath('//*[@id="certPwd"]')
time.sleep(3)

#  Select certi item
element = driver.find_element_by_xpath('//*[@id="1"]')
element.click()

# Input certi PW
element = driver.find_element_by_xpath('//*[@id="certPwd"]')
element.send_keys(certi_pw)

# Click to login
element = driver.find_element_by_xpath('//*[@id="nx-cert-select"]/div[4]/button[1]')
element.click()

# Maximize 
driver.maximize_window()
```




## Lecture index


```python
time.sleep(2)
# 강의 인덱스 설정
contentid = []
class_index = driver.find_elements_by_css_selector('div[contentid]')
for idx, index in enumerate(class_index[2:]):
    contentid.append(index.get_attribute('contentid'))
    print(idx, index.text)
```

    0 교수 소개
    1 강의 개요
    2 창조적 문제해결력
    3 고정관념 벗어나기
    4 데이터 분석의 핵심
    5 분석의 활용사례
    6 개요
    7 일상적 업데이트
    8 사례연구와 활용
    9 분석적 사고와 실행
    10 학습평가
    11 학습정리
