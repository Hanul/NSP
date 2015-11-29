# Node Server Pages
Node Server Pages is a server-side script to create dynamic web pages based on Node.js.

***Node Server Pages is NOT a template engine!***

## Features
- You can instantly start developing server programming with basic knowledge of JavaScript language.
- You can use all the [APIs of Node.js](https://nodejs.org/api/).
- You can use hundreds of [npm modules](https://www.npmjs.com/) out-of-box.
- You can use [COMMON](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-COMMON.md) and [NODE](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-NODE.md) of [UPPERCASE.JS](https://github.com/Hanul/UPPERCASE.JS).
- Pages change instantly once you modify `.nsp` codes.
- Built-in support for multi-core CPUs.

## Install
1. Copy `NSP.js`, `import` folder and `config.json` to wherever you want.
2. Modify `config.json`.
```json
{
	"port": 8080,
	"isDevMode": true,
	"rootPath": "./",
    "uploadURI": ["examples/upload_result.nsp"]
}
```
- `port` is the port of the web server.
- `idDevMode` `true` means development mode. In development mode, resources such as images are not cached.
- `rootPath` specified the root path where `.nsp` files or resources are saved.
- `uploadURI` points to the `URI` to upload to.

## Run
```
node NSP.js
```

## Examples
```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		<%
			var msg = 'Hello World!';
		%>
		<p>{{msg}}</p>
	</body>
</html>
```

## Syntax
### `<%`, `%>`
Allows server-side JavaScript codes to be embedded.

### `{{`, `}}`
Prints formatted expressions, e.g., `{{expression}}`. It is equivalent to `<% print(expression); %>`.

### `<? expression>`, `</?>`
Interprets statements only if `expression` is `true`.

```nsp
<%
    var a = true, b = false;
%>
<? a>
    I'm printed.
</?>
<? b>
    I'm not printed.
</?>
```

### `<~ array -> value>`, `</~>`
Iterates over an array.

```nsp
<%
    var arr = [1, 2];
%>
<~ arr -> v>
    {{v}}
</~>
```

### `<~ data -> value>`, `<~ data -> name: value>`, `</~>`
Iterates over an object.

```nsp
<%
    var data = {
        a : 1,
        b : 2
    };
%>
<~ data -> n: v>
    {{n}} : {{v}}
</~>
```

## Built-in functions
### print
Add contents to the `HTML` document using `print` function.

```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		<%
			print('Hello World!');
		%>
	</body>
</html>
```
Output

```html
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		Hello World!
	</body>
</html>
```

You can also print out JSON objects to make JSON-based APIs.

```nsp
<%
	var data = {
		a : 1,
		b : 2,
		c : 3
	};
	
	print(data); // or {{data}}
%>
```
Output

```json
{"a":1,"b":2,"c":3}
```

### include
You can have another `.nsp` file in the page using `include` function.

- If a variable was registered using `var` keyword, it is only visible in current page.
- If a variable is used in multiple pages with `include`, you can use it by assigning values to `self` that are shared between pages in current request.

`include.nsp`

```nsp
<!DOCTYPE html>
<html>
	<body>
		<% include('include/top.nsp'); %>
		<h1>My first NSP page</h1>
		<% include('include/bottom.nsp'); %>
	</body>
</html>
```

`include/top.nsp`

```nsp
<%
	var local = 'Welcome!';
	
	self.msg = 'Hello World! ' + local;
%>
```

`include/bottom.nsp`
```nsp
<p>{{self.msg}}</p>
```

If you give another function as the second argument of `include` function, you can set the action to be run after the inclusion finishes. Note that the inclusion doesn't start until the sentences that contain `include` function ends.

```nsp
<!DOCTYPE html>
<html>
	<body>
		<%
    		include('include/top.nsp', function() {
    		    console.log(self.msg); // Hello World! Welcome!
    		});
    		
    		console.log(self.msg); // undefined
		%>
		<%
		    console.log(self.msg); // Hello World! Welcome!
		%>
	</body>
</html>
```

You can also contain `.js` file to reuse one `.js` file for both server-side and client-side.

```nsp
<!DOCTYPE html>
<html>
	<body>
		<% include('include/common.js'); %>
		<script src="include/common.js"></script>
	</body>
</html>
```

### pause/resume
You can pause document processing for a while in `callback`s with `pause` function when you deal with database, etc.

```nsp
start
<%
	setTimeout(function() {
	
		print('ok');
		
		resume();
	}, 1000);
	
	pause();
%>
end
```

### upload
Uploads a file to the `URI` that's specified by `uploadURI` in `config.json`.

```nsp
<%
    var paramsList;
    
    // Uploaded file is saved in 'upload_temp_files' folder.
	upload('upload_temp_files', function(_paramsList) {
		paramsList = _paramsList;
		resume();
	});
	
	pause();
%>
<~ paramsList -> params>
	<p>{{params}}</p>
</~>
```

## escape
A statement with backslash(\\) in front of `<%` or `{{` is NOT interpreted. However, since two backslashes(\\\\) in front of `<%` or `{{` is recognized as a backslash, that statement is interpreted.

```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		
		\<%
			// This statement is NOT interpreted.
			var msg = 'Hello World!';
		%>
		\{{msg}}
		
		\\<%
			// This statement is interpreted.
			var msg = 'Hello World!';
		%>
		\\{{msg}}
	</body>
</html>
```

## self.params
`self` object has a sub object called `params`. This object has data passed from `form`s, etc.

`params.nsp`

```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>Params Example</h1>
		<form action="params_result.nsp" method="POST">
			First name: <input type="text" name="fname"><br>
			Last name: <input type="text" name="lname"><br>
			<input type="submit" value="Submit">
		</form>
	</body>
</html>
```

`params_result.nsp`

```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>Params Example</h1>
		<p>
			{{self.params}}
		</p>
		<a href="params.nsp">Back</a>
	</body>
</html>
```

If you typed `Sam` to `fname` and `Ple` to `lname`, `self.params` should be `{"fname":"Sam","lname":"Ple"}`.

## Etc.
### Are you familiar with PHP?
Install [php.js](https://github.com/kvz/phpjs) and use it together with NSP.
```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>NSP + php.js</h1>
		<%
			var php = require('phpjs');
			
			print(php.sprintf('Hey, %s : )', 'you'));
			print(php.parse_url('mysql://kevin:abcd1234@example.com/databasename')['pass']);
			print(php.strtotime('2 januari 2012, 11:12:13 GMT'));
		%>
	</body>
</html>
```

### Benchmark
Benchmarekd for two simple pages below.
```
loadtest -n 100000 -c 100 <url>
```

`hello.nsp`
```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		<%
			var msg = 'Hello World!';
		%>
		<p>{{msg}}</p>
	</body>
</html>
```

`hello.php`
```php
<!DOCTYPE html>
<html>
	<body>
		<h1>My first PHP page</h1>
		<?
			$msg = 'Hello World!';
		?>
		<p><?=$msg ?></p>
	</body>
</html>
```

#### System Spec
- Intel Core i7-4500U CPU 1.8GHz
- 8GB Ram
- Windows 10

#### NSP Result
```
Target URL:          http://127.0.0.1:8123/examples/hello.nsp
Max requests:        100000
Concurrency level:   100
Agent:               none

Completed requests:  100000
Total errors:        0
Total time:          112.43470760199999 s
Requests per second: 889
Total time:          112.43470760199999 s

Percentage of the requests served within a certain time
  50%      109 ms
  90%      126 ms
  95%      136 ms
  99%      166 ms
 100%      398 ms (longest request)
```

#### PHP Result
```
Target URL:          http://127.0.0.1/hello.php
Max requests:        100000
Concurrency level:   100
Agent:               none

Completed requests:  100000
Total errors:        0
Total time:          118.630713735 s
Requests per second: 843
Total time:          118.630713735 s

Percentage of the requests served within a certain time
  50%      115 ms
  90%      130 ms
  95%      141 ms
  99%      157 ms
 100%      301 ms (longest request)
```

## License
[MIT](../../LICENSE)

## Author
[Young Jae Sim](https://github.com/Hanul)