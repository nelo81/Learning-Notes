## OAuth2 介绍

- 基本组成元素

  1. Resource owner (the user): 拥有需要访问资源的用户
  2. Resource server (the API): 拥有需要访问资源的服务器
  3. Authorization server (can be the same server as the API): 授权服务器
  4. Client (the third-party app): 第三方 app

- 授权方式(获取 access_token)

	1. Authorization code: Client 发送带有 client_id 的请求到 Authorization server，例如
	```
	https://authorization-server.com/oauth/authorize
	?client_id=a17c21ed
	&response_type=code
	&state=5ca75bd30
	&redirect_uri=https://example-app.com/cb
	```
	之后会跳转到让 user 登录的界面，登录成功并点击允许后，会跳转到上面请求中的 redirect_uri，redirect_uri 后面会带有一个新的参数，就是 authorization code，例如
	`https://example-app.com/cb?code=Yzk5ZDczMzRlNDEwY`
	然后我们就可以在 `https://example-app.com/cb` 这个接口处理这个参数，就是发送一个带这个 code 的请求获取 access_token，例如
	```
	POST /oauth/token HTTP/1.1
	Host: authorization-server.com

	code=Yzk5ZDczMzRlNDEwY
	&grant_type=code
	&redirect_uri=https://example-app.com/cb
	&client_id=mRkZGFjM
	&client_secret=ZGVmMjMz
	```


	2. Password Grant: Client 发送带有 client_id, client_secret, user_id, password 的请求到 Authorization server，直接获取 access_token，例如
	```
	POST /oauth/token HTTP/1.1
	Host: authorization-server.com
	
	grant_type=password
	&username=user@example.com
	&password=1234luggage
	&client_id=xxxxxxxxxx
	&client_secret=xxxxxxxxxx
	```


	3. Client Credentials: Client 发送带有 client_id 和 client_secret 的请求到 Authorization server，直接获取到 access_token，例如
	```
	POST /token HTTP/1.1
	Host: authorization-server.com
	
	grant_type=client_credentials
	&client_id=xxxxxxxxxx
	&client_secret=xxxxxxxxxx
	```

- 三种方式的使用场景

	1. 先说 Authorization code 方法，它相比其他两个方法的优点，它不需要知道 client_secret，因此在第三方单页应用，即只使用 html, css, js 文件编写的 app，这种方法有效防止用户可以从源码中直接获取到 client_secret。

	2. Client Credentials 方法需要使用到 client_secret，因此适用于源码存放在服务器，且不能被下载的情况，但是这个方法缺点很明显，由于不需要其他用户的登录，因此只能访问到创建这个 client_id 用户的信息，因此不适用于用户查看，操作自己信息的情况，而适用于官方信息。

	3. Password Grant 也有 client_secret 的问题，但是需要用户登录，弥补了 Client Credentials 的一个缺点，因此适用于源码存放在服务器，且不能被下载，并且用户又要获取用户自己信息的情况。