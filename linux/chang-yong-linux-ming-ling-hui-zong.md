# curl

## 发送get请求

* curl "[http://www.baidu.com](http://www.baidu.com)" 如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
* curl -i "[http://www.baidu.com](http://www.baidu.com)" 显示全部信息
* curl -l "[http://www.baidu.com](http://www.baidu.com)" 只显示头部信息
* curl -v "[http://www.baidu.com](http://www.baidu.com)" 显示get请求全过程解析

## 发送post请求

* curl -d "param1=value1¶m2=value2" "[http://www.baidu.com](http://www.baidu.com)"
* curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13521389587","password":"test"}' [http://domain/apis/users.json](http://domain/apis/users.json) // json格式的post请求

