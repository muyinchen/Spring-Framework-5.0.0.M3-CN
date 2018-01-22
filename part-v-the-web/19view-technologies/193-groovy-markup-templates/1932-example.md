### 19.3.2例子

和传统模板引擎不同, 这一个依赖于使用构建器语法的DSL。 以下是HTML页面的示例模板:

```js
yieldUnescaped '<!DOCTYPE html>'
html(lang:'en') {
    head {
        meta('http-equiv':'"Content-Type" content="text/html; charset=utf-8"')
        title('My page')
    }
    body {
        p('This is an example of HTML contents')
    }
}
```



