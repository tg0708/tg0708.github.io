---
layout: single
title:  "get_data_from_knou"
categories: Project
tag: [Web Scraping, Selenium]
toc : true
author_profile : false
---


```python
from custom import selenium_custom # custom
from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
import warnings
warnings.filterwarnings('ignore')

import time
import pyautogui
import pyperclip
import pandas as pd

# Wait until xpath element come
def wait_by_xpath(xpath):
   element = WebDriverWait(driver,10).until(EC.presence_of_element_located((By.XPATH, xpath)))
   return element
```

# Login


```python
selenium_custom.driver_update()
driver_path = 'C:\\Users\\tgkang\\PyData\\크롤링\\101\\chromedriver.exe'
# 자동화 문구 제거
option = webdriver.ChromeOptions()
option.add_experimental_option("useAutomationExtension", False)
option.add_experimental_option("excludeSwitches", ['enable-automation'])
# 드라이버 실행
# driver = webdriver.Chrome(driver_path, chrome_options=option)
driver = webdriver.Chrome(driver_path)
url = "https://www.knou.ac.kr/knou/index.do?epTicket=INV"
driver.get(url)
```

    크롬드라이버 버전이 최신입니다.


```python
# 전체화면
driver.maximize_window()

log_id = '*******'
log_pw = '*******'
wait_by_xpath('//*[@id="btnLogin"]')

# 로그인
btn_login = driver.find_element_by_xpath('//*[@id="btnLogin"]')
btn_login .click()
```


```python
# 아이디 입력
wait_by_xpath('//*[@id="username"]')
user_name = driver.find_element_by_xpath('//*[@id="username"]')
user_name.send_keys(log_id)  
```


```python
# 비밀번호 입력
password = driver.find_element_by_xpath('//*[@id="password"]')
password.send_keys(log_pw)
```


```python
# 로그인 클릭
click_login = driver.find_element_by_xpath('//*[@id="btn_login"]')
click_login.click()
```


```python
# 나의 수강정보
url = 'https://ucampus.knou.ac.kr/ekp/user/study/retrieveUMYStudy.sdo'
driver.get(url)
```


```python
# 수강과목 리스트 조회
wait_by_xpath('/html/body/div[2]/div[1]/div/div/div/div/form/div/div/div[1]/div[2]/div[2]/div/div[1]/div[1]/h3/a')
class_elements = driver.find_elements_by_xpath('/html/body/div[2]/div[1]/div/div/div/div/form/div/div/div[1]/div[2]/div[2]/div/div[1]/div[1]/h3/a')
for class_element in class_elements:
    print(class_element.text)
```

    원격대학교육의이해
    세계의역사
    컴퓨터의이해
    무역학원론
    무역관습개론
    무역보험론
    세계의음식·음식의세계


# Select lecture


```python
# Select lecture
wait_by_xpath('/html/body/div[2]/div[1]/div/div/div/div/form/div/div/div[1]/div[2]/div[2]/div[3]/div[2]/ul/li[5]/a[1]')
select_lecture = driver.find_element_by_xpath('/html/body/div[2]/div[1]/div/div/div/div/form/div/div/div[1]/div[2]/div[2]/div[3]/div[2]/ul/li[8]/a[1]')
select_lecture.click()

# switch frame
time.sleep(1)
driver.switch_to.window(driver.window_handles[1])
driver.maximize_window()

iframe_id = driver.find_element_by_id('ifrmVODPlayer_0')
driver.switch_to.frame(iframe_id)

play_btn =  driver.find_element_by_xpath('//*[@id="player0"]/div[6]/div[1]/div')
play_btn.click()

# Click start button 
try:
    continue_no =  driver.find_element_by_xpath('//*[@id="wp_elearning_play"]')
    continue_no.click()
except: 
    pass
```


```python
# pyautogui 위치값 선택
chrome_icon = 711,1060
class_page = 811,974
subtitle_setting = 1766,913
subtitle_hover = 1789,776
subtitle_korean = 1692,832
```


```python
# Click chrome browser
pyautogui.moveTo(chrome_icon)
time.sleep(2)
pyautogui.click()
time.sleep(2)
# Click class page
pyautogui.moveTo(class_page)
pyautogui.click()
# Click subtitle setting
pyautogui.moveTo(subtitle_setting)
time.sleep(4)
# hover
pyautogui.moveTo(subtitle_hover)
time.sleep(0.5)
pyautogui.click()
time.sleep(0.3)
# Click(Korea subtitle)
pyautogui.moveTo(subtitle_korean)
pyautogui.click()

```

# Get subtitle data


```python
# subtitle list
subtitle_list = []
while True:
    subtitle =  driver.find_element_by_xpath('/html/body/div/div[6]/div[3]/div[1]/div/span')
    # 동일한 택스트가 없을 때만 list에 추가
    if subtitle.text not in subtitle_list:
        subtitle_list.append(subtitle.text)
        # 2초 간격으로 지속적으로 자막 탐색
        time.sleep(2)
        print(subtitle.text)
```

    그래서 정보통신기술 쪽에서는 제 3의 물결이라는 그 책에 대한 이야기와 함께 정보사회 그리고 정보사회를 구동하고 있는 정보통신기술에 대해서 살펴보겠습니다.
    그리고 분야별 정보통신기술에서는 일곱가지 정치 경제 과학 기술 교육 문화 예술 의료 스포츠 뭐 그 밖에도 많이 있습니다마는 그건 여러분이 한번 직접 살펴봐 주시고요
    이런 한 일곱 가지 정도에서 정보통신기술이 어떻게 활용되고 있는지 우리 사회의 곳곳에 지금 숨어서 동작하고 있는 이 정보통신기술에 대해서 같이 살펴보겠습니다.
    이 강의를 다 듣고 나면 여러분은 다음과 같은 것을 하실 수 있어야 됩니다
    그것이 바로 학습 목표, 학습 목표로 하면 이 강의를 만든 사람이 무엇을 전달할 것인가 결국은 맨 마지막에 이것에 대해서 평가를 하겠죠.. `````` 중략


​    
