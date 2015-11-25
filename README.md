# Node Server Pages
Node Server Pages는 Node.js를 기반으로 동적 웹페이지를 생성하기 위해 개발된 서버 스크립트 엔진입니다.

## 특징
- 멀티코어 CPU를 지원합니다.
- [Node.js의 API](https://nodejs.org/api/)를 모두 사용할 수 있습니다.
- [UPPERCASE.JS](https://github.com/Hanul/UPPERCASE.JS)의 [COMMON](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-COMMON.md)과 [NODE](https://github.com/Hanul/UPPERCASE.JS/blob/master/DOC/UPPERCASE.JS-NODE.md)를 사용할 수 있습니다.

## 설치
1. `NSP.js`와 `import` 폴더, `config.json`을 원하는 폴더에 복사합니다.
2. `config.json`을 수정해서 사용합니다.
```json
{
	"port": 8080,
	"isDevMode": true,
	"rootPath": "./"
}
```

## 실행
```
node NSP.js
```

## 코드 예제
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

## 내장 함수
### print
`print`를 사용하여 `HTML` 문서에 내용을 추가합니다.
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
결과
```html

<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		Hello World!
	</body>
</html>
```

JSON 객체도 출력할 수 있습니다. 이를 통해 JSON 기반 API를 만들 수 있습니다.
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
결과
```json
{"a":1,"b":2,"c":3}
```

### include
`include` 함수로 다른 `.nsp` 파일을 포함할 수 있습니다.
- `var` 키워드로 변수를 등록한 경우, 해당 페이지에서만 변수를 사용할 수 있습니다.
- `include` 등으로 여러 페이지에서 변수를 공유하는 경우에는, 현재 요청에서 해석중인 페이지들이 공유하는 `self` 객체에 값을 대입하여 사용할 수 있습니다.

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

`include` 함수의 두번째 파라미터에 함수를 넣으면, 포함할 파일의 내용을 불러온 이후에 처리할 로직을 작성할 수 있습니다. `include` 함수가 포함된 구문이 끝날 때 까지는 파일 내용을 불러오지 않음에 유의하시기 바랍니다.

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

`.js` 파일 또한 포함할 수 있어, 하나의 `.js` 파일을 서버 측과 클라이언트 측에서 동시에 사용하는 등의 개발이 가능합니다.

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
데이터베이스 등을 조작하다 `callback` 처리로 들어갈 경우 `pause` 함수로 문서 해석을 잠시 중단할 수 있습니다. `resume` 함수로 문서 해석을 다시 진행할 수 있습니다.
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

## escape
`<%`나 `{{` 앞에 역슬래시(\\)를 붙히면 해당 구문은 해석하지 않습니다. 또한 `<%` 나 `{{` 앞에 역슬래시를 두개(\\\\) 붙히면 하나의 역슬래시로 판단하고, 코드 구문을 해석합니다.
```nsp
<!DOCTYPE html>
<html>
	<body>
		<h1>My first NSP page</h1>
		
		\<%
			// 이 구문을 해석하지 않음
			var msg = 'Hello World!';
		%>
		\{{msg}}
		
		\\<%
			// 이 구문은 해석됨
			var msg = 'Hello World!';
		%>
		\\{{msg}}
	</body>
</html>
```

## self.params
`self` 객체에는 `params` 라는 서브 객체가 존재합니다. 이는 HTML `form` 등에서 넘어온 데이터를 담고 있습니다.

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

`fname`을 `Sam`, `lname`을 `Ple`로 지정하면 `self.params`는 `{"fname":"Sam","lname":"Ple"}`가 됩니다.

## 기타
### PHP에 익숙한 개발자세요?
[php.js](https://github.com/kvz/phpjs)를 설치하여 NSP와 함께 사용해보세요.
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

### 벤치마크
아래와 같이 간단한 두 페이지를 대상으로 1000번의 호출을 수행해 보았습니다.

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

#### 시스템 사양
- Intel Core i7-4500U CPU 1.8GHz
- 8GB Ram
- Windows 10

단순 비교 결과 위 사양에서 아래와 같이 NSP가 PHP 대비 약 두배정도 성능이 높습니다. 이는 멀티코어를 지원하지 않는 PHP 대비 NSP가 멀티코어를 지원하기 때문으로 보입니다.

#### NSP 결과
```
Max requests:        1000
Concurrency level:   1
Agent:               none
Completed requests:  1000
Total errors:        0
Total time:          93.007305635 s
Requests per second: 11
Total time:          93.007305635 s
Percentage of the requests served within a certain time
  50%      91 ms
  90%      94 ms
  95%      98 ms
  99%      113 ms
 100%      126 ms (longest request)
```

#### PHP 결과

## 라이센스
[MIT](../../LICENSE)

## 작성자
[Young Jae Sim](https://github.com/Hanul)