./vmstorage-prod --help

#### 处理大型合并时所用的最大 cpu 核数

-bigMergeConcurrency int
The maximum number of CPU cores to use for big merges. Default value is used if set to 0

#### 如果时序数据之间的时间差比该配置小，则剔除。当多个 Prometheus 同时写入一个 VictoriaMetric 时，这对减少开销很有帮助。当设置为 0 时，不生效。

-dedup.minScrapeInterval duration
Remove superflouos samples from time series if they are located closer to each other than this duration. This may be useful for reducing overhead when multiple identically configured Prometheus instances write data to the same VictoriaMetrics. Deduplication is disabled if the -dedup.minScrapeInterval is 0

#### 是否拒绝已配置的-retentionPeriod(数据保留时长)之外的查询。 设置后，/api/v1/query_range 对于-retentionPeriod 以外的'from'值的查询，将返回'503 Service Unavailable'错误。 当多个具有不同数据保留时长的数据源隐藏在 query-tee 后面时，该项可能很有用

-denyQueriesOutsideRetention
Whether to deny queries outside of the configured -retentionPeriod. When set, then /api/v1/query_range would return '503 Service Unavailable' error for queries with 'from' value outside -retentionPeriod. This may be useful when multiple data sources with distinct retentions are hidden behind query-tee

#### 是否监听和使用 ipv6，默认只使用 ipv4

-enableTCP6
Whether to enable IPv6 for listening and dialing. By default only IPv4 TCP is used

#### 是否读取命令行以外的环境变量的参数，命令行参数的等级高于环境变量参数。若未设置时，只读取命令行参数。

-envflag.enable
Whether to enable reading flags from environment variables additionally to command line. Command line flag values have priority over values from environment vars. Flags are read only from command line if this flag isn't set

#### 环境变量的前缀，如果设置了-envflag.enable

-envflag.prefix string
Prefix for environment variables if -envflag.enable is set

#### 没有新数据插入之后，在完成每月分区的最终合并之前的延迟。完成最终合并后，通常会降低查询速度和磁盘空间使用率。最终合并的延迟过短可能会导致磁盘 IO 使用率和 CPU 使用率增加（默认为 30 秒）

-finalMergeDelay duration
The delay before starting final merge for per-month partition after no new data is ingested into it. Query speed and disk space usage is usually reduced after the final merge is complete. Too low delay for final merge may result in increased disk IO usage and CPU usage (default 30s)

#### 授权字符串，必须以查询字符串的形式传递给/internal/force_flush 接口

-forceFlushAuthKey string
authKey, which must be passed in query string to /internal/force_flush pages

#### 授权字符串，必须以查询字符串的形式传递给/internal/force_merge 接口

-forceMergeAuthKey string
authKey, which must be passed in query string to /internal/force_merge pages

#### 是否使用 pread()代替 mmap()读取数据文件。默认情况下，mmap()用于 64 位架构，而 pread()用于 32 位架构，因为它们无法读取内存中大于 2^32 字节的数据文件。mmap()通常比 pread()更快地读取小数据块。

-fs.disableMmap
Whether to use pread() instead of mmap() for reading data files. By default mmap() is used for 64-bit arches and pread() is used for 32-bit arches, since they cannot read data files bigger than 2^32 bytes in memory. mmap() is usually faster for reading small data chunks than pread()

#### 外部 http 请求在超过该值后，将被关闭。这可能有助于在负载均衡器后面的服务集群之间分配传入负载。请注意，实际超时可能会增加多达 10％，以防止出现惊群效应（默认为 2m0s）

-http.connTimeout duration
Incoming http connections are closed after the configured timeout. This may help spreading incoming load among a cluster of services behind load balancer. Note that the real timeout may be bigger by up to 10% as a protection from Thundering herd problem (default 2m0s)

####禁用 HTTP 响应压缩以节省 CPU 资源。默认情况下启用压缩以节省网络带宽
-http.disableResponseCompression
Disable compression of HTTP responses for saving CPU resources. By default compression is enabled to save network bandwidth

#### 空闲 http 请求超时时间

-http.idleConnTimeout duration
Timeout for incoming idle http connections (default 1m0s)

#### HTTP server 正常关闭的最大持续时间。高负载服务器可能需要增加值才能正常关闭（默认为 7 秒）

-http.maxGracefulShutdownDuration duration
The maximum duration for graceful shutdown of HTTP server. Highly loaded server may require increased value for graceful shutdown (default 7s)

#### http url 代理前缀。此项在 nginx 等代理时有用

-http.pathPrefix string
An optional prefix to add to all the paths handled by http server. For example, if '-http.pathPrefix=/foo/bar' is set, then all the http requests will be handled on '/foo/bar/\*' paths. This may be useful for proxied requests. See https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus

