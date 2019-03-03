## [A web server](http://docs.studygolang.com/doc/effective_go.html#web_server)

我们以一个web服务器程序作为结尾。其实它是服务器重用的一种。Google的`chart.apis.google.com`提供一种可以将数据自动转为表和图格式的服务。但是使用此服务有点不友好，是因为你需要将数据作为查询`URL`的一部分。此例子中为一种形式的数据提供了一个友好的交互接口：给定一个简短文本，程序就会调用表格服务生成一个二维码（QR码），二维码是一个文本编码的矩阵框。二维码可以被手机摄像头抓取并解析（比如为一个`URL`)，这样就可以为你免去在手机的小键盘上输入`URL`的麻烦。

下面是全部的代码，后续会进行讲解。

```Go

	package main

	import (
		"flag"
		"html/template"
		"log"
		"net/http"
	)

	var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

	// 1.template.New申请一个名为qr的html模板
	// 2.Parse解析模板{{}}
	var tmpl = template.Must(template.New("qr").Parse(templateStr))

	func main() {
		// 解析flag,设置端口：1718
		flag.Parse()
		// 将根路径绑定处理函数：QR
		http.Handle("/", http.HandlerFunc(QR))
		// 启动服务，一旦服务运行就会阻塞
		err := http.ListenAndServe(*addr, nil)
		if err != nil {
			log.Fatal("ListenAndServer:", err)
		}
	}

	// QR 接收请求，并执行模板引擎处理数据s
	func QR(w http.ResponseWriter, req *http.Request) {
		tmpl.Execute(w, req.FormValue("s"))
	}

	// 一个html模板字符串，(由于`chart.apis.google.com`需要越过`GFW`才能访问，
	// 所以进行了调整，使用了`api.qrserver.com`提供的创建二维的服务
	const templateStr = `
	<html>
		<head>
			<title>QR Link Generator</title>
		</head>
		<body>
			{{if .}}
			<img src="https://api.qrserver.com/v1/create-qr-code/?data={{.}}&size=100x100" alt="test" title="qrtest" />
			<!--
			<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}"/>
			-->
			<br>
			{{.}}
			<br>
			<br>
			{{end}}
			<form action="/" name=f method="GET">
				<input maxLength=1024 size=70 name=s value="" title="Text to QR Encode">
				<input type=submit value="Show QR" name=qr>
			</form>
		</body>
	</html>
	`
```

> 备注：运行上述代码，使用`go run`即可，然后访问`http://localhost;1718/s=XXX`

在`main`函数上面的代码应该是十分易懂的。`flag`设置HTTP服务默认端口号；`templ`是模板变量，正是有趣之处，它构建一个`HTML`模。`HTML`模板会被服务解析展示出一个页面，下面进行更为详细的说明。

`main`函数解析`flag`,然后使用我们上面讲到的机制：将`QR`函数绑定到服务的根路径。接着调用`http.ListenAndServe`，它会启动服务，一旦启动就会阻塞，直到接收到请求。

`QR`函数（译者注：真正的处理请求和响应幕后着）接收请求，请求中包含了表单数据，然后执行处理名为`s`的表单数据。

`html/template`模板包是很强大的；此程序只是涉及其功能的冰山一角。本质上是在运行时，它重写了部分`HTML`文本，它是通过替换来自传递给`templ.Execute`数据项，此例子中是表单数据。在模板文本（`templateStr`)，以`{{}}`界定是模板动作。在`{{ .}}`与`{{end}}`之间的代码只有当前的数据，也就是`.`（点）不为空才会执行。也就是说，若字符串是空得，模板部分会被压缩（就是空的）。

其中`{{.}}`展示是web页面的模板查询字符串。`HTML`模板包会自动进行适当的转义，因此文本会安全的呈现出来。

余下的模板文本仅仅是页面载入的`HTML`，若这里讲解的过于匆忙，可以查阅[template 文档](http://docs.studygolang.com/pkg/html/template/) 获得更为详细的讨论。

到此，你有一个有用的web服务，它由简单几行代码加上数据驱动的`HTML`文本构成。`Go`是足够强大的，仅仅用几行代码就可以做出很多东西。