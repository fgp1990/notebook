```js
computed: {
    title() {
      if (this.newIndex == true) {
        return '新增策略'
      } else {
        return '修改策略'
      }
    }
  },
```

这里面 title 就是变量名称。看起来像是一个函数，实际上是一个函数形式的判断逻辑。
这里无比要有 return，保证 title 有一个值
