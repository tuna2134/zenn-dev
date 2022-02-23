---
title: "アプリケーション"
---

## アプリケーションを定義しよう

```python
from sanic import Sanic

app = Sanic("app")
```

## ハローワールドを表示するコードを書こう

では早速hello worldを表示するコードを書こう

```python
from sanic import Sanic
from sanic.response import text

app = Sanic("app")

@app.route("/")
async def main(request):
    return text("hello world")
```

## 実行しよう

最後の行にこれを追加してください。
```python
app.run(host="0.0.0.0", port=80)
```

そのあと
```shell
python3 main.py
```
を実行したあと http://localhost:80/ にアクセスすると表示されると思います。
