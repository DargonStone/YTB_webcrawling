# YTB_webcrawling
김용석
유투브 자동화 크롤링 
import time
import pandas as pd
import bs4
from selenium import webdriver
from selenium.webdriver import Keys
from selenium.webdriver.common.by import By

### 1818147 김용석

### 원하는 주제의 유투브 동영상을 원하는 갯수만큼 데이터 크롤링

## 우튜브는 동적 사이트임으로 selenium을 이용, data parsing의 경우는 beautifulsoup4 이용

## pandas를 이용하여 df프레임 변환 및 csv 파일 저장

## 정보를 입력받을 변수
file_name = input("저장할 파일 이름을 정해주세요")
ytb_title_name = input("원하는 영상의 주제를 입력해주세요")
ytb_video_count = int(input("몇개의 영상을 참고 하시겠습니까?"))

# 작업 상황을 알려줄 변수
count = 1
# 유투브 영상 주소가 들어갈 리스트
url_list = []
# 댓글 정보가 들어갈 리스트
ytb_comment_list = []
########################################################
## chome 실행
driver = webdriver.Chrome()

## 입력 받은 주제를 검색
driver.get('https://www.youtube.com/results?search_query=' + ytb_title_name)

##로딩이 완료 될떄까지 대기
driver.implicitly_wait(10)
time.sleep(5)

##ytb_video_count 에서 입력받은 만큼 end키를 이용하여 페이지 밑으로 스크롤
for i in range(ytb_video_count):
    driver.find_element(By.TAG_NAME, 'body').send_keys(Keys.END)
    time.sleep(2)

## 각 영상의 url주소 속성값 가져오기
ytb_url = driver.find_elements(By.XPATH, '//*[@id="video-title"]')

## 가져온 url주소를 url_list변수에 리스트 형식으로 append
for ytb_url_list in ytb_url:
    url_data = ytb_url_list.get_attribute('href')
    print(url_data)

    url_list.append(url_data)


## href가 없는 None값도 가져와지기 떄문에 list혁식에 있는 모든 None값을 삭제
while None in url_list:
    url_list.remove(None)

print("None 데이터 삭제")

print(url_list)


## ytb_video_count에 입력받은 값 만큼 리스트에 있는 특정 값을 출력하여 접속허고 반복

for i in range(ytb_video_count):

    ## 리스트의 특정 순서에 있는 url주소를 출력
    driver.get(url_list[i])

    time.sleep(5)

    ## 모든 댓글을 가져 오기위해서 스크롤을 끝까지
    last_page_height = driver.execute_script("return document.documentElement.scrollHeight")
    while True:
        driver.execute_script("window.scrollTo(0, document.documentElement.scrollHeight);")
        time.sleep(3.0)
        new_page_height = driver.execute_script("return document.documentElement.scrollHeight")

        if new_page_height == last_page_height:
            break
        last_page_height = new_page_height

    time.sleep(5)

    # bs4 실행
    html = driver.page_source
    # lxml형식으로 parsing
    soup = bs4.BeautifulSoup(html, 'lxml')
    # 모든댓글에 관한 element 출력
    ytb_comment = soup.find_all('yt-formatted-string', {'id': 'content-text'})

    ##찾은 모든 댓글 element를 ytb_comment_list 변수에 list형식으로 저장
    for text_save in ytb_comment:
        data = text_save.text
        print(data)
        ytb_comment_list.append(data)
        ytn_comment_data = {'comment': ytb_comment_list}

## 만약 작업이 완료 된다면 pandas를 이용하여 df을 만들고 file_name으로 입력받은 변수명으로 csv파일 저장
if i == ytb_video_count-1:
    df = pd.DataFrame(ytn_comment_data)
    df.to_csv("%s.csv" % file_name, encoding="utf-8-sig")
    print("%d 개의 작업이 안료 되었습니다." % len(ytb_comment_list))
    
## 아직 작업이 완료가 되지 않았다면 현재 작업 진행도 출력
else:
    print("%d번쨰 작업이 완료 되었습니다." % count)
    count += 1

