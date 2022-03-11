```go
switch tv := m.Tags.(type) {
	case string:
		fmt.Println("enter string")
		m.TagsMap = make(map[string]string)
		qq := strings.Split(tv, ",")
		for _, value := range qq {
			a := strings.Split(value, "=")
			m.TagsMap[a[0]] = a[1]
		}
	case map[string]string:
		fmt.Println("enter string")
		// m.TagsMap = tv
	case map[string]float64:
		fmt.Println("enter float64")
	case []string:
		fmt.Println("enter array")
	case map[string]interface{}:
		fmt.Println("enter interface")
		for k, v := range tv {
			fmt.Println(k, v)
		}
		fmt.Println(tv)

	default:
		fmt.Println("enter valid")
		valid = false
	}
```

这里注意一个问题。case 里面只能识别最开始定义的变量。这段代码里面，就是 tv。
尽管 tv 和 m.Tags 是一个东西，但是 case 里面只能识别 tv 的类型。
