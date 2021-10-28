---
title: uWSGI v.s. Gunicorn in Python
date: 2021-09-08 14:10:00 +0800
categories: [Python, Framework]
tags: [flask]     # TAG names should always be lowercase
---

## Introduction (待補)

- what is WSGI?

- What are the differences between uWSGI and Gunicorn?

## Objectives

- 查看 uWSGI 及 Gunicorn 如何呈現

## 實作

### **環境設定**

```bash
$ pip install flask uwsgi gunicorn
```

### **python file**

建立 `main.py` 檔案，這邊用不用 flask 都可以

- using Flask

```python
from flask import Flask

app = Flask(__name__)
app.debug = True


@app.route('/')
def hello():
    return 'Hello, World!'


if __name__ == "__main__":
    app.run(debug=True)
```

- w/o Flask

```python
def app(environ, start_response):
    data = "Hello, World!"
    start_response("200 OK", [
        ("Content-Type", "text/plain"),
        ("Content-Length", str(len(data)))
    ])
    return iter([data])
```

### **uWSGI**

因為是在本機運行，所以我們使用`--http-socket`即可，如果是還有一層 web server (如 nginx)，則要用`--socket`。

以下是分別運行1個 worker 、3個 worker、3個 worker w/ 4 threads 的結果，跑完之後可以用指令`ps`看一下

- one worker

```bash
$ uwsgi --http-socket 0.0.0.0:5000 -w main:app
```

![Image](/assets/img/post/uWSGI_1w.png)

- one master with 3 workers

```bash
$ uwsgi --http-socket 0.0.0.0:5000 -w main:app --master --workers 3
```

![Image](/assets/img/post/uWSGI_1m3w.png)

```bash
$ uwsgi --http-socket 0.0.0.0:5000 -w main:app --master --workers 3 --threads 4
```

![Image](/assets/img/post/uWSGI_1m3w4t_1.png)
可以看到 process 一樣只有4個(1 master, 3 workers)

![Image](/assets/img/post/uWSGI_1m3w4t_2.png)
但每個 process 底下各有4個 threads

### **Gunicorn**

```bash
$ gunicorn -b :8787 -w 4 --threads 3 main:app
```

![Image](/assets/img/post/gunicorn_1.png)
可以看到 gunicorn 會自動多產生一個 process 當 master

![Image](/assets/img/post/gunicorn_2.png)
若有設定 thread 數量，則會顯示`Using worker: gthread`，不然一般為`Using worker: sync`

---

## _**References**_

- [uWSGI Options](https://uwsgi-docs.readthedocs.io/en/latest/Options.html)

- [uWSGI和Gunicorn对比实践笔记](https://zhuanlan.zhihu.com/p/50857407)

- [QA: uwsgi invalid request block size](https://stackoverflow.com/questions/15878176/uwsgi-invalid-request-block-size)

- [Gunicorn Settings](https://docs.gunicorn.org/en/stable/settings.html)
