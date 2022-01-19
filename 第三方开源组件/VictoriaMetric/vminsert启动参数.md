./vminsert-prod --help
vminsert-20201116-190946-tags-v1.47.0-cluster-0-ga00df790b
Usage of ./vminsert-prod:

#### 导入 csv 数据时的时间戳间隔。最小时间间隔为 1ms，使用较高的时间间隔（如 1s）可以有效降低磁盘占用

-csvTrimTimestamp duration
Trim timestamps when importing csv data to this duration. Minimum practical duration is 1ms. Higher duration (i.e. 1s) may be used for reducing disk space usage for timestamp data (default 1ms)
-enableTCP6
Whether to enable IPv6 for listening and dialing. By default only IPv4 TCP is used
-envflag.enable
Whether to enable reading flags from environment variables additionally to command line. Command line flag values have priority over values from environment vars. Flags are read only from command line if this flag isn't set
-envflag.prefix string
Prefix for environment variables if -envflag.enable is set
-fs.disableMmap
Whether to use pread() instead of mmap() for reading data files. By default mmap() is used for 64-bit arches and pread() is used for 32-bit arches, since they cannot read data files bigger than 2^32 bytes in memory. mmap() is usually faster for reading small data chunks than pread()

#### 监听 Graphite 数据的 tcp 和 udp 地址，为空则不监听

-graphiteListenAddr string
TCP and UDP address to listen for Graphite plaintext data. Usually :2003 must be set. Doesn't work if empty

#### Graphite 数据时间戳间隔时间

-graphiteTrimTimestamp duration
Trim timestamps for Graphite data to this duration. Minimum practical duration is 1s. Higher duration (i.e. 1m) may be used for reducing disk space usage for timestamp data (default 1s)
-http.connTimeout duration
Incoming http connections are closed after the configured timeout. This may help spreading incoming load among a cluster of services behind load balancer. Note that the real timeout may be bigger by up to 10% as a protection from Thundering herd problem (default 2m0s)
-http.disableResponseCompression
Disable compression of HTTP responses for saving CPU resources. By default compression is enabled to save network bandwidth
-http.idleConnTimeout duration
Timeout for incoming idle http connections (default 1m0s)
-http.maxGracefulShutdownDuration duration
The maximum duration for graceful shutdown of HTTP server. Highly loaded server may require increased value for graceful shutdown (default 7s)
-http.pathPrefix string
An optional prefix to add to all the paths handled by http server. For example, if '-http.pathPrefix=/foo/bar' is set, then all the http requests will be handled on '/foo/bar/\*' paths. This may be useful for proxied requests. See https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus
-http.shutdownDelay duration
Optional delay before http server shutdown. During this dealy the servier returns non-OK responses from /health page, so load balancers can route new requests to other servers
-httpListenAddr string
Address to listen for http connections (default ":8480")

#### /api/v1/import 接口接收的最大数据长度，可以通过 max_rows_per_line url 参数修改

-import.maxLineLen max_rows_per_line
The maximum length in bytes of a single line accepted by /api/v1/import; the line length can be limited with max_rows_per_line query arg passed to /api/v1/export
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 104857600)

#### 单次解析 influx 数据的最大长度

-influx.maxLineSize value
The maximum size in bytes for a single Influx line during parsing
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 262144)

#### influx 地址

-influxListenAddr http://<vminsert>:8480/insert/<accountID>/influx/write
TCP and UDP address to listen for Influx line protocol data. Usually :8189 must be set. Doesn't work if empty. This flag isn't needed when ingesting data over HTTP - just send it to http://<vminsert>:8480/insert/<accountID>/influx/write

#### 插入 influx 时的 metric 名称分隔符

-influxMeasurementFieldSeparator string
Separator for '{measurement}{separator}{field*name}' metric name when inserted via Influx line protocol (default "*")

#### 使用 field_name 最为 metric 名称，忽略 measurement 和 -influxMeasurementFieldSeparator

-influxSkipMeasurement
Uses '{field_name}' as a metric name while ignoring '{measurement}' and '-influxMeasurementFieldSeparator'

#### 如果 Influx 行仅包含一个字段，请使用“ {measurement}”而不是“ {measurement} {separator} {field_name}”作为模板名称

-influxSkipSingleField
Uses '{measurement}' instead of '{measurement}{separator}{field_name}' for metic name if Influx line contains only a single field
-influxTrimTimestamp duration
Trim timestamps for Influx line protocol data to this duration. Minimum practical duration is 1ms. Higher duration (i.e. 1s) may be used for reducing disk space usage for timestamp data (default 1ms)

