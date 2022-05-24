# telegraf 自己写 input

telegraf 支持开发者自己开发 input 插件，写的好的会合并到主分支上。
自己写 plugin 需要自定义以下部分：

## 一、register——注册

注册包括 input 插件的注册和它自己使用的数据解析器的注册。

### 1.1 插件注册

文件位置：plugins/inputs/all/all.go

```go
package all
import (
  _ "github.com/influxdata/telegraf/plugins/inputs/script"
)
```

实际上 import 的内容很多，每一个 plugin 都要在这里 import，这里只截取自己写的。
这个 script 是自己写的插件目录

### 1.2 解析器的注册

文件位置：plugins/parsers/register.go

```go
func NewParser(config *Config) (Parser, error) {
  var err error
  var parser Parser
  switch config.DataFormat {
  case "json":
    parser, err = json.New(
      &json.Config{
        MetricName:   config.MetricName,
        TagKeys:      config.TagKeys,
        NameKey:      config.JSONNameKey,
        StringFields: config.JSONStringFields,
        Query:        config.JSONQuery,
        TimeKey:      config.JSONTimeKey,
        TimeFormat:   config.JSONTimeFormat,
        Timezone:     config.JSONTimezone,
        DefaultTags:  config.DefaultTags,
        Strict:       config.JSONStrict,
      },
    )
  case "value":
    parser, err = NewValueParser(config.MetricName,
      config.DataType, config.ValueFieldName, config.DefaultTags)
  ......
  case "openfalcon":
    parser, err = NewSelfDefinedParser()
  default:
    creator, found := Parsers[config.DataFormat]
    if !found {
      return nil, fmt.Errorf("invalid data format: %s", config.DataFormat)
    }

    // Try to create new-style parsers the old way...
    // DEPRECATED: Please instantiate the parser directly instead of using this function.
    parser = creator(config.MetricName)
    p, ok := parser.(ParserCompatibility)
    if !ok {
      return nil, fmt.Errorf("parser for %q cannot be created the old way", config.DataFormat)
    }
    err = p.InitFromConfig(config)
  }
  return parser, err
```

这个文件的核心部分是各种 case，这里只截取了两种。自己写的解析器(parser)要在这里加上一个 case，这里面“openfalcon”就是我自己写的。

## 二、写实际功能

实际功能包括两个部分，上面提到的解析器(parser)和插件(plugin)

### 2.1 parser

parser 是 input 的解析器，里面规定了采集到的数据按照怎样的逻辑进行处理

```go
package script

import (
  "encoding/json"
  "fmt"
  "github.com/influxdata/telegraf"
  "github.com/influxdata/telegraf/metric"
  "time"
)

type Parser struct {
  DefaultTags     map[string]string
  DefaultEndpoint string
  DefaultMetric   string
  Log             telegraf.Logger `toml:"-"`
}

// Parse ParseLine SetDefaultTags这三个方法必须要有，接口那里定义了，那么关联的方法就得写全
func (p *Parser) Parse(buf []byte) ([]telegraf.Metric, error) {
  fmt.Printf("Parse func, %+v\n", string(buf))
  var datas []*Opentsdb
  err := json.Unmarshal(buf, &datas)
  if err != nil {
    return []telegraf.Metric{}, err
  }
  metrics := make([]telegraf.Metric, 0)
  for _, data := range datas {
    err := data.CheckValidity(time.Now().Unix(), p)
    if err != nil {
      p.Log.Errorf("script checked data failed. data: %+v.\n", err.Error())
      continue
    }
    m := metric.New("script", data.Tags, map[string]interface{}{data.Metric: data.Value}, time.Unix(data.Timestamp, 0), data.CounterType)
    metrics = append(metrics, m)
  }
  fmt.Println("metrics: ", metrics[0].Tags(), metrics[0].Fields(), metrics[0].Type())
  return metrics, nil
}

func (p *Parser) ParseLine(line string) (telegraf.Metric, error) {
  fmt.Printf("ParserLine: %+v\n", line)
  metrics, err := p.Parse([]byte(line + "\n"))

  if err != nil {
    return nil, err
  }

  if len(metrics) < 1 {
    return nil, fmt.Errorf("can not parse the line: %s, for data format: json ", line)
  }

  return metrics[0], nil
}

func (p *Parser) SetDefaultTags(tags map[string]string) {
  fmt.Printf("SetDefaultTags: %+v\n", tags)
  p.DefaultTags = tags
}
```

