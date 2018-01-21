### 19.3.2Example

Unlike traditional template engines, this one relies on a DSL that uses the builder syntax. Here is a sample template for an HTML page:

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



