## helloworld

```
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```

要运行这个程序，先将将代码放到名为 hello-world.go 的文件中，然后执行 go run。

```
$ go run hello-world.go
hello world
```

如果我们想将程序编译成二进制文件（Windows 平台是 .exe 可执行文件）， 可以通过 go build 来达到目的。

```
$ go build hello-world.go
$ ls
hello-world    hello-world.go
```

然后我们可以直接运行这个二进制文件。

```
$ ./hello-world
hello world
```

go 会根据所在平台的不同，编译成不同的可执行程序。  
比如你在windows上，就会生成***.exe  
在linux上就是二进制文件  
