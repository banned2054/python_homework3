# Python程序设计#3作业

## 作业题目

基于 aiohttp（https://docs.aiohttp.org/en/stable/）实现一个服务查询客户端，能够访问[#2作业](https://github.com/banned2054/python_homework2)提供的服务。数据获取后进行格式转换：

* JSON结果转换为TEXT格式（字段之间使用空格间隔、记录之间使用换行符间隔）
* XML结果转换为TEXT格式（需求同上）。
* CSV格式转换为TEXT格式（需求同上）。

要求客户端可以通过以上3种格式访问数据服务。

## 作业内容

程序源代码嵌入下方的code block中。

```python
import asyncio

import aiohttp


async def get_json(session, url) :
    async with session.get(url) as response :
        return await response.json( )


async def get_text(session, url) :
    async  with session.get(url) as response :
        return await response.text( )


async def main( ) :
    while (True) :
        file_type = input('输入查询文本种类：[csv,json,xml]\n')
        if (file_type != 'csv' and file_type != 'json' and file_type != 'xml') :
            continue
        begin_year = input('输入起始年份：\n')
        if ((not begin_year.isdigit( )) or int(begin_year) < 1880 or int(begin_year) > 2020) :
            continue
        end_year = input('输入结束年份：\n')
        if ((not end_year.isdigit( )) or int(end_year) < int(begin_year) or int(end_year) > 2020) :
            continue
        sort_type = input('输入排序方式：[up,down]\n')
        if (sort_type != 'up' and sort_type != 'down') :
            continue
        url = 'http://127.0.0.1:2054'
        url += '/' + file_type + '?'
        url += 'begin=' + begin_year + '&'
        url += 'end=' + end_year + '&'
        url += 'sort=' + sort_type
        if (file_type == 'json') :
            async with aiohttp.ClientSession( ) as session :
                result = await get_json(session, url)
                for data in result :
                    data_json = result[ data ]
                    year = data_json[ 0 ][ 'year' ]
                    no_smoothing = data_json[ 1 ][ 'No_Smoothing' ]
                    lowess = data_json[ 2 ][ 'Lowess' ]
                    print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))
        if (file_type == 'xml') :
            async  with aiohttp.ClientSession( ) as session :
                result = await get_text(session, url)
                result = result.replace('<?xml version="1.0" encoding="UTF-8"?>', '')
                result = result.replace('<Document>', '')
                result = result.replace('</Document>', '')
                result = result.replace('</climate>', '')
                datas = result.split('<climate>')
                datas.remove(datas[ 0 ])
                for data in datas :
                    data_split = data.split('\n')
                    year = data_split[ 0 ].replace('<year>', '')
                    year = year.replace('</year>', '')
                    no_smoothing = data_split[ 1 ].replace('<No_Smoothing>', '')
                    no_smoothing = no_smoothing.replace('</No_Smoothing>', '')
                    lowess = data_split[ 2 ].replace('<Lowess>', '')
                    lowess = lowess.replace('</Lowess>', '')
                    print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))
        if (file_type == 'csv') :
            async  with aiohttp.ClientSession( ) as session :
                result = await get_text(session, url)
                for data in result :
                    data_split = data.split(',')
                    year = data_split[ 0 ]
                    no_smoothing = data_split[ 1 ]
                    lowess = data_split[ 2 ]
                    print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))


loop = asyncio.get_event_loop( )
loop.run_until_complete(main( ))

```

## 代码说明

### 查询功能

分别定义了`get_json`和`get_text`这两个查询函数，一个用于查询得到json格式的数据，而由于xml和csv没有对应的格式，所以用text代替查询

get_json:

```python
async def get_json(session, url) :
    async with session.get(url) as response :
        return await response.json( )
```

get_test:

```python
async def get_text(session, url) :
    async  with session.get(url) as response :
        return await response.text( )
```

### 用户使用

程序会无限次提供查询功能，直到程序被关闭，使用`while`进行循环

#### 输入

提示输入，以及判断输入错误后，会`continue`，重新进行输入操作：

```python
file_type = input('输入查询文本种类：[csv,json,xml]\n')
if (file_type != 'csv' and file_type != 'json' and file_type != 'xml') :
	continue
begin_year = input('输入起始年份：\n')
if ((not begin_year.isdigit( )) or int(begin_year) < 1880 or int(begin_year) > 2020) :
	continue
end_year = input('输入结束年份：\n')
if ((not end_year.isdigit( )) or int(end_year) < int(begin_year) or int(end_year) > 2020) :
	continue
sort_type = input('输入排序方式：[up,down]\n')
if (sort_type != 'up' and sort_type != 'down') :
	continue
```

#### 输入转换

将用户输入的信息转换为url

```python
url = 'http://127.0.0.1:2054'
url += '/' + file_type + '?'
url += 'begin=' + begin_year + '&'
url += 'end=' + end_year + '&'
url += 'sort=' + sort_type
```

#### json解析成text

```python
if (file_type == 'json') :
	async with aiohttp.ClientSession( ) as session :
		result = await get_json(session, url)
       	for data in result :
        	data_json = result[ data ]
            year = data_json[ 0 ][ 'year' ]
            no_smoothing = data_json[ 1 ][ 'No_Smoothing' ]
            lowess = data_json[ 2 ][ 'Lowess' ]
			print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))
```

#### xml解析成text

```python
if (file_type == 'xml') :
	async  with aiohttp.ClientSession( ) as session :
    	result = await get_text(session, url)
        result = result.replace('<?xml version="1.0" encoding="UTF-8"?>', '')
        result = result.replace('<Document>', '')
        result = result.replace('</Document>', '')
        result = result.replace('</climate>', '')
        datas = result.split('<climate>')
        datas.remove(datas[ 0 ])
        for data in datas :
        	data_split = data.split('\n')
            year = data_split[ 0 ].replace('<year>', '')
            year = year.replace('</year>', '')
            no_smoothing = data_split[ 1 ].replace('<No_Smoothing>', '')
            no_smoothing = no_smoothing.replace('</No_Smoothing>', '')
            lowess = data_split[ 2 ].replace('<Lowess>', '')
            lowess = lowess.replace('</Lowess>', '')
            print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))
```

#### csv解析成text

```python
if (file_type == 'csv') :
	async  with aiohttp.ClientSession( ) as session :
    	result = await get_text(session, url)
        for data in result :
        	data_split = data.split(',')
            year = data_split[ 0 ]
            no_smoothing = data_split[ 1 ]
            lowess = data_split[ 2 ]
            print('year:' + str(year) + ' No_Smoothing:' + str(no_smoothing) + ' Lowess:' + str(lowess))
```

