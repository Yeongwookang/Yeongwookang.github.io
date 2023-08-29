---
title : Selenium을 활용한 WebScrapper
date : 2023-08-02 00:00:00 +09:00
categories : [Project, Python]
tags : [selenium, python, flask, project] 
---

## 개발 환경

- VSCode로 개발을 진행
- 채용사이트 Indeed를 Scrapping 하는것이 목적
- 크롬을 최신업데이트 하고 pypi에 있는 webdriver-manager 모듈을 활용한 자동 최신화
- Selenium을 사용함 ([공식문서](https://www.selenium.dev/documentation/)를 참조)

<br>

## 결과
Link: [https://github.com/Yeongwookang/MyNotes/tree/main/Projects/Python-Flask/WebScrapper](https://github.com/Yeongwookang/MyNotes/tree/main/Projects/Python-Flask/WebScrapper)
Selenium을 활용하여 키워드와 한번에 검색할 양을 정하면 처음 5개 페이지에 대해 스크랩해서 Excel 파일로 만들어주는 Scrapper를 만들었다.

<br>

## 개발 과정

### 데이터 가공
- find_element 메소드는 WebElement를 반환
- find_elements는 WebElement로 된 List를 반환

위 두가지 요소를 통해 거의 해결이 가능하다.
>하지만, 동적으로 받아오는 경우와 숨겨진 경우는 해결하지 못함
{: .prompt-warning}

#### selenium.common.exceptions.NoSuchElementException
- 한번에 검색하는 갯수가 적게 설정하면 문제가 없었음
- 갯수를 늘리니 NoSuchElementException이 나왔음
- indeed사이트는 mosaic-afterFifthJobResult 처럼 5단위로 공백을 보내고 있었음

>if (card.get_attribute('innerText').replace(" ","") != "") 을 통해 공백을 잘라냈을때 공백이 아니라면이라는 조건을 걸어 해결되었다.
{: .prompt-tip}

#### ,와 \n이 포함되어 csv파일이 일정한 형태를 가지지 않음
모든 문자열에 대해 ,를 대체하였는데도 postDate에서 글자 중간에 \n이 있었다. replace method로 해결하였다.

### csv파일로 만들기
csv 파일로 만들어주는 코드이다. 파이썬 기본 라이브러리의 파일 함수를 사용하였다.
utf-8로만 인코딩하면 엑셀등에서 켰을때 글자가 깨져서 utf-8 sig로 인코딩 하였다.
데이터 가공하기에는 별로라서 다음에는 pandas를 써야겠다.

### Flask

- keyword와 limit을 쿼리로 하는 페이지를 구현
- export 할때 다시 검색하여 하지 않도록 db={} 를 활용하여 임시저장
- 파일로 만들어 다운로드 가능하게 함

Flask는 부트캠프에서는 간단하게만 해보았는데, 작은 규모의 프로젝트에는 꽤나 효과적인것 같다.

<br>
## 고찰

#### 의문점들
- pagination을 끝까지 조회하는 방법은 없나?
- Indeed가 아닌 다른 사이트에서는 검색 쿼리로 limit를 제공하지 않을수도 있지 않나?
- 대량 검색을 할때 wait를 걸어줘도 조회되지 않는 데이터가 존재함