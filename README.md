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

### 쓰기 기능 구현(1)

```jsx
@app.route('/create/')
def create():
    content = '''
        <form action="/create/" method="POST">
            <p><input type="text" name="title" placeholder="title"></p>
            <p><textarea name="body" placeholder="body"></textarea></p>
            <p><input type="submit" value="create"></p>
        </form>
    '''
    return template(getContents(), content)
```

→ input type=”text” 는 입력받는 칸

→ textarea는 내용 입력받는 칸

→ input type=”submit” 제출 버튼

→ method=”GET” 기본이 get방식임, 주소 창에 모든 정보가 입력됨

→ POST방식은 데이터를 입력했을 때 숨기기 가능(보안을 위해서)

### 쓰기 기능 구현(2)

```python
# route에 특별한 조치를 취하지 않으면 이건 get방식으로 보내는거임
# 우리는 post방식을 사용하는거라 조치를 취해야
@app.route('/create/', methods=['GET', 'POST'])
def create():
		# request.method를 print하여 데이터가 어떤 방식으로 전달되는지 확인
    print(request.method)
    # GET방식이라면? 아래의 콘텐츠를 표시
    if(request.method == 'GET'):
        content = '''
            <form action="/create/" method="POST">
                <p><input type="text" name="title" placeholder="title"></p>
                <p><textarea name="body" placeholder="body"></textarea></p>
                <p><input type="submit" value="create"></p>
            </form>
        '''
        return template(getContents(), content)
    # POST방식이라면?
    elif request.method == 'POST':
		    # 함수안에서는 nextId가 전역변수로 선언되었다는 것을 선언해야함
        global nextId
        # request.form[]을 사용해서 title과 body데이터 저장
        title = request.form['title']
        body = request.form['body']
        # topics 딕셔너리에 추가할 데이터 형식 생성
        newTopic = {'id': nextId, 'title': title, 'body': body}
        topics.append(newTopic)
        url = '/read/'+str(nextId)+'/'
        nextId += 1
        # redirect(url)을 사용해서 새로운 데이터, 새로운 url 생성
        return redirect(url)
```

## Day 3

### 수정 기능 구현

```python
# update기능 역시 'GET', 'POST' 둘다 사용함
@app.route('/update/<int:id>/', methods=['GET', 'POST'])
# id를 파라미터로 가져와야 함
def update(id):
    print(request.method)
    if request.method == 'GET':
        title = ''
        body = ''
        for topic in topics:
            if id == topic['id']:
                title = topic['title']
                body = topic['body']
                break
        content = f'''
            <form action="/update/{id}/" method="POST">
                <p><input type="text" name="title" placeholder="title", value="{title}"></p>
                <p><textarea name="body" placeholder="body">{body}</textarea></p>
                <p><input type="submit" value="update"></p>
            </form>
        '''
        return template(getContents(), content)
    elif request.method == 'POST':
        global nextId
        title = request.form['title']
        body = request.form['body']
        for topic in topics:
            if id == topic['id']:
                topic['title'] = title
                topic['body'] = body
                break
        url = '/read/'+str(id)+'/'
        
        return redirect(url)
```

### 삭제 기능 구현

→ 삭제에서는 a태그를 사용하지 않음, POST방식으로 해야함

```python
def template(contents, content, id=None):
    contextUI = ''
    if id != None:
        contextUI = f'''
            <li><a href="/update/{id}/">update</a></li>
            <li><form action="/delete/{id}/" method="POST"><input type="submit" value="delete"></li></form>
        '''
        # 위에 있는 두번째꺼가 추가된 delete 버튼 메서드는 포스트
    return f'''<!doctype html>
    <html>
        <body>
            <h1><a href="/">WEB</a></h1>
            <ol>
                {contents}
            </ol>
            {content}
            <ul>
                <li><a href="/create/">create</a></li>
                {contextUI}
            </ul>
        </body>
    </html>
    '''

@app.route('/delete/<int:id>/', methods=['POST'])
def delete(id):
    for topic in topics:
        if id == topic['id']:
            topics.remove(topic)
            break
    # 상세보기 페이지가 없으니 루트로 가면 됨
    return redirect('/')

```

### 전체코드

```python
from flask import Flask, request, redirect
import random

app = Flask(__name__)
nextId = 4
topics = [
    {'id':1, 'title': 'html', 'body': 'html is ...'},
    {'id':2, 'title': 'css', 'body': 'css is ...'},
    {'id':3, 'title': 'javascript', 'body': 'javascript is ...'}
]

def template(contents, content, id=None):
    contextUI = ''
    if id != None:
        contextUI = f'''
            <li><a href="/update/{id}/">update</a></li>
            <li><form action="/delete/{id}/" method="POST"><input type="submit" value="delete"></li></form>
        '''
    return f'''<!doctype html>
    <html>
        <body>
            <h1><a href="/">WEB</a></h1>
            <ol>
                {contents}
            </ol>
            {content}
            <ul>
                <li><a href="/create/">create</a></li>
                {contextUI}
            </ul>
        </body>
    </html>
    '''
    
def getContents():
    liTags = ''
    for topic in topics:
        liTags = liTags + f'<li><a href="/read/{topic["id"]}/">{topic["title"]}</a></li>'
    return liTags

@app.route('/')
def index():
    return template(getContents(), '<h2>Welcome</h2>Hello, WEB')
    
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
    return template(getContents(), f'<h2>{title}</h2>{body}', id)

# route에 특별한 조치를 취하지 않으면 이건 get방식으로 보내는거임
# 우리는 post방식을 사용하는거라 조치를 취해야
@app.route('/create/', methods=['GET', 'POST'])
def create():
    print(request.method)
    if(request.method == 'GET'):
        content = '''
            <form action="/create/" method="POST">
                <p><input type="text" name="title" placeholder="title"></p>
                <p><textarea name="body" placeholder="body"></textarea></p>
                <p><input type="submit" value="create"></p>
            </form>
        '''
        return template(getContents(), content)
    elif request.method == 'POST':
        global nextId
        title = request.form['title']
        body = request.form['body']
        newTopic = {'id': nextId, 'title': title, 'body': body}
        topics.append(newTopic)
        url = '/read/'+str(nextId)+'/'
        nextId += 1
        return redirect(url)
    
@app.route('/update/<int:id>/', methods=['GET', 'POST'])
def update(id):
    print(request.method)
    if request.method == 'GET':
        title = ''
        body = ''
        for topic in topics:
            if id == topic['id']:
                title = topic['title']
                body = topic['body']
                break
        content = f'''
            <form action="/update/{id}/" method="POST">
                <p><input type="text" name="title" placeholder="title", value="{title}"></p>
                <p><textarea name="body" placeholder="body">{body}</textarea></p>
                <p><input type="submit" value="update"></p>
            </form>
        '''
        return template(getContents(), content)
    elif request.method == 'POST':
        global nextId
        title = request.form['title']
        body = request.form['body']
        for topic in topics:
            if id == topic['id']:
                topic['title'] = title
                topic['body'] = body
                break
        url = '/read/'+str(id)+'/'
        
        return redirect(url)

@app.route('/delete/<int:id>/', methods=['POST'])
def delete(id):
    for topic in topics:
        if id == topic['id']:
            topics.remove(topic)
            break
    # 상세보기 페이지가 없으니 루트로 가면 됨
    return redirect('/')

# debugger active
app.run(port=5001, debug=True)
```
