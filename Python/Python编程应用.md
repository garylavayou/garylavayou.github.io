# Python编程应用

Python应用框架和编程接口。

## 命令行工具

### click

Command Line Interface Creation Kit (click)

- 命令嵌套
- 自动生成帮助文档
- 支持运行时懒加载子命令
- Unix/POSIX命令行约定

https://click.palletsprojects.com/en/7.x/quickstart/

```shell
pip install click 
```

#### 应用模板

```python
import click
@click.group(help="group info.")   # 创建命令组，可以添加子命令
def cli():
   pass

@click.command(name='init', help='command info.', eplilog)
def initdb():
    click.echo('Initialized the database')
cli.add_command(initdb)  # 添加子命令

@cli.command(name='drop')   # 添加子命令
@click.option('-c','--count', default=1, help='number of greetings') # 添加选项
@click.argument('name')  # 添加位置参数
def dropdb(count, name):
    click.echo('Dropped the database')
      
if __name__ == '__main__':
    cli()  # 调用定义的任意命令或分组
```

##### 分组和子命令

使用`@click.command()`定义程序所需处理的命令行参数及其对应的处理函数；如果程序具有多个子命令，则需要使用`@click.group()`定义统一入口，以访问子命令。此外，通过`@Group.group()`可以嵌套定义子命令集合。

> 主程序只能调用一个`click`对象（返回后程序结束），因此必须将子命令的处理方法组织到一个分组中。否则主程序只能调用其中一个子命令的处理方法。

除了根分组外，子命令和子分组的名称默认为对应处理函数名，也可通过`name`参数指定。

##### 参数分类

- **位置参数**：使用`@click.argument()`定义，第一个参数为参数名，并对应作为Python变量名。

- **选项参数**：使用`@click.option()`定义，参数列表`*args`用于定义多个长/短选项名（例如`"-n","--name","name"`）。

  - 其中如果包含没有短划线前缀的名称，则作为该选项的变量名传递给修饰的函数；反之，

  - 第一个长选项用于推导选项变量名（移除前缀并将名称中出现的短划线转换为`_`以构成合法变量名）；

  - 没有长选项时，使用第一个短选项作为变量名。


