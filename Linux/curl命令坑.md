```
curl http://192.168.51.6:4242/api/query?start=24h-ago&m=sum:codis.sessions{codis_name=codis-recsys}
```
这个指令是没有办法执行的，curl会把{}这个东西给吃了。导致请求返回bad request
需要你用单引号把后面的都引用起来