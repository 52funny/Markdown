分享一些查常用的API接口信息

### Bilibili获取ip

`GET: http://api.bilibili.com/x/web-interface/zone`

+ 返回类型 `json`

+ 返回值

```json
{
    "code": 0,
    "message": "0",
    "ttl": 1,
    "data": {
        "addr": "120.243.220.168",
        "country": "中国",
        "province": "安徽",
        "city": "宣城",
        "isp": "移动",
        "latitude": 30.941999,
        "longitude": 118.746819,
        "zone_id": 4407456,
        "country_code": 86
    }
}
```
返回值`data.addr`就是ip信息




