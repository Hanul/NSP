# Node Server Pages
Node Server Pages is a server-side script to create dynamic web pages based on Node.js.

***Node Server Pages is NOT a template engine!***

## Features
- You can instantly start developing server programming with basic knowledge of JavaScript.
- You can use all the [APIs of Node.js](https://nodejs.org/api/).
- You can use hundreds of [npm modules](https://www.npmjs.com/) out-of-box.
- You can use [COMMON](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-COMMON.md) and [NODE](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-NODE.md) of [UPPERCASE.JS](https://github.com/Hanul/UPPERCASE.JS).
- Pages change instantly once you modify `.nsp` pages.
- Built-in support for multi-core CPUs.

## Install
1. Copy `NSP.js`, `import` folder and `config.json` to wherever you want.
2. Modify `config.json`.
```json
{
	"port": 8080,
	"isDevMode": true,
	"rootPath": "./",
    "uploadURI": ["examples/upload_result.nsp"],
    "isNotUsingDCBN" : false
}
```
- `port` is the port of the web server.
- `idDevMode` `true` means development mode. In development mode, resources such as images are not cached.
- `rootPath` specified the root path where `.nsp` files or resources are saved.
- `uploadURI` points to the `URI` to upload to.
- `isNotUsingDCBN` if set to `true`, {{, }} (double curly braces notation) is not allowed.

## Run
```
node NSP.js
```
Now type `http://localhost:8123`, `http://localhost:8123/index.nsp` or `http://localhost:8123/index` in your browser. It's okay to emit `.nsp`.

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

### `{{`, `}}`, `<%=`, `%>`
Prints formatted expressions, e.g., `{{expression}}` or `<%=expression %>`. It is equivalent to `<% print(expression); %>`.

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

### save/load
Saves or loads a variable. Loaded/saved variables are shared everywhere in the NSP server.

```nsp
<%
	save('sample', 'this is an example.');
	load('sample'); // this is an example.
	save('sample', undefined); // Saving 'undefined' deletes the variable.
%>
```

### pause/resume
Document processing is paused for a while in `callback`s with `pause` function when you deal with database, etc.

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

### cookie
Saves or loads a cookie.

```nsp
<%
	// Saves a cookie
	cookie('sample-cookie', 'this is example.');
	cookie('sample-cookie', 'this is example.', 10); // Expired in 10 seconds
	cookie('sample-cookie', 'this is example.', 10, '/'); // Sets path to '/'
	cookie('sample-cookie', 'this is example.', 10, '/', 'www.example'); // Sets domain to www.example
	
	// Loads a cookie
	cookie('sample-cookie');
%>
```

#### Sessions example using `cookie`
An example to implement sessions using `cookie` and `save/load`.

```nsp
<%
	var sessionKey = cookie('session-key');
	
	print(sessionKey);
	
	if (sessionKey === undefined) {
		sessionKey = RANDOM_STR(20);
	}
	
	cookie('session-key', sessionKey, 3600);
%>
<p>{{load(sessionKey)}}</p>
<%
	save(sessionKey, {
		name : 'YJ Sim',
		age : 28
	});
%>
```
The next step could be saving sessions to a file or a database.

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

### escape
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

## Built-in Variables
### self.headers
Holds requested HTTP headers.

### self.method
Holds requested HTTP Method. `ex) GET, POST`

### self.ip
Holds requested IP address in String form. `ex) 127.0.0.1`

### self.params
Holds data passed from `form`s, etc.

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

If you passed `Sam` as `fname` and `Ple` as `lname`, `self.params` should be `{"fname":"Sam","lname":"Ple"}`.

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

## License
[MIT](../../LICENSE)

## Author
[Young Jae Sim](https://github.com/Hanul)