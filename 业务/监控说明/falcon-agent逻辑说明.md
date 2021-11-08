# falcon-agent逻辑说明（看到哪写到哪，没有顺序）
## main-cron.Collect
这个是从funcs.BuildMappers()这个函数里面取设置好的metric。这里面全是基础监控，至于为啥这些是，别的不是，应该是小米的人自己定的。 

如果想屏蔽或者添加，就在funcs.BuildMappers()这个函数里面添加。

## plugin的采集和上传
cron.SyncMinePlugins()这个函数处理。它到指定目录下读取所有的脚本文件，然后添加到定时任务中。添加完直接*.Schedule()就跑起来了。具体执行哪些plugin，中间有一步去系统段获取，就拿到本机应该执行哪些plugin