# openresty-escape
openresty逃逸技术

## openresty 逃逸技术是什么
别想多了，我的resty逃逸技术并不是什么高深的玩意， 并没有利用什么resty漏洞。
只是使用很简单的技术，绕开了openresty的某些限制，仅此而已。

## 特殊任务
我用openresty一年多了，一直没有做什么太有价值的东西。 

前几天，某领导让我用openresty做个金证交易通讯协议kuab.kdgp的demo， 于是一个有点挑战的任务就开始了。

## 到底是什么限制了我
加密解密，包处理，连接池，很快就写好了。 调试时，问题就来了。

- socket 不能跨ctx使用  
  在请求A中创建的sock，在请求B中不能用。 
  >这是长连接协议，全双工读写，sock不能还回去的，更不能频繁创建和关闭。
- 协程也不能跨ctx使用  
  我创建了一个全局协程，专门负责接收数据并解包，根本执行不了。
  
## 突破sock限制
我很快就找到了突破sock限制的办法， 启动一组时钟，死循环， 永不退出， 专门负责socket处理。

## 突破协程限制
我正在研究，尚未实践。 通过openresty源代码发现，协程函数是被替换了，所以才不让跨ctx使用的。
协程原来的函数仍然保留，只是被换了名字，名字前面全都加了下划线。

```
local co_yield = coroutine._yield
local co_create = coroutine._create
local co_status = coroutine._status
local co_resume = coroutine._resume
```
居然调用成功了， 也没发现有什么问题。 但是我仍然担心它们会影响父协程的调度。
