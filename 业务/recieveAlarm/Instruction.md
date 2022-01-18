```flow
st=>start: Start
e=>end: End
kafka=>operation: 启动kafka队列读取进程|past
cycle=>subroutine: 从配置获取topic列表，循环获取|current
read=>operation: 读取数据|past
judge=>condition: topic读取完毕?
hang=>operation: hang

st(right)->kafka->cycle->read(right)->judge(no)->cycle
judge(yes,right)->hang->e
```

```flow
st=>start: Start
e=>end: End
http=>operation: 启动http接口|past
send=>operation: 发n9e、发falcon
st->http->send->e
```
