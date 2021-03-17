---
title: Python学习笔记
date: 2020-11-14 22:49:38
tags:
---
# Python学习笔记——汇率查询
## 前言
之前一直在断断续续的学习python，看过视频，买过书。一直都是按照书上的内容一步一步照着学，结果就是学了之后，过几天不用就又忘记了。要能记住还是要通过自己思考的项目来应用和练习。最近看到了火币网的汇率API接口，准备做一个获取接口汇率数据，进行查询汇率的程序。

## 项目构思
火币的API接口 (https://api.huobi.pro/general/exchange_rate/list) 接口返回数据如下，是一个字典，里面有4个键。其中第二个键’data‘的值是一个包含了各个货币对的汇率的列表，而列表中的各个元素又是一个个字典，每个字典中有四个键值对，包含了当前时间，币对，汇率，汇率数据时间。

API接口返回数据
```json
    {
        "code": 200,
        "data": [
            {
                "data_time": 1605110400000,
                "name": "usd_krw",
                "rate": 1114.5081695677,
                "time": 1605114001194
            },
            --snip--
            {
                "data_time": 1605199140250,
                "name": "btc_cny",
                "rate": 104236.9900000000,
                "time": 1605199140250
            }
        ],
        "message": null,
        "success": true
    }
```
这个项目将基于火币接口获得的汇率数据做一个查询汇率的程序。
整体思路是先通过API接口获取到汇率数据，将汇率保存到一份本地文件中（汇率获取代码），然后在做一个基于读取本地文件数据的汇率查询程序（汇率查询代码）。


## 项目代码

### 汇率获取代码

``` python
import requests
import json

url = 'https://api.huobi.pro/general/exchange_rate/list'
r = requests.get(url)

#显示接口请求状态码
print('status code:', r.status_code)

#将API相应存储在一个变量中
response_dict = r.json()

#将汇率相关的汇率的数据提取出来
data_dicts = response_dict['data']

#分别创建一个币对和汇率的空列表
coinpairs = []
pairrates = []

#将字典中的币对和汇率值分别添加到列表中
for pairs in data_dicts:
    p = pairs['name']
    coinpairs.append(p)
    r = pairs['rate']
    pairrates.append(r)

#将两个列表合并为一个字典，币对为键汇率为值    
rate_dict = dict(zip(coinpairs,pairrates))

#创建一个json文件，将创建的汇率字典存储下来
filename = 'huobi_rates.json'
with open (filename,'w') as f:
    json.dump(rate_dict,f)
```
保存在本地的json文件内容

```json
{
    "usd_krw": 1109.7649251677,
    "usd_jpy": 104.6367043165,
    "usd_gbp": 0.7600429367,
    "usd_eur": 0.8460492255,
    "usd_rub": 77.6297586519,
    "usd_vnd": 23183.8570245314,
    "usd_inr": 74.6111939859,
    "usd_sgd": 1.3487038942,
    "usd_hkd": 7.7533786713,
    
    "cny_ils": 0.5098594911,
    "cny_kzt": 65.1120246007,
    "cny_mad": 1.3844687037,
    "cny_mxn": 3.1088269596,
    "cny_nad": 2.3616427521,
    "cny_qar": 0.5509530543,
    "cny_uyu": 6.4979960933,
    "cny_uzs": 1570.9292623929,
    "usdt_cny": 6.52,
    "btc_cny": 105049.43
}    
```

### 汇率查询代码

```python
import json

#While True打头的循环可以不断运行
while True:
    #输入要查询币对的基础货币或者退出程序
    basecoin = input('请输入基础货币,退出输入quit:')

    #判断是否退出程序
    if basecoin == 'quit':
        break

    #输入币对的计价货币
    quotecoin = input ('请输入计价货币:')

    #将用户的输入拼成币对，便于查询
    get_pair = basecoin + "_" + quotecoin

    #打开之前写下的文件，将内容存储到变量中
    filename = 'huobi_rates.json'
    with open (filename) as f:
        rate_dict = json.load(f)

    #判断币对是否存在
    if get_pair in rate_dict.keys():
        #存在就打印币对键对应的汇率值
        print(get_pair,"Rate:",rate_dict[get_pair])
    #不存在就打印无币对
    else:
        print('无该币对')
```

### 运行效果

```
$ 请输入基础货币,退出输入quit:usd
$ 请输入计价货币:cny
$ usd_cny Rate: 6.6067334987
$ 请输入基础货币,退出输入quit:usd
$ 请输入计价货币:xyz
$ 无该币对
$ 请输入基础货币,退出输入quit:quit
$
```