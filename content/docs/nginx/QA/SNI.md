# 后端是一个IP对多个站点的情况
nginx的后端：通过一个ip提供多个站点服务

1. 502 bad gateway问题
	1. 解决方法：启用proxy_ssl_server_name
		```
		 # 设置 SNI，确保后端服务器返回正确的证书
		 proxy_ssl_name backend.example.com;
		 proxy_ssl_server_name on;
		```
2. 设置proxy_set_header出现404问题
	1. 解决方法：
		删除这个参数，如果你**没有**显式设置 `proxy_set_header Host`，Nginx 在代理请求时会自动使用 `proxy_pass` 目标的主机名（即 `$proxy_host`）作为 `Host` 头。
	2. $host取的值是客户端请求带的Host值，如果客户端不带，则取nginx中server_name的值。而因为后端是多个站点，后端可能找不到对应Host值的站点，因此报404，400等。
	3. $proxy_host取的值是proxy_pass的域名或IP或变量。
		```
		# 传递 Host 头，确保后端识别正确的站点。
		# 删除则使用$proxy_host作为Host头传给后端。
		proxy_set_header Host $host;
		```

3. 如贵出现the referrer does not match the host问题
	1. 解决方法：
		添加这个参数：`proxy_set_header Referer https://example.com$request_uri;`
	2. $request_uri 变量在 Nginx 中表示 完整的请求 URI（包含路径和查询参数，但不包含域名和协议）。它的值来源于 客户端请求的原始 URI，不会进行 URL 解码。
	3. 例：
		```
		请求: GET https://example.com/path/to/page?user=123&action=view

  		$request_uri = "/path/to/page?user=123&action=view"
  		```
