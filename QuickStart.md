
# 小应用
```python
# 导入Flask类,这个类的实例将作为WSGI应用
from flask import Flask

# 第一个参数为应用的模块/包名
app = Flask(__name__)

# route装饰器告诉Flask哪个URL会触发这个函数
@app.route('/')
def hello_world():
     return 'Hello, World'
```

## 运行应用
export FLASK_APP=hello.py
flask run

### 外部可访问
flask run --host=0.0.0.0

### Debug模式
1.激活debugger
2.激活自动reloader
3.在Flask应用上启动debug模式
export FLASK_DEBUG=1
flask run

# 路由
route()装饰器用来将函数绑定到URL
```python
@app.route('/')
def index():
     return 'Index Page'

@app.route('/hello')
def hello():
     return 'Hello, World'
```

## 变量规则
匹配变量以及变量转换
```python
@app.route('/user/<username>')
def show_user_profile(username):
     return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
     return 'Post %d' % post_id
```

## 唯一URL/重定向
```python
# 访问不带/的URL会重定向到带/的URL
@app.route('/projects/')
def projects():
     return 'The project page'

# 访问带/会导致404
@app.route('/about')
def about():
     return 'The about page'
```

## URL构建
使用url_for()生成URL
```python
from flask import Flask, url_for
app = Flask(__name__)
@app.route('/')
def index():
     pass

@app.route('/login')
def login():
     pass

@app.route('/user/<username>')
def profile(username):
     pass

# 
with app.test_request_context():
     print url_for('index')                                        # /
     print url_for('login')                                         # /login

     # 未知变量通过?加在URL后面
     print url_for('login', next='/')                           # /login?next=/
     print url_for('profile', username='John Doe')    # /user/John%20Doe 
```

# HTTP方法
默认只响应GET请求,可以通过route()装饰器的method参数定义
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
     if request.method == 'POST':
          do_the_login()
     else:
          show_the_login_form()
```

# 静态文件
在项目目录下创建static目录,可以用/static访问
```python
# static/style.css
url_for('static', filename='style.css')
```

# 呈现模板
render_template()方法,Flask在templates目录下查找模板
1.如果应用是一个模块,那么templates目录在模块的旁边
/application.py
/templates
     /hello.html

2.如果应用是一个包,那么templates在包内
/application
     /__init__.py
     /templates
          /hello.html
```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
     return render_template('hello.html', name=name)
```

```python
# eg. 模板
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
     <h1>Hello {{ name }}!</h1>
{% else %}
     <h1>Hello, World!</h1>
{% endif %}
```

# 获取请求数据

## Request对象
```python
from flask import request

@app.route('/login', methods=['POST', 'GET'])
def login():
     error = None
     if request.method == 'POST':
          # 使用form属性访问表单数据
          # 如果key不再form属性中,抛出一个KeyError
          if valid_login(request.form['username'], request.form['password']):
               return log_the_user_in(request.form['username'])
          else:
               error = 'Invalid username/password'
     return render_template('login.html', error=error)

# 获取URL中的参数
# (?key=value), 使用args属性
searchword = request.args.get('key', '')
```

## 上传文件
在HTML表单中设置enctype="multipart/form-data"
上传的文件保存在内存或系统的临时目录中
通过request对象的files属性访问文件,和file对象类型,另外有一个save()方法
```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
     if request.method == 'POST':
          f = request.files['the_file']
          f.save('/var/www/uploads/uploaded_file.txt')

# 获取filename属性可以得到文件在客户端上的名字,不过可以被伪造
# werkzeug的secure_filename()方法
from flask import request
from werkzeug.utils import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
     if request.method == 'POST':
          f = request.files['the_file']
          f.save('/var/www/uploads/' + secure_filename(f.filename))
```

## Cookies
通过cookies属性获取request的cookies,是一个字典,包含了客户端发送的所有cookies
通过response对象的set_cookie方法设置cookies
如果想使用session,使用Flask的Session而不是直接使用cookies,更加安全
```python
from flask import request

# 获取cookies
@app.route('/')
def index():
     username = request.cookies.get('username')

# 存储cookies
@app.route('/')
def index():
     # 使用make_response方法自定义response对象
     resp = make_response(render_template(...))
     resp.set_cookie('username', 'the username')
     return resp
```

# 重定向和错误
redirect()
abort()
```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
     return redirect(url_for('login'))

@app.route('/login')
def login():
     abort(401)
     this_is_never_executed()

# 使用errorhandler()装饰器定制error页面
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
     return render_template('page_not_found.html'), 404
```

# 关于Responses
从view函数返回的值自动转成response对象
返回值为string时,转变成response对象,string是response body,状态码200,mimetype为text/html
Flask应用于将返回值转换为response对象的逻辑如下
1.




















































