aiohttp是一个基于asyncio的异步http框架，在高并发的情况下具有很好的性能，这也是我选择使用aiohttp来搭建微信公众平台的原因。但是由于网上关于aiohttp的资料较少，再加上自己对协程的相关概念理解不够深刻，导致在开发过程中吃到了一些苦头，所以在使用aiohttp之前，首先来介绍一下基本概念。

### 协程与线程、进程的关系

一说到协程，很容易就会联想到线程与进程，但协程与它们并不相同。我们知道一个进程可以包含多个线程，它们都由系统进行管理与维护。进程本身并不执行任何代码，当系统创建一个进程时，会自动为其创建一个线程（主线程），由主线程来执行相应代码或创建更多线程。当一个进程所包含的的线程数为0时，该进程也就没有存在的必要了。所以说，进程是线程的容器。

协程是指在一个线程的执行过程中，可以在一个子程序内部中断转而执行别的子程序，在适当的时候再返回来接着执行。但与线程的切换不同，子程序的切换并不由系统控制，而是通过程序来控制。在系统看来，协程还是只有一个线程，这样的好处就是节省了线程切换的开销，也不需要多线程的锁机制，但相应的也就无法利用cpu的多核优势。

打个比方，在多核cpu情况下，一个进程就相当于一间屋子，该进程若是包含两个线程就表示这间屋子里有两个人分别在做自己的事，如果其中一个人一会儿做这件事一会儿做那件事这样来回切换，我们就说在该线程中使用了协程。

### asyncio

asyncio是python的一个异步协程框架，要使用asyncio需要先了解几个概念：

**event loop:** event负责I/O时间通知，loop负责循环处理I/O通知并在就绪时调用回调，当协程被加入到event loop后，event loop会在适当的时候调用该协程，需要注意的是，每个进程只有一个event loop

**async/await:** async def用来定义一个协程函数，在协程函数中调用嵌套协程时可以使用await关键字来挂起当前协程并等待嵌套协程的执行结果

**task:** task负责协程函数在event loop中的执行，在当前task挂起时，event loop会执行其他task，在event loop中，同一时刻只能执行一个task

下面举例说明：
```
import time
import asyncio
# hu 定义协程函数
async def compute(x, y):
	print("=>compute: x = %d,y = %d" % (x, y))
	await asyncio.sleep(1)  # hu 当前task挂起，event loop转而执行其他task
	print("<=compute: x = %d,y = %d" % (x, y))
	return x + y

async def print_sum(x, y):
	print('=>print_sum: x = %d,y = %d' % (x,y))
	result = await compute(x, y) # hu 当前协程阻塞，event loop执行嵌套协程
	print("<=print_sum: %d + %d = %d" % (x, y, result))

# hu 获取event loop
loop = asyncio.get_event_loop()
# hu 获取task列表
tasks = [asyncio.ensure_future(print_sum(1, 1), loop=loop),
		 asyncio.ensure_future(print_sum(2, 2), loop=loop)]
# hu 开始计时
start = time.time()
# hu 等待tasks的执行
loop.run_until_complete(asyncio.wait(tasks))
# hu 结束计时
end = time.time()
loop.close()
print('time: ',end-start,'s')

```
运行结果如下：
```
=>print_sum: x = 1,y = 1
=>compute: x = 1,y = 1
=>print_sum: x = 2,y = 2
=>compute: x = 2,y = 2
<=compute: x = 1,y = 1
<=print_sum: 1 + 1 = 2
<=compute: x = 2,y = 2
<=print_sum: 2 + 2 = 4
time:  1.0000569820404053 s
```
可以看到执行时间非常接近1s。

### 搭建微信公众平台

关于搭建微信公众平台的准备工作，网上的资料很多，这里就不作介绍了，只提一点，就是微信公众平台接口调用仅支持80端口，可以参考我的上一篇博文[《通过Apache反向代理实现微信服务器80端口访问》](http://blog.csdn.net/u011475134/article/details/69951987)。

新建main3.py文件，代码如下：
```
import asyncio
from aiohttp import web
import re

async def getWX(request):
	# hu 响应微信发送的Token验证
	echostr = 'success'
	try:
		echostr = request.query['echostr']  # hu 返回查询字符串中echostr的值
	except Exception as ex:
		pass
	return web.Response(body=echostr.encode('utf-8'))

async def postWX(request):
	info = await request.text()   # hu 读取请求的body
	reg = r'''<xml><ToUserName><!\[CDATA\[(.*?)\]\]></ToUserName>
<FromUserName><!\[CDATA\[(.*?)\]\]></FromUserName>
<CreateTime>(.*?)</CreateTime>
<MsgType><!\[CDATA\[(.*?)\]\]></MsgType>'''
	ToUserName, FromUserName, CreateTime, MsgType = re.findall(reg, info)[0]

	result = 'success'
	# hu 接收消息（文本、语音等等）
	if MsgType.lower() == 'text':        # hu 文本消息
		pass
	elif MsgType.lower() == 'voice':     # hu 语音消息
		pass
	elif MsgType.lower() == 'event':
		reg = r'''<Event><!\[CDATA\[(.*?)\]\]></Event>'''
		Event = re.findall(reg, info)[0]
		# hu 接收事件推送（关注、取消关注等等）
		if Event.lower() == 'subscribe':       # hu 用户关注事件
			pass
		elif Event.lower() == 'unsubscribe':  # hu 取消关注事件
			pass
	return web.Response(body=result.encode('utf-8'))

def __main():
	loop = asyncio.get_event_loop()
	app = web.Application(loop=loop)
	app.router.add_route('GET', '/WeiXin', getWX)
	app.router.add_route('POST', '/WeiXin', postWX)
	web.run_app(app, port=6670) # hu 使用6670端口
	loop.close()

if __name__ == '__main__':
	__main()
```
接下来，你就可以为公众号添加自己想要的功能啦~

说明：

1) 微信消息采用明文方式
2) 考虑到接收的xml消息内容很少，所以直接通过正则提取信息
3) 使用非80端口号之前需要配置反向代理

#### 源码下载：https://github.com/SigalHu/WeiXin/
#### 参考链接
https://pawelmhm.github.io/asyncio/python/aiohttp/2016/04/22/asyncio-aiohttp.html</br>
https://aiohttp.readthedocs.io/en/stable/</br>
https://docs.python.org/3.4/library/asyncio-task.html</br>
http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432090171191d05dae6e129940518d1d6cf6eeaaa969000</br>
http://python.jobbole.com/87310/</br>
http://blog.csdn.net/u014595019/article/details/52295642</br>
https://mp.weixin.qq.com/wiki
