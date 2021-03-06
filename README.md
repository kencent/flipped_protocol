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

# 获取腾讯云对象存储sig
如果fileid参数不为空，则生成一次性sig；否则生成7天有效sig，过期需要主动续期。
```
GET /youtusig?fileid=/1251789367/flipped/test/a.jpg
```

* 200 OK
```
{
	"sig": "xxxxxxxx"
}
```

* 示例
```
curl -k -H"x-uid:13410794959" -H"Authorization:SRP AehgVsne741NRvGGRrrIcktjZtf52/0gFOAlWkSLLAoz8X2XpGhJ1Ccez0e4YA78ZsYHrflvUoRp6odSSgTmnpqVFvbQNqhzl9KJRw52FyQW84AEEUfMK3xIEZ16Lc5D" "https://127.0.0.1/youtusig"
{"sig":"BgSRe92mhr8Qn82U4zXrVhy1GChhPTEyNTE3ODkzNjcmYj1mbGlwcGVkJms9QUtJREw4cE5seW51ZUZneDN5REpqeTF0ZEVpNEZIOG4yYWlmJmU9MTQ5NzA4MDIzNCZ0PTE0OTY0NzU0MzQmcj01MTcwNyZ1PTAmZj0="}
```

# 发表flippedwords
```
POST /flippedwords

{
	"sendto": "13410794959",
	"contents": [
		{"type": "text", "text": "bitch!", "link": "91porn.html"},
		{"type": "picture", "text": "qcloud/fuck", "size": {"width": 100, "height": 200}, " alt": {"type": "text"}},
		{"type": "video", "text": "youtube/dick", "size": {"width": 100, "height": 200}, "cover": "dick.jpg", alt": {"type": "text"}}
		{"type": "audio", "text": "qcloud/suck", "duration": 60, alt": {"type": "text"}}
	],
	"lat": 22.0000,
	"lng": 103.0000
}

# contents是content数组, content的结构为：
#	type: 类型，如text, picture, video。其中text是最基础的类型，所以其他类型为保证向下兼容性都会有一个alt元素，alt是一定是一个类型为text的content。
# 	text: 内容。对于text类型，为文本内容；对于picture类型，为图片地址；对于video类型，为视频地址；
# 	link: 该内容的超链接地址，所有内容均可能有超链接。
# 	size: size信息。对于picture类型，为图片大小；对于video类型，为未全屏播放器大小。
# 	cover: 对于video类型，为视频封面图片。
# 	duration: 对于audio类型，为语音时长。
```

* 200 OK
```
{
	"id": 172
}
```

* 429 Too Many Requests
```
{
	"err": "今天已经发过了，请明天再来！",
	"errcode": 0
}
```

* 示例
```
curl -v -X POST -H"x-uid:13410794959" -d '
{
	"sendto": "13410794959",
	"contents": [
		{"type": "text", "text": "bitch!"}
	],
	"lat": 22,
	"lng": 103
}' "http://127.0.0.1:8080/flippedwords"
```

# 查询flippedwords详情
```
GET /flippedwords/{id}
```

* 200 OK
```
{
	"id"： 754,
	"sendto": "132******27",
	"ctime": 1496479362123,
	"contents": [{"type: "text", "text": "I love you"}]
}
```

# 附近flippedwords
为保证看到的内容与自己相关，提供附近的flippedwords接口。

**注意此接口用户未授权位置时，不能传lat, lng参数**

**返回顺序是id从大到小** 

**接口目前最多返回200条，不支持翻页** 
```
GET /nearby_flippedwords?lat=22&lng=103
```

* 200 OK
```
{
	"flippedwords": [{
		"id"： 754,
		"sendto": "132******27",
		"ctime": 1496479362123,
		"distance": 65,
		"contents": [{"type: "text", "text": "I love you"}]
	},{
		"id"： 257,
		"sendto": "132******25",
		"ctime": 1496478362456,
		"contents": [{"type: "text", "text": "I wanna fuck you"}]
	}],
	"links" [{
		"rel": "previous",
		"method": "GET",
		"uri": "/nearby_flippedwords?lat=22&lng=103"
	}]
}
```

