---
title : Selenium을 활용한 WebScrapper
date : 2023-05-24 00:00:00 +09:00
categories : [Project, Python]
tags : [selenium, python, flask, project] 
---

## Selenium을 활용하여 웹스크래퍼 만들기
Selenium 4.11.2 버전을 import 하여 채용사이트인 indeed를 scrapping 해보았다.

#### 개발 환경 세팅

처음에는 replit으로 개발을 진행했는데, replit은 파이썬 3.10을 사용하고 패키지 설치 및 설정이 힘들어서, VSCode로 개발을 진행했다. 
Chrome driver는 매번 맞춰서 파일을 넣기가 어려울것 같아서 크롬을 최신업데이트 하고 pypi에 있는 webdriver-manager 모듈을 활용하여 자동 최신화하게 했다.

#### Selenium
replit으로 할때는 beautifulsoup를 이용해서 하다가 Selenium의 메소드들을 새로 알아야 했다.
https://www.selenium.dev/documentation/ 공식문서를 참조하면서 해결했다.
    
## 결과
Selenium을 활용하여 키워드와 한번에 검색할 양을 정하면 처음 5개 페이지에 대해 스크랩해서 Excel 파일로 만들어주는 Scrapper를 만들었다.


### 데이터 가공
한줄 평: Selenium은 공식문서가 생각보다 친절하지 않았다.. 
find_element 메소드는 WebElement를 반환하고, find_elements는 WebElement로 된 List를 반환한다는 점만 생각하면 다 할 수 있긴하다.
하지만, 요소를 찾는 방식이 문제가 됐는데, 동적으로 받아오는 경우와 숨겨진 경우가 있으면 좀 곤란하다.
indeed의 경우 그렇지 않았지만, 결과 화면에 데이터가 비는 부분이 있다.



#### selenium.common.exceptions.NoSuchElementException 
한번에 검색하는 갯수가 적게 설정하면 문제가 없었지만, 갯수를 늘리니 NoSuchElementException이 나왔다.
처음에는 코드가 잘못된줄 알고 for문에 index도 해본 결과, indeed사이트는 mosaic-afterFifthJobResult 처럼 5단위로 공백을 보내고 있었다.


#### 해결
if card.get_attribute('innerText').replace(" ","") != "": 을 통해 공백을 잘라냈을때 공백이 아니라면이라는 조건을 걸어 해결되었다.


#### ,와 \n이 포함되어 csv파일이 일정한 형태를 가지지 않음
모든 문자열에 대해 ,를 대체하였는데도 postDate에서 글자 중간에 \n이 있었다.
그 부분도 수정해주니 결과가 나왔다.

#### 의문점
한번에 검색할 양을 늘리면 전체 내용을 긁어오는데 문제는 없지만, pagination은 한번에 5개씩만 보여주기 때문에 전체 내용을 다 긁어오고 싶을 때나 indeed가 아닌 다른 사이트에서는 검색 쿼리로 limit를 제공하지 않을수도 있는데, 이를 어떻게 해결해야할지 생각을 해봐야겠다.

#### 의문점2
대량 검색을 할때 로딩이 잘 되지 않는건가 하고 wait를 걸어줘도 몇몇개는 여전히 비고있다. 
이에대한 이유를 찾아야 할 것이다.



### csv파일로 만들기
csv 파일로 만들어주는 코드이다. 파이썬 기본 라이브러리의 파일 함수를 사용하였다.
utf-8로만 인코딩하면 엑셀등에서 켰을때 글자가 깨져서 utf-8 sig로 인코딩 하였다.



### Flask 

keyword와 limit을 쿼리로 하는 페이지를 구현하였다. export 할때 다시 검색하여 하지 않도록 db={} 를 활용하여 임시저장한 뒤
파일로 만들어 다운로드 가능하도록 하였다.
Flask는 부트캠프에서는 간단하게만 해보았는데, 파일처리 등이 스프링보다는 편한 것 같다.
