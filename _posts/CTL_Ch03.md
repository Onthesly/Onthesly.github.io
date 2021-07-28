# 네이버 영화 리뷰 크롤링 프로젝트

## BeautifulSoup 라이브러리 사용법


```python
from bs4 import BeautifulSoup

html_doc = '''
<html><head><title>The Dormouse's story</title></head>
<body>
<p class='title'><b>The Dormouse's story</b></p>
<p class='story'>Once upon a time there were three little sisters;
                and their names were
<a href='http://example.com/elsie' class='sister' id='link1'>Elsie</a>,
<a href='http://example.com/lacie' class='sister' id='link2'>Lacie</a> and
<a href='http://example.com/tillie' class='sister' id='link3'>Tillie</a>;
and they lived at the bottom of a well.</p>

<p class='story'>...</p>
'''

# Beautifulsoup 객체를 생성한다.
soup = BeautifulSoup(html_doc, 'lxml')

# 객체, 태그 이름
# .태그 이름으로 하위 태그로의 접근이 가능하다.
print('soup.body.p의 결과 : ', soup.body.p)
```

    soup.body.p의 결과 :  <p class="title"><b>The Dormouse's story</b></p>
    


```python
# 객체.태그['속성 이름']
# 객체의 태그 속성은 파이썬 딕셔너리처럼 태그['속성 이름']으로 접근이 가능하다.
print("soup.a['href']의 결과 : ", soup.a['href'])
```

    soup.a['href']의 결과 :  http://example.com/elsie
    


```python
# 객체.name
## name 변수
print('soup.title.name의 결과 : ', soup.title.name)
```

    soup.title.name의 결과 :  title
    


```python
# 객체.string
## string 변수 (참고) NavigableString: 문자열은 태그 안의 텍스트에 상응한다.
## BeautifulSoup은 이런 텍스트를 포함하는 NavigableString 클래스를 사용한다.
print('soup.title.string의 결과 : ', soup.title.string)
```

    soup.title.string의 결과 :  The Dormouse's story
    


```python
# 객체.contents
## 태그의 자식들을 리스트로 반환
print('soup.contents의 결과 : ', soup.contents)
```

    soup.contents의 결과 :  [<html><head><title>The Dormouse's story</title></head>
    <body>
    <p class="title"><b>The Dormouse's story</b></p>
    <p class="story">Once upon a time there were three little sisters;
                    and their names were
    <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
    <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
    <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
    and they lived at the bottom of a well.</p>
    <p class="story">...</p>
    </body></html>]
    


```python
# find() : 태그 하나만 가져옴
'''
find(name, attrs, recursive, string, **kwargs)

[옵션]
name - 태그 이름
attrs - 속성(딕셔너리로)
recursive - 모든 자식 or 자식
string - 태그 안에 텍스트
keyword - 속성(키워드로)

* (주의) class는 파이썬 예약어이므로, class_를 사용한다.
'''
print('soup.find()의 결과 : ', soup.find('a', attrs={'class' : 'sister'}))
```

    soup.find()의 결과 :  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
    