#### HTTP server 关闭之前的可选延迟。在此交易期间，http server 为/health 接口返回了 non-OK 响应，因此负载均衡器可以将新请求路由到其他 server 上

-http.shutdownDelay duration
Optional delay before http server shutdown. During this dealy the servier returns non-OK responses from /health page, so load balancers can route new requests to other servers

#### http 监控地址

-httpListenAddr string
Address to listen for http connections (default ":8482")

#### 是否禁用在日志中写入时间戳

-loggerDisableTimestamps
Whether to disable writing timestamps in logs

#### 每秒错误日志数量的限制。 如果每秒发出的错误数量超过给定的数量，那么剩余的错误将被抑制。零值禁用速率限制（默认值为 10）

-loggerErrorsPerSecondLimit int
Per-second limit on the number of ERROR messages. If more than the given number of errors are emitted per second, then the remaining errors are suppressed. Zero value disables the rate limit (default 10)

#### 日志格式

-loggerFormat string
Format for logs. Possible values: default, json (default "default")

#### 日志等级

-loggerLevel string
Minimum level of errors to log. Possible values: INFO, WARN, ERROR, FATAL, PANIC (default "INFO")

#### 日志输出

-loggerOutput string
Output for the logs. Supported values: stderr, stdout (default "stderr")

#### VictoriaMetrics 可用的缓存大小，如果设置为非 0 值，将覆盖-memory.allowedPercent 参数。值太低可能会增加高速缓存未命中率，这通常会导致更高的 CPU 和磁盘 IO 使用率。值太高可能会从 OS 页面缓存中驱逐过多数据，这将导致更高的磁盘 IO 使用率。

支持以下值的可选后缀：KB，MB，GB，KiB，MiB，GiB（默认 0）
-memory.allowedBytes value
Allowed size of system memory VictoriaMetrics caches may occupy. This option overrides -memory.allowedPercent if set to non-zero value. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 0)

#### VictoriaMetrics 可用的缓存占用系统内存的百分比。默认 60%

-memory.allowedPercent float
Allowed percent of system memory VictoriaMetrics caches may occupy. See also -memory.allowedBytes. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage (default 60)

#### 每个值要存储的精度位数。较低的精度位以精度损失为代价提高了数据压缩比（默认为 64）

-precisionBits int
The number of precision bits to store per each value. Lower precision bits improves data compression at the cost of precision loss (default 64)

#### 数据保留时长，时间戳在该值之外的数据将被自动删除。

支持以下可选后缀：h（小时），d（天），w（周），y（年）。如果未设置后缀，则持续时间以月为单位（默认为 1）
-retentionPeriod value
Data with timestamps outside the retentionPeriod is automatically deleted
The following optional suffixes are supported: h (hour), d (day), w (week), y (year). If suffix isn't set, then the duration is counted in months (default 1)

#### 禁用 RPC 流量压缩。 这样可以减少 CPU 使用率，但会占用更高的网络带宽

-rpc.disableCompression
Disable compression of RPC traffic. This reduces CPU usage at the cost of higher network bandwidth usage

#### 每次搜索返回的最大标签建数量

-search.maxTagKeys int
The maximum number of tag keys returned per search (default 100000)

#### 从/metrics/find 返回的标签值后缀的最大数量（默认为 100000）

-search.maxTagValueSuffixesPerSearch int
The maximum number of tag value suffixes returned from /metrics/find (default 100000)

#### 每次搜索返回的标签值的最大数量（默认为 100000）

-search.maxTagValues int
The maximum number of tag values returned per search (default 100000)

#### 每次搜索扫描可以支持的独立事件序列的最大数量

-search.maxUniqueTimeseries int
The maximum number of unique time series each search can scan (default 300000)

#### 用于小型合并的最大 CPU 内核数。如果设置为 0，则使用默认值

-smallMergeConcurrency int
The maximum number of CPU cores to use for small merges. Default value is used if set to 0

#### authKey，必须以查询字符串的形式传递给/snapshot\* 接口

-snapshotAuthKey string
authKey, which must be passed in query string to /snapshot\* pages

#### 数据存储路径

-storageDataPath string
Path to storage data (default "vmstorage-data")

#### 是否启动 tls

-tls
Whether to enable TLS (aka HTTPS) for incoming requests. -tlsCertFile and -tlsKeyFile must be set if -tls is set
-tlsCertFile string
Path to file with TLS certificate. Used only if -tls is set. Prefer ECDSA certs instead of RSA certs, since RSA certs are slow
-tlsKeyFile string
Path to file with TLS key. Used only if -tls is set
-version
Show VictoriaMetrics version
-vminsertAddr string
TCP address to accept connections from vminsert services (default ":8400")
-vmselectAddr string
TCP address to accept connections from vmselect services (default ":8401")
