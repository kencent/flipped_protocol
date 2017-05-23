# 接口规范
任何业务接口, 后台发现K不存在(过期也是不存在), 则返回401, 并在头部中设置:
`WWW-Authenticate: SRP N_num_bits="2048"`

任何业务接口, 前端需使用srp.K加密(phone + local miniseconds + seq + client(platform, version) + random)得到auth-param, 并设置到头部:
Authorization: SRP auth-param


所有4xx 5xx返回的错误信息在body里面, 如:
```
{
	"err": "获取验证码次数过多",
	"errcode": 0
}
```


# 获取验证码
```
GET /srp/password?phone=xx
```

* 200 OK
```
{
	"N_num_bits": "2048"
	"s": "xx"
}
```

- N_num_bits: srp算法中使用多少位的N
- s: salt

```


# 获取srp.B
```
GET /srp/B?phone=xx&A=xx
```

* 200 OK
```
{
	"B": "xx"
}
```

* 403 Forbidden
```
{
	"err": "非法请求",
	"errcode": 0
}
```



# 检查验证码是否正确, 请求srp.M2
```
GET /srp/M2?phone=xx&M1=xx
```

* 200 OK
```
{
	"M2": "xx"
}
```

* 422 Unprocessable Entity
```
{
	"err": "M1值非法",
	"errcode": 0
}
```











