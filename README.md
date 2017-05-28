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

* 示例
```
curl -H"x-uid:13410794959" "http://127.0.0.1:8082/srp/password"
{"s":"0344e93fcdaa111dc22d7ab3387abf3c","validtime":3000,"countdown":60,"password":"001024","N_num_bits":"2048"}
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

* 示例
```
curl -H"x-uid:13410794959" "http://127.0.0.1:8082/srp/B?A=a54129cec61dbafc08ecb96296c3ffe9e064fd6638b98d3588ab9806e7af23d063d860d6a0e0d4a3cd510df869b6b753e07fb542fc0c16d0704b35f7dfa5370b858d249f4ef34ff2305bc8446abb0c02e279236eed8fc55d06d38977610f3a633536dbd0ef6f2c52fac5fd4e706dec8911c3fb1850fb5929e2a24a80526ead5a9c096d66be3cfd4f6db917bacf29612ac63f81bd6f39fb07c18559bc970cc130f6634c573f8ae058fc69800d772a1a41cc100f691f1e6bf38b0392c4ff03f250d770d7393e450aefe87841e478cb4fac46a6b6e706309204981c23836bb75954619ba927c5cdfecbf5c3c213af82c2e8e6ecacfcf6ef6f65eacd6d5a5c5cd216"
{"B":"4560eb8eb1eb2aad79866eac6488e56e8ec7c0c9a7ba0a1bf24c6cf7e0a0008c8ec78da4d4e060130eea8ec0772808b91e6cc71f6a39cbe45b6bcf2b3739c99167294cead175f30b200783ac51f03b640056bbb9a2a9660a7fa96b683a68afc777bae36b6d0efd676b4c5db6e18acaff78e62e3117a5c22da84a2574b11e589aafff31a9283c9e99d2a0cfc5a7af88d389ca89dd6c718e6670034752dd0abdba5d68efc3dffb123ddf1bb4f1958a7179cd3953bd5b2c703db02e18e3d34a14da616f1d334e0afbd29ba422acf860e9eb406639e97ffdde7f8628d4e5a08b845c2d404931f89bf7ea7aa418afaa8c9cd4fe2fd30a8a7b3dba4940a3cb2d492015"}
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

* 示例
```
curl -H"x-uid:13410794959" "http://127.0.0.1:8082/srp/M2?M1=a8d2d9e2b18b80388835d354c575661f742351e9"
{"M2":"be1c3b0ea3fa50331e844ead7e32fdb16b570704"}
```