> [setuptools integration](https://click.palletsprojects.com/en/7.x/quickstart/#switching-to-setuptools)

通过装饰器声明的参数，必须在对应的处理函数的输入参数中声明。

##### 参数数据类型

`str`（默认类型），`int`，`float`，`bool`

`click.DateTime`

如果未指定类型，则根据默认值`default`推导（如果未提供`default`，则默认为`str`）。

#### 选项参数

```python
@click.option('-c', '--count', default=1, help='number of greetings')
```

> `help`：设置文档的帮助信息；`metavar`：帮助文档中用于表示选项的值。

##### 强制选项

选项通常时可选的，如果需要强制某选项提供，则令`required=True`；

##### 标识选项

- **真值标识选项**：`is_flag=True`：如果选项名称中包含`/`，则定义真值开关的两个选项（变量名为前者）。

  ```python
  @click.option('--shout/--no-shout', default=False)
  ```

- **切换标识选项**：`flag_value=value`，多个选项定义一个变量，其中一个设置为`True`。

  ```python
  @click.option('--upper', 'transformation', flag_value='upper', default=True)
  @click.option('--lower', 'transformation', flag_value='lower')
  ```

##### 多值选项

- 一个选项如果要接受多个参数，使用`nargs=k`声明，该选项变量将保存为`tuple`；也可以通过直接定义多个参数的类型`type=(str,int)`表明选项接受的参数个数。
- 可重复使用的选项`multiple=True`：每次使用选项指定的值都被添加到序列对象；
- **计数选项**：`count=True`：选项的值为选项声明的次数。

##### 取值受限的选项

根据选项配置，对输入值进行校验。

- **单选选项**：

  ```python
  @click.option('--hash-type', 
                type=click.Choice(['MD5', 'SHA1'], 
                case_sensitive=False))
  ```

- **数值范围选项**：`IntRange`/`FloatRange`（默认为闭区间，可通过`min_open`/`max_open`设置开区间）。

  ```python
  @click.option('--count', type=click.IntRange(0, 20, clamp=True)) #*
  @click.option('--digit', type=click.IntRange(0, 10))
  ```

  > `*`：`clamp=True`对于超出范围的输入参数，将其修改为设定的边界。

- **密码选项**：

  ```python
  @click.password_option()  # equivalent password
  @click.option('--password', prompt=True, hide_input=True, confirmation_prompt=True)
  ```


##### 交互式选项与回调函数

`prompt=Ture|message`：当命令行未提供该选项的值，弹出消息提示用户输入该选项的值。

根据用户输入决定程序的后续处理流程：

```python
@click.confirmation_option(prompt='Are you want to continue?')
```

> 等价实现方式：
>
> ```python
> def abort_if_flase(ctx, param, value):
>    if not value:  # value of 'yes'
>       ctx.abort()
> @click.option('--yes','-y', is_flag=True, callback=abort_if_false, 
>               expose_value=False,  # 不向处理流函数传递该参数
>               prompt='Are you want to continue?')
> ```

##### 选项默认值

通过`default`参数设置选项默认值。默认值可以是固定编码值，可以从配置文件或系统环境变量读取。

```python
@option('username', 
        default=lambda: os.environ.get('USER',''), 
        show_default='current user', prompt=True)
```

`default`如果为函数，则可同时与`prompt`结合使用；反之，如果设置了`default`则不会再弹出交互信息。使用`show_default`在文档中显示默认值信息。

`auto_envvar_prefix='PREFIX'`：自动读取指定前缀环境变量`PREFIX_COMMAND_OPTION`（命令名中的`-`使用`_`替换）；

手动指定环境变量：

```python
@click.option('username', envvar='USERNAME')
```

**从一个环境变量中读取多个值**。在命令行利用`multiple=True`可重复使用一个选项以读入多个值；对于从环境变量读取，则尝试将环境变量分割为多个部分（默认使用空白分割，对于`File`和`Path`类型的选项值，则使用平台相关分隔符）。

`default_map`：

##### 优先处理选项

```python
@click.version_option()
```

> 实现方式：使用`eager`选项以优先处理该参数。
>
> ```python
> def print_version(ctx, param, value):
>    if not value or ctx.resilient_parsing: # value of 'version'
>       return
>    ctx.echo('version 1.0')
>    ctx.exit()
> @click.option('--version','-V', is_flag=True, callback=print_version, 
>               expose_value=False, eager=True)
> ```



[Multiple Values from Environment Values](https://click.palletsprojects.com/en/7.x/options/#multiple-values-from-environment-values)

[Callbacks for Validation](https://click.palletsprojects.com/en/7.x/options/#callbacks-for-validation)

##### 位置参数

参数接受的值的个数：

```python
@click.argument('src', nargs=-1)  # 接受任意数量的值（只能由一个此类型）
@click.argument('dst', nargs=1)
```

> `required=True`，可变数量参数至少包括一个参数（不能为空）。

[File Arguments](https://click.palletsprojects.com/en/7.x/arguments/#file-arguments)

`click.File`

[File Path Arguments](https://click.palletsprojects.com/en/7.x/arguments/#file-path-arguments)

`click.Path`

##### 命令嵌套

https://click.palletsprojects.com/en/7.x/commands/

子命令的方法运行时父命令的方法也被执行。

[Group Invocation Without Command](https://click.palletsprojects.com/en/7.x/commands/#group-invocation-without-command)

##### 生成文档

https://click.palletsprojects.com/en/7.x/documentation/

##### 自动补全

> Completion is only available if a script is installed and invoked through an entry point, not through the `python` command. See [Setuptools Integration](https://click.palletsprojects.com/en/7.x/setuptools/#setuptools-integration).

#### 交互式程序中的应用

在Python脚本主模块中，调用`click`定义的任意命令或分组，即可实现相应命令的自动解析处理。

通过上述方式，执行命令后程序将自动退出。对于不自动退出的交互式程序，需要在程序中显式调用该命令行框架并将所需处理的命令行参数传入。

```python
cli.main(['add', '--help'], standalone_mode=False)
```

`standalone_mode=False`：在命令执行完成后继续执行程序，反之直接退出程序。

> *问题：`click`捕获`SystemExit`，使得注册的退出处理函数不会被触发。*

next：https://click.palletsprojects.com/en/7.x/complex/。

### Typer

> [**Typer**](https://typer.tiangolo.com/) is FastAPI's little sibling. And it's intended to be the **FastAPI of CLIs**. ⌨️ 🚀

## HTTP

### HTTP客户端

#### Python `http.client`模块

[`http.client`](https://docs.python.org/3.7/library/http.client.html#module-http.client) is a low-level HTTP protocol client; implement the client side of the HTTP and HTTPS protocols. It is normally not used directly — the module [`urllib.request`](https://docs.python.org/3.7/library/urllib.request.html#module-urllib.request) uses it to handle URLs that use HTTP and HTTPS.

> [`http`](https://docs.python.org/3.7/library/http.html#module-http) is a package that collects several modules for working with the HyperText Transfer Protocol:
>
> - `http.client`;
> - [`http.server`](https://docs.python.org/3.7/library/http.server.html#module-http.server) contains basic HTTP server classes based on [`socketserver`](https://docs.python.org/3.7/library/socketserver.html#module-socketserver)
> - [`http.cookies`](https://docs.python.org/3.7/library/http.cookies.html#module-http.cookies) has utilities for implementing state management with cookies
> - [`http.cookiejar`](https://docs.python.org/3.7/library/http.cookiejar.html#module-http.cookiejar) provides persistence of cookies

```python
http.client.HTTPConnection(host, port=None, timeout=None)
HTTPConnection.request(method, url, body=None, headers={})
HTTPConnection.getresponse()
# HTTPSConnection
```



#### Python `urllib.request`模块

> *Extensible library  of high level API for opening URLs.*

```python
r = urllib.request.urlopen(url_or_request, data=None, timeout=None)
urllib.request.Request(url, data=None, headers={}, method=None)
r.url
r.headers
r.response
r.read(length)  # read response body
```

> *This function always returns an object which can work as a context manager and has the properties url, headers, and status.*
>
> *The default method is 'GET' if data is None or 'POST' otherwise.*



#### urllib3

[Home - urllib3 2.0.0.dev0 documentation](https://urllib3.readthedocs.io/en/latest/)

##### 请求

```python
response = urllib3.request(METHOD, url)
http = urllib3.PoolManager()
response = http.request(METHOD, url,headers=hdr_dict, fields=query_dict)
resp = http.request(
  "POST", url,
  fields={"hello": "world"}, # form data
  body=encoded_data  # JSON string or something else
)
```

##### 响应

```python
response.status
response.headers
response.data
```



#### requests

[Requests: HTTP for Humans™ — Requests is an elegant and simple HTTP library for Python, built for human beings.](https://requests.readthedocs.io/en/latest/)

> *Requests allows you to send HTTP/1.1 requests extremely easily. There’s no need to manually add query strings to your URLs, or to form-encode your POST data. Keep-alive and HTTP connection pooling are 100% automatic, thanks to [urllib3](https://github.com/urllib3/urllib3).*

##### 请求

```python
response = requests.get(
  url, 
  params=query_dict,          # URL参数=> ?key=value&key=value&...
  headers=hdr_dict,           # e.g., {'user-agent': 'my-app/0.0.1'}
  cookies=cookie_dict_or_jar, # jar = requests.cookies.RequestsCookieJar(),
  allow_redirects=True,       # HEAD默认为False
  timeout=0.001
)
```

URL参数：如果参数的值为列表，则生成的URL中将会包含多个同名参数，依次使用列表中的值。

HTTP头部：由于HTTP头部字段名称不区分大小写，因此可以通过任意大小写访问同一字段。

Cookies：以字典或`requests.cookies.RequestsCookieJar`（访问方式与字典相同，提供更完整的设置接口）的形式提供。

SSL证书验证：指定`CA_BUNDLE`文件目录，或受信任CA的证书目录（默认从环境变量`$REQUESTS_CA_BUNDLE`读取）。

```python
requests.get('https://github.com', verify='/path/to/certfile') 
requests.get('https://kennethreitz.org', verify=False) # 关闭验证
```

> `requests`使用`certifi`包提供的证书，用户可以通过更新`certifi`包以维持证书的最新状态。

代理服务器（对`Session`对象设置的代理为默认配置，会被环境变量配置覆盖；为保证代理设置有效，应该在每一次请求时设置）：

```python
proxies = {
    'http://10.20.1.128': 'http://10.10.1.10:5323', # 精确匹配
    'http': 'http://user:pass@10.10.1.10:3128',
    'https': 'http://10.10.1.10:1080',
}
```

> [Advanced Usage —Proxies](https://requests.readthedocs.io/en/latest/user/advanced/#proxies)

`post`方法用于向服务器提交数据。除了`get`方法支持的参数外，还提供参数以传递提交的数据。

```python
response = requests.post(
  url, 
  data = payload_str_or_dict, # data/files优先级高于json
  files = file_dict,
  json = payload_dict,        # -> Content-Type: application/json
  **kwargs
)
```

发送数据格式：

- `data:[Dict(key,value), Tuple(key,value)]`：HTML表单，`value`可以是序列。通过元组方式可重复为同一个`key`指定多个`value`，表单中该`key`的值拼接为一个序列。

  ```python
  data = {'key1': 'value1', 'key2': 'value2'}
  ```

  ```json
  { //...
    "form": {
      "key2": "value2",
      "key1": "value1"
    }, //...
  }

- `data:str`：直接将文本数据作为消息体。

  如果要发送JSON对象，可将其通过`json.dumps()`首先转换为文本，在传递给`data`；这种方式不会设置`Content-Type=application/json`，如果有必要需要手动设置。

- `json：dict`：`posts`方法提供专门的`json`参数以传递JSON对象并正确设置`Content-Type`。

- `files:Dict('file': Tuple)`：将文件以`multipart/form-data`的形式发送。`files`具体使用格式：

  ```python
  files = {
    'file': (
      'report.xls',               # 提交给服务器的文件名
      open('report.xls', 'rb'),   # 待发送的文件数据*
      'application/vnd.ms-excel', # Content-Type
      {'Expires': '0'}
    )
  }
  ```

  > `*`：可以是打开的文件对象，也可以是字符串、字节数组。推荐以`binary`模式打开文件，避免对`Content-Length`估计错误。
  >
  > 如果安装了`requests-toolbelt`库，则支持流式数据传输。

- `files: List[(form_field_name,file_info)]`：以`multipart/form-data`格式一次发送多个文件，其中`form_field_name`通常为与服务器交互的表单字段名，`file_info`的结构与发送单个文件时传递给`files`参数的结构相同。

> 其他HTTP方法：`put, delete, head, options, ...`

##### 响应

```python
response.status_code         # requests.codes提供了状态码枚举变量
response.raise_for_status()  # 将HTTP请求错误抛出为异常(200无任何效果)
response.headers  # HTTP headers (Header names are case-insensitive)
response.cookies  # 
```

通过响应对象也可查看对应的请求信息：

```python
response.url      # 请求URL
response.request.headers
response.request.cookies  # 
```

文本响应：

```python
response.text     # 响应体(文本)
response.encoding # 响应体编码，设置编码以获得正确解码的text
response.json     # 将JSON响应体解码为JSON对象
```

非文本响应：

```python
response.content  # binary（gzip/deflate编码自动解压缩）*
response.raw      # HTTP原始响应（未经处理的接收数据）
```

> `*`：如果安装了`brotli`或`brotlicffi`，则可以自动解压缩`br`编码

分块接收响应数据：

```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size=128):   # iter_lines() 
        fd.write(chunk)
```

请求参数`stream=True`：当使用响应的消息体时，才开始下载数据。

##### 重定向

除`HEAD`方法外，`requests`将自动处理重定向（设置`allow_redirects=False`以禁用自动重定向）；对于`HEAD`方法，可以设置`allow_redirects=True`启用重定向。重定向过程中的响应数据可通过`response.history`查看。

##### 异常

`ConnectionError`：网络问题；

`HttpError`：HTTP返回非成功响应码；

`Timeout`：响应的超时；

`TooManyRedirects`：重定向次数超过限制；

`RequestException`：所有引发的异常的基类。

##### 会话对象

会话对象将保存HTTP请求的某些参数和Cookies，并重用连接池中的TCP连接（HTTP持久连接）。

```python
session = requests.Session()  # 通过session调用requests接口
s.auth = ('user', 'pass')     # 设置会话公用参数
s.headers.update({'x-test': 'true'})
with requests.Session() as s: # 支持上下文管理器
	response = session.get(url)
```

### Flask

The “micro” in microframework means Flask aims to keep the core simple but extensible.

Flask has many configuration values, with sensible defaults, and a few conventions when getting started. By convention, templates and static files are stored in subdirectories within the application’s Python source tree, with the names `templates` and `static` respectively.

Flask itself just bridges to Werkzeug to implement a proper WSGI application and to Jinja2 to handle templating. It also binds to a few common standard library packages such as logging. Everything else is up for extensions.

Flask supports Python 3.6 and newer. `async` support in Flask requires Python 3.7+ for `contextvars.ContextVar`.

```shell
conda create -n flask python=3.7 flask
```

[Quickstart — Flask Documentation (2.0.x) (palletsprojects.com)](https://flask.palletsprojects.com/en/2.0.x/quickstart/)



#### Flask App

##### Project Layout

```shell
flaskproject/
├── flaskproject/
│   ├── __init__.py
│   ├── db.py
│   ├── schema.sql
│   ├── auth.py
│   ├── blog.py
│   ├── templates/
│   │   ├── base.html
│   │   ├── auth/
│   │   │   ├── login.html
│   │   │   └── register.html
│   │   └── blog/
│   │       ├── create.html
│   │       ├── index.html
│   │       └── update.html
│   └── static/
│       └── style.css
├── tests/
│   ├── conftest.py
│   ├── data.sql
│   ├── test_factory.py
│   ├── test_db.py
│   ├── test_auth.py
│   └── test_blog.py
├── venv/
├── .gitignore
├── setup.py
└── MANIFEST.in
```



[Project Layout — Flask Documentation (2.1.x) (palletsprojects.com)](https://flask.palletsprojects.com/en/latest/tutorial/layout/)

[Application Setup — Flask Documentation (2.1.x) (palletsprojects.com)](https://flask.palletsprojects.com/en/latest/tutorial/factory/)

[Deployment Options — Flask Documentation (2.1.x) (palletsprojects.com)](https://flask.palletsprojects.com/en/latest/deploying/#self-hosted-options)

[Template Designer Documentation — Jinja Documentation (3.0.x) (palletsprojects.com)](https://jinja.palletsprojects.com/en/3.0.x/templates/)

##### App Layout

```python
from flask import Flask, request
app = Flask(__name__)
app.config['JSON_SORT_KEYS'] = False    # 响应JSON数据不对字段排序
@app.route("/url_path/<name>", methods=['GET', 'POST'])
def route_process(name="test"):
  req = request.get_data(as_text=True)
  return "success"
if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False,load_dotenv=True,**wsgi_options)
```

> `load_dotenv=True`：加载`.env`或`.flaskenv`中定义的环境变量。

##### 调试运行Flask App

如果Python代码中包含`__main__`代码块，且其中调用了`app.run()`方法，则可以直接运行该代码文件或模块启动Flask程序。反之，通过`flask run`命令自动调用指定Python模块`$FLASK_APP`（默认为`app`）中的`Flask`实例（通常是`app=Flask(__name__)`）的`run()`方法。

```shell
export FLASK_APP=hello
export FLASK_ENV=development
flask run --host=0.0.0.0 --port=5000  --cert=PATH --key=FILE  # => python -m flask run
```

> 这种方式不会执行Flask程序中的`__main__`代码块，因此不能将额外参数传递给要执行的程序。

**调试模式**：如果配置环境变量`FLASK_ENV=development`或运行时参数`debug=True`，将启用调式模式，在代码发生变化时服务器会**自动加载**且在发生异常时在终端显示交互式的调试器（使用`use_evalex=False`禁用调试器，使用`use_reloader=False`禁用自动加载），并在前端显示异常的追踪信息页面。未启用调试模式时，Flask会忽略任意处理过程产生的错误，仅返回通用的错误页面。==不推荐在启用自动加载的情况下通过代码调用`run()`方法（支持不好），而是使用`flask run`的方式==。大规模生产环境部署需要使用专门的WSGI服务器。

[使用VS Code调试Flask App](https://code.visualstudio.com/docs/python/tutorial-flask)。

#### 处理请求

##### 路由绑定

```python
@app.route("PATH", methods=['POST','GET',...],)
```

**路径**：客户端请求通过装饰器声明的路径规则进行分发，路径规则：

- `/path`：路径以“`/`”结尾代表目录，请求省略`/`会被自动重定向到对应目录；反之，代表文件，如果请求包含“`/`”则无法找到相应资源（返回`404 Not Found`）。
- `/path/<ARGUMENT>`：`ARGUMENT`作为响应函数的参数。
- `/path/<int:ARGUMENT>`：限定参数数据类型。

一个响应方法可添加多个路径声明。

`url_for()`方法用于测试当前已声明的处理方法对应的URL。

```python
with app.test_request_context():
    print(url_for('index'))
    print(url_for('login'))   # "index" and "login" 是已定义的处理方法 
```

##### 获取请求信息

```python
from flask import request      # context local object
request.url                    # request.base_url, request.host_url, request.root_url
request.args                   # -> dict: URL查询参数 -> request.query_string
request.method
data = request.data            # binary data
request.get_data(as_text=True) # text data
request.get_json()             # json object -> request.is_json
```

为了避免用户请求数据中包含的注入攻击脚本被执行，使用`escape`将请求数据转换为纯文本。

```shell
from markupsafe import escape
@app.route("/<name>")
def hello(name):
    return f"Hello, {escape(name)}!"
```

##### 构造响应消息

处理方法必须提供返回值（不能返回`None`），响应消息可以是纯文本、JSON对象、HTML文本等类型。

##### 重定向和错误处理

```python
from flask import abort, redirect, url_for
redirect(url_for('static', filename=subpath))
abort(404)
```

##### 会话

除了处理单次请求外，Flask为每个会话提供了会话上下文`session`。[处理方法可在会话上下文中记录用户的登录状态](https://flask.palletsprojects.com/en/2.0.x/quickstart/#sessions)，从而连续处理用户的不同请求。

##### 日志

```python
app.logger.warning('A warning occurred (%d apples)', 42)  # logging.Logger
```



#### 静态文件

静态文件位于Flask程序的工作目录下`static/`目录中。这些文件可以根据用户请求的URL（如`/static/style.css`）被自动响应。

#### HTML模板

利用指定模板（Jinja2）和输入参数自动生成HTML响应。模板文件位于程序目录下的`templates/`目录下。

```python
from flask import render_template
return render_template('hello.html', name=name)
```

#### 文件上传

#### Cookies

#### 部署Flask应用

> *While lightweight and easy to use, Flask’s built-in server is not suitable for production as it doesn’t scale well. You can deploy your Flask application to a WSGI server, where your `Flask` application object is the actual WSGI application.*

Flask实现了一个简易的HTTP WSGI服务器，方便开发者调试运行Flask程序而无须安装专门的WSGI服务器。但内置WSGI服务器功能简单，性能扩展能力差，因此可能无法胜任生产环境大规模服务请求。

[花了两个星期，我终于把 WSGI 给搞明白了 - 知乎用户的文章 - 知乎](https://zhuanlan.zhihu.com/p/269456318)

WSGI服务器包括：Gunicorn、[uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/#)、

##### Run Flask with Gunicorn

```shell
gunicorn -w 4 -b 0.0.0.0:5000 your_project:app # serve your app with 4 workers on port 5000
```

`app`是模块中可调用的Flask应用实例，或者提供创建实例的函数`myproject:create_app()`。

使用gevent（`-k gevent`）或[eventlet](https://eventlet.net/)（`-k eventlet`）提供工作线程的异步支持。

> *[gevent](http://www.gevent.org/) is a coroutine -based Python networking library that uses greenlet to provide a high-level synchronous API on top of the libev or libuv event loop.*

##### Nginx代理WSGI服务器

```nginx
server {
  location /flask {
    proxy_pass         http://127.0.0.1:8000/;
    proxy_redirect     off;
    proxy_set_header   Host                 $host;
    proxy_set_header   X-Real-IP            $remote_addr;
    proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto    $scheme;
  }
}
```



### FastAPI[^flask-fastapi]

[FastAPI (tiangolo.com)](https://fastapi.tiangolo.com/)

```shell
conda install -c conda-forge fastapi uvicorn
# pip install fastapi uvicorn[standard]
```

> FastAPI没有内置开发服务器，因此需要使用`uvicorn`作为ASGI服务器。

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()

@app.get("/")  # 单独声明HTTP方法的处理函数
def home():
    return {"Hello": "World"}
@app.post("/")
def home_post():
    return {"Hello": "POST"}

if __name__ == "__main__":
    uvicorn.run("fastapi_code:app", reload=True) # enable hot-reloading for development.
    # uvicorn run fastapi_code:app --reload
```

##### URL参数

```python
@app.get("/employee/{id}")
def home(id: int):   # 从URL中解析参数，代码支持类型提示
    return {"id": id}
```

查询参数`/employee?department=sales`将自动被解析为处理函数的参数，在处理函数的参数中声明即可使用。

##### 静态文件

```python
from fastapi.staticfiles import StaticFiles
app.mount("/static", StaticFiles(directory="static"), name="static")
```

##### 数据验证

支持使用对象模型对请求参数进行反序列化和校验，并对响应对象进行序列化。

```python
from pydantic import BaseModel
class Request(BaseModel):
    username: str
    password: str
class Response(BaseModel):
    username: str
    email: str
@app.post("/login", response_model=Response) # 自动过滤响应数据模型中未定义字段
async def login(req: Request):
    if req.username == "testdriven.io" and req.password == "testdriven.io":
        return {"message": "success"}
    return {"message": "Authentication Failed"}
```

##### 异步处理模型

支持`asyncio`，避免处理方法阻塞HTTP服务。

```python
@app.get("/")
async def home():
    result = await some_async_task()  # 将在异步任务完成时,继续执行后续代码
    return result
```

后台任务模式：立即返回处理状态，让后台任务继续执行，避免客户端等待（可能需要额外的后台任务查询接口）。

```python
from fastapi import BackgroundTasks
def process_file(filename: str):
    # process file :: takes minimum 3 secs (just an example)
    pass
@app.post("/upload/{filename}")
async def upload_and_process(filename: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(process_file, filename)
    return {"message": "processing file"}
```

##### 模块化路由

```python
# routers/product/views.py
from fastapi import APIRouter
product = APIRouter(prefix='/product')
@product.get("/{id}")
def get_product(id:int):
    pass

# main.py
from routers.product.views import product
app.include_router(product)
```



## 数据库

SQLAlchemy由Core和ORM两套API组成，其中ORM基于Core构建。

- Core：数据库工具集的基础架构，包括管理数据库连接、处理查询和响应、构建SQL查询语句。

- ORM：提供Python对象到数据库表的映射以及对象持久化机制。

  > 对象关系映射（***Object Relational Mapping***，ORM）模式使用描述对象和数据库之间映射的元数据，实现程序中的对象与数据库中的记录的同步与持久化存储。

```python
import sqlalchemy as sqa
print(sqa.__version__)    # -> 1.4.20
```

### 数据库连接管理

```python
db_url='dialect[+driver]://user:password@[host]/dbname[?key=value..],'
engine = sqa.create_engine(db_url, encoding='utf-8', echo=False, future=True)
```

- `echo`：回显执行的SQL命令（如果为`"debug"`，还将输出返回结果）；也可通过`logging`模块调整输出信息的级别。

- `future=True`：完全使用2.0风格API。

> 数据库连接：需要首先安装相应数据的驱动包（DBAPI）。
>
> - SQLite：`"sqlite+pysqlite:///:memory:"`、相对路径`sqlite:///relative/path/to/file.db`、**绝对路径**`sqlite:////absolute/path/to/file.db`。
> - PostgreSQL：`"postgresql+psycopg2://user:passwd@/dbname?host=/var/run/postgresql"`
> - MySQL：`"mysql+pymysql://root:gang2019wsl@/test?unix_socket=/var/run/mysqld/mysqld.sock"`



##### 连接

`Engine`对象用于管理数据库连接`Connection`，从而实现对数据库的并发访问。当连接关闭时，连接的资源实际返回给`Engine`所管理的连接池，同时调用`.rollback()`释放未完成的事务以及锁从而使之能够被再次使用。

```python
with engine.connect() as connection:
    result = connection.execute(sql_command)
```

##### 事务

事务用于保证数据操作过程的正确性和完整性，如果在事务执行过程中发生异常，则可以回退到初始状态，防止对数据库产生非预期操作。`Engine.begin()`或`Connection.begin()`会创建一个事务`Transaction`对象，使用上下文管理器来自动提交事务`.commit()`和处理异常回退`·.rollback()`（不使用上下文则需要手动提交和进行异常回退）。

```python
with engine.begin() as conn:  # -> Transaction
     result = conn.execute(sql_command)
# equals to =>
#     with engine.connect() as connection:
#         with connection.begin():
#             result = connection.execute(sql_command) 
```

### 数据库元数据

SQLAlchemy的**查询和对象映射**需要**数据库元数据**（*database metadata*）支持。元数据以Python对象的形式描述表格和其他Schema层次的对象（SQLAlchemy使用***schema name***来区分表格所属命令空间）。

> PostgreSQL的默认命名空间为`public`。

#### 构造数据库元数据

元数据存储对象`MetaData`用于保存一个数据相关的元数据。

```python
from sqlalchemy import MetaData
metadata_obj = MetaData(bind=engine, schema="default_schema")
```

> `engine`可不提供，在实际执行操作时通过相关接口传入覆盖初始配置。当指定`schema`时，表格将显式通过`schema`来引用。如果需要通过元数据对数据执行实际操作，则==数据库中必须已经存在相应的命名空间==。

向元数据存储中添加表格定义（描述数据类型及其他属性）：

```python
from sqlalchemy.schema import Table,Column,ForeignKey,Sequence
from sqlalchemy import Integer, String
user = Table('user',metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('name', String(16), nullable=False),
    schema='user' 
)
```

访问元数据存储的表格定义：

```python
for n in metadata_obj.tables:  # <-> metadata_obj.sorted_tables
    table:Table = metadata_obj.tables[n]
    print(n)        # -> print(table) -> print(table.name)   
    table.metadata  # a runtime property of MetaData
    table.bind      # bound Engine/Connection
```

访问表格列信息：

```python
col = user.columns.id  # -> user.c.id -> user['id']
col.name  # col.{type,nullable,primary_key,foreign_keys}
col.table # 反向引用列所属的表
for col in user.c:
    print(col)
for primary_key in employees.primary_key:
    print(primary_key)
for fkey in employees.foreign_keys:
    print(fkey)
```



#### 获取数据库元数据

使用SQLAlchemy提供的反射机制（*reflection*）从数据库自动加载元数据。

##### 使用MetaData

[`MetaData`](https://docs.sqlalchemy.org/en/14/core/reflection.html#reflecting-all-tables-at-once)可自动返回连接数据库的所有数据表的元数据对象。

```python
from sqlalchemy import MetaData
metadata_obj = MetaData()
metadata_obj.reflect(bind=engine,**kwargs)
metadata_obj.tables  # 所有表组成的字典
```

使用`Table`可返回指定数据表（及其外键引用的数据表）的元数据。

```python
user = Table('user', metadata_obj, autoload_with=engine)
```

##### 使用Inspector

`Inspector`提供更加底层的接口获取数据库表的元数据。

```python
from sqlalchemy import inspect
from sqlalchemy.engine.reflection import Inspector
inspector:Inspector = inspect(engine)
for table in inspector.get_table_names():  # 获取表名
    for column in inspector.get_columns(table): # 获取列定义(字典)*
    	print(column['name'])
```

> `*`：列定义包含`name`、`type`、`nullable`、`default`、`autoincrement`、`comment`等属性。

检查表格是否存在：

```python
tf = sqa.inspect(sql_engine).has_table(table_name)
```

##### 使用SQL命令

使用[SQL语句从数据库的元数据表中获取](../数据库/PostgreSQL.md#数据库信息)，具体命令与数据库类型相关。不支持PostgreSQL的`pgsql`命令行工具的快捷命令，如`\dt`。

#### 创建和删除数据表

使用元数据定义（批量）创建表。

```python
metadata_obj.create_all(engine, checkfirst=True)
metadata_obj.drop_all(bind=engine)
table.create(bind=engine, checkfirst=True)
table.drop(bind=engine)
```

> `checkfirst=True`：如果表格存在则不执行操作。

### SQL查询API

#### API用法

查询语句的构造方式分为两类：1) 构造SQL命令的文本对象；2) 通过API调用构造命令；

```python
from sqlalchemy.sql import column, select, table  #*
sql_query = text("select x, y from some_table where y > :y") # 文本模式
t = table("some_table", column('x'), column('y'))  
sql_query = select(column('x'), column('y'))\
           .select_from(t)\
           .where(t.c.y > bindparam('y'))  # API构造模式
result = conn.execute(sql_query, {'y':'100'})
result.rowcount  # 查看查询/修改记录的行数
```

虽然SQL命令文本更加简洁，但是通过API调用可在程序更加灵活地参数化构造SQL查询命令（无需关注SQL语法细节）。

##### 查询对象构建

查询过程中的列和表都需要构造为语句对象而不能使用字符串文本直接表示，可用文本模式或API模式构建对象。

```python
col_a, table_x = text('col_a'), text('table_x')    # 文本模式
col_a, table_x = column('col_a'), table('table_x', schema='public') # API模式
```

API模式支持对列或表做更多参数配置（如为表指定所属的schema）。`column`，`table`分别是`ColumnClause`和`TableClause`类型构造方法的封装。区别于元数据定义中使用的`Column`和`Table`类，后者不仅继承前者，还扩展了与数据类型、属性和数据库后端相关的接口。

> 文本模式如果指定`schema_name.table_name`作为表名无法工作，因为该形式会被整体视为默认schema中的表名`public."schema_name.table_name"`。

##### 参数传递

在执行`execute()`方法时，SQL语句中的绑定参数通过`execute()`方法的`*multiparams, **params`参数传递。对于单次执行的命令，使用`execute()`方法的`**params`参数传递SQL语句中对应的绑定参数：

```python
conn.execute(sql, key1=value1, key2=value2)
```

对于无返回值的SQL命令（如`INSERT`、`UPDATE`等，即未添加`RETURNING`表达式），可以批量执行SQL命令，则需要使用`*multiparams`参数（可以==合并为一个字典对象的序列传递==，方便[从`pandas.DataFrame`传递数据](Python数据类型.md#以字典或列表初始化)），批量接收绑定参数对应的参数数据。SQL命令将根据每个字典对象中的参数执行一次查询（底层数据库API也可能对批量查询提供优化）。

```python
conn.execute(
    table.insert(), 
    {'key1': value1, 'key2': value2},
    {'key1': value3, 'key2': value4},
    # ...
) # [{'key1': v1, 'key2': v2}, {'key1': v3, 'key2': v4},...]
```

##### 输出SQL命令

可将通过API构造的SQL命令编译为文本形式并输出，从而方便验证是否正确调用相关API。

```python
stmt: ClauseElement = select(t).where(t.c.id == 5)
print(stmt)
sql: Compiled = stmt.compile(literal_binds=True)
print(sql)
```

`literal_binds=True`：将构造命令中的字面值嵌入表达式（反之在表达式中将以自动生成的绑定参数代替，执行时再使用字面值代替绑定参数）。

##### 表达式参数绑定

文本命令模式：使用`:arg_name`在SQL命令中嵌入运行时参数。

API构造模式：使用`bindparam()`设置运行时参数，其构造结果与文本造模式一致。对于参数为常量的表达式，SQLAlchemy也会将其转换为**自动命名参数**（基于列名和数字编号），并将常量作为参数的默认值在运行时传递给SQL语句。



##### 条件语句

逻辑连接：`and_(*exprs)`、`or_(*exprs)`和`not_(expr)`。支持使用相应的Python运算符`&`、`|`和`~`，但注意使用`()`限定每个表达式的范围。

##### 比较运算符

```python
column('x').is_(None)               # [is_/is_not] -> x IS NULL
column('x').in_([1, 2, 3])          # [in_/not_in]
                                    # any_()/all_()
                                    # between
column('x').like('word')            # like/ilike/notlike/notilike*
column('x').startswith('word')      # startswith/endswith/contains
                                    # match
column('name').regexp_match('word') # dialect specific
```

> `*`：需要添加通配符`%`以模糊匹配。


#### 读取数据

通过`SELECT`命令或`select()`API调用读取数据。

```python
from sqlalchemy import select
select(table).where(cond_expr)  
select(table.c.name, table.c.fullname)
select(name, fullname).select_from(table)
table.select().where(expr)  #*
```

> `*`：此方式不再支持传入列名以选择数据表中的列；同时，仅返回已定义的表对象中的列（`SELECT`命令会自动包含所定义的列）。

##### 选择数据源的所有列

```python
sql = select(text('*')).select_from(table)
```



##### 排序和分组统计

```python
from sqlalchemy import asc,desc,func
select(user).order_by(asc(user.c.name))  # -> user.c.name.asc()
select(user.name, func.count(user.id).label("count"))\
      .group_by(user.name)\
      .having(func.count(user.id) > 10)  # HIVAING是对分组结果进行过滤
```

##### SQL函数

提取字段：`extract("YEAR", t.c.date_created)`；

SQL统计函数：`func.FUNCNAME()`：将对应的函数调用`FUNCNAME()`转换为SQL函数表达式。

##### 连接

连接两个表并返回查询结果。默认根据两个表的外键字段进行连接；可通过`join_from`的第三个参数（`join`的第二个参数）指定连接条件，如`user.c.id == address.c.user_id`。

```python
select(user.c.name, address.c.email_address)\ 
     .join_from(user, address, isouter=False, full=False) #*
select(user.c.name, address.c.email_address).join_from(address,**kwargs) #**
```

> `*`：指定`OUTER`和`FULL`连接方式。
>
> `**`：`join()`方法只指定连接运算的右侧表（左侧根据查询推测）。

##### 获取执行结果

```python
result:CursorResult = connection.execute(sql_query)
for row in result: # -> CursorResult: DBAPI cursor
  print(f"x: {row.x}  y: {row.y}")
for row in result:  # 将行数据返回为对象: 可通过下标或属性访问数据
  x = row[0]
  y = row.y
for x, y in result: # 直接将序列对象展开为列元素
    # ...
for dict_row in result.mappings(): # 将行数据返回为字典
    x = dict_row['x']
    y = dict_row['y']
```

当返回的`CursorResult`被读取完后，DBAPI游标将被自动关闭（因此无法重复读取一个返回结果）。

[Selecting Rows with Core or ORM — SQLAlchemy 1.4 Documentation](https://docs.sqlalchemy.org/en/14/tutorial/data_select.html)

#### 修改记录

##### 插入

```python
from sqlalchemy import insert
stmt = insert(user_table)\ # -> user_table.insert()
      .values(name='name', fullname='Full Name')\
result = conn.execute(sql_query, records_data)  # 数据包含name,fullname字段
```

`Insert.values()`方法对绑定参数进行了简化，如果未显式调用`bindparam()`方法，则==SQL命令的参数名称与`values`方法的关键字参数名称相同==。

> 需要注意：`values()`关键字参数中定义的列必须包含在操作的数据表的列定义中（适用于`insert()`和`update()`方法），否则出错*`CompileError: Unconsumed column names: xxx`*。

##### 更新

根据匹配条件更新数据。如果未发生匹配则不会更改任何数据，如果未指定任何条件，则对所有数据都进行更新。

```python
update(user_table)\  # -> user_table.update()
      .where(user_table.c.name == 'patrick')\
      .values(fullname='Patrick the Star') # values类似于insert()
```

> `where()`和`values()`方法中的列名不能重复，否则出错*`CompileError: Bind parameter 'xxx' conflicts with unique bind parameter of the same name.`*。已经出现在`where()`语句中的列是不需要执行更新操作的，因此无须再出现在`values()`中。

##### 删除记录

```python
delete(user_table)\  # -> user_table.delete()
      .where(user_table.c.name == 'patrick')
```

##### 返回修改记录的信息

```python
stmt.returning(user.c.id, user.c.name)
```

在上述修改数据记录的相关命令后，调用`returning`将返回修改数据的指定字段（==调用`returning`后，执行命令时将不支持批量传递数据==）。根据这些字段可判断对应的数据是否按预期完成操作（某些插入操作可能存在主键重复，或是更新和删除操作存在多个匹配项或未发现匹配项等）。

### ORM API

[ORM Quick Start — SQLAlchemy 1.4 Documentation](https://docs.sqlalchemy.org/en/14/orm/quickstart.html)

##### 定义数据模型

[ORM Mapped Class Overview — SQLAlchemy 1.4 Documentation](https://docs.sqlalchemy.org/en/14/orm/mapping_styles.html#orm-mapping-styles)

声明式定义：

```python
from sqlalchemy import Column,ForeignKey,Integer,String
from sqlalchemy.orm import declarative_base,relationship
Base = declarative_base()  # 创建ORM对象存储
class User(Base):
    __tablename__ = "user_account"
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String)
    addresses = relationship(
        "Address", back_populates="user", cascade="all, delete-orphan"
    )
    def __repr__(self):
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"

class Address(Base):
    __tablename__ = "address"
    id = Column(Integer, primary_key=True)
    email_address = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey("user_account.id"), nullable=False)
    user = relationship("User", back_populates="addresses")
    def __repr__(self):
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"
```

命令式定义：类的字段可动态设置。

```python
from sqlalchemy import Table, Column, Integer, String, ForeignKey
from sqlalchemy.orm import registry

mapper_registry = registry()
user_table = Table(
    'user',
    mapper_registry.metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('fullname', String(50)),
    Column('nickname', String(12))
)
class User:
    pass
  
mapper_registry.map_imperatively(User, user_table)
```

数据模型可用于创建数据对象。

```shell
squidward = User(name="squidward", fullname="Squidward Tentacles")
krabs = User(name="ehkrabs", fullname="Eugene H. Krabs")
```

##### ORM会话

会话用于记录数据库与程序对象间的映射实例。通过上下文管理器实现数据的自动同步。

```python
with Session(engine) as session, session.begin():
  session.execute(select(User))  # 获取数据
  session.add(squidward)         # 修改数据
  session.add(krabs)
# equals to =>
#     with Session(engine) as session:
#         with session.begin():
#             session.add(some_object)
#             session.add(some_other_object)
```

[Session Basics — SQLAlchemy 1.4 Documentation](https://docs.sqlalchemy.org/en/14/orm/session_basics.html#what-does-the-session-do)



### 常见问题

##### 查询语句中存在特殊字符

- `URL`字段中如果存在特殊字符，使用`urllib.parse.quote_plus`进行转义；或者使用`sqa.engine.url.URL.create()`创建URL对象。

- `%`序列在SQL查询语句中会由SQLAlchemy再次执行变量替换，为了防止替换，使用`%%`代替`%`或将查询语句置于`sqlalchemy.text()`避免变量替换。

  > 可能出现错误提示：*`ValueError: unsupported format character (...) at index ...`*

  

## $\LaTeX$文档

### 生成$\LaTeX$代码

#### 文档对象

##### `Document`

```python
doc=Document(
   default_filepath='default_filepath',
   documentclass='article', 
   document_options=["12pt"],   # 文档类的选项列表
   fontenc='T1', inputenc='utf8', 
   font_size="normalsize", 
   lmodern=True, textcomp=True, page_numbers=True, indent=None, 
   geometry_options={"margin": "2.5cm"}, 
   data=None
)
doc.preamble.append(Command('title', 'Awesome Title'))
doc.preamble.append(Command('author', 'Gary Wang'))
doc.preamble.append(Command('date', NoEscape(r'\today')))
doc.append(NoEscape(r'\maketitle')) # from pylatex.utils import NoEscape
```

> `NoEscape`将字符串直接添加到文档内容中，不会对其进行转义解释。



 `generate_pdf` 

##### 章节

使用`doc.append(item)`向章节中添加内容，添加内容不会自动包含切换换段落。

```python
from pylatex import Section,Subsection,Subsubsection
with doc.create(Section('A section')):
    doc.append('Some regular text and some ') # 添加段落
    doc.append(italic('italic text. '))       # from pylatex.utils import italic
    with doc.create(Subsection('A subsection')):
        doc.append('Also some crazy characters: $&#{}')
```

##### 文本内容

文本中的特殊字符会在生成的$\LaTeX$文档中被还原未转义序列（例如，`"\"->"\textbackslash"`）。如果不需要转换使用`NoEscape(text)`。

##### 图表

```python
with doc.create(Table(position='!htbp')):
   with doc.create(Center()):
      with doc.create(Tabular('rccl')) as table:
         table.add_hline()
         table.add_row((1, 2, 3, 4))
         table.add_hline(1, 2)
         table.add_empty_row()
         table.add_row((4, 5, 6, 7))
         table.add_hline()
```

##### 数学环境

数学公式内容需要放在数学环境对象中。

```python
doc.append(Math(data=[r'2\times\alpha=9'], escape=False))
```

`Alignat`封装`align`环境，支持多行公式、编号等功能。

```python
with doc.create(Alignat(aligns=2, escape=False)):
   doc.append(r'c^2&=a^2+b^2\\')
   doc.append(r'x&=\frac{a}{b}')
```

矩阵对象`Matrix`可以基于`numpy`==二维数组==构造，可作为公式环境中的内容：

```python
a = np.array([[100, 10, 20]]).T
M = np.matrix([[2, 3, 4], [0, 0, 1], [0, 0, 2]])
with doc.create(Alignat(aligns=2, numbering=False)):
   doc.append(Matrix(M))
   doc.append(Matrix(a))
   doc.append('=')
   doc.append(Matrix(M*a))
```

##### 列表

```python
with doc.create(Itemize()) as itemize:
   itemize.add_item("the first item")
   itemize.append(Command("ldots"))
with doc.create(Enumerate(enumeration_symbol=r"\arabic*)",
                          options={'start': 20})) as enum:
   enum.add_item("the first item")
```



#### 格式

##### 对齐

`Center`对象定义居中对齐环境（`\begin{center}...\end{center}`）。

#### 命令对象

```python
Command(command=None, arguments=None, options=None, packages=None)
# \usepackage{packages}
# \command[options]{arguments}
```

```python
doc.append(Command("maketitle"))
doc.append(NoEscape(r'\maketitle'))
```



#### 生成源码

```python
doc.generate_tex(filepath=None)  # 将源码保存未tex文件
tex = doc.dumps()                # 将源码输出为字符串
```

### 编译文档

```python
doc.generate_pdf(
   filepath=None,      
   clean=True,         # 是否删除编译生成的辅助文件
   clean_tex=True,     # 是否删除生成的tex文件
   compiler=None,      # 编译引擎名，默认依次尝试latexmk和pdflatex
   compiler_args=None, # 编译引擎选项
   silent=True         # 是否输出编译日志
)
```

> 如果要使用`latexmk/xelatex`编译，需要指定编译选项`-xelatex`（检测到`latexmkrc`配置没有生效）。

## 参考资料

[^flask-fastapi]: [Moving from Flask to FastAPI | TestDriven.io](https://testdriven.io/blog/moving-from-flask-to-fastapi/).
[^fastapi]: [*FastAPI framework, high performance, easy to learn, fast to code, ready for production*](https://fastapi.tiangolo.com/)