#### 因为-maxConcurrentInserts 导致的插入队列最大等待时间

-insert.maxQueueDuration duration
The maximum duration for waiting in the queue for insert requests due to -maxConcurrentInserts (default 1m0s)
-loggerDisableTimestamps
Whether to disable writing timestamps in logs
-loggerErrorsPerSecondLimit int
Per-second limit on the number of ERROR messages. If more than the given number of errors are emitted per second, then the remaining errors are suppressed. Zero value disables the rate limit (default 10)
-loggerFormat string
Format for logs. Possible values: default, json (default "default")
-loggerLevel string
Minimum level of errors to log. Possible values: INFO, WARN, ERROR, FATAL, PANIC (default "INFO")
-loggerOutput string
Output for the logs. Supported values: stderr, stdout (default "stderr")

#### 最大并发插入数，默认值在大多数情况下都是最优解，因为他最大程度的减少了并发开销。和-insert.maxQueueDuration 配合使用

-maxConcurrentInserts int
The maximum number of concurrent inserts. Default value should work for most cases, since it minimizes the overhead for concurrent inserts. This option is tigthly coupled with -insert.maxQueueDuration (default 8)

#### 单个 Prometheus remote_write API 请求的最大长度

-maxInsertRequestSize value
The maximum size in bytes of a single Prometheus remote_write API request
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 33554432)

#### 每个时序数据能够接受的最大 label 数量

-maxLabelsPerTimeseries int
The maximum number of labels accepted per time series. Superflouos labels are dropped (default 30)
-memory.allowedBytes value
Allowed size of system memory VictoriaMetrics caches may occupy. This option overrides -memory.allowedPercent if set to non-zero value. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 0)
-memory.allowedPercent float
Allowed percent of system memory VictoriaMetrics caches may occupy. See also -memory.allowedBytes. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage (default 60)
-opentsdbHTTPListenAddr string
TCP address to listen for OpentTSDB HTTP put requests. Usually :4242 must be set. Doesn't work if empty
-opentsdbListenAddr string
TCP and UDP address to listen for OpentTSDB metrics. Telnet put messages and HTTP /api/put messages are simultaneously served on TCP port. Usually :4242 must be set. Doesn't work if empty
-opentsdbTrimTimestamp duration
Trim timestamps for OpenTSDB 'telnet put' data to this duration. Minimum practical duration is 1s. Higher duration (i.e. 1m) may be used for reducing disk space usage for timestamp data (default 1s)
-opentsdbhttp.maxInsertRequestSize value
The maximum size of OpenTSDB HTTP put request
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 33554432)
-opentsdbhttpTrimTimestamp duration
Trim timestamps for OpenTSDB HTTP data to this duration. Minimum practical duration is 1ms. Higher duration (i.e. 1s) may be used for reducing disk space usage for timestamp data (default 1ms)

#### 标签重置的配置文件路径

-relabelConfig string
Optional path to a file with relabeling rules, which are applied to all the ingested metrics. See https://victoriametrics.github.io/####relabeling for details

#### 副本数，即在-storageNode 要复制多少个副本。当-replicationFactor>1 时，vmselect 必须以-dedup.minScrapeInterval=1ms 运行，以进行重复数据的删除。~~vmselect 的-dedup.minScrapeInterval 可以大一些~~

-replicationFactor int
Replication factor for the ingested data, i.e. how many copies to make among distinct -storageNode instances. Note that vmselect must run with -dedup.minScrapeInterval=1ms for data de-duplication when replicationFactor is greater than 1. Higher values for -dedup.minScrapeInterval at vmselect is OK (default 1)
-rpc.disableCompression
Disable compression of RPC traffic. This reduces CPU usage at the cost of higher network bandwidth usage

#### vmstorage 节点列表

-storageNode array
Address of vmstorage nodes; usage: -storageNode=vmstorage-host1:8400 -storageNode=vmstorage-host2:8400
Supports array of values separated by comma or specified via multiple flags.
-tls
Whether to enable TLS (aka HTTPS) for incoming requests. -tlsCertFile and -tlsKeyFile must be set if -tls is set
-tlsCertFile string
Path to file with TLS certificate. Used only if -tls is set. Prefer ECDSA certs instead of RSA certs, since RSA certs are slow
-tlsKeyFile string
Path to file with TLS key. Used only if -tls is set
-version
Show VictoriaMetrics version