这是自己写的 parser。全是核心，一样都不能少，你可以把内容空着，但是该有的方法，结构体都不能少。

#### 2.1.1 type Parser struct

定义了当前 parser 的一些基本参数。这个例子里面其实只有 Log 用到了，其余的都是充数的。可以看作是一个初始化的东西。
注意这个定义，`Parser`这个词一定得是这个，大小写、拼写都不能错。因为这个关联了 telegraf 关于 parser 的一个接口，如果关联不上，这个 parser 就找不到，也就无法使用。go 里面，基本都是通过接口来调用方法，就没看到说直接调用结构体的方法，所以这个地方绝对不能改。

#### 2.1.2 Parse

这是 parser 的核心方法，这里规定了具体的解析逻辑。
注意入参和返回值的类型，**不可以改**。
返回值定死了，不能改。那么你的解析结果必须按照要求返回。
我没有仔细去看这个 telegraf.Metric 是怎么工作的，我猜测是 input 往这个结构里面写入，output 从这里面读取。

##### telegraf.Metric 的使用

这也是一个接口，但是这个数据存到了 telegraf/metrc/metrc.go 里面 metric 这个结构体里面了，但是它怎么就变成 telelgraf.Metric，不知道。只知道，需要存数据的时候，调用对应的方法就行。

先定义

```go
func New(
  name string,
  tags map[string]string,
  fields map[string]interface{},
  tm time.Time,
  tp ...telegraf.ValueType,
) telegraf.Metric {...}

m := metric.New("script", data.Tags, map[string]interface{}{data.Metric: data.Value}, time.Unix(data.Timestamp, 0), data.CounterType)
m.AddFields("metric": "value")
m.AddTag("key": "value")
```

这里把 New 这个函数的定义拿出来了，里面的逻辑点点点替代。第一个是 plugin 的名字，在 telegraf 原生的采集逻辑里面，这个名字表示采集数据的大类，例如 cpu、disk、mem 之类。fields 就是 openfalcon 里面所定义的 metric，不过这里的 fields 是一个字典，它的值可以是任何类型(不知道它怎么搞定的，反正目前我接触的都是 int)。tags 就是标签，tm 是时间戳(注意类型)，tp 是采集类型(注意类型)

tags 和 fields 都是字典，即，可以一次性输入多个值。需要注意 fields，fields 对应 openfalcon 中的 metric 和 value。所以，多个值意味着，一次传输了多个 metric，同时其余参数保持不变，这个在特定条件下可以简化数据格式化。

metric 还提供了多个方法，AddField()、AddTags()
这两个方法一次只能加一组数据

#### 2.1.3 ParseLine

没有找到调用的地方，暂时不管

#### 2.1.4 SetDefaultTags

好像也没有什么用

### 2.2 plugin 调用

需要在 plugins/inputs/目录下新建一个同名目录
至于里面文件的名字可以随便写，建议同名，不要整些没用的。
样例：

```go
package script

import (
  "bytes"
  "encoding/json"
  "fmt"
  "github.com/influxdata/telegraf"
  "github.com/influxdata/telegraf/config"
  "github.com/influxdata/telegraf/internal"
  "github.com/influxdata/telegraf/plugins/inputs"
  "github.com/influxdata/telegraf/plugins/parsers"
  "github.com/kballard/go-shellquote"
  osExec "os/exec"
  "time"
)

// Script struct should be named the same as the Plugin
type Script struct {
  // Script for a mandatory option to set a tag
  DeviceName string `toml:"device_name"`

  // Config options are converted to the correct type automatically
  NumberFields int64 `toml:"number_fields"`

  // We can also use booleans and have diverging names between user-configuration options and struct members
  EnableRandomVariable bool `toml:"enable_random"`

  // Script of passing a duration option allowing the format of e.g. "100ms", "5m" or "1h"
  Timeout config.Duration `toml:"timeout"`

  // Telegraf logging facility
  // The exact name is important to allow automatic initialization by telegraf.
  Log telegraf.Logger `toml:"-"`

  // This is a non-exported internal state.
  count int64

  parser    parsers.Parser
  SelfParam string `toml:"selfparam"`

  ScriptPath string `toml:"scriptpath"`
}

// Usually the default (Script) configuration is contained in this constant.
// Please use '## '' to denote comments and '# ' to specify default settings and start each line with two spaces.
const sampleConfig = `
  # 这里面的内容是自定义配置的，可以随便搞。
