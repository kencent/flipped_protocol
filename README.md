# 接口规范
任何业务接口, 后台发现K不存在(过期也是不存在), 则返回401, 并在头部中设置:
`WWW-Authenticate: SRP realm="flipped", N_num_bits="2048"`

任何业务接口, 前端需使用srp.K加密(phone + local miniseconds + seq + client(platform, version) + random)得到auth-param, 并设置到头部:
Authorization: SRP auth-param

任何业务接口，需要设置头部用户标识
x-uid: phone


所有4xx 5xx返回的错误信息在body里面, 如:
```
{
	"err": "获取验证码次数过多",
	"errcode": 0
}
```


# 获取验证码
```
GET /srp/password
```

* 200 OK
```
{
	"N_num_bits": "2048",
	"s": "xx",
	"countdown": 60,
	"validtime": 300,
}

# N_num_bits: srp算法中使用多少位的N
# s: salt
# countdown: 倒计时，xx秒内不能重新获取
# validtime: xx秒内验证码有效
```

* 429 Too Many Requests
```
{
	"err": "获取验证码次数过多，请稍后重试",
	"errcode": 0
}
```



# 获取srp.B
```
GET /srp/B?A=xx
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
GET /srp/M2?M1=xx
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
	"err": "验证码错误，还有2次机会",
	"errcode": 0
}

在由于key过期，重新生成key的流程中，如果收到验证码错误的返回，应该直接引导用户重新获取验证码
在获取验证码后，尝试用验证码登录的流程中，如果收到验证码错误的返回，则应该给予用户重新输入验证码的机会，后台最多允许3次，达到3次即返回429
```

* 429 Too Many Requests
```
{
	"err": "输入错误的验证码达到3次，请重新获取验证码",
	"errcode": 0
}
```










