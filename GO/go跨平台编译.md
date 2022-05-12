# 跨平台编译

```linux
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
```

windows 下：

```cmd
SET CGO_ENABLED=0;SET GOOS=linux;SET GOARCH=amd64;go build
```

这个没有验证成功。这个 SET 就没有效果
