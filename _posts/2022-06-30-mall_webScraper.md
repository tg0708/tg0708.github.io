---
layout: single
title:  "mall_webScraper"
categories: Study
tag: [Web Scraping, Requests]
toc : true
author_profile : false
sidebar:
    nav: "docs" 
---


# Import


```python
from bs4 import BeautifulSoup
import requests
import re
import pandas as pd
```

# Create DataFrame


```python
col = ['Date', 'Category', 'Mall','Time','Item','Price','Image']
df = pd.DataFrame(columns=col)
df
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
      <th>Date</th>
      <th>Category</th>
      <th>Mall</th>
      <th>Time</th>
      <th>Item</th>
      <th>Price</th>
      <th>Image</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



# Parse HTML


```python
# Input date
date = input('Please input a date : ')
res = requests.get(f'https://www.hsmoa.com/?date={date}&site=&cate=')
soup = BeautifulSoup(res.content, "html.parser")
```

    Please input a date : 2022-06-30


# Get Data


```python
container = soup.select('div.timeline-item')
for idx, item in enumerate(container):
    # Item
    item_content = item.select_one('.font-15').text.strip()

    # Convert html to string    
    convert_str = str(item)
    convert_str = convert_str.split('>')[0]

    # Mall
    mall_content = convert_str.split('timeline-item ')[1].split(" ")[0]

    # Category
    p = re.compile('[가-힣]')
    result = p.findall(convert_str)
    category_content = "".join(result)

    # Time
    try :
        time_contents = item.select_one('.font-12').text.strip()
        time_start = time_contents.split("\n")[0]
        time_end = time_contents.split("\n")[1].strip()
        time_content = time_start + time_end
    except:
        time_content = "-"

    # Price
    price_contents = item.select_one('.strong.font-17').text.strip()

    # Image
    str_img = str(item.select_one('img.lazy'))
    img_content = str_img.split('smart/')[1].split('"')[0]

    # Parse items to dict
    dict_info = {'Date':date, 'Category': category_content,'Mall':mall_content,
             'Time':time_content,'Item':item_content,'Price':price_contents,'Image':img_content}

    df = df.append(dict_info, ignore_index=True)

df
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
      <th>Date</th>
      <th>Category</th>
      <th>Mall</th>
      <th>Time</th>
      <th>Item</th>
      <th>Price</th>
      <th>Image</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-06-30</td>
      <td>식품건강</td>
      <td>immall</td>
      <td>0시 00분 ~0시 50분</td>
      <td>화통 직화 ★닭발 세트★ (닭발8팩+근위1팩)</td>
      <td>39,900원</td>
      <td>http://cdn.image.buzzni.com/2022/06/21/w08NEO7...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-06-30</td>
      <td>가전디지털</td>
      <td>nsmallplus</td>
      <td>0시 03분 ~0시 43분</td>
      <td>[삼성카드5%할인]LG 코드제로 A9S(AS9271IB/SB/VB)</td>
      <td>699,000원</td>
      <td>http://cdn.image.buzzni.com/2022/05/12/cD4oCoZ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-06-30</td>
      <td>식품건강</td>
      <td>wshop</td>
      <td>0시 25분 ~1시 26분</td>
      <td>[50％세일] 밸런스 프로틴 하이셀 슬림S 7통 + 전용스푼 2개 + 텀블러</td>
      <td>98,000원</td>
      <td>http://cdn.image.buzzni.com/2022/01/21/a8gbwvJ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-06-30</td>
      <td>패션잡화</td>
      <td>hmall</td>
      <td>0시 35분 ~1시 15분</td>
      <td>핀에스커 22 SUMMER 첼시 레인부츠</td>
      <td>59,000원</td>
      <td>http://cdn.image.buzzni.com/2022/06/24/aR6Gen0...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-06-30</td>
      <td>식품건강</td>
      <td>bshop</td>
      <td>0시 36분 ~1시 36분</td>
      <td>[닥터린] (방송에서만) 초임계 알티지 오메가3 18개월</td>
      <td>177,300원</td>
      <td>http://cdn.image.buzzni.com/2021/08/13/V3abKC8...</td>
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
      <th>464</th>
      <td>2022-06-30</td>
      <td>화장품미용</td>
      <td>hmall</td>
      <td>23시 50분 ~1시 00분</td>
      <td>[최다 선세럼 5개 구성] 최신상 달바 선세럼</td>
      <td>79,900원</td>
      <td>http://cdn.image.buzzni.com/2022/05/18/6uzt0Ke...</td>
    </tr>
    <tr>
      <th>465</th>
      <td>2022-06-30</td>
      <td>패션잡화</td>
      <td>hnsmall</td>
      <td>23시 50분 ~2시 00분</td>
      <td>[후라밍고] 22SS 올데이 썸머 큐롯 팬츠 2종</td>
      <td>44,100원</td>
      <td>http://cdn.image.buzzni.com/2022/06/30/Rklc7O4...</td>
    </tr>
    <tr>
      <th>466</th>
      <td>2022-06-30</td>
      <td>화장품미용</td>
      <td>lottemall</td>
      <td>23시 50분 ~1시 00분</td>
      <td>[한정판][NEW 마데카 앰플 시즌3] 센텔리안24 마데카 멜라 캡처 앰플 프로 히...</td>
      <td>188,000원</td>
      <td>http://cdn.image.buzzni.com/2022/06/27/6vGUkSb...</td>
    </tr>
    <tr>
      <th>467</th>
      <td>2022-06-30</td>
      <td>가전디지털</td>
      <td>gsshop</td>
      <td>23시 55분 ~1시 00분</td>
      <td>LG 디오스 오브제 컬렉션 더블매직스페이스 글라스</td>
      <td>3,890,000원</td>
      <td>http://cdn.image.buzzni.com/2022/06/13/i7wkcys...</td>
    </tr>
    <tr>
      <th>468</th>
      <td>2022-06-30</td>
      <td>보험</td>
      <td>nsmall</td>
      <td>23시 55분 ~0시 15분</td>
      <td>라이나생명(무)골라담는간편건강보험</td>
      <td>상담/렌탈</td>
      <td>http://cdn.m.ui.hsmoa.com/media/insu1/nsmall.png</td>
    </tr>
  </tbody>
</table>
<p>469 rows × 7 columns</p>
</div>

