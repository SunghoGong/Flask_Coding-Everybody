# Flask Web Framework(<a href="https://opentutorials.org/course/4904">생활코딩</a>)

## Day 1
### 개발환경 세팅

1. 터미널에서 python3 [파일명].py + 엔터

```python
from flask import Flask
import random

app = Flask(__name__)

# 라우터
@app.route('/')
def index():
    return 'random :  <strong>' + str(random.random()) + '</strong>'

# 기본 port는 5000인데, 이미 사용중일때는 5001로
# debugger active, debug=True
# 실제 서비스에는 False로 해놔야 함
app.run(port=5001, debug=True)
```

### Routing(라우팅)

```python
@app.route('/example/<id>/')
def example(id):
	return f'example {id}'
```

→ @app.route를 이용해서 경로를 지정. 그 아래의 함수가 요청을 처리할 함수로 지정됨

### 홈페이지 구현

```python
from flask import Flask
import random

app = Flask(__name__)

topics = [
    {'id':1, 'title': 'html', 'body': 'html is ...'},
    {'id':2, 'title': 'css', 'body': 'css is ...'},
    {'id':3, 'title': 'javascript', 'body': 'javascript is ...'}
]

@app.route('/')
def index():
    liTags = ''
    for topic in topics:
        liTags = liTags + f'<li><a href="/read/{topic["id"]}/">{topic["title"]}</a></li>'
    return f'''<!doctype html>
    <html>
        <body>
            <h1><a href="/">WEB</a></h1>
            <ol>
                {liTags}
            </ol>
            <h2>Welcome</h2>
            Hello, Web
        </body>
    </html>
    '''
    
@app.route('/read/<id>/')
def read(id):
    return f'read {id}'

# debugger active
app.run(port=5001, debug=True)
```

→ 홈페이지의 표시될 정보를 topics라는 파이썬 리스트에 저장한 후 반복문을 사용해 코드를 간결하게 바꿈

→ 리스트, 딕셔너리 자료형이 많이 쓰일듯

### 읽기 기능 구현

```python
from flask import Flask
import random

app = Flask(__name__)

topics = [
    {'id':1, 'title': 'html', 'body': 'html is ...'},
    {'id':2, 'title': 'css', 'body': 'css is ...'},
    {'id':3, 'title': 'javascript', 'body': 'javascript is ...'}
]

# 템플릿을 함수화함
def template(contents, content):
    return f'''<!doctype html>
    <html>
        <body>
            <h1><a href="/">WEB</a></h1>
            <ol>
                {contents}
            </ol>
            {content}
        </body>
    </html>
    '''
    
# liTags를 함수화 함
def getContents():
    liTags = ''
    for topic in topics:
        liTags = liTags + f'<li><a href="/read/{topic["id"]}/">{topic["title"]}</a></li>'
    return liTags

@app.route('/')
def index():
    return template(getContents(), '<h2>Welcome</h2>Hello, WEB')
    
# 기본적으로 str형태로 받아오는데
# <int:id>를 사용해서 정수형으로 받아오도록 함
@app.route('/read/<int:id>/')
def read(id):
    print(type(id))
    title = ''
    body = ''
    for topic in topics:
        if id == topic['id']:
            title = topic['title']
            body = topic['body']
            break
    print(title, body)
    return template(getContents(), f'<h2>{title}</h2>{body}')

# debugger active
app.run(port=5001, debug=True)
```

## Day 2
