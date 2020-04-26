# 快速静态html编写

- 生成html模板：！+Enter键
- 下级元素：div>p
- 同级元素：div>p+a
- 多个元素：div>p*3
- 带内容：div{hello}
- 带序号：div{hello$}*5
- 带3位数的序号：div{hello$$$}*6
- 序号带初始值：div{hello$$@3}*3
- 类：div.view
- id：div#view
- 属性：a[href=# id=aa class=view]

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <!-- div>nav>p -->
    <div>
        <nav>
            <p></p>
        </nav>
    </div>
    <!-- div>p+a -->
    <div>
        <p></p>
        <a href=""></a>
    </div>
    <!-- div>nav>p*3 -->
    <div>
        <nav>
            <p></p>
            <p></p>
            <p></p>
        </nav>
    </div>
    <!-- div>(nav>p*2)*2 -->
    <div>
        <nav>
            <p></p>
            <p></p>
        </nav>
        <nav>
            <p></p>
            <p></p>
        </nav>
    </div>
    <!-- div>p{hello} -->
    <div>
        <p>Hello</p>
    </div>
    <!-- div{$}*5 -->
    <div>1</div>
    <div>2</div>
    <div>3</div>
    <div>4</div>
    <div>5</div>
    <!-- div{Hello$}*3 -->
    <div>hello1</div>
    <div>hello2</div>
    <div>hello3</div>
    <!-- div{hello$$}*5 -->
    <div>hello01</div>
    <div>hello02</div>
    <div>hello03</div>
    <div>hello04</div>
    <div>hello05</div>
    <!-- div{hello$$@3}*3 -->
    <div>hello03</div>
    <div>hello04</div>
    <div>hello05</div>
    <!-- #view -->
    <div id="view"></div>
    <!-- .view -->
    <div class="view"></div>
    <!-- a[href=# id=aa class=bg] -->
    <a href="#" id="aa" class="bg"></a>
</body>
</html>
```