```python
# find_all() : 해당 태그가 여러 개 있을 경우 한꺼번에 모두 가져온다. 그 객체들의 리스트로 반환한다.
'''
find_all(name, attrs, recursive, string, limit, **kwargs)

[옵션]
Limit -몇 개까지 찾을 것인가? find_all()로 검색했을 때, 수천, 수만 개가 된다면 시간이 오래 걸릴 것이다.
이때 몇 개 까지만 찾을 수 있도록 제한을 둘 수 있는 인자다.

'''
print('soup.find_all()의 결과 : ', soup.find_all('a', limit=2))
```

    soup.find_all()의 결과 :  [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
    

## 영화 '블랙 위도우'의 리뷰 제목 크롤링


```python
import requests
from bs4 import BeautifulSoup

# '블랙 위도우'의 네이버 영화 리뷰 링크
url = 'https://movie.naver.com/movie/bi/mi/review.nhn?code=184318'
# html 소스 가져오기
res = requests.get(url)
# html 파싱
soup = BeautifulSoup(res.text, 'lxml')

# 리뷰 리스트
ul = soup.find('ul', class_='rvw_list_area')
lis = ul.find_all('li')

# 리뷰 제목 출력
count = 0
for li in lis:
    count += 1
    print(f'[{count}th] ', li.a.string)
```

    [1th]  스칼렛 요한슨 재계약 기원
    [2th]  스칼렛 요한슨은 세상에서 제일 예쁘다.
    [3th]  멍멍이 소리 한번 해보고싶었습니다
    [4th]  [영화감상] 블랙 위도우 (Black Widow, 2020)
    [5th]  [ "It's okey" 의미가 온전히 해석되다. 블랙위도우 리뷰 ]
    [6th]  호우
    [7th]  I adore you. 나타샤 사랑해 행복하자
    [8th]  제발 극장가가 반등했으면 - 블랙위도우
    [9th]  [영화] 블랙 위도우 / 마블 그냥 사랑해요 ㅠㅠ / 쿠키 O 스포 X
    [10th]  블랙 위도우(2021)
    

## 크롤링 결과를 파일에 저장


```python
# 결과를 파일에 저장
with open('reviews.txt', 'w') as fp:
    count = 0
    for li in lis:
        count += 1
        fp.write(f'[{count}th] '+str(li.a.string)+'\n')
```

## 리뷰가 적힌 날짜 크롤링


```python
import requests
from bs4 import BeautifulSoup

# '블랙 위도우'의 네이버 영화 리뷰 링크
url = 'https://movie.naver.com/movie/bi/mi/review.nhn?code=184318'
# html 소스 가져오기
res = requests.get(url)
# html 파싱
soup = BeautifulSoup(res.text, 'lxml')

# 리뷰 리스트
ul = soup.find('ul', class_='rvw_list_area')
lis = ul.find_all('li')

# 리뷰 날짜 출력
count = 0
for li in lis:
    count += 1
    print(f'[{count}th] ', li.span.em.string)
```

    [1th]  2019.05.18
    [2th]  2019.05.14
    [3th]  2019.07.06
    [4th]  2021.07.08
    [5th]  2021.07.09
    [6th]  2019.05.12
    [7th]  2019.07.29
    [8th]  2021.07.04
    [9th]  2021.07.12
    [10th]  2021.07.10
    

## 리뷰 내용 크롤링


```python
# 리뷰에 적힌 내용 크롤링
import requests
from bs4 import BeautifulSoup

# '블랙 위도우'의 네이버 영화 리뷰 링크
url = 'https://movie.naver.com/movie/bi/mi/review.nhn?code=184318'
# html 소스 가져오기
res = requests.get(url)
# html 파싱
soup = BeautifulSoup(res.text, 'lxml')

# 리뷰 리스트
ul = soup.find('ul', class_='rvw_list_area')
lis = ul.find_all('li')

# 리뷰 날짜 출력
count = 0
for li in lis:
    count += 1
    print(f'[{count}th] ', li.p.string)
```

## 원하는 페이지까지의 리뷰 제목 크롤링 


```python
import requests
from bs4 import BeautifulSoup

# '블랙 위도우'의 네이버 영화 리뷰 링크
url = 'https://movie.naver.com/movie/bi/mi/review.nhn?code=184318'
# 페이지별 리스트를 만들어보자
page = 2
urls = ['https://movie.naver.com/movie/bi/mi/review.nhn?code=184318&page='+str(i+1) for i in range(page)]

# html 소스 가져오기
resources = [requests.get(i) for i in urls]

# html 파싱
soups = [BeautifulSoup(i.text, 'lxml') for i in resources]

# 리뷰 리스트
uls = [i.find('ul', class_='rvw_list_area') for i in soups]
lis = [i.find_all('li') for i in uls]

# 리뷰 제목 출력
count = 0
for lis_in in lis:
    for i in lis_in:
        count += 1
        print(f'[{count}th] ', i.a.string)
```

    [1th]  스칼렛 요한슨 재계약 기원
    [2th]  스칼렛 요한슨은 세상에서 제일 예쁘다.
    [3th]  멍멍이 소리 한번 해보고싶었습니다
    [4th]  [영화감상] 블랙 위도우 (Black Widow, 2020)
    [5th]  [ "It's okey" 의미가 온전히 해석되다. 블랙위도우 리뷰 ]
    [6th]  호우
    [7th]  I adore you. 나타샤 사랑해 행복하자
    [8th]  제발 극장가가 반등했으면 - 블랙위도우
    [9th]  [영화] 블랙 위도우 / 마블 그냥 사랑해요 ㅠㅠ / 쿠키 O 스포 X
    [10th]  블랙 위도우(2021)
    [11th]  2021년 7월 13일 [블랙 위도우]
    [12th]  [MMR MORNING의 영화 리뷰] 블랙 위도우
    [13th]  블릭위도우 솔직리뷰 - 마블시리즈 꼭 보고 봐야하는가?
    [14th]  신장투석시 고통은 어느정도인가요?
    [15th]  블랙 위도우
    [16th]  Black Widow
    [17th]  210714 전주 신시가지 영화관 엠스퀘어 CGV 서전주 블랙 위도우 Black Widow
    [18th]  블랙 위도우 (Black Widow, 2021) - ‘히어로, 딸, 언니, 친구였던 나타샤의 삶’ [영화 후기,리뷰/신작,개봉작,상영작, 마블 영화 추천/줄거
    [19th]  개봉예정_ 블랙 위도우_액션, 모험, SF _스칼릿 조핸슨,문충추천작
    [20th]  개봉예정영화_ 블랙 위도우_액션, 모험, SF _어벤져스히어로_문충추천작
    