`

// Description will appear directly above the plugin definition in the config file
func (s *Script) Description() string {
  return `This is an Script plugin`
}

// SampleConfig will populate the sample configuration portion of the plugin's configuration
func (s *Script) SampleConfig() string {
  return sampleConfig
}

// Init can be implemented to do one-time processing stuff like initializing variables
func (s *Script) Init() error {
  return nil
}

func (s *Script) SetParser(parser parsers.Parser) {
  s.parser = parser
}

type Openfalcon struct {
  Metric      string  `json:"metric"`
  Endpoint    string  `json:"endpoint"`
  Tags        string  `json:"tags"`
  Timestamp   int64   `json:"timestamp"`
  Value       float32 `json:"value"`
  CounterType string  `json:"counterType"`
  Step        int64   `json:"step"`
}

// Gather defines what data the plugin will gather.
func (s *Script) Gather(acc telegraf.Accumulator) error {
  //todo:
  //1. 获取策略，这台机器上应该执行哪些脚本，及脚本的位置和入参；
  //2. 循环执行这些脚本，并获取返回值；
  //3. telegraf需要自己填写metric、tags、value，要把这三个参数从返回值里面拿出来
  //   对应到telegraf的形式就是 metric——>measurement+'_'+fields的key；value——>fields的value
  //   这个metric有待验证，之前看到的是output这么干，input似乎没有这个操作，看代码也没有
  //回来需要找一下配置文件的值是怎么写入程序的

  splitCmd, err := shellquote.Split(s.ScriptPath)
  if err != nil {
    fmt.Println(err)
  }

  cmd := osExec.Command(splitCmd[0], splitCmd[1:]...)

  var (
    out    bytes.Buffer
    stderr bytes.Buffer
  )
  cmd.Stdout = &out
  cmd.Stderr = &stderr

  runErr := internal.RunTimeout(cmd, time.Duration(s.Timeout))
  if runErr != nil {
    fmt.Println("last: ", runErr.Error())
  }

  aaa := &[]Openfalcon{}
  json.Unmarshal(out.Bytes(), aaa)

  metrics, err := s.parser.Parse(out.Bytes())
  if err != nil {
    s.Log.Errorf("script %s parses error: %s\n", s.ScriptPath, err.Error())
  }
  for _, metric := range metrics {
    acc.AddMetric(metric)
  }

  return nil
}

// Register the plugin
func init() {
  inputs.Add("script", func() telegraf.Input {
    return &Script{
      SelfParam: "origin",
    }
  })
}
```

文件内容必须包含以下内容：都要注意入参和返回值，类型一定要正确

#### 2.2.1 type Script struct

这个名字随便写，自己记着就行。
这个东西和 parser 里面的有个重要区别，这里面的内容是关联到配置文件的！

```toml
[inputs.script]
selfparam = "123123123123123"
scriptpath = "./sys_thread.py"
timeout = "1s"
data_format = "openfalcon"
```

这是自己加的配置。这里面 selfparam 和 scriptpath 是可以传到这个结构体里面，然后拿来用的。

#### 2.2.2 SampleConfig

这个是用来生成默认配置文件的，最好是写上。文档建议把所有配置都写在这里，方便别人用。

#### 2.2.3 Description

返回介绍

#### 2.2.4 Init()

初始化。这里是 plugin 调用的必经之路，且只调用一次，如果有什么一次性配置，可以写在这里。

#### 2.2.5 SetParser(parser parsers.Parser)

设置 parser，这个地方是可以不设置的，如果不设置，上面关于 parser 注册的两部分，就会走 default 分支，它就采用公共的解析方式。具体什么逻辑还没有看，作为自己写的插件，还是自己写 parser 比较好，都已经定制化了，这点就写上吧

#### 2.2.6 Gather(acc telegraf.Accumulator) error

最重要的部分。
Gather 定义了你如何采集你想要的数据。并且采集完成的数据要在这里进行解析然后发送给 accumulate。

Gather 里面必须要有 acc.AddMetric()。这是数据推送给 accumulate 的唯一途径，不推就无法发送(这个词准确性有待商榷)给 output。

#### 2.2.7 init()

go 语言特性，init 必定执行，无需调用。这里就是把这个 input plugin 挂到总的 inputs 列表里面。
这里初始化 Script 这个结构体