* 示例
```
curl -H"x-uid:13410794959" "http://127.0.0.1:8080/nearby_flippedwords?lat=22&lng=103"
{"flippedwords":[{"sendto":13410794959,"lat":22,"id":1000002,"contents":"[{\"text\":\"bitch3!\",\"type\":\"text\"}]","lng":103},{"sendto":13410794959,"lat":22,"id":1000001,"contents":"[{\"text\":\"bitch2!\",\"type\":\"text\"}]","lng":103},{"sendto":13410794959,"lat":22,"id":1000000,"contents":"[{\"text\":\"bitch!\",\"type\":\"text\"}]","lng":103}],"links":[{"rel":"previous","uri":"\/nearby_flippedwords?lng=103&lat=22","method":"GET"},{"rel":"next","uri":"\/nearby_flippedwords?lat=22&id=1000000&lng=103","method":"GET"}]}
```


# 查询发给我的flippedwords
客户端需要缓存发给我的flippedwords，如客户端请求参数中id=103，则服务器端认为id小于等于103的flippedwords客户端均已接收了，服务器端会更新发送给我的id小于等于103的flippedwords的状态（该状态后续有可能回显给发送人），并且将id大于103的flippedwords返回。

**返回顺序是id从小到大，客户端需从大到小展示数据，每次拿最大的id(或links.previous)向服务器查询数据** 

```
GET /my_flippedwords?id=103
```

* 200 OK
```
{
	"flippedwords": [{
		"id"： 104,
		"ctime": 1496479362234,
		"contents": [{"type: "text", "text": "I wanna fuck you"}]
	}, {
		"id"： 153,
		"ctime": 1496475362456,
		"contents": [{"type: "text", "text": "I love you"}]
	}],
	"links" [{
		"rel": "previous",
		"method": "GET",
		"uri": "/my_flippedwords?id=153"
	}]
}
```

* 示例
```
curl -H"x-uid:13410794959" "http://127.0.0.1:8080/my_flippedwords?id=1000001"
{"flippedwords":[{"sendto":"13410794959","lat":22,"id":1000002,"contents":"[{\"text\":\"bitch3!\",\"type\":\"text\"}]","lng":103}],"links":[{"rel":"previous","uri":"\/my_flippedwords?id=1000002","method":"GET"}]}
```

# 查询我发表的flippedwords
按发表时间从大到小排序，支持分页
```
GET /mypub_flippedwords
```

* 200 OK
```
{
	"flippedwords": [{
		"id"： 153,
		"ctime": 1496479362567,
		"sendto": "13210794925",
		"contents": [{"type: "text", "text": "I wanna fuck you"}],
		"status": 100
	}, {
		"id"： 104,
		"ctime": 1496475362678,
		"sendto": "13270244925",
		"contents": [{"type: "text", "text": "I love you"}],
		"status": 200
	}],
	"links" [{
		"rel": "previous",
		"method": "GET",
		"uri": "/mypub_flippedwords"
	},{
		"rel": "next",
		"method": "GET",
		"uri": "/mypub_flippedwords?id=104"
	}]
}

# status取值：
# 	0 新发表
# 	100 已下发给接收人客户端
# 	200 接收人已读
```

# 查询自上次打开应用，我发出去的，新的被对方已读的flippedwords
不分页，返回全部。
客户端缓存上次打开应用时的毫秒数
```
GET /read_flippedwords?last_open_time=1496475362678
```

* 200 OK
```
{
	"flippedwords": [{
		"id"： 153,
		"ctime": 1496479362567,
		"sendto": "13270244925",
		"contents": [{"type: "text", "text": "I wanna fuck you"}]
	}, {
		"id"： 104,
		"ctime": 1496475362678,
		"sendto": "13270244925",
		"contents": [{"type: "text", "text": "I love you"}]
	}]
}
```


# 反馈
```
POST /feedbacks

{
	"contents": [{}, {}]
}

# contents格式同发表flippedword（POST /flippedwords）
```

* 示例
```
curl -v -X POST -H"x-uid:13410794959" -d '
{
	"contents": [
		{"type": "text", "text": "bitch!"}
	],
	"lat": 22,
	"lng": 103
}' "http://127.0.0.1:8080/feedbacks"
```





