---
layout: post
title: Flask introduction & set up
date: 2024-05-21
description: "Flask framework learning"
tag: Flask - Python
---

## Set up env (virtual environment)

[tutorial](https://tutorial.helloflask.com/)

```shell
$ python -m venv env  # Windows

$ python3 -m venv env  # Linux or macOS
```

In python, there is no central lib management tools like Maven in Java, and it is not possible to have different version of same lib, so we need to create a env to manage the lib we use in this project

now we created the env, we need to activate it

```shell
$ source env/bin/activate  # Linux or macOS

$ .\env\Scripts\activate  # Windows
```

if you successfully activated the env, you will see the env name in the terminal

```shell
(env) $
```

if you want to deactivate the env, just type `deactivate` in the terminal

## Install Flask

```shell
$ pip install Flask

$ pip install flask==2.1.3 # install specific version
```

## Create a simple Flask app

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

save the code above to a file named `app.py`, and run the app

```shell
$ export FLASK_APP=app.py  # Linux or macOS
> $env:FLASK_APP = "hello.py"  # Windows

$ flask run
```
