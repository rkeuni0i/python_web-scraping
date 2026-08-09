# 06. Web Scraping

`requests`와 `BeautifulSoup`(bs4)을 이용해 웹 페이지를 가져오고 원하는 정보를 추출하는 웹 스크래핑을 학습한 실습 코드입니다.

## 학습 내용

- `requests.get()`으로 웹 페이지 가져오기, `res.raise_for_status()`로 오류 확인
- 응답 HTML을 파일로 저장하기
- `BeautifulSoup`로 HTML 파싱: `soup.title`, `soup.find()`, `soup.find_all()`, `soup.select()`, `soup.select_one()`
- 실제 페이지(네이버 웹툰, 네이버 금융 시세, 환율표)에서 원하는 데이터 추출
- `pd.read_html()`로 HTML 표(table)를 바로 `DataFrame`으로 변환하기
- User-Agent 헤더를 지정해 차단을 피하는 방법

## 실습 파일

| 파일 | 학습 내용 |
|---|---|
| `01_naver_comic_and_stock_scraping.ipynb` | bs4 기본 탐색 + 네이버 웹툰/주식 시세 스크래핑 |
| `02_bs4_basics_naver_comic.ipynb` | bs4의 `find`/`select` 기본 메서드 탐색 |
| `03_naver_stock_price_scraping.ipynb` | 네이버 금융 일별 시세 표 스크래핑, `pd.read_html` |
| `04_bs4_selectors_and_stock_scraping.ipynb` | bs4 셀렉터(`select`, `select_one`) 심화 + 시세 스크래핑 |
| `05_exchange_rate_table_scraping.ipynb` | 네이버 금융 환율 표를 `pd.read_html`로 가져오기 |

이 폴더의 노트북들은 **재실행하지 않고 원본 노트북에 저장되어 있던 실행 결과를 그대로 유지**했습니다. 웹 스크래핑은 실행 시점의 페이지 구조와 실시간 데이터(주가, 환율 등)에 따라 결과가 달라지기 때문에, 다시 실행하면 원본과 다른 결과나 다른 오류가 나올 수 있습니다.

## Troubleshooting

### 1. `FeatureNotFound: Couldn't find a tree builder with the features you requested: html5lib` (실제 발생한 오류)

**문제**

`04_bs4_selectors_and_stock_scraping.ipynb`에서 `pd.read_html()`이 내부적으로 `html5lib` 파서를 사용하려다 실패했습니다.

**원인**

`html5lib` 파서를 사용하려면 `html5lib` 패키지가 별도로 설치되어 있어야 하는데, 실습 당시 환경에는 설치되어 있지 않았습니다.

**해결**

`pip install html5lib`로 패키지를 설치하거나, `pd.read_html(..., flavor="lxml")`처럼 이미 설치된 다른 파서를 명시적으로 지정하면 해결됩니다.

### 2. `TypeError: 'NoneType' object is not callable` (실제 발생한 오류)

**문제**

`01_naver_comic_and_stock_scraping.ipynb`에서 `parsing_list[i].find_All("td", calss_="num")`처럼 `find_all`을 `find_All`로(대문자 `A`) 잘못 입력해 오류가 발생했습니다. 참고로 `class_`도 `calss_`로 오타가 나 있습니다.

**원인**

BeautifulSoup의 `Tag` 객체는 실제로 존재하지 않는 속성/메서드 이름에 접근하면 무조건 `AttributeError`를 내는 대신, 같은 이름의 하위 태그를 찾아보고 없으면 `None`을 반환하는 경우가 있습니다. `find_All`이라는 메서드는 없고 같은 이름의 태그도 없으므로 `None`이 반환되었고, 그 `None`을 함수처럼 `(...)`로 호출하려다 "`NoneType` object is not callable" 오류가 났습니다.

**해결**

`find_all`(소문자 `a`, `ll`)과 `class_`로 정확히 입력하면 해결됩니다. 대소문자를 정확히 지키는 것이 중요하다는 점을 보여주는 사례입니다.

### 3. `TypeError: 'NoneType' object is not subscriptable`

**문제**

`01_naver_comic_and_stock_scraping.ipynb`의 네이버 웹툰 페이지 접근 코드에서도 오류가 함께 기록되어 있습니다. 같은 셀의 `print()` 결과(`<title>네이버 웹툰</title>`, `None`, `<body><div id="root"></div></body>`)는 정상적으로 출력된 것으로 보아, 이 오류는 화면에 보이는 코드가 수정되기 전의 이전 실행에서 남은 결과일 가능성이 있습니다(NumPy 폴더의 사례처럼, Jupyter는 셀 코드를 수정해도 이전 출력이 자동으로 지워지지 않습니다).

**원인 / 배운 점**

네이버 웹툰 페이지처럼 자바스크립트로 화면을 그리는(SPA) 사이트는 `requests`로 가져온 원본 HTML에 `<div id="root"></div>`처럼 빈 뼈대만 들어있고 실제 콘텐츠는 없는 경우가 많습니다. 이런 페이지에서는 원하는 태그를 찾지 못해 `None`이 반환되고, 그 `None`을 다시 인덱싱하거나 호출하려고 하면 오류로 이어지기 쉽습니다.

**해결**

정적 HTML 스크래핑으로는 자바스크립트 렌더링 결과를 가져올 수 없으므로, 이런 사이트는 API를 직접 호출하거나(가능한 경우) Selenium 같은 브라우저 자동화 도구가 필요합니다. 또한 셀을 수정한 뒤에는 다시 실행해 출력 결과도 함께 갱신하는 습관이 오류 원인을 헷갈리지 않는 데 도움이 됩니다.
