通过上文[《使用python-aiohttp搭建微信公众平台》](http://blog.csdn.net/u011475134/article/details/70147484)，我们已经可以响应来自微信服务器的请求，接下来，我们为公众号增加一个在线点歌的功能。

由于本人平时听歌用的是网易云音乐，所以就在网上搜了一下，还真找到不少，再考虑到这里只需要用到网易云音乐的关键词搜索，最终锁定了这篇文章[《音乐API推荐–网易音乐API–百度音乐API》](http://www.fddcn.cn/music-api-wang-yi-bai-du.html)，先通过Fiddler抓包看看这篇文章介绍的方法效果咋样。
```
GET http://s.music.163.com/search/get/?type=1&s=彩虹&limit=1&offset=0
```
| 参数    | 取值 | 说明 |
| ------ |------| ---- |
| type   | 1    | 无  |
| s      | 彩虹 | 关键词 |
| offset | 0    | 偏移量 |
| limit  | 1    | 最大返回结果数 |
其中的offset类似翻页功能，比如一页有十首歌曲，就令limit=10,offset=0，返回第一页，设置limit=10,offset=10来返回第二页。

返回值如下：
```
{
    "result": {
        "songCount": 3919,
        "songs": [
            {
                "id": 413831818,
                "name": "彩虹",
                "artists": [
                    {
                        "id": 1207037,
                        "name": "猫猫村长",
                        "picUrl": null
                    }
                ],
                "album": {
                    "id": 34706315,
                    "name": "彩虹",
                    "artist": {
                        "id": 0,
                        "name": "",
                        "picUrl": null
                    },
                    "picUrl": "http://p1.music.126.net/e2P6AA498KaJpQ_BGrfWGw==/17993507788684015.jpg"
                },
                "audio": "http://m2.music.126.net/pDFup5BuOFC28iCopRj2QQ==/3251255884973388.mp3",
                "djProgramId": 0,
                "page": "http://music.163.com/m/song/413831818"
            }
        ]
    },
    "code": 200
}
```
在返回的json数据中，songCount表示以彩虹为关键词搜索到的歌曲总数，songs是返回的歌曲列表，其中，name表示歌曲名；artists与album分别包含歌手与唱片信息；audio与page分别为歌曲与歌曲页面的链接。既然返回结果没问题，接下来我们就用aiohttp搭建客户端来发送GET请求，并从返回值中提取歌曲名、歌手名、唱片图片与歌曲页面等信息。

新建文件netease_music3.py，代码如下：
```
import asyncio
from aiohttp import ClientSession

__MUSIC_NUM = 1   # hu 返回的最大歌曲数

async def __fetch(url,data,loop):
	try:
		async with ClientSession(loop=loop) as session:
			# hu 发送GET请求，params为GET请求参数，字典类型
			async with session.get(url, params=data,timeout=5) as response:
				# hu 以json格式读取响应的body并返回字典类型
				return await response.json()
	except Exception as ex:
		print('__fetch:%s' % ex)

async def getMusicInfo(keyword,offset, loop):
	global __MUSIC_NUM
	urlFace = 'http://s.music.163.com/search/get'
	dataMusic = {'type': '1',
				's': keyword,
				'limit': str(__MUSIC_NUM),
				'offset': str(offset)}
	result = None
	try:
		task = asyncio.ensure_future(__fetch(urlFace, dataMusic,loop),loop=loop)
		taskDone = await asyncio.wait_for(task,timeout=5)
		if 'result' not in taskDone:
			return result

		for song in taskDone['result']['songs']:
			if result is None:
				result = [{'name':song['name'],
						   'artist':song['artists'][0]['name'],
						   'picUrl':song['album']['picUrl'],
						   'audio':song['audio'],
						   'page':song['page']}]
			else:
				result.append({'name': song['name'],
							   'artist': song['artists'][0]['name'],
							   'picUrl': song['album']['picUrl'],
							   'audio': song['audio'],
							   'page': song['page']})
	except Exception as ex:
		print('getMusicInfo:%s' % ex)
	return result

def __main():
	loop = asyncio.get_event_loop()
	music = '彩虹'
	player = '乔楚熙'
	task = asyncio.ensure_future(getMusicInfo(music+player,1,loop),loop=loop)
	taskDone = loop.run_until_complete(task)
	for key,value in taskDone[0].items():
		print(key+':'+value)
	loop.close()

if __name__ == '__main__':
	__main()
```
返回值：
```
name:彩虹
artist:乔楚熙
picUrl:http://p1.music.126.net/dAzMzFp6ucMVcSSPn0cFqg==/19049038951250054.jpg
audio:http://m2.music.126.net/5f0_bUm6eMrx4o_ZN4OFQg==/2074778441644241.mp3
page:http://music.163.com/m/song/4873072
```
关于asyncio模块的基本概念可参考[《使用python-aiohttp搭建微信公众平台》](http://blog.csdn.net/u011475134/article/details/70147484)，这里就不一一注释了。接下来，我们在微信公众平台上添加在线点歌功能。

打开main3.py文件（见[《使用python-aiohttp搭建微信公众平台》](http://blog.csdn.net/u011475134/article/details/70147484)），导入文件：
```
import netease_music3
```
在if语句中添加代码如下：
```
    if MsgType.lower() == 'text':        # hu 文本消息
        reg = r'''<Content><!\[CDATA\[(.*?)\]\]></Content>
<MsgId>(.*?)</MsgId>'''
        Content, MsgId = re.findall(reg, info)[0]
        resp = await netease_music3.getMusicInfo(Content, 0, request.app.loop)
        if resp is None:
            Content = '要不换首歌吧，这个真找不到啊'
            result = '''<xml>
<ToUserName><![CDATA[%s]]></ToUserName>
<FromUserName><![CDATA[%s]]></FromUserName>
<CreateTime>%s</CreateTime>
<MsgType><![CDATA[%s]]></MsgType>
<Content><![CDATA[%s]]></Content>
</xml>''' % (FromUserName, ToUserName, CreateTime, MsgType, Content)
        else:
            MsgType = 'news'
            MusicCount = len(resp)
            musics = [None] * MusicCount

            for ii, music in zip(range(MusicCount), resp):
                musics[ii] = '''
<item>
<Title><![CDATA[%s]]></Title>
<Description><![CDATA[%s]]></Description>
<PicUrl><![CDATA[%s]]></PicUrl>
<Url><![CDATA[%s]]></Url>
</item>
''' % ('%s-%s' % (music['name'], music['artist']), music['artist'], music['picUrl'], music['page'])
            result = '''<xml>
<ToUserName><![CDATA[%s]]></ToUserName>
<FromUserName><![CDATA[%s]]></FromUserName>
<CreateTime>%s</CreateTime>
<MsgType><![CDATA[%s]]></MsgType>
<ArticleCount>%d</ArticleCount>
<Articles>
%s
</Articles>
</xml>''' % (FromUserName, ToUserName, CreateTime, MsgType, MusicCount, ''.join(musics))
```
运行以上代码就可以在公众号里输入关键词点歌啦~
#### 源码下载：https://github.com/SigalHu/WeiXin/
#### 参考链接
https://aiohttp.readthedocs.io/en/stable/</br>
http://blog.csdn.net/u014595019/article/details/52295642
